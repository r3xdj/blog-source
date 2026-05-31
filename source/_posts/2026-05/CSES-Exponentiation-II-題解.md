---
title: CSES Exponentiation II 題解
tags:
  - C++
  - math
categories:
  - OI
  - CSES
  - Mathematics
cover: /img/cses.jpg
top_img: /img/cses.jpg
date: 2026-05-31 22:37:40
updated: 2026-05-31 22:37:40
description:
---


> 本題會用到上次在[***CSES Exponentiation 題解 | R3X's Blog***](/2026/05/31/CSES-Exponentiation-%E9%A1%8C%E8%A7%A3/)講到的東西，並假設讀者都已看過。

# 題目

- Time limit: 1.00 s
- Memory limit: 512 MB

Your task is to efficiently calculate values $a^{b^c}$ modulo $10^9+7$.
Note that in this task we assume that $0^0=1$.

| Input | Output |
| --- | --- |
|The first input line contains an integer $n$: the number of calculations.<br/>After this, there are $n$ lines, each containing three integers $a, b$ and $c$. |Print each value $a^{b^c}$ modulo $10^9+7$.|

Constraints
- $1 \le n \le 2 \cdot 10^5$
- $0 \le a,b,c \le 10^9$

Example:
| Input | Output |
| --- | --- |
|3<br/>3 7 1<br/>15 2 2<br/>3 4 5<br/>|2187<br/>50625<br/>763327764|

# 解法

這題跟前一題差不多，我們只要額外想如何處裡$a$上面的$b^c$即可。因為$M = 10^9+7$是質數，所以我們可以用費馬小定理，即$a^{M-1}\equiv 1\pmod{M}$。於是我們只要先計算出$t\equiv b^c \pmod{M-1}$(這樣能有效的讓$a$的指數變小)，再計算$a^t$即可。指數計算則用我們上次提到的快速冪。

## AC Code

```cpp
#include<bits/stdc++.h>
using namespace std;

const int M = (int)1e9 + 7;
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

int main(){
  ios::sync_with_stdio(0), cin.tie(0);
  ll n, a, b, c;
  cin >> n;
  while(n--){
    cin >> a >> b >> c;
    cout << fast_pow(a, fast_pow(b,c,M-1),M) << "\n";
  }
}
```
