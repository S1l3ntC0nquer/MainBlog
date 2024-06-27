---
title: Cryptography Notes
mathjax: true
thumbnail: /images/cryptography.webp
categories:
    - [CyberSec, CTF]
tags:
    - Cryptography
    - CTF
    - CyberSec
    - Notes
---

{% notel red 公告 %}
這篇文章會先發布公開，但是有內容都還在完善中，等補完內容後會把這個公告撤掉。
{% endnotel %}

# 前言

## 為甚麼會想寫這篇文

因為自己在學習的時候一直覺得密碼學是個很困難的領域，所以決定自己把一些筆記給記錄下來。一來是幫助自己以後可以回來翻，其次也希望可以幫助到正在學習密碼學的人！加油啦！

## 柯克霍夫原則 Kerckhoffs's principle

> "A cryptosystem should remain secure even if an adversary knows all the details of the system, except for the secret decryption key." — **_Kerckhoffs’s principle_**

柯克霍夫原則是由 Auguste Kerckhoffs 在 19 世紀提出的，其內容主要就是說明一個好的加密系統即使其運作原理全部都是公開的，只要金鑰未公開，他就應該要是安全的。資訊理論的創始人 Claude Shannon 將其改說為以下說法。

> "The enemy knows the system" — **_Claude Shannon_**

也就是說，永遠預設敵人知道你的整個加密系統是如何運作的，包括其中的算法以及各種原理。一個加密系統要在這樣的情境下還能保證其加密資料的安全性才稱得上是安全的。

## 領域展開：密碼學的三大領域

在密碼學中的任何知識幾乎都是圍繞著**編碼**、**雜湊**、**加密**這三個領域，所以就先來分別簡單聊聊他們各自都是甚麼吧！

### 編碼 Encoding

編碼的目的是為了資料的傳輸或表達，把原本的資料**換一種方式**表達而已。因為他只是把資料轉換成另一種形式，所以編碼後的資料仍然可以被解碼（Decode），也就是說整個過程完全是**可逆的**。

所以這裡要特別注意。**編碼不是加密！編碼不是加密！編碼不是加密！**

### 雜湊 Hashing

雜湊，又名哈希（對就是直接從 Hash 直接翻過來）。雜湊的話主要是用於數據校驗以及數據完整性的驗證，還有一些其他的安全應用比如數位簽章等。雜湊算法會有以下的特性：

1. **不可逆性**：雜湊的算法是不可逆的，無法透過已知的雜湊值（哈希值 Hash values）回推原始資料。
2. **固定長度輸出**：不管輸入的數據長度為何，哈希函式都會生成固定長度的哈希值。比如 SHA-256 的輸出長度就是固定 256 個 Bits。
3. **唯一性**：理想情況下，兩個不同的輸入會產生不同的輸出（避免碰撞）。

### 加密 Encryption

加密就相對好理解啦！就是用於保護數據的機密性（Confidentiality），確保只有被授權的用戶才能訪問數據。其加密和解密的過程都**依賴於密鑰**（Key），密鑰的保密性也會直接的影響到數據的安全性。

就好比是一個保險箱，要打開它需要一把鑰匙，而那把鑰匙就是密鑰。如果密鑰被盜取，那麼保險箱也就毫無用處了。

# 常見編碼 Common Encoding

## ASCII Encoding

他的全名是 American Standard Code for Information Interchange，反正不是很重要，就是一個挺常見的編碼方式。以下是一張 ASCII 表：

## Base64 Encoding

### 簡介

Base64 通常用於傳輸二進制數據的場合，例如在電子郵件中嵌入圖像、文件等。它的範圍是從使用`A-Z`、`a-z`、`0-9`，共 62 個字符，加上兩個額外字符`+`和`/`，共 64 個字符。

### 原理

由於 $\log_{2}64=6$，所以每 6 個

## URL Encoding

# 常見雜湊函式 Common Hash Functions

# 古典密碼學 Classical Cryptography

# 對稱式加密 Symmetric Cryptography

# 非對稱式加密 Asymmetric Cryptography

# 公鑰加密 Public-key Cryptography
