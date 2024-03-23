---
title: "Macro Mayhem -  East Africa Intervasity CTF - Semis"
date: 2024-03-23 20:55:00 +03
categories: [VBA_Macro]
tags: [malware analysis, macro, vba, ole, oletool ]
image: /assets/img/Posts/(logo.png)
---

### Challenge Description:
With each layer peeled back, new challenges arise, testing your skills and ingenuity as you race against time to neutralize the threat and safeguard the digital landscape from Macro Mayhem. Only those with the keenest of eyes will unravel its secrets.

## Challenge Walkthrough
We have been provided with a zipped challenge file. Our first step would be to check at the type of file we are dealing with. Check it out with the file command:
```
k4p3re@DESKTOP-DFKV1EF:/mnt/c/Users/K4p3re/OneDrive/SecurityResearch/Maseno UniCTF$ file secr3tfil3.zip
secr3tfil3.zip: data
```
When the `file` command returns `data` for a file, it means that the command was unable to determine the file type based on its contents. This typically occurs when the file does not have any distinctive headers or signatures that can be used to identify its format.

To further analyze the file, we need to use other tools and techniques, like inspecting the content of the file manually by looking at its hex values for headers or signatures that can assist us identify it's format

I utilzed an hex editor; you can use any editor out there to inspect the file. For my case I will use `HxD` for my analysis
Opening our zip file in the editor, we get this display
![alt text](magik-1.png)

The string IHDR definitely hints that this is a PNG file. IHDR (Image Header Chunk): It stores basic image information, 13-bytes long, must be the first chunk in a PNG data stream, and there must be only one file header chunk in a PNG data stream.

The first 4 bytes are set to `.eCf`, a non standard header. Using any hex editor, replacing the 4 bytes with the PNG magic bytes will give us an image file. PNG magic bytes; `89 50 4E 47`

![alt text](magik2-1.png)
Looking at the header now, we now have a file format. Save the new file as a PNG file

