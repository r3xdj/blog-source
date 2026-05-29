---
title: Projects
date: 2026-05-26
update: 2026-05-27
layout: page
---

# 📦 Projects

這裡記錄了我開發的開源工具、軟體以及設計的 CTF 題目。

---

## 🖥️ Software & Tools Development

### 🛠️ cmd-protocol

一個讓使用者能直接透過瀏覽器網址列（自訂 Protocol Handler）觸發並執行 Windows 系統命令的輕量化工具。

- **GitHub Repository:** [r3xdj/cmd-protocol](https://github.com/r3xdj/cmd-protocol)
- **技術棧:** `Python` / `Windows Registry`
- **專案簡介:** 靈感來自[I made my own protocol](https://www.youtube.com/watch?v=46pBeyHKQuQ)，由於原影片未提供 `C` 的程式，我便自己用 `Python` 做一個。藉由註冊 Windows 登錄檔（`.reg`）實現了自訂的 `cmd://` 協定。當使用者在瀏覽器網址列輸入指令時，會自動調用後端的 Python 執行檔進行 URL Decode，並在跳出安全確認視窗後於本地端執行對應的終端機命令。

---

## 🚩 CTF Challenges

### 🎲 Misc

#### 📁 File Manager Series (Total: 3 Challenges)

一個圍繞著檔案管理系統設計的 Misc 題組。

- **Challenge Repo:** [r3xdj/file-manager-challs](https://github.com/r3xdj/file-manager-challs)
- **題目狀態:**
  - 🛠️ Challenge: **Released**
  - 📝 Official Writeup: [File Manager Official Writeup | R3X's Blog](https://r3xdj.github.io/blog/2026/05/27/File-Manager-Official-Writeup/)
- **難度：**
  - **File Manager 1:** Easy
  - **File Manager 2:** Hard
  - **File Manager 3:** Hard

### 🕵️‍♀️ Forensics

#### 🖼️ Find My Size

考驗在 `png` 圖片的長、寬與 CRC 循環冗餘校驗都被修改後，如何還原出原始正確長寬。

- **Challenge Repo:** [r3xdj/FindMySize-chall](https://github.com/r3xdj/FindMySize-chall)
- **題目狀態:**
  - 🛠️ Challenge: **Released**
  - 📝 Official Writeup: [Find My Size Official Writeup | R3X's Blog](http://r3xdj.github.io/blog/2026/05/28/Find-My-Size-Official-Writeup/)
- **難度：** Medium

### 🔐 Crypto

#### 🧮 Eisenstein's RSA

在非整數環上的 RSA 題目

- Challenge Repo: [r3xdj/Eisenstein-s-RSA-chal](https://github.com/r3xdj/Eisenstein-s-RSA-chal)

- **題目狀態:**
  
  - 🛠️ Challenge: **Released**
  
  - 📝 Official Writeup: ✒️**Writing**

- **難度:** Hard