---
title: CSES Dice Combinations 題解
tags:
  - C++
  - DP
categories:
  - OI
  - CSES
  - Dynamic Programming
cover: /img/CSES.jpg
top_img: /img/CSES.jpg
date: 2026-06-03 13:06:13
updated: 2026-06-03 13:06:13
description:
---


> ~~DP? 想成高中數學的遞迴就好啦~~

# 題目

- Time limit: 1.00 s
- Memory limit: 512 MB

Your task is to count the number of ways to construct sum n by throwing a dice one or more times. Each throw produces an outcome between 1 and  6.
For example, if n=3, there are 4 ways:

- 1+1+1
- 1+2
- 2+1
- 3

| Input | Output |
| --- | --- |
|The only input line has an integer $n$.|Print the number of ways modulo $10^9+7$.|

Constraints
- $1 \le n \le 10^6$

Example:
| Input | Output |
| --- | --- |
|3|4|

# 解法

## 分析

這是DP入門題
給定$n\in\N$，令$\mathrm{dp}[n]$表示欲求的方法數。

首先處理$n=1,2,\cdots,6$的情形，此時是整數的有序拆分
($n\geq 7$時就不是了，以$n=7$作為舉例，直接算整數拆分的方法數會多算到「擲出一個$7$」這個方法，但骰子點數只有$1$~$6$)
對於整數$n$的整數拆分，可以看成有$n$顆球一字排開，如此共有$n-1$個空隙，每個空隙都可選擇放/不放一個隔板，此方法數與整數拆分方法數一一對應。如此共有$2^{n-1}$種方法。

而當$n=k\geq 7$，欲組出總和為$k$，可以是以下幾種情形：
    - 前幾次已組出$k-1$，最後一次擲出$1$
    - 前幾次已組出$k-2$，最後一次擲出$2$
    - 前幾次已組出$k-3$，最後一次擲出$3$
    等等，一直到
    - 前幾次已組出$k-6$，最後一次擲出$6$

於是
$$
\begin{cases}
\mathbb{dp}[n]=2^{n-1}, \space n=1,2,\cdots,6  \\
\mathrm{dp}[n]=\mathrm{dp}[n-1]+\mathrm{dp}[n-2]+\cdots+\mathrm{dp}[n-6],\space n\geq 7
\end{cases}
$$

喔然後記得模$10^9+7$

## AC Code

```cpp
#include<bits/stdc++.h>
using namespace std;

const int M = 1e9+7;
const int N_MAX = 1e6;

long long dp[N_MAX+1];
int main(){
  int n;
  cin >> n;
  for(int i=1;i<=6;i++){dp[i] = 1 << (i-1);}
  for(int i=7;i<=n;i++){
    for(int j=1;j<=6;j++){
      dp[i] = (dp[i] + dp[i-j]) % M;
    }
  }
  cout << dp[n];
}
```

這裡用了一個技巧，`1 << (i-1)`。其中`<<`是左移算子，表示在二進位表示中將整個數字往左移`i-1`格，然後在後面補$0$。
比如`1 << 5`的結果等於$100000_2=2^5=32_{10}$。所以`1 << (i-1)`其實就是$2^{i-1}$的意思。
