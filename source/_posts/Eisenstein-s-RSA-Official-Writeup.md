---
title: Eisenstein's RSA Official Writeup (簡要版)
tags:
  - crypto
  - RSA
  - CTF
categories:
  - Cyber
  - CTF Writeup
description: 在非整數環上的RSA題目
date: 2026-05-29 16:03:17
updated: 2026-05-29 16:03:17
cover:
top_img:
---


# 題目資訊

- Challenge Repo: [r3xdj/Eisenstein-s-RSA-chal](https://github.com/r3xdj/Eisenstein-s-RSA-chal)

# Writeup

---

Eisenstein's RSA

難度：Medium

類別：Crypto

> 請直接執行 cipher ，針對其輸出進行破解。

---

我們先來看題目給的`cipher.py`

```python
#!/usr/bin/python3
import gmpy2
from Cryptodome.Util import number
from FLAG import flag

class eint:
    def __init__(self, a, b):
        self.a = gmpy2.mpz(a)
        self.b = gmpy2.mpz(b)
    def __add__(self, other):
        return eint(self.a + other.a, self.b + other.b)
    def __sub__(self, other):
        return eint(self.a - other.a, self.b - other.b)
    def __mul__(self, other):
        return eint(self.a * other.a - self.b * other.b,
                    self.a * other.b + self.b * other.a - self.b * other.b)
    def norm(self):
        return self.a ** 2 - self.a * self.b + self.b ** 2
    def conj(self):
        return eint(self.a - self.b, -self.b)
    def __mod__(self, other):
        num = self * other.conj()
        den = other.norm()
        q_a = (2 * num.a + den) // (2 * den)
        q_b = (2 * num.b + den) // (2 * den)
        q = eint(q_a, q_b)
        return self - (q * other)
    def __repr__(self):
        return f"({self.a} + {self.b}w)"

def pow_mod(base: eint, exp: int, mod: eint) -> eint:
    res = eint(1, 0)
    base = base % mod
    while exp > 0:
        if exp % 2 == 1:
            res = (res * base) % mod
        base = (base * base) % mod
        exp //= 2
    return res

def quick_generate_e_prime(nbits):
    while True:
        a = number.getRandomNBitInteger(nbits // 2)
        b = number.getRandomNBitInteger(nbits // 2)
        p = a**2 - a*b + b**2
        if number.isPrime(int(p)):
            return eint(a, b)

def encode_msg(s1: str, s2: str) -> eint:
    a = number.bytes_to_long(s1.encode())
    b = number.bytes_to_long(s2.encode())
    return eint(a, b)

def decode_msg(m: eint) -> tuple[str, str]:
    """
    Oops, my cat deleted this part :/
    """

def decrypt(c: eint, N: eint, d: int) -> tuple[str, str]:
    """
    Oops, my cat deleted this part :/
    """

if __name__ == "__main__":
    mid = len(flag) // 2
    flag1 = flag[:mid]
    flag2 = flag[mid:]

    p = quick_generate_e_prime(512)
    q = quick_generate_e_prime(512)
    N = p * q

    e1 = number.getPrime(128)
    e2 = number.getPrime(128)
    while gmpy2.gcd(e1, e2) != 1:
        e1 = number.getPrime(128)
        e2 = number.getPrime(128)

    # d1 = Hay who deleted this part?
    # d2 = It wasn't by my cat...

    phi = (p.norm() - 1) * (q.norm() - 1)
    m = encode_msg(flag1, flag2)
    c1 = pow_mod(m, e1, N)
    c2 = pow_mod(m, e2, N)
    print(f"N: {N}\ne1: {e1}\nc1: {c1}\ne2: {e2}\nc2: {c2}")
```

簡單來說，題目定義了一個新的數據類型`eint`，並基於它去做RSA。(不同於一般的RSA是在整數環上操作)。看加密主程式(第64~86行)可知這題考的是RSA的共同模數攻擊，也就是用同一個模數$N$，兩個不同的公鑰$e1, e2$對相同的密文$m$進行加密，此時我們不須知道私鑰$d$即可還原出明文$m$。

## 艾森斯坦整數

### 基本運算

`cipher.py`中定義的`eint`類型即為艾森斯坦整數$\mathbb{Z}[\omega]=\{a+b\omega:a,b\in\mathbb{Z}\}$，其中$\omega=\frac{1+\sqrt3i}{2}$為$x^3=1$的單位根(因此$\omega^2+\omega+1=0$)。

對於兩艾森斯坦整數$\alpha=a+b\omega,\beta=c+d\omega$，其加、減、乘、除、共軛、範數(Norm)分別定義如下(即`cipher.py`中`eint`類型的方法和魔術方法)：

| 運算  | 定義/計算                                                                                     |
| --- | ----------------------------------------------------------------------------------------- |
| 共軛  | $\bar{\alpha}=a+b\bar{\omega}=a+b\omega^2=(a-b)-b\omega$                                  |
| 範數  | $N(\alpha)=\alpha\bar{\alpha}=a^2-ab+b^2$                                                 |
| 加法  | $\alpha+\beta=(a+c)+(b+d)\omega$                                                          |
| 減法  | $\alpha-\beta=(a-c)+(b-d)\omega$                                                          |
| 乘法  | $\alpha\beta=(a+b\omega)(c+d\omega)=ac+(ad+bc)\omega+bd\omega^2=(ac-bd)+(ad+bc-bd)\omega$ |
| 除法  | $\frac{\alpha}{\beta}=\frac{\alpha\bar{\beta}}{N(\beta)}=$ 剩下的你自己算吧XD                     |

在艾森斯坦整數上一樣有帶餘除法、費馬小定理那些，所以一樣能在上面跑RSA

(先相信就好，因為其中涉及很多抽象代數的東西，我擇日再寫一篇文章詳細說明)

> 我們稱可以定義出帶餘除法的整域(即沒有零因子且具有乘法單位元的交換環)為歐幾里得整域，或歐式整域(ED)
> 
> 實際上，所有的歐式整域都可以在上面訂出一個RSA加密演算法

知道這件事情之後，那個`eint`就只是一個嚇人的外殼而已，我們只要照著傳統RSA的共模攻擊來打擊可。

> Hmm ~~這麼簡單 看來是該出個Revenge了~~

不過在那之前，我們要先說明艾森斯坦整數的帶餘除法實務上怎麼操作：

給定$a,b\in\Z[\omega]$，我們希望找到$q,r\in\Z[\omega]$使得$a=bq+r$且$N(r)<N(b)$。

(存在性和唯一性的證明我們一樣先跳過，這相當於證明$\Z[\omega]$是ED)

欲找出$q,r$，先計算$\frac{a}{b}=x+y\omega$，其中$x,y\in\R$，接著我們取$q$為和$\frac{a}{b}$最接近的艾森斯坦整數，$r=a-bq$。

### 模反元素

在艾森斯坦整數上，我們也可以仿造整數上的擴展歐幾里得算法來找出模反元素：

```python
def eint_invert(a: eint, m: eint) -> eint:
    # 找 a * x + m * y = 1
    r0, r1 = a, m
    x, y = eint(1, 0), eint(0, 0)
    while r1.a != 0 or r1.b != 0:
        q, r = divmod(r0, r1)
        r0, r1 = r1, r
        x, y = y, x - (q * y)

    # 此時 r0 是單位元 u ，是 [1, -1, w, -w, w^2, -w^2] 中其中一個
    # a * x ≡ u => a * (x * u_inv) ≡ 1
    u_inv = r0.conj()
    x = x * u_inv

    return x % m
```

值得注意的是，不同於整數Units只有$1$和$-1$，艾森斯坦整數的Units有$1, -1, \omega, -\omega, \omega^2, -\omega^2$六種，因此擴展歐幾里得演算法跑完得到的$ax\equiv u\pmod{N}$，$u$不一定是$1$。因此我們需要左右同乘以$u^{-1}$，得到我們真正要的模反元素是$xu^{-1}$。顯然對於艾森斯坦整數的六個Units，其模反元素即其共軛複數。

## 共模攻擊

在RSA中，當同一個密文$m$被相同的模數$N$，兩個不同的公鑰$e_1,e_2$加密(為了簡單起見，這裡假設$\gcd(e_1,e_2)=1$(`cipher.py`中也是這樣寫的))，得到$c_1,c_2$，我們可以在不知道私鑰$d$的情形下直接還原出明文。

因為

$$
c_1\equiv m^{e_1}\pmod{N}\\
c_2\equiv m^{e_2}\pmod{N}
$$

我們可以用擴展歐幾里得演算法找到$x,y\in\Z$使得$xe_1+ye_2=1$，如此

$$
m\equiv m^{xe_1+ye_2}\equiv c_1^xc_2^y\pmod{N}
$$

## exploit

理解了以上，就可以來寫exploit了。這裡的快速冪比`cipher.py`中多處裡了負指數的情形。

完整exploit如下：

```python
#!/usr/bin/python3
from pwn import *
import gmpy2
import re
from Cryptodome.Util.number import long_to_bytes

class eint:
    def __init__(self, a, b):
        self.a = gmpy2.mpz(a)
        self.b = gmpy2.mpz(b)
    def __add__(self, other):
        return eint(self.a + other.a, self.b + other.b)
    def __sub__(self, other):
        return eint(self.a - other.a, self.b - other.b)
    def __mul__(self, other):
        return eint(self.a * other.a - self.b * other.b,
                    self.a * other.b + self.b * other.a - self.b * other.b)
    def norm(self):
        return self.a ** 2 - self.a * self.b + self.b ** 2
    def conj(self):
        return eint(self.a - self.b, -self.b)

    def __divmod__(self, other):
        # 歐幾里得除法
        num = self * other.conj()
        den = other.norm()
        q_a = (2 * num.a + den) // (2 * den)
        q_b = (2 * num.b + den) // (2 * den)
        q = eint(q_a, q_b)
        r = self - (q * other)
        return q, r

    def __mod__(self, other):
        return self.__divmod__(other)[1]

    def __repr__(self):
        return f"({self.a} + {self.b}w)"

# 以擴展歐幾里得算法求模反元素
def eint_invert(a: eint, m: eint) -> eint:
    # 找 a * x + m * y = 1
    r0, r1 = a, m
    x, y = eint(1, 0), eint(0, 0)
    while r1.a != 0 or r1.b != 0:
        q, r = divmod(r0, r1)
        r0, r1 = r1, r
        x, y = y, x - (q * y)

    # 此時 r0 是單位元 u ，是 [1, -1, w, -w, w^2, -w^2] 中其中一個
    # a * x0 ≡ u => a * (x0 * u_inv) ≡ 1
    u_inv = r0.conj()
    x = x * u_inv

    return x % m

def pow_mod(base, exp, mod):
    if exp < 0:
        base = eint_invert(base, mod)
        exp = -exp
    res = eint(1, 0)
    base = base % mod
    while exp > 0:
        if exp % 2 == 1:
            res = (res * base) % mod
        base = (base * base) % mod
        exp //= 2
    return res

# --- 執行 Exploit ---

p = process(["python3", "cipher.py"])

def parse_eint(s: str) -> eint:
    nums = re.findall(r'-?\d+', s)
    return eint(nums[0], nums[1])

p.recvuntil(b"N: ")
N = parse_eint(p.recvline().decode())
p.recvuntil(b"e1: ")
e1 = int(p.recvline().strip())
p.recvuntil(b"c1: ")
c1 = parse_eint(p.recvline().decode())
p.recvuntil(b"e2: ")
e2 = int(p.recvline().strip())
p.recvuntil(b"c2: ")
c2 = parse_eint(p.recvline().decode())

log.info(f"N={N}\ne1={e1}\ne2={e2}\nc1={c1}\nc2={c2}")

# 共模攻擊
g, s, t = gmpy2.gcdext(e1, e2)
# m = c1^s * c2^t (mod N)
m_eint = (pow_mod(c1, s, N) * pow_mod(c2, t, N)) % N

# 還原字串
def decode_msg(m: eint) -> tuple[str, str]:
    a_num = int(m.a)
    b_num = int(m.b)
    return long_to_bytes(a_num).decode(errors='ignore'), long_to_bytes(b_num).decode(errors='ignore')

flag = "".join(decode_msg(m_eint))

success(f"Flag: {flag}")
```

跑完就可以得到Flag了！

```
ICYSTAR{EUclid34n_DoMAiN_I$-p0w3Rfu1!!!}
```

> 
