---
title: picoCTF 2022 Forensics Writeup
date: 2025-04-06 12:04:46
updated: 2025-04-06 18:15:00
tags:
  - forensics
  - CTF
categories:
  - Cyber
  - CTF Writeup
cover: /img/picoCTF.png
top_img: /img/picoCTF.png
---

參考影片：[PicoCTF 2022 Forensics
](https://www.youtube.com/playlist?list=PL1H1sBF1VAKUOp_TVZiOTGt4nB74So8sv)

題目：
https://play.picoctf.org/practice?category=4&difficulty=2&originalEvent=70&page=1&search=

# Torrent Analyze
題目提供了一個pcap檔案，我們可以用wireshark進行封包分析
題目提到是進行BT下載(所以分析BT-DHT)，目標是找出下載檔名。在[維基百科](https://en.wikipedia.org/wiki/Mainline_DHT#Operation)上我們學到裡面會有一個SHA-1的infohash，所以我們在wireshark設定filter:`bt-dht contains "info_hash"`
可以看到有好幾個不同的Hash，哪個才是我們要的呢？根據提示4檔案為.iso檔案，大小很大，因此會傳較多的封包，故可判斷出我們要的hash是`e2467cbf021192c241367b892230dc1e05c0580e`
將其copy出來
![image](https://hackmd.io/_uploads/r1bN-p1A1x.png)
丟去搜尋就可以發現~~都是別人的write up~~檔案名稱為`ubuntu-19.10-desktop-amd64.iso`

- 參考資料：https://hackmd.io/@SBK6401/BynwYfvxT

# Loocky Here
用wget下載檔案，`cat anthem.flag.txt | ag "picoCTF"`就可找到

# Enhance!
題目給了一張svg檔案，用文字編輯器開啟可看出85~120行藏有Flag
(可搜尋大括號：`cat drawing.flag.svg | ag "{"`)
```svg=
         id="tspan3748">p </tspan><tspan
         sodipodi:role="line"
         x="107.43014"
         y="132.08942"
         style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;"
         id="tspan3754">i </tspan><tspan
         sodipodi:role="line"
         x="107.43014"
         y="132.09383"
         style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;"
         id="tspan3756">c </tspan><tspan
         sodipodi:role="line"
         x="107.43014"
         y="132.09824"
         style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;"
         id="tspan3758">o </tspan><tspan
         sodipodi:role="line"
         x="107.43014"
         y="132.10265"
         style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;"
         id="tspan3760">C </tspan><tspan
         sodipodi:role="line"
         x="107.43014"
         y="132.10706"
         style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;"
         id="tspan3762">T </tspan><tspan
         sodipodi:role="line"
         x="107.43014"
         y="132.11147"
         style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;"
         id="tspan3764">F { 3 n h 4 n </tspan><tspan
         sodipodi:role="line"
         x="107.43014"
         y="132.11588"
         style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;"
         id="tspan3752">c 3 d _ a a b 7 2 9 d d }</tspan></text>
```
接著需要將其取出，用以下指令可輸出，用ag抓出我們要的行
`cat drawing.flag.svg | ag "</tspan"`
輸出如下：
```
         id="tspan3748">p </tspan><tspan
         id="tspan3754">i </tspan><tspan
         id="tspan3756">c </tspan><tspan
         id="tspan3758">o </tspan><tspan
         id="tspan3760">C </tspan><tspan
         id="tspan3762">T </tspan><tspan
         id="tspan3764">F { 3 n h 4 n </tspan><tspan
         id="tspan3752">c 3 d _ a a b 7 2 9 d d }</tspan></text>
```
我們進一步將訊息截斷，只取我們要的部分
`cat drawing.flag.svg | ag "</tspan" | cut -d ">" -f2 | cut -d "<" -f1`
這裡`-d`參數用以指定截斷處的字元，`-f1`或`-f2`指定要留下前面那一段還是後面那一段
輸出如下：
```
p 
i 
c 
o 
C 
T 
F { 3 n h 4 n 
c 3 d _ a a b 7 2 9 d d }
```
接著要移除換行符與空白，我們可以用tr將他們刪除掉
`cat drawing.flag.svg | ag "</tspan" | cut -d ">" -f2 | cut -d "<" -f1 | tr -d " " | tr -d "\n"`
得到的便是flag

# St3g0
先用`file`檢查一下
```
pico.flag.png: PNG image data, 585 x 172, 8-bit/color RGBA, non-interlaced
```
沒什麼異狀
再用`exiftool`，也無異狀
使用`binwalk`得到輸出
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 585 x 172, 8-bit/color RGBA, non-interlaced
41            0x29            Zlib compressed data, default compression
```
使用`zsteg`，便得到我們的flag

# File types
用`flie`確認，發現副檔名`.pdf`是騙人的，他是一個`shell script`。用`bat Flag.pdf`確定這個script是沒問題的，我們chmod給權限並執行
```
x - created lock directory _sh00046.
x - extracting flag (text)
./Flag.pdf: 119: uudecode: not found
restore of flag failed
flag: MD5 check failed
x - removed lock directory _sh00046.
```
於是我們安裝`uudecode`
```bash
sudo apt install sharutils -y
```
再試一次，得到一個名為flag的檔案，用`file`確認發現為current ar archive
## ar
`man ar` 看如何將其解壓縮
`ar x flag`
## cpio
```
mv flag flag.cpio
cpio -i < flag.cpio
```
## bzip2
```
bzip2 -d flag
```
## Gzip
```
mv flag.out flag.gz
gzip -d flag.gz
```
## lzip
```
lzip -d flag
```
## lz4
```
mv flag.out flag.lz4
lz4 -d flag.lz4
```
## lzma
```
mv flag flag.lzma
lzma -d flag.lzma
```
## lzop
```
mv flag flag.lzop
lzop -d flag.lzop
```
## lzip
```
lzip -d flag
```
## xz
```
mv flag.out flag.xz
xz -d flag.xz
```
## Hex
終於，我們拿到了文字檔，`cat`後結果看起來是16進位
```
7069636f4354467b66316c656e406d335f6d406e3170756c407431306e5f
6630725f3062326375723137795f39353063346665657d0a
```
用xxd解碼`cat flag | xxd -p -r`即得flag。

> 一直解壓縮的部分跟bandit Level 12 -> 13 挺像的

# Eavesdrop
這題也是封包分析題，一樣丟進wireshark分析
> TCP協議會用三次握手建立連接，四次揮手段開連接
> Arp協議是用來取得目標ip的mac地址的
> DHCP協議用來分配ip
可以看這兩個影片：
> - [一条视频讲清楚TCP协议与UDP协议-什么是三次握手与四次挥手?](https://youtu.be/Iuvjwrm_O5g)
> - [一条视频讲清楚什么是ARP协议-ARP攻击又是什么](https://youtu.be/mOVWyogzKvY)

有以上先備知識，數據又沒加密，可以一邊看他們聊天內容一邊解題
所以劇情應如下：
## 劇情
### No.1~2
10.0.2.15請求ip位置
### No.3~4
為了傳送訊息，使用arp協議取得目標的mac位置
### No.5~11
No.5~7為10.0.2.15和10.0.2.4 TCP三次握手的過程
No.8~11是使用arp協議互相取得對方mac位置
### No.12~No.19

> 10.0.2.4:Hey, how do you decrypt this file again? #No.12
> 10.0.2.15:You're serious? #No.14
> 10.0.2.4:Yeah, I'm serious #No.16
> 10.0.2.15:\*sigh\* **openssl des3 -d -salt -in file.des3 -out file.txt -k supersecretpassword123** #No.18

通信過程中會有一些Arp，我們都知道他是幹嘛的，對解題沒什麼幫助所以之後都將跳過。

### No.24~37

> 10.0.2.4:Ok, great, thanks. #No.24
> 10.0.2.15:Let's use Discord next time, it's more secure. #No.26

(你也知道現在這樣不安全啊)
> 10.0.2.4:C'mon, no one knows we use this program like this! #No.28
> 10.0.2.15:Whatever. #No.30
> 10.0.2.4:Hey. #No.32
> 10.0.2.15:Yeah? #No.34
> 10.0.2.4:**Could you transfer the file to me again?**

看來是10.0.2.15接著會傳檔案給10.0.2.4，裡面便有本題的答案

### No.38~47
看起來是10.0.2.15向35.224.170.84發起HTTP請求，可能只是單純瀏覽別的網站或是要下載要傳給10.0.2.4的檔案(我不知道XD)
### No.48~
> 10.0.2.15:Oh great. Ok, over 9002? #No.48
> 10.0.2.4:Yeah, listening. #No.52

這裡的9002我不知道是什麼，可能是port之類的？

No.57 10.0.2.15傳了48bytes的data，之後說
> 10.0.2.15: Sent it #No.59

所以No.57發送的這個data應該就是他傳輸的檔案，我們要的東西
將其以Hex Stream複製出來，得到
```hex
53616c7465645f5f3c4b26e8b8f91e2c4af8031cfaf5f8f16fd40c25d40314e6497b39375808aba186f48da42eefa895
```

將其寫入檔案，並用No.18提到的解密方法解密
```bash=
#寫入檔案
echo "53616c7465645f5f3c4b26e8b8f91e2c4af8031cfaf5f8f16fd40c25d40314e6497b39375808aba186f48da42eefa895" | xxd -p -r > file.des3
#解密
openssl des3 -d -salt -in file.des3 -out file.txt -k supersecretpassword123
#查看
cat file.txt
```

# Redaction gone wrong
超簡單，當學生都做過的事
把pdf打開，用選取把黑框的部分反白就是答案

# Packets Primer
簡單題，一樣開wiresharp分析封包，點開第四個封包就是答案
複製出來後可用`tr -d " "`把空格去掉

# 
