---
layout: post
title: My OSCP Experience
category: Security
tags: [oscp, hacking, security, review]
---

When I was young, around the age of 12, I thought that becoming a Certified Ethical Hacker was THE goal in life I wanted to accomplish. Ten years pass by and I achieved that goal, only to find that it was much less fulfilling and technically satisfying than I originally thought. I gave up on certifications until hearing of the Offensive Security Certified Professional (OSCP). It sounded like a fantastic aptitude test to prove to future employers, and myself, that I maintained the mindset of a professional penetration tester.

## My Background

I'm 27, and I started learning to hack and code when I was about 10 years old. I learned everything I could learn through a dial-up AOL account. Once I had decided I wanted to learn to hack, everything I read seemed to suggest that I needed to learn to code first. So, I did. The career paths I observed while in high-school seemed to suggest that I would have to get lucky to land a penetration testing job leaving college. Instead, I decided to pursue a degree in software engineering , and aim towards the security field. This lead to me getting my Bachelors and Masters in Software Engineering with a focus in Information Assurance, and landing a job in a mixed role of penetration testing and software development.

## What is the OSCP

The Offensive Security Certified Professional certification is the world's first completely hands-on offensive information security certification. The OSCP challenges the students to prove they have a clear and practical understanding of the penetration testing process and life-cycle through an arduous twenty-four hour certification exam. In order to gain access to this exam, you are required to take the companion course, [Penetration Testing with Kali Linux](https://www.offensive-security.com/information-security-training/penetration-testing-training-kali-linux/).

From what I've gathered, about thirty-two thousand people have registered for this course up to this point in time. However, I would guess that only half (at most) of those people have passed the exam. I have spoken with many people about it, and the average amount of exam attempts before passing seems to be about 3. Some people pass the first try, and some people take 4 or 5 attempts before passing. Yikes! This is definitely an exam that pushes you to really understand what you're doing. That's exactly the reason I wanted to take it on!

### Penetration Testing with Kali Linux

Penetration Testing with Kali Linux (PWK) is half of the reason this certification is so valuable. PWK is a set of course materials in the format of a PDF, and accompanying videos, coupled together with a lab of roughly 55 machines that will allow you to learn and hone your skills to develop a foundation for breaking into the penetration testing field. More on this later!

## Who Should Take the OSCP

I would say that the OSCP is a great certification for anyone wanting to stake their claim as a penetration tester. Since the OSCP exam is hands-on, it proves that the certification holder can actually understand the basic concepts of mapping networks, enumerating services, finding and modifying exploits, and successfully gaining access to vulnerable systems. It also demonstrates a basic knowledge of network and operating system interactions. If you are looking to prove to someone that you deserve a spot in the penetration-testing realm, or if you're just interested in learning what it takes to become a penetration tester, I would say that the OSCP is for you!

## What Are the Prerequisites

There isn't a firm list of prerequisites to take the OSCP. Based on my experience from the course, you will need to have some basic knowledge of the following areas:

* Command Line Usage
* Linux
* Windows
* Networking
* Programming (only VERY basic knowledge)
* Google-fu (being able to self-teach through research/Googling)
* An appetite to learn!

Again, let me reiterate in saying that you only need a very basic knowledge of the above subjects. Completing courses for Python and Command Line on sites like [Code Academy](https://www.codeacademy.com) would be plenty. If you're unfamiliar with Linux, I recommend downloading a virtual machine of Kali or Ubuntu and just playing with the command line for a day or two. If you have a Mac, you can also use the Terminal app to get familiar with the command line. It is very similar to what you will use in the course.

## My Lab Experience

### Signing Up

I signed up for my course access on March 23rd. Upon signing up, you will receive an e-mail containing a VPN connectivity test pack, as well as a link to download a specific version of the Kali VM (curated by Offensive Security). Once you get that VM up and running, you'll connect to the lab VPN and ensure that your connection speeds will be adequate for the coursework and exam. This is the same VM most people will choose to use in the labs/exam so don't delete it when you're finished with the connectivity test.

Soon, after completing my connectivity test, I received an e-mail stating that I had been officially signed up for the course, and my start date would be April 17th!

### PDF / Videos

I had to work on the 17th, but that night I eagerly retrieved my course materials. I had already decided that I would NOT touch the labs until I finished the PDF and Video walkthroughs. So, I began working through the course materials starting with chapter 1.

I maintained that trajectory for a whole two days. I made it to around chapter 10 before I moved on to the labs. The PDF and videos were great, but the things after chapter 10 were things I was fairly comfortable with already, and I decided that when I got stuck in the labs I would come back and resume the materials. My development background definitely assisted me in bypassing the rest of the PDF for now. I was just too excited to pop my first box!

### Goals

Now that I was moving on to the actual labs, I felt the need to layout my goals. I had several, but my main goals were to tackle the "big three", which are Pain, Sufferance, and Humble, as well as to get through to the Admin network. If I could accomplish these goals, I would feel confident going into the exam.

Looking back, I would say that anyone who can complete these goals on their own should have no problem passing the exam.

#### Metasploit Usage

I was really focused on not using Metasploit for a while, but then decided that I would rather focus on compromising as many boxes as possible instead. To others considering whether to use Metasploit or not, I would say don't worry about it. If you are able to compromise some boxes with manual exploits, don't get hung up trying to compromise EVERY box with manual exploits. Do what you need to get through the entire lab. If you hit the 50 box mark, the exam will be a breeze either way.

### The Games Begin

The first stretch of lab time was pretty rough. I got a couple of boxes early on, but then things started to break down. I realized that what I thought was good enumeration, was actually terrible. An nmap scan is not enough. You need to really nail down your approach to scanning a box, enumerating the services running, enumerating what is accessible by the services running, and then once you gain access to something realize that it's time to start the enumeration process over entirely! This was my first and most valuable lesson.

Bonus: Always revert a box before beginning scanning :) This cost me about 8 hours of lab time ha.

