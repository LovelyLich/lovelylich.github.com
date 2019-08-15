---
layout:     post
title:      "中国剩余定理"
date:       2019-07-05 15:00:00
author:     "Yi"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 算法
---

### 1. 简介（内容摘自维基百科）
中国剩余定理给出了以下一元线性同余方程组：

$$
(S) : \begin{cases}
   x \equiv a_1 & (\text{mod }m_1) \\
   x \equiv a_2 & (\text{mod }m_2) \\
   & \vdots \\
   x \equiv a_n & (\text{mod }m_n)
\end{cases}
$$

有解的判定条件，并用构造法给出了在有解情况下解的具体形式。
中国剩余定理说明：假设整数$m_1,m_2,\ldots,m_n$两两互质，则对任意的整数：$a_1,a_2,\ldots,a_n$,方程组(S)有解，并且通解可以用如下方式构造得到：

1. 设$M=m_1 \times m_2 \times \ldots \times m_n=\displaystyle\prod_{i=1}^n m_i$是整数$m_1,m_2,\ldots,m_n$的乘积，并设$M_i=\frac{M}{m_i}$，即$M_i$是除$m_i$之外的n-1个整数的乘积
2. 设$t_i=M_i^{-1}$为$M_i$模$m_i$的数论倒数：$t_iM_i \equiv 1 (\text{mod }m_i)$
3. 方程组(S)的通解形式为:

$$
x=a_1t_1M_1+a_2t_2M_2+\ldots+a_nt_nM_n+kM
$$

### 2. 解释

以模$m_1$为例，由于$t_1M_1$模$m_1$等于1，所以$a_1t_1M_1$模$m_1$时即为余$a_1$，另其余项由于$M_i$的存在，都含有因子$m_1$，所以其余项都能被整除，故项$a_1t_1M_1$支撑了方程组(S)中的第一个方程成立，其余类似。故第三步得到的即为方程组的通解。

### 3. 求模的逆元
同样，以$m_1$为例，为寻找$M_i$模$m_i$的模逆元，即寻找满足:

$$
t_iM_i \equiv 1 (\text{mod } m_i)
$$
的$t_i$。

另一方面，设exgcd($M_i, m_i$)为扩展欧几里算法的函数，则可得$M_ix + m_iy = g$, $g$为$M_i,m_i$的最大公约数。
由于:$M_ix + m_iy = 1$,两边同时取模$m_i$得

$$
\begin{aligned}
(M_ix+m_iy)\%m_i&=1 \\
(M_ix)\%m_i&=1
\end{aligned}
$$

根据模逆元的定义，所以x即为$M_i$模$m_i$的逆元。
而x可以通过扩展欧几里德递归调用得到。
另注意：扩展欧几里德得到的x是$M_i$模$m_i$的其中一个逆元，为取最小正整数逆元，取 $x \text{ mod } n$ 即可。

### 4. 扩展欧几里德算法

定义: 扩展欧几里德算法对于任意的$a \ge b \ge 0$,能够计算出$x,y,d$，使得$ax+by=d=gcd(a,b)$

算法实现：

```python
function exgcd(a, b)
Input: Two positive integers a and b with a>=b>=0
Output: Integers x, y, d such taht d=gcd(a,b) and ax+by=d
if b=0: return (1, 0, a)
(x1, y1, d) = exgcd(b, a%b)
return (y1, x1-floor(a/b)y, d)
```

所以对于$M_i$模$m_i$的逆元可通过调用exgcd($M_i, m_i$)返回的x得到

### 5. POJ例题：Problem1006 Biorhythms
**解析：**设对于某个case输入 $p,i,e$ 给出三个达到顶峰时的时间，并且从 $d$ 开始，经过 $x$ 天后，三者同时达到顶峰，则有方程：

$$
\begin{cases}
   p+23m_1 &= d+x \\
   i+28m_2 &= d+x \\
   e+33m_3 &= d+x
\end{cases}
$$

所以对于每个case，即为给出了$p, i, e, d$,欲求x的值。两边同时$mod  \ m_i$即可转化为同余方程组形式，同时设$y=d+x$, 即为：

$$
\begin{cases}
   y \equiv p & (mod\ 23) \\
   y \equiv i & (mod\ 28) \\
   y \equiv e & (mod\ 33)
\end{cases}
$$

所以，根据中国剩余定理即可解出。

**实现代码:**

```c++
//
//  main.cpp
//  Problem1006
//

#include <iostream>
#include <math.h>
using namespace std;

//扩展欧几里德公式
void exgcd(long long a,long long b, long long *x, long long *y, long long *d) {
    cout << "start compute: " << "a=" << a <<" b=" << b << " x=" << *x << " y=" << *y << " d=" << *d <<  endl;
    if (b==0)
    {
        *x=1;
        *y=0;
        *d = a;
        return;
    } else {
        exgcd(b, a%b, x, y, d);
        long long t=*y;
        *y = *x - floor(a/b) * (*y);
        *x = t;
        cout << "inverse multipler of " << b << "mod "<< a%b << " is: " << *x << " y=" << *y<< " d=" << *d <<  endl;
    }
}
// 中国剩余定理
long long crt(int n, long long *a, long long *m){
    long long M=1;
    long long ret=0;
    //计算最小公倍数
    cout << "a[0]="<< a[0] << "m[0]=" << m[0] << endl;
    cout << "a[1]="<< a[1] << "m[1]=" << m[1] << endl;
    cout << "a[2]="<< a[2] << "m[2]=" << m[2] << endl;
    for (int i=0;i<n;i++) {
        M*=m[i];
        cout << "M="<< M << endl;
    }
    
    //利用通解求结果
    for (int i=0;i<n;i++) {
        //计算M_i的逆元
        long long x, y, d;
        long long M_i=M/m[i];
        exgcd(M_i,m[i], &x, &y, &d);
        cout << "i="<<i << " M_i=" << M_i <<  " m[i]=" << m[i] << " x=" << x << endl;
        //x是M_i%m[i]意义下的逆元
        ret+=M_i*x*a[i];
        ret%=M;
    }
    return ret;
}

int main(int argc, const char * argv[]) {
    long long m[3] = {23, 28,33};
    long long a[3] = {0};
    long long d;
    int i = 0;
    while(true) {
        i++;
        cin>>a[0]>>a[1]>>a[2]>>d;
        if (a[0]==a[1] && a[1]==a[2] && a[2]==d && d==-1)
            break;
        long long ret = crt(3, a, m);
        cout << "Case "<< i << ": the next triple peak occurs in " << ret << " days." << endl;
    }
    return 0;
}
```


