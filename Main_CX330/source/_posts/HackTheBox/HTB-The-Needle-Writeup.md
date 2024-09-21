---
title: "[HTB] The Needle Writeup 🪡"
date: 2024-09-22 01:31:01
cover: >-
    https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/help-you-at-solving-hackthebox-htb-challenges-machines.png
categories:
    - - HackTheBox
    - - CTF
tags:
    - HTB
    - Hardware
    - HackTheBox 
---

# 0x00 Challenge Info

> As a part of our SDLC process, we've got our firmware ready for security testing. Can you help us by performing a security assessment?

# 0x01 Analyze

We will get a file called `firmware.bin`, and we can use *binwak* to extract the data from the bin file.

```bash
binwalk -e firmware.bin
```

And the operation will create a directory called `_firmware.bin.extracted`. So we can go in there and check what is inside. We will find out it's actually messy in there, that means we cannot easily get what we want.

Besides the file, we also got an IP from the challenge, let's connect to it to see what service is it.

```bash
┌──(kali㉿kali)-[~/CTF/_firmware.bin.extracted]
└─$ nc 94.237.59.63 59877 
��������
ng-1276942-hwtheneedle-czk1p-5786c5548b-gnm6v login: 
```

It's a login service! So we should find something about the login info.

```bash
grep -rn login ./_firmware.bin.extracted
```

![Result](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240922015003138.png)

If we check the result carefully, we can see that there's an username called `Device_Admin`, and also, the password is in the `$sign` variable. So the next step is to find the `sign` file.

```bash
┌──(kali㉿kali)-[~/CTF]
└─$ find ./_firmware.bin.extracted -name sign
./_firmware.bin.extracted/squashfs-root/etc/config/sign
```

That's it! We are ready to exploit it!

# 0x02 Exploit

Since we found the sign file, we can cat it out to peek it.

```bash
┌──(kali㉿kali)-[~/CTF]
└─$ cat ./_firmware.bin.extracted/squashfs-root/etc/config/sign
qS6-X/n]u>fVfAt!
```

So our password is `qS6-X/n]u>fVfAt!`. Let's connect to the server and login as `Device_Admin` & `qS6-X/n]u>fVfAt!`. Once we login the server, we can simply cat the flag.txt out!

# 0x03 Pwned

![Pwned](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240922015742719.png)

