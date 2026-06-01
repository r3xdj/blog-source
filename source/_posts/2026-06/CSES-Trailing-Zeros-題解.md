---
title: CSES Trailing Zeros 題解
tags:
  - C++
  - math
  - factorial
categories:
  - OI
  - CSES
  - Introductory Problems
cover: /img/CSES.jpg
top_img: /img/CSES.jpg
date: 2026-06-01 18:19:36
updated: 2026-06-01 18:19:36
description:
---


# 題目

- Time limit: 1.00 s
- Memory limit: 512 MB

Your task is to calculate the number of trailing zeros in the factorial n!.
For example, $20!=2432902008176640000$ and it has $4$ trailing zeros.

| Input | Output |
| --- | --- |
|The only input line has an integer $n$.|Print the number of trailing zeros in $n!$.|

Constraints
- $1 \le n \le 10^9$

Example:
| Input | Output |
| --- | --- |
|20|4|

# 解法

## 分析

這題十分經典，是要求$n!$尾部有幾個$0$。
注意到$n!$尾部$0$的個數$=$滿足$10^k | n!$的$k$的最大值(也就是$n!$能被$10$整除幾次)$=n!$的質因數分解中$5$的次方數(因為$10=2\cdot 5$，而又顯然$n!$的質因數分解中$5$的次數會比$2$來的少)

接著，$n!$的質因數分解中$5$的次數即為$1$~$n$中每個數個別被$5$整除的次數之和。
$1$~$n$中，
- 被$5$整除的數有$\lfloor\frac{n}{5}\rfloor$個
- 被$5^2$整除的數有$\lfloor\frac{n}{5^2}\rfloor$個
- 被$5^3$整除的數有$\lfloor\frac{n}{5^3}\rfloor$個
等等，於是所求即
$$
\lfloor\frac{n}{5}\rfloor+\lfloor\frac{n}{5^2}\rfloor+\lfloor\frac{n}{5^3}\rfloor+\cdots=\sum\limits_{k=1}^\infty\lfloor\frac{n}{5^k}\rfloor
$$
此即勒讓德定理(Legendre's Theorem)：
$$
\nu_{p}(n!)=\sum\limits_{k=1}^\infty\lfloor\frac{n}{p^k}\rfloor
$$
其中$\nu_p(x)$表示滿足$p^k|x$的最大正整數$k$(即$x$被$p$整除幾次)

在計算時，我們可以用以下性質簡化計算：
$$
\lfloor \frac{x}{ab} \rfloor=\lfloor \frac{\lfloor \frac{x}{a}\rfloor}{b} \rfloor
$$

## AC Code

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
  int n;
  cin >> n;
  int z = n, ans = 0;
  while(z != 0){
    z /= 5;
    ans += z;
  }
  cout << ans;
}
```

很簡單吧~
