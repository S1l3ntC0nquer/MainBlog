---
title: Cryptography Notebook 密碼學任督二脈
mathjax: true
categories:
    - Notebooks
tags:
    - Cryptography
    - CTF
    - CyberSec
    - Notes
thumbnail: /images/cryptography.webp
date: 2024-06-27 20:05:01
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

### 簡介

這應該大家都很熟悉吧！他的全名是 American Standard Code for Information Interchange，反正不是很重要，就是一個挺常見的編碼方式。以下是一張 ASCII 表：

![ASCII Table from GeeksforGeeks](https://hackmd.io/_uploads/rJ6NTsq8A.png)

### Python Code

```python=
# 將字符串轉換為 ASCII 編碼
def ascii_encode(input_string):
    return [ord(char) for char in input_string]

# 將 ASCII 編碼轉換回字串
def ascii_decode(ascii_codes):
    return ''.join(chr(code) for code in ascii_codes)

# 示例
input_string = "This is ASCII encoding"
ascii_encoded = ascii_encode(input_string)
ascii_decoded = ascii_decode(ascii_encoded)

print("Original string:", input_string)
print("ASCII encoded:", ascii_encoded)
print("ASCII decoded:", ascii_decoded)
```

## Base64 Encoding

### 簡介

Base64 通常用於傳輸二進制數據的場合，例如在電子郵件中嵌入圖像、文件等。它的範圍是從使用`A-Z`、`a-z`、`0-9`，共 62 個字符，加上兩個額外字符`+`和`/`，共 64 個字符。

### 原理

Base64 編碼這個名稱代表著它**基於 64 個可列印字元**所形成的編碼。由於 $\log_{2}64=6$，所以每 6 個位元（Bit）為一個基本單元，對應著一個可列印字元。每 3 個位元組（Byte）為 24 個位元，相當於 4 個 Base64 基本單元，代表每 3 個位元組可以由 4 個可列印字元表示。下圖就是每個可列印字元所對應的索引值。

![Base64 encoding table from GeeksforGeeks](https://hackmd.io/_uploads/r1KgQh5LR.png)

### Python Code

```python=
import base64


def base64_encode(data):
    # 使用 base64 模組的 b64encode 函式進行編碼
    encoded_bytes = base64.b64encode(data)
    # 將編碼後的位元組轉換為字串並回傳
    return encoded_bytes.decode("utf-8")


def base64_decode(encoded_data):
    # 將 Base64 編碼的字串轉為位元組
    encoded_bytes = encoded_data.encode("utf-8")
    # 用 base64 的 b64decode 解碼
    decoded_bytes = base64.b64decode(encoded_bytes)
    # 回傳解碼後的字串
    return decoded_bytes.decode("utf-8")

# 測試
data_to_encode = b"This is Base64 encoding"

encoded_data = base64_encode(data_to_encode)
print("Base64 Encoded Data:", encoded_data)

decoded_data = base64_decode(encoded_data)
print("Base64 Decoded Data:", decoded_data)
```

### 延伸

除了 Base64 編碼以外，這個 Base 家族還有許多例如 Base16（Hex）、Base32、Base58（用於 Bitcoin）等不同的編碼方式。如果有興趣的話歡迎閱讀[這篇文章](https://blog.csdn.net/Sciurdae/article/details/133642336)。

## URL Encoding

### 簡介

URL Encoding 也稱作為 Percent-encoding，是一種將 URL 中的特殊字符和非 ASCII 字符轉換為百分號（%）後跟兩個十六進制數字的形式，以確保這些字符在 URL 中能夠被正確解析和傳輸。

### Python Code

```python=
import urllib.parse

# URL 編碼
def url_encode(input_string):
    return urllib.parse.quote(input_string)

# URL 解碼
def url_decode(encoded_string):
    return urllib.parse.unquote(encoded_string)

# 示例
input_string = "This is URL encoding"
url_encoded = url_encode(input_string)
url_decoded = url_decode(url_encoded)

print("Original string:", input_string)
print("URL encoded:", url_encoded)
print("URL decoded:", url_decoded)
```

## 小結

這裡面其實在打 CTF 的時候最常用到的就是 Base64 了，所以其實只要熟悉一下 Base64 的原理還有代碼，在比賽的時候可以快速編碼解碼就可以啦！

# 常見雜湊函式 Common Hash Functions

## MD5

## SHA-256

# 古典密碼學 Classical Cryptography

# 對稱式加密 Symmetric Cryptography

# 非對稱式加密 Asymmetric Cryptography
