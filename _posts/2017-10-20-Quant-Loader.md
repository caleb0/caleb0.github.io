---
layout: post
title: Quant Loader
---

Quant Loader is a piece of Russian malware that originated from exploit.in during the September of 2016. It has gained notoriety in underground circles for being a stable and reliable botnet. It is priced at $550 for the full license, quite expensive for this quality, or rather lack thereof. 

The first thing that Quant does is sleep for 180 seconds (og 100% fud reboot antivirus runtime bypass). 
![quant_sleep]({{ site.baseurl }}/images/quant_sleep.png)

It then proceeds to decrypt all the strings at once to ensure that reversing it is very easy. **URLDownloadToFileA**? Why is this "professional dll loader/dropper" using win32api functions to preform it's core task? Other interesting strings include the **":Zone.Identifier"**, presumably to strip the identifier flag when the file is downloaded off the internet. To determine whether the system is 32 bit or 64 bit, it reads the "ProgramFiles Dir (x86)". If there is an "x86" in the program files directory, it is a 64 bit system. 

The malware then generates it's unique ID by reading the MachineGuid key at *"SOFTWARE\Microsoft\Cryptography"*, reading only the numbers, then reading 8 numbers starting from the 5th. It will use this GUID to create a new folder at the **Appdata\Roaming\<Guid>\svchost.exe** path, and copying an instance of itself to the path.
![quant_guid]({{ site.baseurl }}/images/quant_guid.png)

Quant checks for a number of antiviruses by comparing registry keys, exiting if any of them are found.

* Kaspersky
  * "kis" "SOFTWARE\KasperskyLab\LicStrg"
* Panda
  * "FirewallName" "SOFTWARE\Panda Software\Setup" 
* Norton Security
  * "TaskbarGroupIcon" "SOFTWARE\CLasses\Applications\NS.exe" 
* Dr Web
  * "DisplayName" "SYSTEM\ControlSet001\services\DrWebLwf"
* Bitdefender
  * "InstallPath" "SOFTWARE\Bitdefender Agent\Install"
* Bullguard
  * "SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths\bullguard.exe"

It then edits the firewall to allow outbound traffic using the following command

`
netsh.exe advfirewall firewall add rule "name=Quant" "program=c:\users\appdata\<uid>\svchost.exe" dir=Out action=allow'
`

![quant_firewall]({{ site.baseurl }}/images/quant_firewall.png)

And also changes the permission of the folder and executable using a very undetected and elegant method, launching an instance of cmd.exe

`cmd.exe /c echo Y|CACLS "c:\users\<user>\appdata\roaming\<uid>\svchost.exe" /P "user:R"
cmd.exe /c echo Y|CACLS "c:\users\<user>\appdata\roaming\<uid>" /P "user:R"`


It also launches the new dropped executable using a non-resolved ShellExecuteA, before sleeping for a number of minutes. Very nice.

Then, it uses a very stealthy and undetected startup method, the runkey

`
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
`

It pings the panel every 1 minute, and for some reason it decrypts the panel url every single time it wants to send a request? 

![quant_panel]({{ site.baseurl }}/images/quant_panelcontact.png)

It uses WinInet functions to send a get request **(InternetOpenA, InternOpenUrlA, InternetReadFile)**. One might wonder if Microsoft knows they are enabling hackers to make dangerous pieces of malware like this one. 

#### String Encryption

The entire config, including the panel url, is stored in 512 byte blocks. In order to decrypt the configuration, Quant Loader uses a primitive method of string encryption, simply subtraction from a hardcoded key.
In this particular sample the key is:

`
23b4d1c1bbad099329ebc045b8861b9d
`

![quant_str]({{ site.baseurl }}/images/quant_str_encrypt.png)

Here is pseudocode for the algorithm. 
![quant_pseudo]({{ site.baseurl }}/images/quant_pseudo.png)

There is also a bug in the string encryption method. One might wonder how such a bug would make it into a self-proclaimed "professional" piece of malware. Very strange. It skips the first byte of the key, and reads an extra byte off the end. This doesn't matter however, as the block of memory has already been memset to NULL, so the extra byte read is just a 0x00 byte. 

In conclusion, this was a very disappointing and technically unsophisticated piece of malware, especially for the hefty price. It doesn't even resolve api's, and uses external programs like cmd.exe and netsh.exe to fulfil it's purposes. It is a miracle how this malware still proves to be effective and a popular choice amongst the underground community. 
