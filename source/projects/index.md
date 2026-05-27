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
一個圍繞著檔案管理系統設計的 Misc 題組，考驗解題者對系統邏輯與特定邊界條件的敏銳度。
- **Challenge Repo:** [r3xdj/file-manager-challs](https://github.com/r3xdj/file-manager-challs)
- **題目狀態:** - 🛠️ Challenge: **Released**
  - 📝 Official Writeup: **In Progress**
- **難度：**
  - **File Manager 1:** Easy
  - **File Manager 2:** Hard
  - **File Manager 3:** Hard

### 🕵️‍♀️ Forensics

#### 🖼️ Find My Size
考驗在 `png` 圖片的長、寬與 CRC 循環冗餘校驗都被修改後，如何還原出原始正確長寬。
- **Challenge Repo:** [r3xdj/FindMySize-chall](https://github.com/r3xdj/FindMySize-chall)
- **題目狀態:** - 🛠️ Challenge: **Released**
  - 📝 Official Writeup: **In Progress**
- **難度：** Medium

### 🔐 Crypto

#### 🧮 Eisenstein's RSA
將傳統的公鑰密碼學體系延伸至非傳統代數結構中的進階密碼學挑戰。
- **題目狀態:** 🧪 **Under Development**
- **題目簡介:** 本題嘗試將標準的 RSA 加密演算法移植到**艾森斯坦整數（Eisenstein Integers, $\mathbb{Z}[\omega]$）** 的複數平面的格點結構上。解題者需要探討在此代數結構下，傳統質因數分解與相關攻擊手法（如特定格基減少演算法或同餘性質）的質變與適用性。