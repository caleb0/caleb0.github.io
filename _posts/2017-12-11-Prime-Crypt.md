---
layout: post
title: Prime Crypt - The Babushka Crypter
---

Hello, and welcome back to another episode of insidemalware. sorry i haven't been posting as much as I would like, had exams, but now i have lots of time to analyse fun malware :)

![info](https://i.imgur.com/IqDBc0Q.png)
As we can see this is a .net binary (136kb) with no packing.
<!--more-->
[VirusTotal Scan (4/67, not bad)](https://www.virustotal.com/#/file-analysis/YmM3ODMxMzEzNzk2OThlY2RiYTIxNzliMDI0MjE3NjY6MTUxMjk2MDAzMQ==)

Let's try and unpack it in dnSpy. 
Strings, classes and function names are all obfuscated and have random names, however this shouldn't be a problem as there doesn't seem to be any control flow obfuscation. 
It gets the packed binary from it's resources section and decrypts it using this primitive algorithm
![decrypt](https://i.imgur.com/duB0LVv.png)

The key in this binary is the result of ```Encoding.ASCII.GetBytes("CEREEOVUIMCVOEOBRMEUTOONVRETRMUBIRVZNVRIEMZIXMEETRVXOTCENMIZOMRCUTOIIMR");```
For some reason he has two argumets for the encryption key, "enc" and "roshav". This suggests we are dealing with a very inexperienced malware developer. 

It then calls the entry point of this decrypted bin. 
By dumping the decrypted resouce information, we are left with another .net binary (92kb).
[VirusTotal (23/67, uh oh)](https://www.virustotal.com/#/file-analysis/NGViYTE2ZjVhYmRkMTQ5NGQ5NDBkMDc2MzEwNmVlYTU6MTUxMjk2MDA2OQ==)

This one was made by someone called "zero", and hosting his development environment in OneDrive! Gotta make sure you have those malwares backed up in the cloud in the event of a disaster.
```c:\users\zeros\onedrive\documents\visual studio 2017\projects\classicrefud\classlibrary1\obj\release\graznataguz.pdb```
Interesting that the project name is called "classic refud". "refudding" is the process of making a piece of malware undetected from anti-viruses again, when it is inevitably detected. 


Again, let's open it up in dnSpy. None of the strings or function names are encrypted, evident by the phallic themed decryption routine. 
![decrypt-phallic](https://i.imgur.com/ligKA7p.png)
Perhaps our malware author should seek some therapy to address his insecurities.

Some generic anti-analysis checks are performed
![antis](https://i.imgur.com/nHtv0Py.png)
The author has also decided to wrap everything in an empty try-catch loop so nothing can go wrong, a theme we are seeing recurr more frequently in amateur malware development. 

Two more binaries are dumped from this .exe
A .net binary that is nothing more than a GUI for some sort of a serial number checker. 
A .net dll called "RunLib" is decrypted from and loaded into memory and hanldes your generic runPE functions, injection routines and some more malicious functions. 
![runlib](https://i.imgur.com/DRfHc0Z.png)
[VirusTotal 12/66](https://www.virustotal.com/#/file/4c1b4ca5b210ce156564d39117a31f1853522a790e400b7a5afc700f9171bd4b/details)

Some beautiful hardcoded file checks
![Hardcoded](https://i.imgur.com/AqRov3O.png)

A very pAvastBypass
![Avast](https://i.imgur.com/ihzkBMj.png)

Interestingly, the coder does not seem to know how to write process hollowing for a native binary
![runpe](https://i.imgur.com/5MQ93ty.png)

If the process is not .net, it writes it all to a temp path and starts it...
and the way it detects a .net binary is very very sophisticated
![net-detection](https://i.imgur.com/sUoouCJ.png)

The persistance method is quite unusual, it uses the Task Scheduler, however it writes to an .xml file before importing to schtasks.exe. Just seems like more dropped files that could be detected?

In conclusion, this malware is very unprofessional, and the author obviously has some insecurities about his penis size. The fact that this "crypter" is as effective and successful as it is in the underground hacker community is a dangerous indication that even amateur malware developers can cause great harm. 
