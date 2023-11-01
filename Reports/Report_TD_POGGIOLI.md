# Report TD TryHackME / Flavio POGGIOLI

## Table of contents

- [Report TD TryHackME / Flavio POGGIOLI](#report-td-tryhackme--flavio-poggioli)
  - [Table of contents](#table-of-contents)
  - [Nmap](#nmap)
  - [Burp](#burp)
  - [SQLmap](#sqlmap)
  - [Metasploit](#metasploit)


## Nmap

[TryHackMe Nmap](https://tryhackme.com/room/furthernmap)

Nmap is an open-source network scanning and discovery tool. 
It is used to scan ports, and services running on a network. 
Nmap offers scriptable automation, and various scanning techniques with special switchies, making it an essential tool for network administrators and security professionals.

It offers a wide range of options for TCP, UDP and ICMP scans. 

In TCP with some specific switches, you can scans ports to determine if they are open or closed depending on which TCP flags (SYN, ACK, RST) are sent back.

With ICMP, you can proceed to a network scanning with the switches "-sn" to determine which hosts are up or down.

NSE (Nmap Scripting Engine) is a powerful scripting language based on Lua programming language.
It allows you to write scripts and automate some scanning vulnerabilities, or even exploit them.

Main features:
- Host discovery and OS detection
- Port scanning and service identification
- Scriptable with NSE (Nmap Scripting Engine)
- Flexible output formats
- Extensive community support

For more information and documentation, visit the [Nmap website](https://nmap.org/) or in the terminal with the command:

```bash
$ nmap -h
```

## Burp

[TryHackMe Burp](https://tryhackme.com/room/burpsuitebasics)

Burp is a Java-based framework designed to serve as a comprehensive solution for conducting web application penetration testing. It serves for the security testing of web and mobile applications, including the REST API of theses applications.

Main features : 
- Proxy
- Repeater : capturing, modifying, and resending the same request multiple times.
- Intruder : allows for spraying endpoints with requests. It is commonly utilized for brute-force attacks or fuzzing endpoints.
- Comparer : allows for comparing two requests or responses.

Burp comes with a lot of options for the proxy feature (WebSockets, Response Interception...).

The feature Site Map allows you to see a tree structure of the website you are testing, and the requests you have made.

A browser based on Chromium is also provided with full support of the proxy to use it without any of the modifications we just had to do.


## SQLmap

[TryHackMe SQLmap](https://tryhackme.com/room/sqlmap)



## Metasploit


[TryHackMe Metasploit](https://tryhackme.com/room/metasploitintro)