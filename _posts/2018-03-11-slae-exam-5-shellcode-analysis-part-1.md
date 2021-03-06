---
title: SLAE Exam 5 Shellcode Analysis - Part 1
category: Security
tags: [slae, shell, assembly, programming]
---

Man, I've been slacking. It's currently 8:45PM, I'm sipping on some sweet Colombian medium-roast coffee, and it's way too late for that. I've gotta get this SLAE wrapped up though! So let's jump into assignment 5, which is all about analyzing third-party shellcode.

## Problem Statement

* Take up at least 3 shellcode samples created using Msfpayload for linux/x86
* Use GDB/Ndisasm/Libemu to dissect the functionality of the shellcode
* Present your analysis

For this assignment, I decided to break the blog post up into 3 separate posts. This will allow us to dive into each of the msfvenom payloads, while not causing an information overload on one monolithic page! The first payload we will research, will be `linux/x86/read_file`.

#### Generating the Files

If we want to analyze one of these payloads, there are two things we need to do. The first is to generate the payload using *msfvenom*. Once we have our payload in it's raw form (shellcode), we can use *ndisasm* to do our analysis. I chose *ndisasm* because this is the only analysis option given that doesn't execute any of the code to be analyzed (simulated or not).

To accomplish these two tasks, I used the following commands:

```bash
root@kali:~/courses/slae/exam/assignment5# msfvenom -p linux/x86/read_file PATH=/etc/motd -a x86 --platform linux > read_file.raw

root@kali:~/courses/slae/exam/assignment5# ndisasm -b 32 read_file.raw > read_file_analyzed.nasm
```

Once we have the ndisasm'ed file, we can read the assembly line-by-line to understand what's happening.


## Analysis

Let's go ahead and just start walking through the shellcode generated by ndisasm. I'll write comments to explain what each instruction is doing, and then summarize the overall purpose of each section.

#### Jump - Call - Pop!

One of the techniques described in the course is the Jump-Call-Pop technique. This allows shellcoders to easily find the address of a set of data. How? When the call method is executed, it places the address of the instruction directly after the call instruction onto the stack. Then, when whatever method that was called returns, the address of that instruction will be popped into EIP, and code execution will continue where it left off. In our case, instead of waiting until we return from the call, we pop as soon as we call in order to store the address of our target data into a register for immediate use. Let's take a look at it:

```
00000000      jmp short 0x38        ; JMP (jmp-call-pop)
```

This is simply telling us to jump to the instruction at position 0x38.

```
00000038      call dword 0x2        ; CALL (jmp-call-pop)

0000003D      das                   ; /
0000003E      gs jz 0xa4            ; etc
00000041      das                   ; /
00000042      insd                  ; m
00000043      outsd                 ; o
00000044      jz 0xaa               ; td
00000046      db 0x00               ; 0x0
```

This is where the magic happens. When the instruction at position 0x38 is executed, it places 0x0000003D onto the stack, and moves to the instruction at position 0x2.

```
00000002      mov eax,0x5           ; SYSCALL(5) // man 2 open
00000007      pop ebx               ; POP (jmp-call-pop) "/etc/motd" stored in ebx from jump-call-pop method
```

The instruction at 0x2 is irrelevant to this discussion, but the very next instruction, `pop ebx`, places the address 0x3D into ebx. And if you noticed, 0x3D just happens to contain the string for the file we want to read, `/etc/motd`. So now, we have a char* loaded into EBX. Pretty cool!



#### Open the Target File

The following is the next set of instructions to be executed:

```
00000002      mov eax,0x5           ; SYSCALL(5) // man 2 open
00000007      pop ebx               ; POP (jmp-call-pop) "/etc/motd" stored in ebx from jump-call-pop method
00000008      xor ecx,ecx           ; Clear ECX
0000000A      int 0x80              ; Call OPEN("/etc/motd")
```

We load the value of 0x5 into EAX, pop EBX loads a char* referencing our target file as previously mentioned, and then we clear out ECX. Finally, `int 0x80` is our trigger for making a syscall. The number loaded in EAX is mapped to a function, in this case open, and the call `open("/etc/motd", 0)` is made. More information on this call can be found in the `man 2 open` man page.

If any of this is unclear, it may be a good idea to visit my previous posts on SLAE before continuing.


#### Read the Target File

Now that we know the general flow, we'll look at the next function that is called - read:

```
0000000C      mov ebx,eax           ; call to open returns FD, store FD in EBX
0000000E      mov eax,0x3           ; SYSCALL(3) // man 2 read
00000013      mov edi,esp           ; Store stack pointer in EDI
00000015      mov ecx,edi           ; Store stack pointer in ECX (*buf for read destination)
00000017      mov edx,0x1000        ; Number of bytes to read
0000001C      int 0x80              ; Call READ(3, esp, 0x1000)
```

Here we see that the value in EAX, which holds the return value of the open function, is stored in EBX. Open returns a file descriptor for the opened file. So, EBX is holding our target file descriptor. We then move the value 3 into EAX to indicate we want to call the read function. Since read takes a buffer to read data into, we store a pointer to the stack in ECX, and tell the read function to read 0x1000 bytes by storing that value into EDX. This obviously means we will only read the first 0x1000 bytes of our target file (which should be enough).



#### Write the Results

Now that we have our file contents loaded into memory, we need to display them to the attacker.

```
0000001E      mov edx,eax           ; Returns number of bytes read
00000020      mov eax,0x4           ; SYSCALL(4) // man 2 write
00000025      mov ebx,0x1           ; FD to write to (stdout)
0000002A      int 0x80              ; Call WRITE(stdout, esp, numberOfBytesRead)
```

Here we see the return value from read, which is the number of bytes read, stored in EDX. We then move the value 4 into EAX to indicate we'd like to call the write function. Storing the value 1 in EBX indicates that we'd like to write to stdout. Finally, ECX still holds the pointer to the location in memory where we read the target file contents to.

#### Exit

Now that we've opened, read, and written the target file contents, the last thing to do is exit!

```
0000002C      mov eax,0x1           ; SYSCALL(1) // man 2 exit
00000031      mov ebx,0x0           ; return code of 0
00000036      int 0x80              ; Call exit(0)

```

## Wrapping Up

The `read_file` payload was a nice and easy payload to get us started. The following two posts will be much more fun, and we'll really get to dive into analyzing shellcode even more. Until then, happy hacking!


###### SLAE Exam Statement

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification:

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/

Student ID: SLAE-1158