### First 30-days (16 compromises)

My first thirty days involved a lot of painful lessons. I had no order in which to start attacking the boxes, so I just discovered the addresses in scope, and started scanning. One of the most valuable lessons I learned in this stage was to only scan what's necessary. My first few scans, I started out by doing a full version scan on all ports! This was a waste of time. I eventually nailed down my process to use three nmap scripts. The first determined all TCP ports that were seemingly open. The next script would do version/OS enumeration using only the ports found by the first script. The third was simply a quick UDP scan. This was adopted from another user's script, but I can't seem to find the link. If anyone has a link to the original please let me know so that I can give him credit :) Here is a link to my [nmap scripts](https://gist.github.com/audrummer15/7c8c3dc54d5c21d588a7b1ba1b4ef66d).

After a month, I had tackled 16 boxes, but I wasn't feeling extremely confident yet. The boxes I had fully compromised were:

r00t

---

**kevin** \| **jd** \| **alice** \| **susie** \| **helpdesk** \| **dj** \| **fc4** \| **beta** \| **tophat** \| **barry** \| **bob** \| **alpha** \| **payday** \| **phoenix** \| **ralph** \| **gamma**

---

### Month Two (12 compromises)

The second month of labs was all over the place for me. I was traveling a lot for work (practically the whole month). But I did manage to squeeze some time in for the labs. This section of the labs boiled down to one HUGE lesson. STICK TO YOUR PROCESS!!

Seriously, I wasted so much time hopping around, bouncing between this service and that service, and straying away from the process. When I say process here, I mean things such as finding a web server, and immediately checking robots.txt, running nikto, and running dirb. Those are first three things I always do when finding an open web port. Once I nailed down my process, the boxes from this month fell easily, and I was able to compromise two of the "big three".

r00t

---

**pain** \| **leftturn** \| **bethany** \| **dotty** \| **sherlock** \| **hotline** \| **mike** \| **sean**  \| **timeclock** \| **humble** \| **joe** \| **mail**

---

### The Last Leg (20 compromises)

When it came down to the last month, I'll be honest. I wasn't feeling ready. I had heard several stories first-hand of skilled people failing the exam multiple times. I really wanted to take these last 30 days and solidify my knowledge and process so that I would feel remotely competent enough to attempt the exam.

I was able to successfully compromise sufferance, and gain access to the admin network. FINALLY! All goals met. Was it some crazy battle between me and these boxes? Sort of. But it was more of a composure battle with me and myself! Seriously, the ways into these boxes are not tricky. There are no crazy techniques I needed to know. The lab tested for a basic set of skills and processes. Once these skills and processes were honed in, nothing could stop me.

I'll add here, for reference, that this month I really took advantage of the multiple subnet setup to practice pivoting. There were many ways I learned to pivot, but the most useful was definitely SSH tunnels, and I was glad to demonstrate my understanding of them to compromise... well, a few special boxes :)

I spent the last week and a half of my lab time finishing up the PDF/Videos, doing the exercises, and getting my documentation ready for the lab report. At the time of this writing, you can get an extra 5 points if you successfully answer ALL of the execise questions in the PDF, and do a full write-up of how you compromised 10 boxes in the labs. There is one edge-case where these 5 points could mean passing or failing, so I decided to do them. These were a little frustrating because they required specific knowledge about boxes in the labs, but if you compromised most of the labs then you'll be able to answer them.

r00t

---

**sufferance** \| **gh0st** \| **pedro** \| **niky** \| **jeff** \| **oracle** \| **kraken** \| **brett** \| **core** \| **observer** \| **slave** \| **master** \| **nina** \| **john** \| **carrie** \| **internal** \| **mario** \| **luigi** \| **tricia** \| **314159265**

---

## Exam Time

The days before my exam, I reworked 3 of the buffer overflow problems from the courseware, and documented each step in that process thoroughly. I also made sure to have a nice page of "quick scripts" that I could copy and paste into a terminal, such as a `wget` powershell script, ftp download script, etc... These were all from techniques I used in the labs. Windows was also not my strongest privesc platform (as shown by the labs), so I made sure to have a list of quick-win checks for privilege escalation on a windows box.

