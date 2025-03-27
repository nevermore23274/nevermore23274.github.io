---
title: Hack the Box Cyber Apocalypse CTF 2025 - SealedRune
published: 2025-03-21
description: 'The first reverse engineering challenge for the CTF'
image: './covers/htb-ctf-2025-re-sealedrune.png'
tags: ["Reverse Engineering", "Cyber Security"]
category: Writeups
draft: false 
---

# Programs/Applications Used

- Kali Linux (virtual machine)
- Radare2

# Useful Links

- https://book.rada.re/intro/intro.html

# Preliminary Steps

I use Arch as my host system but decided I'd like to try using a virtual machine as opposed to a Docker image this time. So I setup [Quickemu](https://github.com/quickemu-project/quickemu) and then [Quickgui](https://github.com/quickemu-project/quickgui) so that I'd be able to have a Kali Linux virtual machine.

`radare2` comes preinstalled so there was no need to install anything on the virtual machine.

# Analysis

The challenge gives a single downloadable file, so first I get some basic information about the file:

![File information](@assets/htb-ctf-2025/sealedrune/1.png)

I note that the executable isn't stripped, meaning it still contains all symbol information. (function names, variable names, source file references, debugging symbols, etc.) For some context, a "stripped" binary would be like a book with all the chapter titles and headings removed, so just a wall of text. So what we have is an executable file that should be relatively easy to parse through.

So next will be to simply run the file and see what happens.

![Initial run](@assets/htb-ctf-2025/sealedrune/2.png)

I pressed enter a few times to see if that'd affect anything, but nothing happened. So in the end, it essentially looks for a specific input before giving something. (in this case a refusal) So now to fire up `radare2` and see what's going on inside.

I'll use the command `r2 -d challenge` to start and then `aaa` in order to analyze everything in the binary. I'll follow that up with `afl` to list all the functions that are found.

![Radare2 analyze and list](@assets/htb-ctf-2025/sealedrune/3.png)

While `afl` does generate a lot of output, something of note would be the `main` function. So, I'll set a breakpoint at `main` and begin debugging.

I start with `dc` to execute until the breakpoint and then use `pdf` to disassemble the `main` function. But I notice something in `main`:

![Main function](@assets/htb-ctf-2025/sealedrune/4.png)

While I'm still unsure what it does, `anti-debug` sounds pretty ominous as a default. But I'll keep it in mind. For now, I'll continue on with checking the registers(`ds`) and memory at the register (`px`):

![Registers](@assets/htb-ctf-2025/sealedrune/5.png)

I didn't see much of interest. But looking back at the main function I can at least see what it's doing: First the program calls an anti-debugging function. It then will call `display_rune` which is the message asking for my input. It uses `call sym.imp.__isoc99_scanf` to take the user input, then verifies it with `call sym.check_input`. Given that I seek to find what answer is expected of me, I'd likely want to check what is inside `check_input`. So I'll navigate (seek) to the function and disassemble.

![Decode secret](@assets/htb-ctf-2025/sealedrune/6.png)

And right off the bat, I see a function named `sym.decode_secret` that returns a pointer (in `rax`) to a dynamically allocated string. User input (stored in `[var_18h]` is then compared to this decoded secret string via `strcmp`. If the input matches, it gives a print statement then calles `sym.decode_flag`, if it fails it gives a different print statment.

So, now to set a breakpoint at `sym_decode_secret()` and see what we can find.

![Debug stop](@assets/htb-ctf-2025/sealedrune/7.png)

But upon trying to continue execution, a strange message is given. This is likely the `anti-debug` function found earlier. So, the next step will be to patch the `sym.anti_debug` function with `NOP` instructions so that it does nothing.

I'll seek to the `anti_debug` function and write an assembly instruction `ret` at the location. Then I should be able to set a breakpoint at `check_input` and continue until the next breakpoint.

![Patch](@assets/htb-ctf-2025/sealedrune/8.png)

This seems to have worked, and it now hit the `sym.check_input` after me giving it a random phrase. I should now be able to continue through the breakpoint up to the `decode_secret`. Once there, I can display the value of the `rax` register and hopefully see what the string is.

![Failed](@assets/htb-ctf-2025/sealedrune/9.png)

But I notice that the `rax` is filled with zeros and is empty. What may have happened is `sym.decode_secret` was ran in isolation by `radare2` and thus stepped all the way to the end without the necessary state that would normally be setup with previous instructions. So instead, I'll try to fully exectue it in context rather than skipping directly to its return address.

First I'll close out the `radare2` instance and rerun it on the binary so it clears all breakpoints. Then get set back to the `sym.check_input` breakpoint specifically:

![Reset](@assets/htb-ctf-2025/sealedrune/10.png)

Now that things are set back up, I decided to use `call sym.decode_secret` and then call/break on it explicity. Once it hit the breakpoint, I started using `ds` to carefully step through instructions, but didn't see anything. So instead I opted to use `dcr` which continues execution until it returns. Upon doing so, it now gives me the secret string point in `rax` which means I just needed to display the `rax` register and string.

![Secret answer](@assets/htb-ctf-2025/sealedrune/11.png)

With that answer, I can now rereun the binary and get the flag for completion.

![Flag](@assets/htb-ctf-2025/sealedrune/12.png)

The secret spell is `HTB{run3_m4g1c_r3v34l3d}`.
