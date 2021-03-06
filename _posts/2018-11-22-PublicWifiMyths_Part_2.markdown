---
layout: post
title:  "The Myths Of Public WiFi - Pt2 Can Your Private Information Be Stolen?"
date:   2018-11-22 14:56:03 -0400
categories: jekyll update
---

<br>[Part 1 (Can Your Traffic Be Monitored?)][part-1]

<b>Legal Disclosure: Don't do this on networks other than your own without written consent from the network administrator. Otherwise, you would be committing an illegal act.</b>

In our last article, we learned how to capture traffic. We learned that URL's can be monitored. If the information is encrypted, then can the attacker steal any information? No, absolutely not. The only hope they have is keeping the captured packets and waiting for someone to break the encryption, then publicly announcing their methods. Since 2009, HTTPS has been broken 3 times over the course of 9 years. The most recent crack for HTTPS was in 2014. But what if traffic is unencrypted? We've seen the Wireshark logs, and it's just raw packet data. How can we extract anything meaningful out of them?

<b>Roadmap</b>

-Poison the ARP Tables.

-Capture passwords with Wireshark.

-Spoof sessions with Wireshak.

-Learn to capture FTP passwords.

<b>Capturing HTTP Passwords</b>

Let's start by opening a new terminal window and poison the ARP.

{% highlight ruby %}
poison
{% endhighlight %}

Open your web browser and log into a non-secure website. I personally like to use an old game I used to play called [Neopets][neopets.com]
Once you open the login page, open a new terminal tab and run.

{% highlight ruby %}
tshark_http
{% endhighlight %}

With tshark running, go back to the browser and the respective login page and enter in a username or password. You are not required to have an account. Just enter anything. Once the password is rejected, end tshark with Ctrl+C.

Open the pcap file in your home folder and take note of all the data. You will notice that there is plenty of data going in and out. In order to get to the good stuff, you will need to use a filter. You can filter all HTTP request by typing the following into the Wireshark filter.

{% highlight ruby %}
http.host == www.neopets.com && http.request.method
#For reference, == means equals and && means AND in Wireshark format.
{% endhighlight %}

You are basically filtering all packets sent to neopets.com, then further filtering that selection by all packets that contain a session. Do not forget to include the www. There should be a limited number of packets to choose from. Look for either the words POST or GET, then look for something that looks like it could contain a password. Once you do that, expand the packet's "HTML Form URL Encode" field. Below you should be able to find the username and password.

![image tooltip](/blog/images/wifi/wiresharkpass.JPG)

note: not all non-secured login pages listed in the URL bar of your browser are insecure. Some have HTTPS request embedded but the browser will only pick up the initial page. Case and point, if you were to sniff traffic from [Pew Reasearch][Pew] then you would see that the login request uses port 443 (HTTPS). This does not mean that the service is safe because an attacker can still capture sessions.

<b>Capturing HTTP Sessions</b>

If you are an attacker and you don't have the opportunity to capture the initial login, you can still log in with the user's session credentials. A session is basically a string of numbers that lets the web server know that you have already logged in. You can open any HTTP stream then filter by

{% highlight ruby %}
http.host == www.neopets.com && http.cookie
{% endhighlight %}

Examine any of the selected packets, then open the "HyperText Transfer Protocol" section. Look under "cookie." A cookie is basically a session in HTTP speak. You can right-click the cookie and copy the printable value. Place it in a text file somewhere for safe keeping. Make sure you get rid of the <i>cookie: </i> at the beginning of the string. You should copy the user agent field as well in case the cookie does not work. Place the user agent into a separate line in your text file.

![image tooltip](/blog/images/wifi/wiresharkcookie.JPG)

Once you have copied everything, go to the website your target user is visiting. You will need to write the newly acquired cookie to your session. Go to the URL and type in

{% highlight ruby %}
javascript:document.cookie="VALUE";
#VALUE is the string you just copied from Wireshark
#NOTE: This will not work if the user has logged out or he has been idle for so long the cookie expires
{% endhighlight %}

If everything has been done correctly, then you should be able to use the victim's account. If you still can not access the account, then try to change the user agent with your web browser. This can be done with [User-Agent Switcher][User-AgentSwitcher] or simply Google how to do it in your respective web browser of choice.

<b>Exporting Raw HTTP Files</b>

If you would like to see everything the user has viewed, simply open the captured files in Wireshark, then <i>file > export objects > HTTP</i>

You can choose what to export or you can simply export everything and see what an attacker could see if he was sniffing your own HTTP traffic.

<b>Capturing FTP Passwords</b>

We start by repeating the capture process.

{% highlight ruby %}
tshark_ftp
{% endhighlight %}

Log into an FTP server of choice. Unlike HTTPS, you will not have to worry about FTP being encrypted. Once you have authenticated, quit tshark with Ctrl+C. Open the files and use the filter

{% highlight ruby %}
ftp.request
{% endhighlight %}

FTP is very straightforward so the username and password should appear in the description of the packet header.

<b>What About SSLStrip?</b>

The common misconception is that SSLStrip is a tool that can unmask HTTPS and convert it to HTTP. That would be true if and only if the victim's website is using outdated SSL certificates. While there are several articles claiming, and even video tutorials demonstrating the impacts of SSLStrip, I can guarantee that such a tool is only effective on outdated versions of HTTPS. The video demonstrations only appear to work because they control the target machine and trust fake certificates. It's very unlikely that you will be able to trick the average user into installing fake certificates. Modern browsers will display a gigantic splash screen warning the user of risk. The "accept" button is hidden in text detail. They can't simply continue on impulse without actively reading the warning. While it is possible that some users will be extremely determined to install just about anything regardless of common sense, SSLStrip will be beyond the scope of this tutorial series. Since the latest HTTPS breach, I have never once seen an outdated certificate. They are extremely rare. They are less common and less practical than attacking the more common unencrypted HTTP.

<b>Conclusion</b>

If your data is not encrypted, it is exposed. If it is encrypted, then it is safe. Several people use the same credentials for other services. If an attacker gets your username and password, then they could test it against common services and in some cases, even your email. You don't want to reset all your passwords in the event an attacker captures yours.

[Pt3. Can Your Private Information Be Stolen?][part-3]

[part-1]: https://danielloosec.github.io/blog/jekyll/update/2018/11/22/PublicWifiMyths_Part_1.html
[part-3]: https://danielloosec.github.io/blog/jekyll/update/2018/11/22/PublicWifiMyths_Part_3.html
[neopets.com]: http://www.neopets.com
[User-AgentSwitcher]: http://useragentswitcher.org
[Pew]: http://www.pewresearch.org/profile/
