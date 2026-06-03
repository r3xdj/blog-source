---
title: CSES - Counting Divisors 題解
tags:
  - C++
  - math
  - sieve
categories:
  - OI
  - CSES
  - Mathematics
description:
cover: /img/CSES.jpg
top_img: /img/CSES.jpg
---

> 我把這題想的太複雜了
> 但這題真的可以很深

# 題目

- Time limit: 1.00 s
- Memory limit: 512 MB

Given $n$ integers, your task is to report for each integer the number of its divisors.
For example, if $x=18$, the correct answer is $6$ because its divisors are $1,2,3,6,9,18$.

| Input | Output |
| --- | --- |
|The first input line has an integer $n$: the number of integers.<br/>
After this, there are $n$ lines, each containing an integer $x$.|For each integer, print the number of its divisors.|

Constraints
- $1 \le n \le 10^5$
- $1 \le x \le 10^6$

Example:
| Input | Output |
| --- | --- |
|3<br/>16<br/>17<br/>18|5<br/>2<br/>6|

以下我將我自己的解法和在網路上查到的解法全部整理在以下，並重述/重寫程式。

# 解法一：暴力的一個一個判斷是不是因數

來源：[CSES Problem Set — Counting Divisors 題解 - HackMD](https://hackmd.io/@winliu/Bkp2UPMop)

想法：依序判斷$1$~$x$中的數能否整除$x$
優化：若$k$為$x$的因數，則$\frac{x}{k}$亦為$x$的因數，所以實際上只要檢查到$\sqrt{n}$即可。

## AC Code

記得開long long

```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long

ll divnum(int x){
  int cnt=0;
  for(int i=1;i*i<=x;i++){
  if(x%i==0){
    cnt += 2;
  }
  if(i*i==x) cnt--; // 這時i=x/i，所以i會被多算一次，故把他扣掉
  }
  return cnt;
}

int main(){
  ios::sync_with_stdio(0), cin.tie(0);
  int n;
  cin >> n;
  while(n--){
  int x;
  cin >> x;
  cout << divnum(x) << "\n";
  }
}
```

## 複雜度

### 時間複雜度

跑一次`divnum(x)`需要$\Theta(\sqrt{x})$，查詢$n$次，設每次的$x$為$x_1,x_2,\cdots,x_n$，則總共的時間複雜度為
$$
\sum\limits_{i=1}^{n}{\Theta(\sqrt{x_i})}=O(n\sqrt{\max{x_i}})
$$

### 空間複雜度

只用了常數個變數，所以空間複雜度是$\Theta(1)$

# 解法二：依序把1~N的倍數的因數個數加一

來源：[CSES Problem Set — Counting Divisors 題解 - HackMD](https://hackmd.io/@winliu/Bkp2UPMop)~
想法：維護一個陣列，Index $n$表示$n$的因數個數。依以下步驟動態更新陣列內容
- 將$1$的倍數因數個數都$+1$
- 將$2$的倍數因數個數都$+1$
- 將$3$的倍數因數個數都$+1$
- 將$4$的倍數因數個數都$+1$

等等，一直到
- 將$N$的倍數因數個數都$+1$

## AC Code

```cpp
#include <iostream>
using namespace std;

const int N=1e6+1;

int divisorCount[N];
int main() {
  int n, x;
  for (int i=1; i<N; i++)
    for (int j=i; j < N; j += i)
      divisorCount[j] += 1;
  cin >> n;
  for (int i = 0; i < n; i++) {
    cin >> x;
    cout << divisorCount[x] << '\n';
  }
}
```

## 複雜度

### 時間複雜度

填完陣列需要
$$
\Theta(N\sum_{k=1}^{N}{\frac{1}{k}})=\Theta(N\log{N})
$$
而查詢故需要$\Theta(n)$，$n\leq N$
所以總複雜度為
$$
\Theta(N\log{N})
$$

### 空間複雜度

顯然是$\Theta(N)$。

# 解法三：


# 解法四：

# 解法五：線性篩求積性函數
