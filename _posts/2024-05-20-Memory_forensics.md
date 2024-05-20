---
title: "Memory Maze -  East Africa Intervasity CTF - Finals"
date: 2024-05-20 20:55:00 +03
categories: [Memory Forensics]
tags: [forensic analysis, memory forensics ]
image: /assets/img/Posts/logo.png
---

### Inro
In the ever-evolving landscape of cybersecurity, mastering the intricacies of memory forensics has become crucial for identifying and mitigating sophisticated threats.  This challenge is meticulously crafted to simulate real-world scenarios, pushing contestants to dive deep into volatile memory analysis, detect malicious activity, and uncover hidden artifacts. Through this exercise, I aim to foster a deeper understanding of memory forensics, promote collaborative problem-solving, and elevate the skill level of budding cybersecurity professionals.

### Challenge Description:
Unravel the Secrets of the Seized System. The Memory Maze beckons, daring analysts to delve deeper and unlock its secrets. Let the quest begin!

Challenge File can be found here.(to be added)

## Recommendation;
Install Volatility 3: Ensure you have Volatility 3 installed. You can find the installation instructions and download it from the Volatility 3 GitHub repository.


## Challenge Walkthrough

1. What is the md5 hash value of the RAM image?
To determining the MD5 hash value of the provided RAM image,you need to utilize a hashing tool to compute the MD5 checksum. By executing the appropriate command (`md5sum` on Unix/Linux/macOS or `CertUtil` on Windows), they can generate the hash, which serves as a unique identifier for the file.

Using the md5sum:
```
md5sum memdump.mem
```
Ans: `db2d8f1b447e14ee4b39fc34f3bc5c6a`

2. Give the system time the RAM image was captured
Volatility 3 does not require you to specify a profile as it automatically detects it. However, you still need to check which OS plugins are applicable.
Determine System Time with `windows.info` Plugin: This plugin scans the memory image and outputs detailed system information, including the current system time.

```
python3 vol.py -f memdump.mem windows.info
```
![alt text](image.png)

The `System time` field in the output represents the system time at the moment the RAM image was captured.

Ans: `2021-04-30 17:52:19`

3. How many active network connections were established at the time of acquisition? (Provide the number)

Using the `windows.netstat` Plugin: This plugin lists network connections, including active connections.
To make the process of determining the number of active network connections more interesting and focus on only established connections, you can use `grep` to filter the output from Vol3. Specifically, you can filter for connections with a state of `ESTABLISHED`.

Here’s how to do it:

Use the `windows.netstat` plugin to list all network connections and then use `grep` to filter for established connections.

```
python3 vol.py -f memsump.mem windows.netstat | grep ESTABLISHED

```
![alt text](established.png)
Count the number of lines in the output to determine the number of established connections.

To do a summary of the challenge question, this is how your final command would look like to give you a direct answer for the number of established connections:
```
python3 vol.py -f memdump.mem windows.netstat | grep ESTABLISHED | wc -l
```
This pipeline filters for established connections and then counts the number of lines, giving you the total number of established connections directly.

Ans: `10`

4. Give the FQDN and city that Chrome have an established network connection with (FQDN:City) 
Given that Chrome appeared in the previous question among the established connections, you can directly filter and identify the specific connections related to Chrome. Here's how to find the Fully Qualified Domain Name (FQDN) and city

## Step 1: Identify Established Connections for Chrome
Run the `windows.netstat` plugin to list all network connections and filter for those related to `Chrome` with the ESTABLISHED state:
```
python3 vol.py -f memdump.mem windows.netstat | grep ESTABLISHED | grep chrome
```
![alt text](chrome.png)

## Step 2: Resolve IP Addresses to FQDN
For the foreign IP address listed in the output, use a tool like `nslookup` to resolve the IP address to its FQDN:
```
nslookup 185.70.41.130
```
## Step 3: Geolocate IP Addresses
Use a geolocation service to determine the city associated with the IP address. You can use services like `ipinfo.io` or `geoiplookup`.
```
curl ipinfo.io/185.70.41.130
```
Output:

```
{
  "ip": "185.70.41.130",
  "hostname": "185-70-41-130.protonmail.ch",
  "city": "Zürich",
  "region": "Zurich",
  "country": "CH",
  "loc": "47.3667,8.5500",
  "org": "AS62371 Proton AG",
  "postal": "8000",
  "timezone": "Europe/Zurich",
  "readme": "https://ipinfo.io/missingauth"
} 

```

With the resolved FQDNs and geolocated cities, all you need to do is compile the results:

Ans: `protonmail.ch:Zurich`

5. What is the SHA256 hash value of process memory for PID 3536?
Dump the Process Memory using `pslist` with `--dump` option:
When using Volatility's `pslist` plugin, you can directly specify the `--dump` option to dump the memory of the process you're interested in. This saves you the step of manually extracting the PID from the pslist output and then using the memdump plugin separately.

```
python3 vol.py -f memdump.mem -o PID3536 windows.pslist --pid 3536 --dump  
```
For my case, I started with creating a directory named `PID3536` to dump my PID data in.

Your next step is to Use a hashing tool like `sha256sum` to calculate the SHA256 hash of the memory dump file:
```
sha256sum 3536.FTK\ Imager.exe.0x400000.dmp
```
Output:
/home/kali/Pictures/sha256.png

Ans: `d4904652dafb61c331de9ac22b8000f5ac0dd7d6304f051097f7a3449b3fd45a`

6.  What is the complete file path and name of the most recent file opened in Notepad?
To retrieve this, you can use Volatility's `cmdline` plugin. This plugin extracts command line arguments passed to processes, this includes the Notepad process, which can contain information about the files being opened.
Whenever you execute a command in Windows, whether it's a built-in command like `dir` or an external program like Notepad (notepad.exe), the command is first processed by `cmd.exe`. For example, when you type `notepad` in the Run dialog and press Enter, `cmd.exe` is responsible for locating the `notepad.exe` executable, parsing any arguments or parameters you provide, and then executing it.

```
python3 vol.py -f memdump.mem windows.cmdline | grep notepad
```
From the above command, we can see that notepad opened a file called `accountNum`:

c:\Users\K4p3re\OneDrive\SecurityResearch\maze\notepad.png

Ans: `C:\Users\JOHNDO~1\AppData\Local\Temp\7zO4FB31F24\accountNum`

7. Identify the process ID for the "brave.exe"
To identify the PID for the "brave.exe" process, you can utilize the `pslist` plugin. This plugin lists all running processes along with their corresponding PIDs. You can then search the output for the "brave.exe" process and note its PID.

```
python3 vol.py -f memdump.mem windows.pslist 
```
Once you've found the "brave.exe" process in the output, the number in the PID column next to it is the Process ID you're looking for.

Ans: `4856`

8. 
When handling this challenge, I did a different approach to investigating how long a program was being used. It was also a learning streak for me. The `userassist` plugin in volatility.
The UserAssist key in the Windows Registry stores information about user activity related to executed programs. This includes details such as the number of times a program has been run and when it was last executed. Analyzing the UserAssist key can provide insights into user behavior and usage patterns of various programs.

Command used:
```
python3 vol.py -f memdump.mem windows.registry.userassist 
```
The userassist plugin parses the ntuser.dat hive, which will provide the actual time the Brave user was used: 
![alt text](brave.png) 

The time duration appears under the column `Focus count`

Ans: `04:01:54`



Until next time, fellow cyber sleuths, may your forensic trails be lined with breadcrumbs of insight and your digital adventures be filled with endless possibilities!

![alt text](out.gif)