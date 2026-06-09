---
title: Kali Linux Setup For CTF
date: 2025-04-06 07:54:17
updated: 2026-06-09 11:32:42
categories: 
  - Cyber
tags:
  - CTF
  - Kali Linux
description: 在Kali Linux上安裝/設定CTF常用的工具
cover: /img/Kali.jpg
top_img: /img/Kali.jpg
---

# Kali Linux Setup for CTF
https://www.kali.org/docs/general-use/metapackages/

```bash
sudo apt update
sudo apt full-upgrade -y
sudo kali-tweaks
```
接著選擇需要的元件，可以直接選`kali-linux-everything`全裝
> 但這空間也是占有點多，還是要考慮一下w

# General
```bash=
#解壓縮軟體
sudo apt install -y lzip lz4 lzop 7zip unar
```
```bash=
sudo apt install lsd bat fzf silversearcher-ag libarchive-tools ffmpeg audacity npm -y
#lsd:ls加強版 #bat:cat加強版 #fzf:檔案搜尋 #ag:faster than grep
```
加入
```bash
alias bat=batcat
```
至`~/.zshrc`

加入
```bash
bindkey -s '\e\e' '\C-asudo \C-e'
```
至`~/.zshrc`，如此便可按兩下`esc`快速在指令前加入`sudo`

- Lazyvim
1. 安裝字形
```bash=
cd ~
mkdir .fonts
cd .fonts
wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf
wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf
wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf
wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf
sudo fc-cache -f -v
```
重新啟動後更改終端機字體
```bash=
# 安裝neovim
sudo apt install neovim -y
# 安裝Lazyvim
# required
mv ~/.config/nvim{,.bak}

# optional but recommended
mv ~/.local/share/nvim{,.bak}
mv ~/.local/state/nvim{,.bak}
mv ~/.cache/nvim{,.bak}
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
nvim
```
- 中文輸入法
```bash=
#裝字體
sudo apt install fonts-arphic-uming fonts-noto-cjk -y
#裝框架、輸入法
sudo apt install fcitx5 fcitx5-chewing zenity im-config -y
sudo  apt install fcitx5-frontend-gtk2 fcitx5-frontend-gtk3 fcitx5-frontend-gtk4 fcitx5-frontend-qt5 fcitx5-frontend-qt6 -y
#設定
sudo im-config
```
然後要進fcitx5設定把原住民移掉改加入英文(美式)，不然單引號會打成其他字元

參考：[打造 Kali Linux 2021 中文桌面環境(字型、中文輸入法) (Kali Linux 2024 也適用)](https://hack543.com/kali-linux-2021-tradictional-chinese-env/)

- wxHexeditor
```bash=
sudo apt install wxhexeditor -y
```
- GHex
```bash=
sudo apt install ghex -y
```
- GIMP(修圖軟體)
```bash
sudo apt install gimp -y
```
- zbarimg (QR code 解析)
```bash
sudo apt install zbar-tools python3-zbar -y
```

# Web
- [r3dir](https://github.com/Horlad/r3dir): 重新導向用以繞過SSRF
```bash
pipx install r3dir
```

## Firefox extension
- [Tampermonkey](https://addons.mozilla.org/zh-TW/firefox/addon/tampermonkey/): 執行js腳本
- [Cookies-Editor](https://addons.mozilla.org/zh-TW/firefox/addon/cookie-editor/): Cookies編輯器
- [DotGit](https://addons.mozilla.org/zh-TW/firefox/addon/dotgit/): .git資料夾洩漏檢查
- [Absolute Enable Right Click & Copy](https://addons.mozilla.org/en-US/firefox/addon/absolute-enable-right-click/)
- [HackTools](https://github.com/LasCC/HackTools): Web滲透測試工具
- [Wappalyzer](https://addons.mozilla.org/en-US/firefox/addon/wappalyzer/): 分析網站組成

# Crypto

# Forensic
```bash=
sudo apt install -y foremost # kali-linux-everything有
```
- volatility3
https://pypi.org/project/volatility3/
```bash
pipx install volatility3
```

## zip-cen-op (zip偽加密工具)
```bash
pipx install zip-cen-op
```

## 隱寫術
```bash=
sudo apt install steghide -y
gem install zsteg
sudo apt install stegcracker -y
```
加入
`export PATH=/home/kali/.local/share/gem/ruby/3.3.0/bin:$PATH`至`~/.zshrc`
### 安裝stegsolve
```bash=
sudo mkdir /opt/Stegsolve
cd /opt/Stegsolve
sudo wget http://www.caesum.com/handbook/Stegsolve.jar -O stegsolve.jar
sudo nano /usr/local/bin/stegsolve
```
加入
```bash=
#!/bin/bash
java -jar /opt/Stegsolve/stegsolve.jar
```
然後
```bash=
sudo chmod +x /usr/local/bin/stegsolve
sudo chmod +x /opt/Stegsolve/stegsolve.jar
```
- 加入menu
```
sudo nano /usr/share/applications/stegsolve.desktop
```
加入
```desktop=
[Desktop Entry]
Version=1.3
Name=Stegsolve
Comment=Steganography Image Analysis Tool
Exec=java -jar /opt/Stegsolve/stegsolve.jar
Icon=accessories-image-viewer
Terminal=false
Type=Application
Categories=kali-forensics;
```
```bash
sudo update-desktop-database
```

# Pwn
- pwntools
```bash
sudo apt install python3-pwntools -y
```

## GDB
```bash
sudo apt install gdb -y
```
### plugin
- gef: https://github.com/hugsy/gef
- pwndbg: https://github.com/pwndbg/pwndbg
```bash=
git clone https://github.com/pwndbg/pwndbg ~/pwndbg
cd ~/pwndbg
./setup.sh
```
> 調整`~/.gdbinit`來決定要使用哪個(把不用的註解掉)

# Reverse
```bash=
sudo apt install ghidra -y # kali-linux-everything有
```

# Misc
- SSTV Decoder
https://github.com/colaclanth/sstv

## OSINT
- [Geowifi](https://github.com/GONZOsint/geowifi)
- [locatebssid]()https://github.com/kjbaker-uk/locatebssid
