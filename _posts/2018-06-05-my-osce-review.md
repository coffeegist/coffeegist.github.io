---
title: My OSCE Review
category: Security
tags: [osce, hacking, security, review]
---

About a year ago, I received the most satisfying e-mail I had ever received. It was from [Offensive-Security](https://offensive-security.com), and it was stating that I had successfully obtained my Offensive Security Certified Professional ([OSCP]({% post_url 2017-09-04-my-oscp-experience %})) certification on my first try! I was ecstatic, and exhausted. I had put in a hard two months of studying beforehand, and it was time to take a break. But, after 10 months of focusing on other areas of life, I decided it was time to come back and grab my Offensive Security Certified Expert (OSCE) certification, and this is how I did it.


## Prerequisites

I think it's fair to assume that if you're reading this, you must be interested in learning more about the exam either because you are considering taking it, you are currently in the labs, or maybe you're in the middle of your exam! In any case, I feel it's best to lay out some of what I feel like are prerequisites for this course. If you think you'd like to take this course on, but feel you may be missing some of these prerequisites, feel free to check out my [mentorship program](/mentorships/) and we can chat one-on-one to discuss what steps you can take to accomplish any goals you have!

First and foremost, you need to have SOME idea of how programming works. Some people might say this isn't necessary, but I am 100% convinced that my programming background allowed me to pass the OSCE on my first try with 12 hours to spare, even while making boneheaded mistakes. Just knowing how programming works will take you a long way. If you're looking for suggested languages, I would say studying up on C and PHP would be sufficient. Just a basic knowledge of these languages will do. C isn't used throughout the labs, but understanding C helps you understand how assembly works as well. Knowing how to build python scripts (and programs) will definitely help you throughout the course as well, but I wouldn't call it a prerequisite.

I would also almost call the OSCP another prerequisite. It's not necessarily essential, but it sure helped me out.

Finally, the question I get asked the most is, "Do I need to know assembly?". It's a difficult question to answer truthfully, but I always respond with the question, why not?! Sure, you'll get walked through some basic assembly during the course, but it is not a primer on assembly. You're either expected to know it, or learn it. So, for people with zero assembly experience, I would recommend [Assembly Primer for Hackers](https://www.youtube.com/watch?v=K0g-twyhmQ4&t=16s). If you have any assembly experience at all, I would recommend jumping straight into [SecurityTube Linux Assembly Expert](http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/index.html). I completed this course before starting the OSCE, and I can guarantee it's worth it.


## Registration

Before you can sign up to begin your journey, you must pass a registration challenge. This is an excellent tool to gauge your readiness. In my case, I passed several parts of the pre-registration challenge quite trivially. However, upon coming to the last section, I felt a little shaky. I completed it, but I didn't do it well and I felt like there were probably better ways to do it. This is what prompted me to sign up for the SLAE, and I'm glad I did. Listen to your brain, if the registration challenge causes you to feel unready, it's because you are. Go do some other self-study, and come back to this when you're ready.


## The Labs

The labs for the OSCE companion course, Cracking the Perimeter (CTP), were very different from the Pentesting With Kali (PWK) labs. In PWK you're given access to some labs, and told to GO! You have to do discovery, reconnaissance, the whole picture! With CTP, the PDF contains walkthroughs of various scenarios, and the labs provide a place to duplicate these walkthroughs. Topics covered in the labs [include](https://www.offensive-security.com/documentation/cracking-the-perimeter-syllabus.pdf):

* Web Application Testing
* PE File Modification
* Antivirus Bypass
* ASLR Bypass
* Egg Hunters In-Depth
* Finding Zero-Days
* Structured Exception Handlers
* GRE Tunneling

I'll be honest, the labs were a little disappointing at first. I was expecting some required problem solving, but it seemed to be straight up walkthroughs. I was familiar with a few of the concepts already, so I didn't have to think too hard at first.

However, when I started trying to automate some of the techniques I was learning, the challenge started to set in. This is when I really started to enjoy the course. I used python to automate several steps within the courseware, and it wound up being some pretty cool projects that I'm glad I worked on.

Module 0x08 was by far my most enjoyable time in the labs. I was able to learn a lot through walking through this exercise a couple of times, as well as trying to automate it. This is the module I practiced on the most. The rest of the course I did once or twice, and felt like I understood it well enough.

This is where the CTP and PWK differed the most. When my PWK lab time ran out, I wished I had more. At the end of the first 30 days of CTP, I felt like I had soaked in everything I could from the labs. I would say if you have 30 days to dedicate to it, then that's all you need for CTP. If you'll be in and out, 60 days is plenty. But with my lab time coming to an end, I decided to schedule my exam with an 0800 start time and just go for it.

## Exam

The dreaded exam. Everyone I work with had already terrified me of it, reviews I read online made it seem almost certain I wouldn't pass. But, the day came, and I rolled out of bed around 0630. I grabbed a shower, made breakfast, grabbed some Monticello Sunrise Ethiopian coffee from my local roaster, and sat down at my desk awaiting the exam email which includes instructions and a set of credentials.


#### 01 x 0800 - Let's Get It

The e-mail arrived precisely at 0800, and I began reading through it to make sure I understood the objectives. I set up my VPN, re-read the instructions and objectives, documented what I needed to, and I was off and running by 0830, after what I thought should be the easiest set of objectives in the exam.


#### 01 x 1100 - Trouble in Paradise

One hour had went by, and I wasn't 100% sure what was going wrong, but I was still confident. Two hours go by, and I begin to panic. I thought I understood what to do, I knew how to do it, but I just couldn't. Was it me? Was it the exam? Was everything functioning properly in the environment? Clearly not, and I spent a third hour verifying that it must be something I don't see. But at the three hour mark, I decided it was time to move on and change up the objectives I was targeting.


#### 01 x 1200 - I See a Shell!

This new set of objectives seemed more my speed. I read the tasks, began working, and immediately began seeing ways of completing the challenges. A little CTP sauce here, a little back-knowledge there, and bam! I have found a way of completing a small objective! This was fantastic, as it was the first blood I had drawn, but I was still far off. From this, I felt it would be easy to continue to complete the objectives on my current list.


#### 01 x 1500 - Finishing is Hard

By 1400 I had decided that I should stop thinking that things will be easy. Nothing is easy in this test. Well, it is, but you have to think about it properly first :) I spent three hours trying to finish my currently chosen objectives before deciding that I was just being dumb, and would have to move on to some new objectives. These seemed very familiar, so I was hoping there wouldn't be too many twists along the way.


