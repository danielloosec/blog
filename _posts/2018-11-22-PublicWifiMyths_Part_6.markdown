---
layout: post
title:  "The Myths Of Public WiFi - Pt6 Securing Yourself"
date:   2018-11-22 14:56:03 -0400
categories: jekyll update
---

<br>[Part 5 (If I See The Lock In My Browser's URL, Does That Mean I'm Safe?)][part-5]

We have learned the advantages the hacker has when he is on the same public network as you are. There is plenty of work that goes into the initial setup but once the system is running, he can become very responsive. The actual risk of compromise itself is extremely small. It's not much higher than a victim on his personal secure LAN becoming compromised because he chose to click on some shady links while ignoring all the warnings. At some point, you may need to leave the safety of your own network fortress and go into the wild west of public LAN networks. There are plenty of steps you can take to defend yourself completely.

<b>Update Everything</b>

This is one of the most important yet least recommended solutions. If some malicious code successfully executes on your browser or someone injects malicious code into your system, then it's likely due to unpatched software. Browser developers, operating system developers, and antivirus developers are very responsive when it comes to security vulnerabilities.

<b>Turn Off Network Discovery</b>

You won’t be needing to share files on public WiFi. Especially now that we have more secure online services for such a task (Dropbox, SpiderOak, SSH, etc).

Instructions:

<br>[Windows 10][ndwin10]
<br>[Windows 7/Vista][ndwin7]
<br>[OSX][ndosx]

<b>Test Your Device For Vulnerabilities With Nessus</b>

I have mentioned [Nessus][nessus] software in my [first hack][first-hack-pt1] article. It is extremely useful for identifying vulnerabilities and the threat outcome on your device.

<b>Use Cloudflare's DNS</b>

By using Cloudflare’s DNS, you can prevent DNS hijacking. As an added bonus, it will make your internet even faster because you will be using top of the line infrastructure to resolve DNS requests. When you first access a router, you may need to disable this in order to access the public WiFi's terms of service page.

Instructions:

<br>[Windows 10][dnswin10]
<br>[Windows 7][dnswin7]
<br>[Windows Vista][dnswinvista]
<br>[OSX][dnswinosx]
<br>[iOS][dnsios]
<br>[Android][dnsandroid]

<b>Install HTTPS Everywhere</b>

HTTPS Everywhere is a plugin invented by one of the most trusted non-profit names in cybersecurity. The EFF. that takes an HTTP request and redirect you to an HTTP equivalent. Keep in mind that this will not work if the website is not configured to use HTTPS. It will only work if the website you are visiting does not default to HTTPS. You can use the plugin to block all non-encrypted traffic after you have agreed to the public WiFi's terms of service page.

Download: [HTTPS Everywhere][https]

<b>Install NoScript Lite</b>

NoScript is a plugin that will disable Javascript. If an attacker redirects you to a fake webpage or even gets you to connect to a fake AP then that webpage could be a page attempting to execute malicious code. If that is the case, malicious code is mostly executed with Javascript. Be sure to keep this off. You will notice your web pages loading faster since Javascript is also used to tracking scripts. If a webpage breaks while loading, you whitelist the page and reload it. The world is attempting to move away from Javascript web pages but some websites still depend on it.

Download: [NoScript Lite][noscript]

<b>Install Brave</b>

You won't be able to get the previous two plugins for mobile browsers. Fear not, there is a more secure browser that allows HTTPS redirects and Javascript blocking. It is called Brave.

Download [Brave][brave]

<b>Uninstall Flash and Java</b>

Much like javascript, Flash and Java can execute malicious code. You can not simply "update" them. Flash is notorious for [getting hit all the time.][flash] Both are meaningless in today's world. Everyone is trying to cut them out and Adobe themselves mentioned that Flash will be discontinued. Uninstall them both.

<b>Disable Automatic WiFi</b>

Keep this off. If a user attempts to deauthenticate you from the network, your computer will not reconnect to his evil-twin hotspot. If you are needed to type in a password before using the WiFi, then make sure you don't see two of the same Access points on the list. If you do, then alert the owner.

<b>Do Not Accept Unsigned Certificates</b>

If you are on a website that you know very well, then do not accept unsigned certificates if asked. Good news is the browser will make it hard for you to continue unless you actively read the warning message.

<b>BUY a VPN</b>

If you buy a VPN, then all your traffic will be encrypted. This includes your DNS information so a hacker won't even be able to look at what websites you have been visiting. Even if you connect to his router, it will not matter. Do NOT get a free VPN. They are free for a reason. The [EFF][eff] recommends using [That One Privacy Site][thatone] in order to review which security features your VPN has. I personally use [NordVPN][nord]. They give out coupon codes all over the internet.


[part-5]: https://danielloosec.github.io/blog/jekyll/update/2018/11/22/PublicWifiMyths_Part_5.html

[first-hack-pt1]: https://danielloosec.github.io/blog/jekyll/update/2018/04/16/MS08_067_Part_1.html

[nessus]: https://www.tenable.com/downloads/nessus

[ndwin10]: https://www.windowscentral.com/how-configure-network-discovery-windows-10-0
[ndwin7]: https://www.home-network-help.com/disable-file-sharing.html
[ndosx]: http://www1.udel.edu/topics/virus/security/mac/macsecure.html

[dnswin10]: https://www.windowscentral.com/how-change-your-pcs-dns-settings-windows-10
[dnswin7]: https://support.opendns.com/hc/en-us/articles/228006987-Windows-7-Configuration
[dnswinvista]: https://torguard.net/knowledgebase.php?action=displayarticle&id=188
[dnsios]: https://appleinsider.com/articles/18/04/22/how-to-change-the-dns-server-used-by-your-iphone-and-ipad
[dnsandroid]: http://osxdaily.com/2015/12/05/change-dns-server-settings-mac-os-x
[https]: https://www.eff.org/https-everywhere
[brave]: https://brave.com/download/
[nord]: https://nordvpn.com
[noscript]: https://mybrowseraddon.com/noscript-lite.html
[flash]: https://www.theregister.co.uk/2016/06/16/adobe_36_flash_flaws/
[eff]: https://ssd.eff.org/en/module/choosing-vpn-thats-right-you
[thatone]: https://thatoneprivacysite.net