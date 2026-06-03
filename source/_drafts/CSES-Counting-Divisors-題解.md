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

# 解法三：建質數表，再用標準分解式求因數個數

這個是我的解法，但很醜。我的想法是先建一個質數表，再看$x$分別被每個質數整除幾次(即$\nu_p(x)$)
則所求為
$$
\sum\limits_{i}(\nu_{p_i}(x)+1)
$$

> 如果你不知道為什麼，請去問你高一數學老師

## 作法

### 質數表：埃拉托斯特尼篩法

篩質數一個快速又簡單的方法就是我們國一就學過的**埃拉托斯特尼篩法**
程式碼如下：

```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long

const int MAX_X = 1e6;

bool not_prime[MAX_X + 1] = {1,1};
int prime[MAX_X + 1];
int cnt = 0;

void make_prime_list(){
  int N = MAX_X;
  for(int i=2;i<=N;i++){
    if(!not_prime[i]){
      prime[++cnt] = i;
      for(int j=2*i;j<=N;j+=i) not_prime[j] = 1;
    }
  }
}

bool is_prime(int x){return !not_prime[x];}
```

其複雜度為$O(N\log\log N)$(好像要用到Mertens' Theorems，所以我不會證)

> 耶我終於能自己直接寫出埃氏篩了

### $\nu_p(x)$

計算$\nu_p(x)$的函式如下：

```cpp
int v(int p, ll &x){ // v代表\nu
  int ans = 0;
  while(x%p==0){
    x /= p;
    ans += 1;
  }
  return ans;
}
```

之所以用`&x`是為了讓這個函式能修改外部參數(也就是直接動態修改傳進去的$x$的值)，`&x`表示傳遞變數`x`的記憶體地址，而非單純複製其值。
單次執行的時間複雜度為$O(\log_p(x))$

### `divnum`

計算正因數個數

```cpp
ll divnum(ll x){
  if(x==0)return 0;
  if(x==1)return 1;
  ll ans = 1;
  for(int i=1;i<=cnt;i++){
    int p = prime[i];
    if(1LL*p*p>x)break;
    ans *= v(p,x)+1;
  }
  if(x>1)ans *= (1+1);
  return ans;
}
```

這邊解釋一下`if(x>1)ans *= (1+1);`這行。迴圈跳出時代表$x$不再有小於$\sqrt{x}$的正因數，這表示要嘛$x=1$，不然就是$x$本身就是一個質數。所以如果$x>1$，就代表$x=某質數^1$，故對正因數個數貢獻為$\times(1+1)$。

複雜度的部分，跑迴圈要時$O(\pi(\sqrt{x}))=O(\frac{\sqrt{x}}{\ln\sqrt{x}})$，而所有的$\nu_p(x)$呼叫加總不超過$O(\log_2{x})$，又不難證明
$$
\lim{x\to\infty}{\frac{\log_2{x}}{\frac{\sqrt{x}}{\ln\sqrt{x}}}}=0
$$
因此總複雜度為
$$
O(\frac{\sqrt{x}}{\ln\sqrt{x}})
$$


## AC Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long

const int MAX_X = 1e6;

bool not_prime[MAX_X + 1];
int prime[MAX_X + 1];
int cnt = 0;

void make_prime_list(){
  int N = MAX_X;
  for(int i=2;i<=N;i++){
    if(!not_prime[i]){prime[++cnt] = i;}
    for(int j=1;j<=cnt;j++){
      int p = prime[j];
      if(1LL*i*p>N) break;
      not_prime[i*p] = 1;
      if(i%p == 0)break;
    }
  }
  return;
}

bool is_prime(int x){return !not_prime[x];}

int v(int p, ll &x){ // v代表\nu
  int ans = 0;
  while(x%p==0){
    x /= p;
    ans += 1;
  }
  return ans;
}

ll divnum(ll x){
  if(x==0)return 0;
  if(x==1)return 1;
  ll ans = 1;
  for(int i=1;i<=cnt;i++){
    int p = prime[i];
    if(1LL*p*p>x)break;
    ans *= v(p,x)+1;
  }
  if(x>1)ans *= (1+1);
  return ans;
}

int main(){
  ios::sync_with_stdio(0), cin.tie(0);
  int n;
  cin >> n;
  make_prime_list();
  while(n--){
    int x;
    cin >> x;
    cout << divnum(x) << "\n";
  }
}
```

## 複雜度

### 時間複雜度

$$
O\left(N \log \log N + Q \cdot \frac{\sqrt{x_{\max}}}{\log \sqrt{x_{\max}}}\right)
$$

### 空間複雜度

$$
\Theta(n)
$$

> 真的很醜

# 解法四：建最大質因數表，再依序除以最大質因數求因數個數

來源：[【題解】CSES 1713 Counting Divisors – Yui Huang 演算法學習筆記](https://yuihuang.com/cses-1713/)
想法：ㄧ樣是用標準分解式，但在預處理時將`not_prime`中的值改為紀錄該數的**最大質因數**，然後一步步除掉，同時統計質因數的次方
比如$x=60$，步驟會是：
1. 令ans=1
2. $60$的最大質因數為$5$，目前有$5^1$，還剩$60/5=12$
3. $12$的最大質因數為$3$，和$5$不同，又因為每次都是除掉最大質因數，所以可知標準分解式中$5$的次方一定只有$1$次，故將ans乘以$1+1=2$，目前有$3^1$，還剩$12/3=4$ ($5$這個質因數處理完畢)
4. $4$的最大質因數為$2$，和$3$不同，所以再將ans乘以$1+1=2$，目前有$2^1$，還剩$4/2=2$ ($3$這個質因數處理完畢)
5. $2$的最大質因數為$2$，和$2$相同，所以目前有$2^2$，還剩$1$
6. 碰到$1$了，此時有$2^2$，所以將ans再乘以$2+1=3$，得到最終答案為$12$

## AC Code

> 跟他原版程式不太一樣，因為我有用自己的方法再寫一次
> 一是因為不知道這能不能直接轉載
> 二是我不會`vector`

```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1e6+1;

int max_prime_factor[N];
int main(){
  ios::sync_with_stdio(0), cin.tie(0);
  for(int i=2;i<N;i++){
    if(!max_prime_factor[i]){
      for(int j=i;j<N;j+=i) max_prime_factor[j]=i;
    }
  }
  int n;
  cin >> n;
  while(n--){
    int x, mp, ans=1, t=0;
    cin >> x;
    while(x!=1){
      mp = max_prime_factor[x];
      x /= mp; t+=1;
      if(max_prime_factor[x] != mp){
        ans *= t+1;
        t=0;
      }
    }
    cout << ans << "\n";
  }
}
```

## 複雜度

### 時間複雜度

預處理需要$\Theta(N\log\log N)$(等我哪天學了Mertens' second theorem不然我還是不會證)
計算正因數個數需要$O(n\log{x_{max}})$
所以總時間複雜度為
$$
O(N\log\log N + n\log{x_{max}})
$$

### 空間複雜度

顯然是
$$
\Theta(N)
$$

# 解法五：線性篩求積性函數

這是裡面時間複雜度最低的做法，十分驚人
> 我搞了好久才弄懂……

來源：[筛法 - OI Wiki](https://oi-wiki.org/math/number-theory/sieve/#%E7%AD%9B%E6%B3%95%E6%B1%82%E7%BA%A6%E6%95%B0%E4%B8%AA%E6%95%B0)

## 線性篩

## AC Code

## 複雜度

### 時間複雜度

### 空間複雜度

# 後記
