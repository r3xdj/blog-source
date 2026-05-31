---
title: CSES Exponentiation 題解
tags:
  - C++
  - math
categories:
  - OI
  - CSES
  - Mathematics
date: 2026-05-31 21:30:11
updated: 2026-05-31 21:30:11
description:
cover:
top_img:
---


# 題目

- Time limit: 1.00 s
- Memory limit: 512 MB

Your task is to efficiently calculate values $a^b$ modulo $10^9+7$.
Note that in this task we assume that $0^0=1$.

| Input | Output |
| --- | --- |
|The first input line contains an integer $n$: the number of calculations.<br/>After this, there are $n$ lines, each containing two integers $a$ and $b$. |Print each value $a^b$ modulo $10^9+7$.|

Constraints
- $1 \le n \le 2 \cdot 10^5$
- $0 \le a,b \le 10^9$

Example:
| Input | Output |
| --- | --- |
|3<br/>3 4<br/>2 8<br/>123 123<br/>|81<br/>256<br/>921450052|

# 解法

## 快速冪

### 理論

要快速計算乘方，這題顯然就是快速冪。
我們以一個例子解釋快速冪的邏輯。假設我們要計算$a^32$，我們其實不需要真的將$a$做$32$次乘法，而可以依序計算$a^2$、$a^4=(a^2)^2$、$a^8=(a^4)^2$、$a^{16}=(a^8)^2$、$a^{32}=(a^{16})^2$。這樣可以讓指數的大小本身亦以指數增長，進而讓$a^n$的時間複雜度從$O(n)$降到$O(\log n)$

對於一般(非$2$的次冪)的指數$n$，快速冪的邏輯如下：
欲計算$a^n$，先將$n$以二進位表示(這麼做是為了用指數律將$a^n$拆成若干個$a^{k_i}$相乘，其中每個次方$k_i$都是$2$的冪次，而由上我們可知$2$的冪次很好算)。舉例來說，若$n=13_{10}=1101_2$，則$a^n=a^{13}=a^8a^4a^1$。接下來便是照我們上面說的一次把$a^8, a^4, a^1$都算出來，再相乘即可。

但其實實務上我們不是這樣做的，實務上做法是：
如果指數是偶數(設$n=2k$)，那麼$a^n=(a^2)^k$；
如果指數是奇數(設$n=2k+1$)，則把$a^n$拆成$aa^{2k}=a(a^2)^k$。
如此我們每經一次操作，指數都會變成約原先的一半。

### 處理模數

題目額外要求了要將答案模$10^9+7$，就是因為計算過程中出現的數字和答案可能太大。但這不困難，只要將上述快速冪中每個乘法步驟都取模即可。

### C++ Code

綜合以上，我們有快速冪的程式如下：

```cpp
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
```

## AC Code

此題的完整程式碼如下：

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
  ll n, a, b;
  cin >> n;
  while(n--){
    cin >> a >> b;
    cout << fast_pow(a,b,M) << "\n";
  }
}
```
