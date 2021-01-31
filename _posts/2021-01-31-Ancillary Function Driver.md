---
layout: post
title: Ancillary Function Driver
date:   2021-01-31
categories:  PE MSDOS
published: true
---

Started the kernel fuzzing in Windows Server 2008 R2 , 32-Bit. I identified this version of windows using vulnerable or unpatched version of Ancillary Function Driver(AFD).

----
[](#header-1)**Definition**
----

The ancillary function driver supports windows sockets application in the afd.sys file. The afd.sys driver runs in a kernel mode and manages the Winsock TCP/IP communication protocols.

----
[](#header-2)**Analysis**
----

Major Flaw in AFD Driver is it was improperly validating input passed from the user mode to the kernel mode, through which any user can get Administrator Rights.
For exploiting this vulnerability , the user should have Login Credentials as a Local User.

----
 [](#header-3)**Bug Mechanism**
---- 
>> Once Execution is done , Process waits for a user to enter an argument, with successfull execution of a process, user will able to exploit Privilege Escalation.

***BUG***

Once the Execution is started :

>>It calls “ZwQuerySystemInformation” with user arguments such as -> **InfoType = SystemModuleInfo**

After command execution , it will list the loaded drivers with there memory addresses in kernel mode.

![](https://yashomer1994.github.io/yash007.github.io/assets/afd/1.png)

>> We identified the Entry of "**ntoskrnl.exe**" or "**ntkrnlpa**" in the list the memory is loaded in the kernel space from "**“_SYSTEM_MODULE_INFORMATION**".

![](https://yashomer1994.github.io/yash007.github.io/assets/afd/2.png)

>> **LoadLibrary()** will Load the module in user mode and start searching for the address of "**HalfDispatchTable**".


![](https://yashomer1994.github.io/yash007.github.io/assets/afd/3.png)

>> Instructions are used to get the “**HalDispatchTable +4**” in module “ntkrnlpa.exe” in kernel space.

![](https://yashomer1994.github.io/yash007.github.io/assets/afd/4.png)

----
 [](#header-4)**Exploit**
---- 

1.The exploit will start by fetching the address to “NtDeviceIoControlFile” API from NTDLL. Once the address is fetched it performs an inline function hooks to the API.

>> Code is used to Perform the Hook:

\\
    	MOV BYTE PTR DS:[ESI],68

    /*ESI points to start of the address of NtDeviceIoControlFile*/

        MOV DWORD PTR DR:[ESI+1],
        MOV BYTE PTR DR:[ESI+5]

    /* above instructions used to inject instructions in to the address space of NtDeviceIoControlFile */ 
\\







