---
title: CSES Bracket Sequences I 題解
tags:
  - C++
  - math
  - Catalan number
  - number theory
  - modulus
  - gcd
categories:
  - OI
  - CSES
  - Mathematics
cover: /img/CSES.jpg
top_img: /img/CSES.jpg
date: 2026-06-01 19:43:04
updated: 2026-06-01 19:43:04
description:
---


# 題目

- Time limit: 1.00 s
- Memory limit: 512 MB

Your task is to calculate the number of valid bracket sequences of length $n$. For example, when $n=6$, there are $5$ sequences:

- ()()()
- ()(())
- (())()
- ((()))
- (()())

| Input | Output |
| --- | --- |
|The only input line has an integer $n$.|Print the number of sequences modulo $10^9+7$.|

Constraints
- $1 \le n \le 10^6$

Example:
| Input | Output |
| --- | --- |
|6|5|

# 解法

這題很顯然是在考卡特蘭數

## 卡特蘭數

### 定義

卡特蘭數$C_n$表示有A, B各$n$個，A永不落後B的排列方法數。組合學上有很多問題都跟卡特蘭數有關。

### 公式

我們可以將卡特蘭數問題轉換成$n\times n$網格上，從左下到右上角的路徑方法數，最終可推得

$$
C_n = \binom{2n}{n}-\binom{2n}{n+1} = \frac{1}{n+1}\binom{2n}{n}, n\geq 1
$$

(網路上有很多教學，在此就不贅述了，反正推過你最後還是得把結果記起來)

### 遞迴

卡特蘭數的遞迴式如下：
$$
\begin{cases}
C_0=1\\
C_{n+1}=C_0C_n+C_1C_{n-1}+\cdots+C_{n-1}C_1+C_nC_0=\sum\limits_{i=0}^{n}C_{i}C_{n-i}, n\geq 0
\end{cases}
$$

## 原題分析

顯然$n$為奇數時答案為$0$，而$n$為偶數時所求即$C_{\frac{n}{2}}$。
接著我們只需想辦法算出$C_k~\rm{mod}~(10^9+7)$即可。而
$$
C_k=\frac{1}{k+1}\binom{2k}{k}=\frac{1}{k+1}\cdot\frac{(k+1)(k+2)\cdots 2k}{1\cdot 2\cdots k}=\frac{(k+2)(k+3)\cdots 2k}{1\cdot 2\cdots k}
$$

我們令$\mathrm{num}=(k+2)(k+3)\cdots 2k$(代表分子，numerator)、$\mathrm{den}=1\cdot 2\cdots k$(代表分母，denominator)，則
$$
C_k=\frac{\mathrm{num}}{\mathrm{den}}\Rightarrow\mathrm{den}\cdot C_k=\mathrm{num}
$$

接著兩邊同時mod $(10^9+7)$，假設模完他變成$AC_k\equiv B\pmod{10^9+7}$，那麼$C_k\equiv A^{-1}B\pmod{10^9+7}$

所以我們還需要寫一個用來求模反元素的函式。

## 求解模反元素

若$(a,M)=1$(此時才必有唯一的模反元素)，則稱
$$
ax\equiv 1\pmod{M}
$$
的解$x$為$a$模$M$的反元素，記作$a^{-1}$

### 方法一：費馬小定理+快速冪

