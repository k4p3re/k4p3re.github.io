---
title: "Attacktive Directory -  TryHackme"
date: 2023-11-13 20:55:00 +03
categories: [AD Exploits]
tags: [AD-exploitation, win, main, vuln, p64, 64-bit ]
image: /assets/img/Posts/ad.png
---

> Welcome to a thrilling adventure into the intricate world of Active Directory security! In this blog post, we'll be delving into the immersive and hands-on experience offered by the Active Directory lab on TryHackMe. Whether you're a seasoned cybersecurity professional or a curious enthusiast eager to enhance your skills, this journey promises to unravel the mysteries of AD security in a practical, real-world environment.

## AD Pentest Toolset
Conducting a penetration test on an Active Directory (AD) environment requires a set of specialized tools to effectively assess its security. Here's a list of essential tools for an AD penetration test:

`BloodHound:`

BloodHound is a powerful tool for visualizing and analyzing relationships within an Active Directory environment. It helps identify attack paths and potential security risks, making it an invaluable asset for penetration testers.

`enum4linux:`

enum4linux is a tool designed to enumerate information from Windows and Samba systems. It specifically focuses on gathering details from SMB (Server Message Block) services, providing insights into shares, users, and other information within an Active Directory environment.

`Neo4j:`

While not a traditional penetration testing tool, Neo4j can be useful in the context of analyzing and visualizing data relationships. In an Active Directory environment, tools like BloodHound often use graph databases like Neo4j to represent complex relationships between AD objects, helping penetration testers identify and understand attack paths.

`Mimikatz:`

This is a post-exploitation tool that allows penetration testers to extract plaintext passwords, hashes, and Kerberos tickets from memory, enabling the identification of potential security vulnerabilities.

`PowerShell Empire:`

PowerShell Empire is a post-exploitation framework that facilitates the deployment of agents on target systems. It allows testers to execute PowerShell-based payloads and perform various actions on compromised systems.

`PowerSploit:`

PowerSploit is a collection of Microsoft PowerShell scripts that aids penetration testers in post-exploitation activities. It includes modules for various tasks, such as privilege escalation, code execution, and credential theft.

`CrackMapExec (CME):`

CME is a post-exploitation tool that automates a variety of tasks in an Active Directory environment. It supports actions like lateral movement, execution of commands, and credential manipulation.

`Responder:`

Responder is a network analysis tool designed to capture and relay authentication requests. It helps identify and respond to Network NTLM challenges, making it useful for performing man-in-the-middle attacks in an AD environment.

`Impacket:`

Impacket is a collection of Python scripts for working with network protocols, including those used in Active Directory environments. It allows penetration testers to perform various tasks, such as relaying authentication, dumping credentials, and more.

`LDAP Enumeration Tools (e.g., ldapsearch, ldapenum):`

These tools are essential for enumerating information from LDAP services. Enumeration is a crucial step in understanding the structure of an Active Directory environment, including users, groups, and permissions.

`Nmap:`

Nmap is a versatile network scanning tool that can be used to discover hosts, open ports, and identify services within an Active Directory network. It helps in the initial reconnaissance phase of a penetration test.

`Veil Framework:`

Veil Framework is a collection of tools designed for generating and delivering payload executables for Windows environments. It aids in creating undetectable payloads for use in penetration tests.

`Aorato PTH Tool:`

This Pass-the-Hash (PTH) tool is designed to perform various actions using pass-the-hash attacks. It's particularly useful for testing the effectiveness of security measures against credential theft.

Let's dive into the challenge:
## Task 3: AD Enumeration
The goal of enumeration is to gather details about users, groups, computers, shares, and other objects within the AD infrastructure. Tools like enum4linux, ldapsearch, and custom PowerShell scripts are commonly used during the enumeration phase. Enumeration serves as the foundation for subsequent penetration testing activities, helping testers devise effective strategies for privilege escalation, lateral movement, and overall compromise. 

Our initial step in this machine is do an nmap scan on our target.

```shell
nmap -sVC [IP]
```
![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/18ad8c9b-9ea4-428c-806d-1ad8aa816ad9)
1. What tool will allow us to enumerate port 139/445?
   
Ans: enum4linux
2. What is the NetBIOS-Domain Name of the machine?

This requires one to use the enum4linux tool to do enumeration in order to identify the NetBIOS-Domain name.

```shell
enum4linux [IP]
```
![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/f3a1183c-22f4-4ce2-aa6e-40f5763357e7)

Ans: THM-AD
4. What invalid TLD do people commonly use for their Active Directory Domain?

