---
title: CSES Missing Number 題解
tags:
  - C++
categories:
  - OI
  - CSES
  - Introductory Problems
cover: /img/CSES.jpg
top_img: /img/CSES.jpg
description:
---

# 題目

> ~~我覺得把題目複製過來是最累的一件事~~

- Time limit: 1.00 s
- Memory limit: 512 MB

You are given all numbers between $1,2,\ldots,n$ except one. Your task is to find the missing number.

| Input | Output |
| --- | --- |
|The first input line contains an integer $n$.<br/>The second line contains $n-1$ numbers. Each number is distinct and between $1$ and $n$ (inclusive).|Print the missing number.|

Constraints
- $2 \le n \le 2 \cdot 10^5$

Example:
| Input | Output |
| --- | --- |
|5<br/>2 3 1 5|4|

# 解法

## 解法一

### 分析

這一個是我的做法
我很暴力的建一個布林陣列，用來標記哪些數字出現過了。當輸入$k$時，將陣列中Index為$k$的那項設為True
最後再掃一遍陣列的Index $1$~$n$，看哪一個是False就是少掉的數字。

### AC Code

```cpp
#include<bits/stdc++.h>
using namespace std;
 
bool Number[200001];
long long n;
int main(){
    std::ios::sync_with_stdio(0);
    std::cin.tie(0);
    cin >> n;
    for(int i=1;i<n;i++){
        int x;
        cin >> x;
        Number[x] = 1;
    }
    for(int i=1;i<=n;i++){
        if(Number[i] == 0){cout << i; break;}
    }
}
```

## 解法二

這是我在網路上看到的：[CSES-Missing Number - HackMD](https://hackmd.io/@apcser/HJ5R16POR)
計算輸入的$n-1$個數的總和，再用$1$到$n$的總和去扣，得到的答案就是少掉的數字。
好處是可以大大降低空間複雜度。

程式自己練習吧反正不難。
