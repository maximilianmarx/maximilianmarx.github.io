---
title: "Malware: Living off the land - creating fileless PowerShell malware"
author: Maximilian Marx
date: 2021-09-19 12:08:00 +0100
categories: [Malware Development]
tags: [Process injection, Malware, PowerShell]

image: /assets/img/fileless-malware-powershell-header.jpg
---

# Introduction

The Windows API refers to functionality exposed by built-in system DLLs (eg. kernel32.dll - which exposes multiple functions that can be used by developers to interact with Windows).

There are a few ways to interact with the Windows API using Powershell:
- Via the *Add-Type* cmdlet in order to to compile C# code (the officially documented method)
- Using *Reflection* to dynamically define a method which calls the Windows API function

# Add-Type and its drawbacks for malware creators

**Add-Type** is a **cmdlet** allowing us to define .NET types that will be available within the PS session.
If you use it, it will compile the C# code on the fly once the script is executed.

From an attackers perspective this method should be avoided.
Why? Because the C# code gets temporarly written to the disk in order to then be compiled using **csc.exe**.

This way we will leave a forensic footprint and might get caught (either by the blue team or even AV).

I created a basic [**Process Injection**](https://github.com/maximilianmarx/shellcode-injection/blob/main/PSInject.ps1) PowerShell script
If we execute it, we'll quickly see that PowerShell creates a temprary file to store the C# code in order to compile it to a DLL using *csc.exe*.
![img-description](/assets/img/fileless-pe-createfile.png)_Temporary Files created by PowerShell_

# The Workaround

To get around the problem that **Add-Type** will leave a forensic footprint, we can use **Reflection**.

Reflection is basically the "same" process performed by the Add-Type cmdlet and the C# compiler.

In order to understand how Reflection works, we have to dig deeper into the .NET Framework.
A large portion of .NET Framework is built into the Windows API, but not all of it is exposed to be public (but instead **private** and therefore we cannot access it directly).

If you want to dig deeper into the .NET Framework, use "**Find-WinAPIFunction**" by [**Matt Graeber**](https://github.com/maximilianmarx/dailies/blob/main/Find-WinAPIFunction.ps1).

The fileless PowerShell Process Injection script I created is based mainly on the great work of Matt Graeber - you can read more about it [**here**](http://web.archive.org/web/20210305193403/www.exploit-monday.com/2012/05/accessing-native-windows-api-in.html).

Within the blog post of Matt he goes into great detail on how to use reflection for accessing the Win32 API and how reflection works in general.

# Demo

We can modify the aforementioned PowerShell script using the technique described by Matt Graeber in order to be completely fileless.
If we execute the modified script, we'll see that it still works and we get a reverse shell:

![img-description](/assets/img/fileless-demonstration.gif)_Proof-of-Concept fileless process injection_

If we analyze the script with Process Explorer again, we see that not a single file got created.
![img-description](/assets/img/fileless-pe-nocreate.png)_No Files created_

### References
- <https://devblogs.microsoft.com/scripting/use-powershell-to-interact-with-the-windows-api-part-1>
- <https://devblogs.microsoft.com/scripting/use-powershell-to-interact-with-the-windows-api-part-2>
- <https://devblogs.microsoft.com/scripting/use-powershell-to-interact-with-the-windows-api-part-3>
- <http://web.archive.org/web/20210305193403/www.exploit-monday.com/2012/05/accessing-native-windows-api-in.html>
- <https://www.redteam.cafe/red-team/powershell/using-reflection-for-amsi-bypass>

### Further Reading
- <https://isc.sans.edu/forums/diary/Fileless+Malicious+PowerShell+Sample/23081/>
- <https://www.netscylla.com/blog/2018/01/26/Dridex-Loader-Technique-Used-For-MSF-Shells.html>

### Image Sources
- Banner <https://news-cdn.softpedia.com/images/news2/malware-created-with-microsoft-powershell-is-on-the-rise-503103-4.jpg>