A quick google search of an invalid TLD commonly used in non-production AD environments is the `.local

Ans: .local

## Task 4: Enumerating Users Using Keberos
Enumerating users via Kerberos involves leveraging the Kerberos authentication protocol to gather information about valid user accounts within an Active Directory (AD) environment. We will use a tool called kerbrute to bruteforce discovery of users, passwords and also password spray.
You can download the tool here: https://github.com/ropnop/kerbrute

Run the help section of the tool to see available arguments.
```shell
./kerbrute_linux_amd64 --help
```
![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/9bbe429e-312e-42a9-8da9-c490c5703408)

What command within Kerbrute will allow us to enumerate valid usernames?

Ans: userenum

We now run the command to enumerate the machine.
```shell
./kerbrute_linux_and64 userenum --dc [IP] -d spookeysec.local <path_to_user_list>
```
![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/d21e37d2-c2e4-40ed-bd33-df0070480196)

What notable account is discovered? (These should jump out at you)

Ans: svc-admin

What is the other notable account is discovered? (These should jump out at you)

Ans: backup

## Task 5: Abusing Kerberos

Abusing Kerberos involves exploiting vulnerabilities or weaknesses in the Kerberos authentication protocol to gain unauthorized access or compromise security within an Active Directory (AD) environment. Some of the methods  for abusing kerberos include the following:

#### Kerberoasting

In this attack, an attacker can compromise a user account and extract the Kerberos ticket-granting ticket (TGT) that can be used to impersonate the user and gain access to sensitive resources.

#### Golden Ticket

If a malicious actor has access to the password hash of the KRBTGT account, they can generate what is known as a golden ticket. This particular type of ticket empowers the attacker to fabricate authentication credentials for any account present in the Active Directory. Once in possession of a golden ticket, the attacker can leverage it to request Ticket Granting Service (TGS) tickets, providing access to designated resources. The acquisition of TGS requires interaction with the Key Distribution Center (KDC), which operates on domain controllers within the Active Directory domain.

#### AS-REP Roasting

AS-REP Roasting is a security attack that aims to extract user hashes that can be later subjected to offline brute-force attempts. When a user has the "Do not use Kerberos pre-authentication" setting enabled, an attacker can obtain a Kerberos AS-REP encrypted with the user's RC4-HMAC'd password. Subsequently, the attacker can attempt to crack this ticket offline, potentially revealing the user's password.
We are going to attempt and abuse this feature in kerberos. 

The script `GetNPUsers.py` found within `impacket` can be employed to fetch a list of domain users lacking the "Do not require Kerberos preauthentication" setting and request their Ticket Granting Tickets (TGTs) without the need for knowledge of their passwords. Subsequently, there is an opportunity to try to decrypt the session key included in the ticket, aiming to uncover the user's password through cracking attempts.

1. We have two user accounts that we could potentially query a ticket from. Which user account can you query a ticket from with no password?

Since we have only two unusual user accounts i.e `svc-admin` and `backup`, we try for both to check which one we can query a ticket from without a password.
It only works for the `svc-admin` user.

```shell
./GetNPUsers.py -no-pass -dc-ip [IP] spookeysec.local/svcadmin
```
![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/82288896-3b0a-40ee-9e04-402b07b2d852)

Ans: svc-admin

2. Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC? (Specify the full name)

A quick search gives us the following results:
<img width="654" alt="image" src="https://github.com/k4p3re/k4p3re.github.io/assets/49836387/3f06feb6-429e-46f3-beba-5c05ecffb5fd">

Ans: Kerberos 5, etype 23, AS-REP

4. What mode is the hash

Ans: 8200

6. Now crack the hash with the modified password list provided, what is the user accounts password?

We use john and the password list provided in the challnege to the crack the hash.

```shell
john --wordlist=<path to the password file> --format=krb5asrep  <hash file>
```
![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/267eb768-86d5-43f8-914d-854644a10e2a)

Ans: management2005

## Task 6: Enumeration (Back to Basics)

1. What utility can we use to map remote SMB shares?

Ans: smbclient

2. Which option will list shares?

Ans: -L 

3. How many remote shares is the server listing?

```shell
smbclient -L [IP] -U svc-admin%management2005
```
![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/85a695f8-a6f6-4c76-9b64-7bbc3e78d3e6)

Ans: 6

4. There is one particular share that we have access to that contains a text file. Which share is it?

Ans: backup

5. What is the content of the file?

![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/3042c9a4-e8e2-4d52-a7c4-56ac92f17567)
Ans: YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw

6. Decoding the contents of the file, what is the full contents?

![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/5f8a1ff8-3696-4766-bc05-fd64e34cda95)

Ans: backup@spookysec.local:backup2517860

## Task 7: Domain Privilege Escalation
Since we now have more information and creds, we now exploit the privileges within the domain.

We will make use of the `secretdump.py` a script within `impacket`Python library. It allows the extraction of secrets (NTDS.dit, SAM and .SYSTEM registry hives) from multiple Windows systems simultaneously. Additionally, it incorporates multithreading support to expedite operations, providing a more efficient and faster extraction process.

1. What method allowed us to dump NTDS.DIT?

![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/cab78d2b-8683-4201-9fc4-68d504646dae)
Ans: DRSUAPI

2. What is the Administrators NTLM hash?

Ans: 0e0363213e37b94221497260b0bcb4fc

3. What method of attack could allow us to authenticate as the user without the password?

Ans: Pass The Hash

4. Using a tool called Evil-WinRM what option will allow us to use a hash?

Ans: -H

## Task 8: Flags
WinRM(Windows Remote Management), serves as Microsoft's implementation of the WS-Management Protocol—a standard SOAP-based protocol designed to facilitate interoperability among hardware and operating systems from various vendors. Microsoft has integrated WinRM into its operating systems, aiming to simplify the tasks of system administrators.

This functionality is available on Microsoft Windows Servers, typically accessible through port 5985, and requires appropriate credentials and permissions for utilization. Consequently, WinRM becomes a valuable tool during the post-exploitation phase of hacking or penetration testing, allowing users to leverage its capabilities provided they have the necessary credentials and permissions.

We utilize the Evil-WinRM tool for as to get a Remote Windows powershell as the Administrator since we already have the hash of the Admin.

Run the below command to get connection:

```shell
evil-winrm -i [IP] -u Administrator -H <Hash>
```

![image](https://github.com/k4p3re/k4p3re.github.io/assets/49836387/a445ba19-4a7c-4755-b4c9-8e071645ad4e)

After getting the remote connection, all flags can be found on the Desktop directory of the users.

- svc-admin
- backup
- Administrator

And we pwned the AD machine !

Thanks for reading, Suggestions & Feedback are appreciated !
