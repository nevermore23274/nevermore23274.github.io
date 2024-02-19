---
title: Hack the Box Reverse Engineering Challenge 1 Writeup
published: 2024-02-19
description: 'Discussion on solving Reverse Engineering Challenge 1 - Behind the Scenes on HTB'
image: './covers/htb-re-1.png'
tags: ["Reverse Engineering", "Cyber Security"]
category: Writeups
draft: false 
---

# Programs/Applications Used
- Kali Linux
- Ghidra
- `strace`
- `ltrace`

# Useful Links
- https://stackoverflow.com/questions/18410344/program-received-signal-sigill-illegal-instruction
- https://cplusplus.com/reference/cstring/strncmp/
- https://cplusplus.com/reference/cstring/strlen/
- https://osandamalith.com/2019/02/11/linux-reverse-engineering-ctfs-for-beginners/

# Preliminary Steps

The first step to begin with is reading challenge description and checking the downloads. We have a binary file, so can start by just trying to run it.

![Binary after unpacking and initial zip](@assets/htb-photos/screencaps/htb-re-1/sc1.png)

# Binary Analysis

Upon trying to run the file, we simply get another prompt.

![Binary run](@assets/htb-photos/screencaps/htb-re-1/sc2.png)

So now to use `strace` and `ltrace` to see what they give:

![Trace command outputs](@assets/htb-photos/screencaps/htb-re-1/sc3.png)

Seeing this `SIGILL`, I looked up what that means (see above) and then opened up the executable in Ghidra and headed to the main().

![Initial Ghidra run](@assets/htb-photos/screencaps/htb-re-1/sc4.png)

Here I saw the `UD2` mentioned in the stackoverflow post, so began to decompile each one (Right click, decompile):

![Decompile step](@assets/htb-photos/screencaps/htb-re-1/sc5.png)

At this point, while decompiling each one, I noticed the `strlen` function, and then noticed the `strncmp` external call as well below this. I noticed that in the first `strncmp` function it had the plain text ‘Itz’ at the end of the variable as well:

![Functions](@assets/htb-photos/screencaps/htb-re-1/sc6.png)

![Functions](@assets/htb-photos/screencaps/htb-re-1/sc7.png)

Going down each `strncmp`, I also found the print that states what format the flag would be printed in:

![Flag found](@assets/htb-photos/screencaps/htb-re-1/sc8.png)

After doing this, the flag came out to be:

`HTB{Itz_0nLy_UD2}`