#### 01 x 1700 - Sure-Shot

It took two hours, some head banging, and some moments of panic, but I finally finished a full set of objectives from start to finish. I definitely over-complicated this set of objectives, but a win is a win, and I gladly took it at 1700. My wife walked in just as I realized I had successfully completed it. We celebrated and she cheered me on, but it was time to get back to the basics. Maybe now, with some confidence, I can go back and try to figure out what went wrong with my previous set of objectives.


#### 01 x 1830 - Finishing isn't as Hard as I Thought

It took me a little while longer to realize that I was trying to make this harder than it was. I went back to the basics, pretended I knew nothing, and started there. Within about 10 minutes I was able to successfully complete this set of objectives, and I was HALF WAY there! A full 45 out of 90 points! Now, I still felt like I may not pass, but at least this was a start. Let's move back to what should have been my easiest set of objectives.


#### 01 x 2000 - Back to the Easy Stuff?

With my new-found hope and the idea that I MIGHT actually pass this on the first try, I began looking back at my first set of objectives. I thought I had some new methods to try. I spent an hour baking these new ideas, but none proved fruitful. What in the world?! How is this so hard? It should be so easy, and I just can't do it. Whatever, let's pick some new objectives...


#### 01 x 2300 - On to the Beast

I had a small number of objectives that I hadn't looked at thus far, so I decided to focus on these. Wow, this was the real deal, but didn't seem like it should be too hard. I began with what I knew, but nothing seemed to work. I tried a few new techniques, and still nothing worked. It was starting to get late, I had been zoned in for a long time, and I didn't seem to be making progress. I made the hardest decision of the day...


#### 02 x 0200 - Off to Bed

