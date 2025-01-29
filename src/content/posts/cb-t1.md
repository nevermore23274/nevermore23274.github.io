---
title: Codebreaker 2024 - Task 1
published: 2024-10-03
description: 'Writeup for Codebreaker Challenge - Task 1'
image: './covers/cb-t1.png'
tags: ["File Forensics", "Cyber Security"]
category: Writeups
draft: false 
---

# Programs/Applications Used
- Kali Linux Container (Docker)
- Libre Office
- `file`
- `pandas`

# Useful Links
- https://www.libreoffice.org/discover/what-is-opendocument/

# Preliminary Steps

The first step to begin with is reading challenge description, checking out the downloads, and making a container for tooling. To begin with the environment:

![Docker container](@assets/codebreaker2024-photos/task1/sc0.png)

To build the container and be dropped into it, you can use: `docker build -f docker/Dockerfile -t kali-env-t1 . && docker run -it --rm -v "$(pwd)":/usr/project kali-env-t1`. Some might disagree with the `--break-system-packages` flag, but for a test environment I see no issue with it.

# File Analysis

Then to look over the actual file, you can use `file` just to see if anything is off about it.

![The basic file analysis](@assets/codebreaker2024-photos/task1/sc1.png)

This generally means it's a zip archive with a MIME type, this can yield towards being an OpenDocument format.

# File Analysis and Conversion

So the first step was to simply open up the file and look over it. Opening it directly led to it looking strange, so instead did `cp shipping.db shipping.odb` so that I could open it nicely in Libre Calc. Once opened, I noted that there's 10 columns and over 1000 rows.

![LibreOffice Calc analysis](@assets/codebreaker2024-photos/task1/sc2.png)

As there's no way I'm going to look over all of this and notice one cell off, I decided to save the file as CSV and use `pandas` to analyze the data for me. Column 9 is the one actually needed for the challenge, however the first and second are the ones most likely to showcase the strange behavior.

# Script Creation and Execution

I wrote this script in order to analyze the file and build out a simple output of the data:

![Script for analysis](@assets/codebreaker2024-photos/task1/sc3.png)

This is a pivot table, meaning it creates a matrix showing counts of occurences for each combination of the first and second column entries. (note that the script also saves a copy of the pivot table for later looks) Once you run this script, it yields:

![Script output](@assets/codebreaker2024-photos/task1/sc4.png)

Which the main take away is that Guardian Armaments pops up twice, and one of those addresses only comes up in the entire database once. Using the search function in LibreOffice Calc for the address tells me it's on line 655.

![Answer](@assets/codebreaker2024-photos/task1/sc5.png)

As it's noted, the answer to the actual question is: `GUA0930252`
