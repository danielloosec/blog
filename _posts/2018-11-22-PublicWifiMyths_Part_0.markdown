---
layout: post
title:  "The Myths Of Public WiFi - Pt0 Introduction"
date:   2018-11-22 14:56:03 -0400
categories: jekyll update
---

<b>Legal Disclosure: Don't do this on networks other than your own without written consent from the network administrator. Otherwise, you would be committing an illegal act.</b>

There has been plenty of confusion surrounding the safety of public WiFi. What is the risk, and how concerned should you be? Short answer, this risk is very minimal, yet it is still significant. Many authority figures have provided guides on the risk of public WiFi. Start by reading them.

[FTC][ftc]

[Homeland Security][dhs]

[Symantec][symantec]

[Kaspersky][kaspersky]

Read them? Excellent. There are a few misconceptions that people infer after reading these risks. Am I saying I know better than the top of the line security professionals? Of course not. Most of them are written to protect people who are very unfamiliar with technology. If you are a normal guy who knows how to fix your own computer when it breaks, then you're going to be ok for the most part. If you are the CEO of Cloudflare or the President of the United States then you will want to avoid the risk altogether. Many of them also want to err on the side of caution. For example, the rules explain that HTTPS is safe but they tell you to avoid online banking even when all major banks use HTTPS? Why is that? We are about to look into the perspective of an attacker’s abilities, limits, and the amount of risk you are willing to mitigate for yourself. But for now, there are 5 things you must always consider.

<b>1: The probability of an individual monitoring your traffic.</b>
Part of that is going to depend on how crowded the area is. If you are in a coffee shop with a handful of people, the odds are not high. If you are at a Las Vegas hotel, then the odds are higher. Yet the odds of it affecting you in any meaningful way are incredibly low.

<b>2: How valuable the information is.</b>
If you are an incredibly wealthy individual making financial transactions, then you can assess that your that your banking or business information is valuable. If you don't have much to lose or anything of worth to an attacker, then you won't have to be as cautious. If you are using a service for a company you are employed at, then the attacker might find value in breaching the company than you personally. At that point, you could be held responsible for negligence depending on how they choose to assess the situation. No one has to go full tinfoil as long as they understand the risk and choose a rational level of security.

<b>3: Is your application encrypted?</b>
Speaking for websites, you should always make sure the green lock appears in the upper left corner. This means the website is encrypted with HTTPS (as opposed to HTTP) and that is good. I can assure you that all popular mainstream websites have strong encryption. There's a lot of debate as to whether or not all websites always need to be encrypted. The short answer is yes, websites must ALWAYS be encrypted. Even if no login is required or the login screen is the only encrypted page, then you could be susceptible to DNS or Session Hijacking. We will prove that these threats are very real in later articles. Keep in mind that is is possible for an attacker to keep encrypted data, then decrypt it once the public learns how to break that version of the encryption. Yet this is highly unlikely. Also, keep in mind that other applications on your computer may not be safe. If an attacker gets the username and password for an unencrypted application, then they can use that password to log into other accounts that share the same username and password. You may have some applications running in the background that broadcast raw information that can be stolen.

<b>4: The amount of time an attacker is willing to spend stealing your information.</b>
As mentioned above, not everyone has valuable information. Most attackers will give up after a certain period of time. If you are running an unencrypted application, then it's still unlikely that the attacker is going to spend time making heads or tails of the information exclusive to the application let alone develop an exploit for it.

<b>5: If you need to enter information into a router in order to access the network</b>
Paid WiFi, even over HTTPS can be a risk. Hackers have ways of making fake versions that look like the real ones. They can even certify it for encryption free of charge with programs like [Let's  Encrypt][lets-encrypt] for free. If you have to provide your credit card number or anything sensitive, then refer to the first two steps to assess your risk.

If you are not interested in playing the role of an attacker, then skip to the [final article][part-6]

<b>What You Will Need</b>

-A copy of Kali Linux

-VirtualBox in order to host Kali in a virtual machine.

-A target system. I suggest installing a second virtual machine with [Windows XP as I have done][first-hack-pt1]. This is the ideal testing environment to ensure that the scripts are set up correctly.

-Your own router

-Internet access

[Pt1. Can Your Traffic Be Monitored?][part-1]

[lets-encrypt]: https://letsencrypt.org
[first-hack-pt1]: https://danielloosec.github.io/blog/jekyll/update/2018/04/16/MS08_067_Part_1.html
[part-1]: https://danielloosec.github.io/blog/jekyll/update/2018/11/22/PublicWifiMyths_Part_1.html
[part-6]: https://danielloosec.github.io/blog/jekyll/update/2018/11/22/PublicWifiMyths_Part_6.html
[ftc]: https://www.consumer.ftc.gov/articles/0014-tips-using-public-wi-fi-networks
[dhs]: https://www.dhs.gov/sites/default/files/publications/Best%20Practices%20for%20Using%20Public%20WiFi.pdf
[symantec]: https://us.norton.com/internetsecurity-wifi-public-wi-fi-security-101-what-makes-public-wi-fi-vulnerable-to-attack-and-how-to-stay-safe.html
[kaspersky]: https://usa.kaspersky.com/resource-center/preemptive-safety/public-wifi