I decided to go to bed. I was 18 hours in at this point, with only half of the credit I needed to ace the exam. I didn't feel great about it, but I also didn't feel terrible. As I lay down to get some rest, I had a last thought, and ran back into the den to automate a few things to run overnight before laying back down and going to sleep. I slept pretty well knowing that my computer never stopped working on the exam for me :)


#### 02 x 0700 - I See a Path!


At 0700 I awoke with a fresh mind, ready to attack the day. The first thing I did was check the computer to see if it had worked hard for me overnight, and it had! I saw data that I felt would be crucial in passing the exam. I was ecstatic, but decided to get a shower, food, and coffee in my system before sitting back down at my desk to tackle the current objectives.


#### 02 x 1700 - If This Doesn't Work...

I feel like I'm making progress for the first hour, and then I hit a road block. I find a way around the road block, and then hit another issue another hour later. At this point, I start to wonder if I'm running down a rabbit hole, but I keep pushing because in my mind I can't see a reason why this shouldn't work. I automate some more things, and seem to make more progress. I'm a few short steps away now, and I feel victory in my hands. I can't seem to grab it. I spend another 4 hours only to realize that I made some assumptions about my automation that cost me precious time. I went back to verify things manually, and once that was complete, I had done everything I knew to do. If this didn't work, I'd be clueless, with a full day wasted. I almost didn't even want to try it, because if it didn't work I'd be devastated. My wife walked in, I told her where I was, and she watched with anticipation as I made my final adjustments and made my attempt.

SUCCESS!! At 33 hours into my test, I've managed to gain enough points to pass! I was ecstatic. We celebrated and relaxed together for about an hour, and then I decided that it was time to ace this thing.


#### 02 x 1800 - Finish Him!

Circling back to the first set of objectives I chose, I wanted them. I explored proof of concepts, double checked everything, wrote things, rewrote things, everything seemed to checkout. It was only when it explaining it to myself that I realized how dumb I was being. I had completely tunnel visioned and not thought about the objectives' entire context. What an idiot. I now knew EXACTLY what needed to be done, and was able to put it together in about 15 minutes. But would it work?


#### 02 x 2000 - Perfection!!!

Yes!! 90 out of 90 points! A perfect score on my first OSCE attempt. I literally wasn't sure if I would be able to pass the exam that day, but hard work paid off, and I was able to clear it with 12 hours to spare! This was definitely one of the most celebrated moments in my certification journey so far, and I relaxed in it for quite some time. We had dinner, watched movies, and enjoyed the fact that the worst was over.


#### 02 x 2200 - Documentation

A couple hours later, I decided that I needed to go ahead and do my documentation while I still had access to the lab. This way, if I missed any screenshots, I could still go grab them. This proved plenty fruitful as I wound up fixing at least 5-10 screenshots. I wrapped up preliminary reporting around 0300 of day 3, and called it a night.


#### 03 x 1000 - Submission

The next day I woke up around 0930, checked my documentation with a clear mind, and after a few very small tweaks, I submitted the documentation to OffSec for review.

## Coffee Break!

It only took one day for them to get back with me when the most fantastic news,

```We are happy to inform you that you have successfully completed the Cracking the Perimeter certification exam and have obtained your Offensive Security Certified Expert (OSCE) certification.
```

What an incredible feeling to know that you put in some hard work, but it paid off and you've made it. I'm thankful to OffSec for such a great course, and thankful for the community around it for keeping the integrity intact so that it still actually means something to complete the OSCE. Since the only other OffSec classes are offered at BlackHat, I think I'll be done for now. Rumors are floating that AWAE may be coming to the internet soon, and if so, I'll definitely be jumping on that! Thanks for reading my review of the experience I had with the OSCE. Remember to sign up for our mailing list if you're interested in courses that teach you some of the back-knowledge I used to pass the exams! Until next time, happy hacking!

##### Extra Resources

I've heard the following resources are good if you are looking to prepare for the OSCE journey, exam, or if your'e just looking for supplemental knowledge:

* [CoreLan Exploit Writing Tutotials](https://www.corelan.be/index.php/articles/)
* [FuzzySecurity Windows Exploit Development Tutorial Series](http://www.fuzzysecurity.com/tutorials.html)