Lets again run the `file` command against our new png file to learn more about the file
```
k4p3re@DESKTOP-DFKV1EF:/mnt/c/Users/K4p3re/OneDrive/SecurityResearch/Maseno UniCTF$ file file.png
file.png: PNG image data, 2560 x 1440, 8-bit/color RGB, non-interlaced
```
On analyzing image files, one can use several tools to check for image metadata to try find any hidden information on the image or embedded files. As a CTFer I would go ahead an run tools like `exiftool`, `strings`, `binwalk`
When you run `binwalk` you get that the image has a zip file embedded on it
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 2560 x 1440, 8-bit/color RGB, non-interlaced
165           0xA5            Zlib compressed data, default compression
3044          0xBE4           Zlib compressed data, best compression
```

Now lets extract our file for further analysis. Well for this one I used a GUI tools called `OpenStego` to extract the file from the image
![alt text](opensteg-1.png)

This extracts our file of interest; Up for an interesting analyis journey!

When you try to unzip the file archive you get prompted for the file password
```
k4p3re@DESKTOP-DFKV1EF:/mnt/c/Users/K4p3re/OneDrive/SecurityResearch/Maseno UniCTF$ unzip Secr3tf1le.zip
Archive:  Secr3tf1le.zip
[Secr3tf1le.zip] sample.bin password:
```
Don't get stuck, let's get back to our png image and open it; On the progress bar we can see a string, well it's the password we can use to unzip our file

## Macro Analysis 
Lets get a little background on Macros and its analysis; Doing the `file` command:
```
sample.bin: Composite Document File V2 Document, Little Endian, Os: Windows, Version 10.0, Code page: 1252, Template: Normal.dotm, Revision Number: 1, Name of Creating Application: Microsoft Office Word, Create Time/Date: Wed Jul 22 23:12:00 2020, Last Saved Time/Date: Wed Jul 22 23:12:00 2020, Number of Pages: 1, Number of Words: 3, Number of Characters: 21, Security: 0
```
Information from this tells us that this is a Microsoft Office Word document saved in the DOCX format on a Windows 10 system. The document, created using the Normal.dotm template, contains one page with three words and 21 characters. 

After gathering the fundamental data, the next step is to ascertain if the sample file contains any macros. To determine the presence of macros within the document streams, we can utilize `oledump.py` for analysis.

## OLE Files
An OLE file can be likened to a miniature file system or a compressed archive, similar to a Zip file. Within an OLE file, there are streams of data resembling individual files, each with a designated name. For instance, in a Microsoft Word document, the primary stream containing the text content is typically named "WordDocument".

OLE files can encompass storages, which act as directories housing streams or other storages. For example, a Word document with embedded VBA macros may contain a storage named "Macros".

Certain streams within OLE files can contain properties, which are specific values used for storing metadata or other document-related information such as title, author, and creation date.

## Oletools Overview
One of the interesting open-source tools for extracting embedded macros from MS Office documents is Oletools; a collection of python tools created for malware static analysis

## Utilized tools For the Challenge
- `oledump.py` is a program to analyze OLE files (Compound File Binary Format). These files contain streams of data. oledump allows you to analyze these streams.
- `olevba.py` is a script to parse OLE and OpenXML files such as MS Office documents (e.g. Word, Excel), to detect VBA Macros, extract their source code in clear text, and detect security-related patterns such as auto-executable macros, suspicious VBA keywords used by malware, anti-sandboxing and anti-virtualization techniques, and potential IOCs
NOTE: There are many scripts within the oletools package that you can explore

## Questions
1. How many streams contain macros in the document, and list them {Format:xx;00,00,00,00...}
`python3 oledump.py sample.bin`
```
Ans: 13, 15, 16
```
Here we run the `oledump.py` to analyze the streams that contain Macro; They will be labeled as `M` in the data stream.
```
k4p3re@DESKTOP-DFKV1EF:/mnt/c/Users/K4p3re/OneDrive/SecurityResearch/CTF$ python3 oledump.py /mnt/c/Users/K4p3re/OneDrive/SecurityResearch/Maseno\ UniCTF/sample.bin
  1:       114 '\x01CompObj'
  2:      4096 '\x05DocumentSummaryInformation'
  3:      4096 '\x05SummaryInformation'
  4:      7119 '1Table'
  5:    101483 'Data'
  6:       581 'Macros/PROJECT'
  7:       119 'Macros/PROJECTwm'
  8:     12997 'Macros/VBA/_VBA_PROJECT'
  9:      2112 'Macros/VBA/__SRP_0'
 10:       190 'Macros/VBA/__SRP_1'
 11:       532 'Macros/VBA/__SRP_2'
 12:       156 'Macros/VBA/__SRP_3'
 13: M    1367 'Macros/VBA/diakzouxchouz'
 14:       908 'Macros/VBA/dir'
 15: M    5705 'Macros/VBA/govwiahtoozfaid'
 16: m    1187 'Macros/VBA/roubhaol'
 17:        97 'Macros/roubhaol/\x01CompObj'
 18:       292 'Macros/roubhaol/\x03VBFrame'
 19:       510 'Macros/roubhaol/f'
 20:       112 'Macros/roubhaol/i05/\x01CompObj'
 21:        44 'Macros/roubhaol/i05/f'
 22:         0 'Macros/roubhaol/i05/o'
 23:       112 'Macros/roubhaol/i07/\x01CompObj'
 24:        44 'Macros/roubhaol/i07/f'
 25:         0 'Macros/roubhaol/i07/o'
 26:       115 'Macros/roubhaol/i09/\x01CompObj'
 27:       176 'Macros/roubhaol/i09/f'
 28:       110 'Macros/roubhaol/i09/i11/\x01CompObj'
 29:        40 'Macros/roubhaol/i09/i11/f'
 30:         0 'Macros/roubhaol/i09/i11/o'
 31:       110 'Macros/roubhaol/i09/i12/\x01CompObj'
 32:        40 'Macros/roubhaol/i09/i12/f'
 33:         0 'Macros/roubhaol/i09/i12/o'
 34:     15164 'Macros/roubhaol/i09/o'
 35:        48 'Macros/roubhaol/i09/x'
 36:       444 'Macros/roubhaol/o'
 37:      4096 'WordDocument'
 ```
2. Which event triggers the initiation of macro execution?
` python3 olevba.py /mnt/c/Users/K4p3re/OneDrive/SecurityResearch/CTF/sample.bin`
```
Ans: Document_open()
```
Utilize `olevba.py` to extract source code in clear text, and detect security-related patterns such as auto-executable macros
Looking at the data stream from the beginning of the file, the MACRO stream `13` contains the starting point of the macros from the defined subroutine:
    `Private Sub` -  Defining the function/subroutine
    `Document_Open` - This run the function defined as the file gets opened
    `boaxvoebxiotqueb` - Function name which defined the functionality to be run on Document Open.
    `End Sub` - Ending the function definition.
![alt text](exec-1.png)

3. What is the name of the user-form contained within this document?
```
Ans: roubhaol
```
The `.frm` extension in a stream suggests that it's associated with a form object.
![alt text](form-1.png)

4. Which stream is storing the base64 encoded stream?
```
Ans: 34
```
## DeepDive
In this particular stream, we can see an encoded string printed; Thats the string that starts with `p`. Adversaries always try to leverage LOLBins(Living Off the Land) binaries to camouflage their malicious activity
Initially, LOLBins were commonly used in a post-exploitation basis, to gain persistence or escalate privileges. However, the local system binaries or the preinstalled tools on a machine are now being used to bypass detection and aid in malware delivery. Which means that malicious actors can use these LOLBins to achieve their goals, without relying on specific code or files.

LOLBins are often Microsoft signed binaries. Such as Certutil, Windows Management Instrumentation Command-line (WMIC). They can be used for a range of attacks, including executing code, to performing file operations

Taking a closer look at the encoded string, we can identify a particular string being repeated `2342772g3&*gs7712ffvs626fq`. We can try eliminating it from the payload to see its effect. We can do this by using the `Find/Replace` recipe in Cyber chef
![alt text](base64-1.png)

We are able to observe that this was a powershell script having a base64 encoded payload and it was stored in stream 34.

5. Which malware variant was the document trying to deploy?
```
Ans: emotet
```
For this question, one can either get the file hash upload the file on the Malware Information sharing platforms to find out more about the malware. Examples of such platforms are: VirusTotal, MalwareBazaar etc
VT analyzes the file are shows that this is an `emotet` malware variant;
![alt text](threat-1.png)


That's the challenge in nutshell. I hope you had fun peeling every layer of this challenge.

Cheers!!
