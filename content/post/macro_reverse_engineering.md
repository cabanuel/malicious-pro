+++
title = "Watching My Macros"
date = "2024-04-18"
author = "NuclearFarmboy"
description = "For once I am trying to look at a malware dropper in Excel rather than trying to write it."
draft = true
+++

{{< image src="/img/macro_reverse_engineering/alwayshasbeen.png" alt="And always will be..." position="center" style="width: 500px; height:auto;" >}}

# Let's Reverse Engineer a Malicious Excel Macro!
### The Backstory
A few weeks ago I had some free time after playing enough *Helldivers II* for one evening (I totally didn't have to stop because I got a hand cramp), I decided to do my other computer hobby: CTFs. As you may all know, a Capture The Flag (CTF) competition is an event where you can put your computer skills to the test and solve fun(?) challenges and potentially win cool prizes! In my case I am not a very competitive person so I do these more often than not to just stretch the ol' brain and learn something cool, try some new technique, or to just muck about because I got a hand cramp playing *Helldivers II*. 

##### (This post was not sponsored by *Helldivers II*, Super Earth, or Managed Democracy) 

This particular weekend I was looking at the ["UTCTF 2024"](https://www.isss.io/utctf/) competition despite the fact that I, as a Fightin' Texas Aggie, refuse to acknowledge that the University of Texas at Austin is a place of higher education. Regardless of these religious beliefs, I figured it would be fun to do a challenge or two, or at least attempt to. 

### The Challenge
This CTF was done jeopardy style, which meant that instead of doing some attacking and/or defending services or hosts in a live network, the game masters instead have a board of challenges separated by categories where you can go in and select whatever challenge you are interested in! This year's categories were: 
* Cryptography 
* Forensics 
* Reverse Engineering 
* Web 
* Misc.

Now I'll be real honest, I am not a cryptographer. I like math, have an engineering degree, but even I am like "nah dawg". As for Forensics, unfortunately all of my *CSI: Miami* knowledge is not very useful for this type of forensics. Reverse Engineering? Oooh, now that's a maybe. 

To be fair I did spend some time in Burp Suite and doing web stuff but I kept coming back to one Reverse Engineering challenge called "Fruit Deals". The prompt is simple:


{{< image src="/img/macro_reverse_engineering/fruit_deals.png" alt="I make the best deals" position="center" style="width: 500px; height:auto;" >}}

{{< caption caption="I have seen people enable macros for less." >}}

So the assignment seems simple enough right? Download the ```deals.xlsm```, see what malware it downloads (or tries to), and bing bam boom, we got ourselves a flag. 

Now I wouldn't be doing my boys *Danger Chad* and *Tarik The Malware Chef* proud (their reversing level is over 9000) if I didn't take a systematic approach to this and actually try to do this like, y'know, **A PROFESSIONAL**. So to do that, I came up with an attack plan and began trying to figure out what exactly was inside...

### Coming Up With a Plan

At first the plan was to just open the Excel spreadsheet in a Windows machine and just YOLO it, and see whatever that macro downloaded and executed. Now you're probably thinking, 

***"But Cervando, that is reckless! you don't know what the malware does!"*** 

To which I just say 

***"Trust me, I'm a Pro."***

Which is just a rebrand of my catchphrase I've said since I was a child *"I'm about to try something stupid here."* which never failed to terrify my parents and teachers.

Nevertheless, I do concede that just clicking the **YOLO** *"Enable Macros"* button on the spreadsheet doesn't seem the most methodical approach so I decided to do something closer to the scientific method and make my 6th grade science teacher Mr. Connor proud. 

### "Cervando's Approach For Examining Potentially Malicious Spreadsheets While Being Only a Slight Idiot"™️

#### Step 1. "Strings on Strings"
I figured I would do the basic thing everyone does whenever you find an unknown file and just Use ```strings``` to examine the file and see if anything interesting is happening. The full output of strings is available at:

https://github.com/cabanuel/malicious-pro-post-data/blob/main/macro_reverse_engineering/deals_strings.txt 

Looking through it, we see a lot of junk as expected, but doing some browsing 


#### Step 2. "X Gon' Give It To Ya"
Fun fact: modern **Microsoft Office** file formats with the "x" like ```.docx```, ```.pptx```, etc. are actually compressed ```.zip``` files full of XML. So for Step 2, I decided to convert 