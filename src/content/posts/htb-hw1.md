---
title: Hack The Box Hardware Challenge 1 Writeup
published: 2024-02-19
description: 'Discussion on solving Hardware Challenge 1 - Debugging Interface on HTB'
image: './covers/htb-hardware-1.png'
tags: ["Hardware", "Cyber Security"]
category: Writeups
draft: false 
---

# Preliminary Steps

First, to begin with the basic challenge description and downloads. After which comes the analysis of the files themselves. What came in the .zip is simply a .sal file which is readable via a program called Logic.

![Files after unpacking](@assets/htb-photos/screencaps/htb-hardware-1/sc1.png)

After looking around here is the installation as well the documentation, for Logic. As was noted from the description we’ll be using async.
- https://support.saleae.com/logic-software/sw-installation#ubuntu-instructions
- https://support.saleae.com/protocol-analyzers/analyzer-user-guides/using-async-serial

# File(s) Analysis

Opening the .sal (capture file) we see this:

![.SAL contents](@assets/htb-photos/screencaps/htb-hardware-1/sc2.png)

Now, noting the Analyzers tab on the right side of the application, I’ll enter in the default values given via the documentation:

![Analyzer Values](@assets/htb-photos/screencaps/htb-hardware-1/sc3.png)

Which yields:

![Analyzer Output](@assets/htb-photos/screencaps/htb-hardware-1/sc4.png)

And in the data window:

![Analyzer Output in the Data window](@assets/htb-photos/screencaps/htb-hardware-1/sc5.png)

Noting that most of the data and values doesn’t make much sense, the documentation states that this is due to the baud rate. So, using an extension that somewhat “auto” measures the baud rate will yield this:

![Baud Rate with Extension](@assets/htb-photos/screencaps/htb-hardware-1/sc6.png)

So, now to change the baud rate in order to see what changes:

![Changed Rate](@assets/htb-photos/screencaps/htb-hardware-1/sc7.png)

Which changes the data to what is needed for the flag:

![Flag](@assets/htb-photos/screencaps/htb-hardware-1/sc8.png)
