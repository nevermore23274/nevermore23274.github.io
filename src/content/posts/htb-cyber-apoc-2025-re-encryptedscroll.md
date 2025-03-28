---
title: Hack the Box Cyber Apocalypse CTF 2025 - EncryptedScroll
published: 2025-03-21
description: 'The second reverse engineering challenge for the CTF'
image: './covers/htb-ctf-2025-re-sealedrune.png'
tags: ["Reverse Engineering", "Cyber Security", "HTB Cyber Apocalypse 2025"]
category: Writeups
draft: false 
---

# Programs/Applications Used

- Kali Linux (virtual machine)
- Radare2

# Useful Links

- https://book.rada.re/intro/intro.html

# Preliminary Steps

N/A

# Analysis

The challenge, same as the last, gives a single downloadable file, so first I get some basic information about the file:

![File information](@assets/htb-ctf-2025/encryptedscroll/1.png)

As mentioned in the first challenge, the only real thing of note here is that the executable isn't stripped. So it should be another relatively easy read.

So next will be to simply run the file and see what happens.

![Initial run](@assets/htb-ctf-2025/encryptedscroll/2.png)

I pressed enter a few times to see if that'd affect anything, but nothing happened. So in the end, it essentially looks for a specific input before giving something. (in this case a refusal) So now to fire up `radare2` and see what's going on inside.

I'll use the command `r2 -d challenge`. However, given how I solved the last challenge that was somewhat like this one, I think I've decided on a general way of handlig these types of files.

- Analyze binary: `aaa`
- List functions, locate entry/main: `afl`
    - If present, then patch anti-debug: with `s sym.anti-debug` and `wa ret`
- Set breakpoints at critical functions (like input checks): `db sym.check_input`
- Run program to hit breakpoints: `dc`
- Step into decoding functions with: `db sym.decode_secret`, `dc` and/or `dcr`
- Inspect returned values and pointers: `dr rax` and `ps @ rax`

So, let's give the chosen methodology a shot.

![Radare2 analyze and list](@assets/htb-ctf-2025/encryptedscroll/3.png)

So here it's seen that there's an `entry0` and `main` function, but also the `sym.anti_debug` that was seen in the previous challenge. So to continue our method, we'll patch the `sym.anti_debug`, then set a debug at `main` and execute to the function. Though I also noticed `sym.decrypt_message` from the screenshot that could be of interest to look over. So I'll go ahead and set a breakpoint there, then step through and inspect the strings.

![sym.decrypt_message strings](@assets/htb-ctf-2025/sealedrune/4.png)

I used: `aaa`, `s sym.anti_debug`, `wa ret` to essentially overwrite the debugger block. Then used `db sym.decrypt_message`, `dc` and input the word `test` to hit the breakpoint for analysis. Then used `ds` and `pdf` so we can start analyzing `sym.decrypt_message`.

So taking those strings I can write a quick little python script to decode it. The script being these 3 lines:

```python
encoded = "IUC|t2nqm4`gm5h`m5h`5s2uin4u2d~"
decoded = ''.join(chr(ord(c) - 1) for c in encoded)
print(decoded)
```

Which then outputs:

![Flag](@assets/htb-ctf-2025/sealedrune/5.png)