由於模數$M=10^9+7$為質數，由費馬小定理知
$$
a^{M-1}\equiv 1\equiv ax\pmod M\Rightarrow x\equiv a^{M-2}\pmod M
$$
而$a^{M-2}$可由[上次提過的快速冪](http://localhost:4000/2026/05/31/CSES-Exponentiation-%E9%A1%8C%E8%A7%A3/#%E5%BF%AB%E9%80%9F%E5%86%AA)算出。

實現程式碼如下：

```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long

ll fast_pow(ll base, ll exp, int m){
  ll res = 1LL;
  while(exp > 0){
    if(exp%2==1){res = (res*base) % m;}
    base = (base*base) % m;
    exp /= 2;
  }
  return res;
}
ll invert(ll a, ll M){ // 僅當M為質數時可用
  return fast_pow(a, M-2, M)
}
```

當然，就算$M$不是質數，我們還是可以用歐拉定理推出
$$
x\equiv a^{\phi(M)-1}\pmod M
$$
但這等於我們還要先去算出歐拉函數，通常不會比較方便。因此此時較為建議以下方法二。

### 方法二：擴展歐幾里得算法

> 打CTF也打多久了，怎麼連個擴展歐幾里得都不會寫
> 我也是搞好久才懂

這個演算法十分重要，打Crypto時也會遇到(騙你的，這太基礎了題目大概是不會考這個w)

我們都知道歐幾里得演算法(也就是輾轉相除法)可以用來求兩數的最大公因數，那擴展歐幾里得算法是拿來幹嘛的呢？
貝祖定理告訴我們，對於所有$a,b\in \Z$且$ab\neq 0$，存在$x,y\in\Z$使得
$$
ax+by=\gcd(a,b)
$$
擴展歐幾里得演算法就是用來找出這組整數$(x,y)$的。
以下說明用到了遞迴的概念。
先將$a$以$b$除，得到
$$
a=bq+r \Rightarrow r=a-bq
$$
由輾轉相除法原理，$\gcd(a,b)=\gcd(b,r)$，令為$k$。
現在我們想像$k$已寫成$b,r$的線性組合(貝祖定理)$k=bx^\prime+ry^\prime$。
ㄟ，那不就把$r=a-bq$代入再移項，就能整理成$ax+by=k$的樣子了嗎？(注意，$q$是已知數，為$\lfloor\frac{a}{b}\rfloor$)如下
$$
k=bx^\prime+ry^\prime=bx^\prime+(a-bq)y^\prime=ay^\prime+b(x^\prime-qy^\prime)
$$
所以我們只要知道$x^\prime, y^\prime$，就能知道$x=y^\prime, y=x^\prime-qy^\prime$，其中$q=\lfloor\frac{a}{b}\rfloor$
那我們怎麼求出$x^\prime, y^\prime$呢？沒錯，用擴展歐幾里得演算法。(所以說是遞迴嘛)

欲遞迴，我們記得處理base case，函式才停得下來。最簡單的base case即當$b=0$，返回$(x,y)=(1,0)$。
(因為輾轉相除法會一直做到整除，也就是最後一個$r=0$為止)

以下程式：

```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long

tuple<ll, ll, ll> exgcd(ll a, ll b){
  if(b==0) return {a, 1, 0};
  auto [gcd, x1, y1] = exgcd(b, a%b);
  ll x = y1;
  ll y = x1 - (a/b)*y1;
  return {gcd, x, y};
}
```

好，但這跟我們求模反元素有什麼關係呢？請看以下：
因為$a,M$互質，我們可以用擴展歐幾里得演算法求出$x,y\in\Z$使得
$$
ax+My=1\Rightarrow ax=1-My
$$
兩邊同時模$M$，瞬間得到$x$即我們要找的模反元素。

```cpp
ll invert(ll a, ll m){
  auto [gcd, x, y] = exgcd(a, m);
  if (gcd != 1) throw invalid_argument("Does not exist."); // a, m互質才有模反元素
  return (x%m + m)%m;
}
```

## AC Code

好耶！我們將以上全部組合起來，得到以下：

```cpp
#include<bits/stdc++.h>
using namespace std;

#define ll long long
const int M = 1e9 + 7;

tuple<ll, ll, ll> exgcd(ll a, ll b){
  if(b==0) return {a, 1, 0};
  auto [gcd, x1, y1] = exgcd(b, a%b);
  ll x = y1;
  ll y = x1 - (a/b)*y1;
  return {gcd, x, y};
}

ll invert(ll a, ll m){
  auto [gcd, x, y] = exgcd(a, m);
  if (gcd != 1) throw invalid_argument("Does not exist.");
  return (x%m + m)%m;
}

int main(){
  ios::sync_with_stdio(0), cin.tie(0); // IO加速
  int n;
  cin >> n;
  if(n % 2 == 1){cout << 0; return 0;}
  n /= 2;
  ll den=1; // 分母
  ll num=1; // 分子
  for(int i=1;i<=n;i++) den = (den*i)%M;
  for(int i=n+2;i<=2*n;i++) num = (num*i)%M;
  cout << invert(den, M)*num % M;
}
```

AC！收工！