The night before the exam, I relaxed. Cooked dinner and watched TV with my wife, and got to bed early. I had scheduled my exam for 0800, and I wanted to get a good nights sleep due to the potential of being up for a solid 24 hours the next day. It was pretty hard to get to sleep due to the anticipation, but somehow I managed.

### Exam Strategy

In the exam, they give you the point value for each box, and you must acquire 70 points to pass. You can get partial credit for a low-privilege shell, and root/system access will give you full credit. I decided that my strategy for this exam would be to start high and work low. So I would go after the boxes with the highest point value first, and work my way down until I receive the 70 points.

### Getting Started [0700 hours]

I started the next morning with a shower, followed by a quick breakfast and a cup of coffee (Chemex, Single Origin Guatemalan). I sat down at my desk about 10 minutes early waiting for the VPN connectivity pack to be e-mailed to me. 0800 rolled around... 0810... 0815... After waiting what I thought was an adequate amount of time on the e-mail, I eventually had to hit up the support chat to ask about it. Apparently the e-mail was sent to the wrong e-mail address (my fault). Once we got the address straightened out, I promptly received my connectivity pack, loaded it into Kali, read through the exam materials, and began my exam.

### Confidence Boost [0830 hours]

Upon connecting to the exam I had already determined that I would be targeting the highest-point boxes first. So, I took on box number one. I was fairly confident about this box based on the description given by Offensive Security. I had a process written out in case I found a box like this. So I started working through my process.

About 30 minutes into working through my process something went wrong. I wasn't getting the results I expected. Are they tricking me? Is there a twist? Is it harder than the things I saw in the labs? Another 30 minutes pass, and I start from scratch. Working back through my process I found the step I missed (probably due to nerves) and I rooted the box no problem! 25 points.

Lesson learned: Have a process, know your process, stick to your process.

### Unexpected Win [0930 hours]

Based on my strategy I moved on to the second 25 point box. This box was a little more unfamiliar. So, I started from the process I nailed down while in the labs. Discover running services, enumerate services, research vulnerabilities. Nothing. Enumerate further. I found my entry point fairly quickly, and was able to get a low-privilege shell rather easily!

When landing on the box I was able to find my escalation path within a couple of minutes based on the experience I gained through the labs. Turning that path into root took about 30 minutes. Bam, 50 points in just two hours! This test was turning out to be a breeze, and I was confident I would conquer the rest of the exam in the next few hours.

### No Man's Land [1000 hours]

The next box I went after was 20 points, and things did not go as smoothly. I scanned and enumerated the machine, and I thought I found my way in but it didn't end up working out. I looked at the other services on the box, enumerated some more, and still came up with nothing. I went back to that first exploit that I thought should have worked, but I couldn't make it work.

At this point, I hopped over to the next 20 point box. Scanning and enumerating it as part of my normal process, but I couldn't find anything there either. After about an hour, I moved on to the 10 point box thinking that a quick win would get me back on my feet. Nothing again. I can't find anything!

Around 1230 I had begun to panic. Bouncing from box to box to box looking for any possible way in. I kept coming up with the same info for the first 20 point box.

### Success! [1930 hours]

Finally, almost 10 hours later, I didn't know what else to do. So I downloaded a copy of the vulnerable software to my Windows box provided by Offensive-Security, and started reversing the exploit I was using to see why it wasn't working. The exploit worked. What the heck? So I threw it at the box again, and it worked! It's about time I got another low-privilege shell.

Now was the hard part. Escalating. I spent a couple of hours escalating this one, but I finally got it around 2200! That's it! 70 points, I officially have enough points to pass my OSCP and the pressure is off.

### Flawless Victory [2300 hours]

I spent the next hour banging away at the 10 point box. I was so dumb. I should have gotten this one immediately, but the test can get to you. Anyways, in around an hour I had obtained administrative privileges. It was a very simple path.

Going for gold, I switched back over to the last 20 point box around 1100. This box required me to go outside of my normal process, but it was nothing I hadn't seen in the labs. So I guess my process was slightly off. It took about an hour to find my foothold on the box, and another hour to escalate. After 17 gruesome hours there I was. 100 points in the bag, screenshots galore, and some writing to do.

I did a quick take on my exam report that was due the next day, and got to bed around 0400. A successful day, and the feeling of accomplishment was incredible.

## Coffee Break!

Throughout my life few things have given me the sense of accomplishment, pride, and confidence as successfully completing the OSCP did. I was able to see massive improvements in my penetration testing knowledge not only in theory, but also in practice. This course gave me the credentials I needed to break into working with the penetration testing team I do development for. It also gave me the confidence to do penetration tests with a certain level of confidence. By no means have I hit a point where I can sit back and coast. However, I've proven to myself and my colleagues that I have the mindset and determination needed to be a professional penetration tester. Until next time, Happy Hacking!
