---
layout: post
title: Aquius XMR Miner
---
SHA256: 2F6FD3CD31A63EFA096DC29898E55F5F485D4A98B0AE65BD027AA471C7380465
MD5: 710F990ACAF9299D71FD43775D5C9932


Aquius miner is an XMR miner advertised on many popular infamous hacking forums, at a relatively steep $20 for a basic build, compared to it's competitors. 
**Sales thread**
![sales_thread]({{site.baseurl}}/images/aquius_xmr/sales.png)

From some preliminary analysis, we can determine that the binary is written in C# and obfuscated with some sort of packer. Nothing de4dot can't handle.
![exeinfo]({{site.baseurl}}/images/aquius_xmr/exeinfo.png)

Another interesting note is the large size of the binary, and the fact that it needs admin rights to execute. I didn't see anything in the binary that requires admin priviges though haha. 
![size]({{site.baseurl}}/images/aquius_xmr/size.png)
I initially thought the large binary size was because the miner carried both 64 and 32 bit files, however further anaylsis will reveal that this is simply because 
the author did not compress the mining binary ([The opensource XMR miner XMRig](https://github.com/xmrig/xmrig), btw) that the miner executes. 

The binary is trivial to unpack, dnSpy does the job fine. All the variable names are unobfuscated, even the strings used in the binary that should definitely by encrypted are stored in plaintext *(note the hardcoded binary name)*
![str]({{site.baseurl}}/images/aquius_xmr/strings.png)

In the main function, the miner hides itself by calling the ShowWindow API function... Could the developer not have compiled without a window? And the "intellimode" simply adds a 25 second sleep to the execution. nice. 
![main]({{site.baseurl}}/images/aquius_xmr/main.png)
It also copies itself to the `Environment.GetFolderPath(Environment.SpecialFolder.Startup);` location for persistance. Very detected

It them copies the XMR binary, which is encrypted by an arbitrarily complex algorithm and stored in memory (to prevent signature detections) but this feature is ultimately futile as the miner is dropped directly to the disk `(Environment.SpecialFolder.LocalApplicationData + the hardcodedname.exe)`
![decrypt]({{site.baseurl}}/images/aquius_xmr/decrypt.png)

For some reason the author has selected a certain number of pools that the miner can mine to, instead of allowing the client to use whatever pool the want. 
![pools]({{site.baseurl}}/images/aquius_xmr/pools.png)

The only other point of interest in the malware is the way that it executes the mining binary. It uses a very detected and public runPE (CreateProcess -> NtUnmap -> NtWriteVM -> NtResumeThread) to 'execute in memory' and also passes several arguments into the function, which seems to suggest that the developer is donating 1% of all profits to themselves! Even after the client has paid some $20 for the binary. Disappointing, but not very surprising. Or, the developer has been stumped on how to remove the donate function in the original binary, and wants to minimise the damage done. Sad!
![runpe]({{site.baseurl}}/images/aquius_xmr/launch_miner.png)
