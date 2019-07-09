---
layout:     post
title:      "从套接字的Pipe Broken异常说起"
date:       2019-07-06 11:00:00
author:     "Yi"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - TCP 协议
---

### 引子

公司开发的新功能，要求支持Nginx层非HTTP请求都透传, 但测试部门报告说有偶现的异常，错误是:

>send() failed (32: Broken pipe)

首先我们需要明确的是: 这种Broken Pipe 在什么情况下触发。这篇文章记录我的探索过程。

### 思考
我们知道：TCP连接建立好后，如果没有数据包传输，连接双方并不会做心跳保持连接。所以此时，TCP连接双方都默认对方仍然存活。

但如果其中一方（比如客户端）发生异常，比如coredump、异常掉电、程序exit(-1)等情况时，在这些情况下，服务器需要等到何时才能知晓该异常呢？

### 测试验证

#### 验证coredump情况
##### 测试 1 
为验证coredump，我们需要手工构造异常，我是用除0异常来让程序直接崩溃退出。客户端代码逻辑就是建立连接，接收数据，并且直接coredump。

代码如下:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

void error(const char *msg)
{
        perror(msg);
        exit(0);
}

int main(int argc, char *argv[])
{
        int sockfd, portno, n;
        struct sockaddr_in serv_addr;
        struct hostent *server;

        char buffer[256];
        if (argc < 3) {
                fprintf(stderr,"usage %s hostname port\n", argv[0]);
                exit(0);
        }
        portno = atoi(argv[2]);
        sockfd = socket(AF_INET, SOCK_STREAM, 0);
        if (sockfd < 0)
                error("ERROR opening socket");
        server = gethostbyname(argv[1]);
        if (server == NULL) {
                fprintf(stderr,"ERROR, no such host\n");
                exit(0);
        }
        bzero((char *) &serv_addr, sizeof(serv_addr));
        serv_addr.sin_family = AF_INET;
        bcopy((char *)server->h_addr,
                        (char *)&serv_addr.sin_addr.s_addr,
                        server->h_length);
        serv_addr.sin_port = htons(portno);
        if (connect(sockfd, (struct sockaddr *) &serv_addr, sizeof(serv_addr)) < 0)
                error("ERROR connecting");

	//建立连接后接收数据，并直接coredump
        printf("connected, recv and coredump\n");

        n  = recv(sockfd, buffer, 256, 0);
        buffer[n] = 0;
        printf("resp=%s\n", buffer);

        n = 3/0;
        return 0;
}
```

服务端代码逻辑则是在接受连接并发送数据后，等待客户端coredump掉以后，尝试send到套接字。服务端代码如下：

```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <time.h>

int main(int argc, char *argv[]) {
        int listenfd = 0, connfd = 0;
        int ret = 0;
        struct sockaddr_in serv_addr;

        char sendBuff[1025];
        time_t ticks;

        listenfd = socket(AF_INET, SOCK_STREAM, 0);
        memset(&serv_addr, 0, sizeof(serv_addr));
        memset(sendBuff, 0, sizeof(sendBuff));

        serv_addr.sin_family = AF_INET;
        serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
        serv_addr.sin_port = htons(5000);

        bind(listenfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr));

        listen(listenfd, 10);

        connfd = accept(listenfd, (struct sockaddr*)NULL, NULL);
        printf("got connected socket\n");

        ticks = time(NULL);
        snprintf(sendBuff, sizeof(sendBuff), "%.24s\r\n", ctime(&ticks));
        write(connfd, sendBuff, strlen(sendBuff));

        //已发送数据，等待客户端coredump
        printf("have sent data to client, waiting for client coredump..\n");
        sleep(20);

        //客户端已coredump，此时尝试发送数据包到客户端
        printf("send data to coredumped socket...\n");
        ticks = time(NULL);
        snprintf(sendBuff, sizeof(sendBuff), "%.24s\r\n", ctime(&ticks));
        ret = send(connfd, sendBuff, strlen(sendBuff), 0);
        printf("send returned %d, err: %s\n", ret, strerror(errno));
        
        return 0;
}
```
测试时，先启动服务器，然后启动客户端尝试连接，并接收数据和coredump，以下是服务器端抓包结果：

```
15:15:28.838856 IP 10.10.121.101.42642 > 10.10.121.102.5000: Flags [S], seq 1555385680, win 29200, options [mss 1460,sackOK,TS val 285610676 ecr 0,nop,wscale 7], length 0
15:15:28.838891 IP 10.10.121.102.5000 > 10.10.121.101.42642: Flags [S.], seq 3125264688, ack 1555385681, win 28960, options [mss 1460,sackOK,TS val 285600862 ecr 285610676,nop,wscale 7], length 0
15:15:28.838959 IP 10.10.121.101.42642 > 10.10.121.102.5000: Flags [.], ack 1, win 229, options [nop,nop,TS val 285610676 ecr 285600862], length 0
15:15:28.839279 IP 10.10.121.102.5000 > 10.10.121.101.42642: Flags [P.], seq 1:27, ack 1, win 227, options [nop,nop,TS val 285600862 ecr 285610676], length 26
15:15:28.839349 IP 10.10.121.101.42642 > 10.10.121.102.5000: Flags [.], ack 27, win 229, options [nop,nop,TS val 285610676 ecr 285600862], length 0
15:15:28.941654 IP 10.10.121.101.42642 > 10.10.121.102.5000: Flags [F.], seq 1, ack 27, win 229, options [nop,nop,TS val 285610701 ecr 285600862], length 0
15:15:28.944159 IP 10.10.121.102.5000 > 10.10.121.101.42642: Flags [.], ack 2, win 227, options [nop,nop,TS val 285600889 ecr 285610701], length 0
15:15:48.839470 IP 10.10.121.102.5000 > 10.10.121.101.42642: Flags [P.], seq 27:53, ack 2, win 227, options [nop,nop,TS val 285605862 ecr 285610701], length 26
15:15:48.839603 IP 10.10.121.101.42642 > 10.10.121.102.5000: Flags [R], seq 1555385682, win 0, length 0
```
可以看到客户端在coredump时，会由客户端操作系统立即向服务器发送FIN.ACK包。
另外，在服务器可以看到，send()调用是成功返回：

```
root@u102:~# ./server
got connected socket
have sent data to client, waiting for client coredump..
send data to coredumped socket...
send returned 26, err: Success
```
另外，如果将此处send()改成recv()，则结果是：

```
root@u102:~# ./server
got connected socket
have sent data to client, waiting for client coredump..
recv() on coredumped socket, returned : 0, err:Success
```
##### 测试1结论
综上：客户端coredump后，会由客户端操作系统立即向服务器发送FIN.ACK包。该行为对于服务器而言，是正常的连接单方断开行为。所以，这并不影响服务器向客户端发送数据，此时send()调用返回成功。而如果此时recv()，同样也是正常的，服务器仅通过recv返回0，知悉客户端断开了连接，并不关心如何断开的。
总而言之，服务器此时仅知道，客户端响应了FIN.ACK包。

##### 测试 2
前面是客户客户端接收数据后coredump，假设coredump时套接字缓冲区有未接收数据呢？比如我们在客户端调用recv()之前就coredump。下面是抓包结果：

```

```
这是由内核哪部分代码实现的呢？答案是tcp_close()。

##### 溯源寻根

