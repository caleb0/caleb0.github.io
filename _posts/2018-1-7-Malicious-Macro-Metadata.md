---
layout: post
title: Using .doc metadata section maliciously
---
References: https://www.carbonblack.com/2017/08/28/threat-analysis-word-documents-embedded-macros-leveraging-emotet-trojan/

In this episode of InsideMalware, we will explore how the metadata section of a word document can be weaponized to avade scantime antivirus detection. It is important to note that this technique is only possible on the older .doc structure, not the newer .docx file version. 

### Background of .doc structure
"Metadata" referes to all the data surrounding the data itself. The .doc structure includes allocated fields for a number of different pieces of metadata, including the Title, Author, Date created, date last edited, and comments. 

![metadata](https://i.imgur.com/HyOsrKX.png)
![hex](https://i.imgur.com/DIMS75l.png)
The metadata sections in the .doc file are all stored in a block of data near to the end of the file. The hexedecimal representation of the metadata can be seen below. It uses the byte `1E (dec 30)` to act as a "delimiter" to separate the different blocks. The delimiter takes up a full integer (4 bytes) which is immediately proceeded by two hex numbers defining the size of the section, or a signed short. **However, the endian-ness of the signed short is reversed. For example, a block with 516 bytes, typically represented as `0x204`, would be `04 02`.** The size of each block is rounded to the nearest 4 bytes. Using this information, we can easily modify the metadata of our .doc file *programatically*. 

The VBA macro language has built in functions to retrieve meta data from a .doc file. You can read more about the functions [here](https://msdn.microsoft.com/en-us/library/4e0tda25.aspx), but essentially the call is `ThisDocument.BuiltInDocumentProperties("Title")`, to retrieve the metadata title of the document. 

The flow of this attack is as follows:
1. Retrieve obfuscated powershell command from metadata
2. Deobfuscate powershell command
3. Execute powershell command in hidden window using `Shell` VBA command

I have found this method to be extremely effective and easy to modify, as instead of having to write a new macro for each exploit, you can simply change the obfuscated powershell command that is written to the .doc to be executed (given that the command is less than 32,767 bytes, enough for shellcode or a small binary :D). Also, this method can be used to hide suspicious function imports for more complicated malicious macros that would otherwise be detected by antivirus via suspicious strings, such as the networking object, `Microsoft.XMLHTTP`, or the file system object, `Adodb.Stream`. 

This method also happens to be undetected by nearly all antivirii after some very slight obfuscation (adding 1 to all characters in powershell string, then subtracting in macro), except Nano Antivirus which detects the `Shell` string in the macro (I think this could be fixed with some creative programming)

Overall this is a very elegant use of the metadata section in .doc files and I expect it to be used more in the future. Antivirus companies can easily detect this with the `ThisDocument.BuiltInDocumentProperties` function in VBA. 
