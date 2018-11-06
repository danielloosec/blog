---
layout: post
title:  "The Myths Of Public WiFi - Pt1. Can Your Traffic Be Monitored?"
date:   2018-08-29 14:56:03 -0400
categories: jekyll update
---

<b>Legal Disclosure: Don't do this on networks other than your own without written consent from the network administrator. Otherwise, you would be committing an illegal act.</b>

There has been plenty of confusion surrounding the safety of public WiFi. What is the risk, and how concerned should you be? Short answer, this risk is very minimal, yet it is still significant. We will look into the perspective of an attacker’s abilities, limits, and the amount of risk you are willing to mitigate for yourself. But for now, there are 5 things you must always consider.

<b>1: The probability of an individual monitoring your traffic.</b>
Part of that is going to depend on how crowded the area is. If you are in a coffee shop with a handful of people, the odds are not high. If you are at a Las Vegas hotel, then the odds are higher. Yet the odds of it affecting you in any meaningful way are incredibly low.

<b>2: How valuable the information is.</b>
If you are an incredibly wealthy individual making financial transactions, then you can assess that your that your banking or business information is valuable. If you don't have much to lose or anything of worth to an attacker, then you won't have to be as cautious. If you are using a service for a company you are employed at, then the attacker might find value in breaching the company than you personally. At that point, you could be held responsible for negligence depending on how they choose to assess the situation. No one has to go full tinfoil as long as they understand the risk and choose a rational level of security.

<b>3: Is your application encrypted?</b>
Speaking for websites, you should always make sure the green lock appears in the upper left corner. This means the website is encrypted with HTTPS (as opposed to HTTP) and that is good. I can assure you that all popular mainstream websites have strong encryption. There's a lot of debate as to whether or not all websites always need to be encrypted. The short answer is yes, websites must ALWAYS be encrypted. Even if no login is required or the login screen is the only encrypted page, then you could be susceptible to DNS or Session Hijacking. We will prove that these threats are very real in later articles. Keep in mind that is is possible for an attacker to keep encrypted data, then decrypt it once the public learns how to break that version of the encryption. Yet this is highly unlikely. Also, keep in mind that other applications on your computer may not be safe. If an attacker gets the username and password for an unencrypted application, then they can use that password to log into other accounts that share the same username and password. You may have some applications running in the background that broadcast raw information that can be stolen.

<b>4: The amount of time an attacker is willing to spend stealing your information.</b>
As mentioned above, not everyone has valuable information. Most attackers will give up after a certain period of time. If you are running an unencrypted application, then it's still unlikely that the attacker is going to spend time making heads or tails of the information exclusive to the application.

<b>5: If you need to enter information into a router in order to access the network</b>
Paid WiFi, even over HTTPS can be a risk. Hackers have ways of making a fake verions that look like the real ones. They can even certify it for encryption free of charge with programs like [Let's  Encrypt][lets-encrypt] for free. If you have to provide your credit card number or anything sensative, then refer to the first two steps to assess your risk.

<b>Roadmap</b>

-What you will need.

-A copy of Kali Linux

-VirtualBox in order to host Kali in a virtual machine.

-A target system. You can just use your main operating system as the target.

-Your own router

-Internet access

Kali Linux comes with a tool called Wireshark. It is extremely useful for monitoring data on your device. You can use this tool to monitor either a specific device on the network or all devices on a network if you so with.

You can capture traffic through a network with a three-step process.

-Setting your network adapter to bridge

-Creating command shortcuts with .bashrc

-Poisoning the arp tables on the router.

-Forwarding the IP.

-capturing a copy of the TCP traffic flow with Wireshark.

ARP tables are like spreadsheets on the router. They assign IP’s to MAC addresses. IP addresses need to know which device to forward traffic to, and ARP tables provide the directory. As the attacker, we are going to make the victim’s IP point to our hardware instead of theirs. Once their traffic is redirected to our computer, we will forward the traffic back to them. In short, their traffic will need to flow between our computer before reaching its destination, hence the name “Man In The Middle” Attack.

<b>Configure VirtualBox Bridge Adapter</b>

If you followed any Kali Linux/VirtualBox installation tutorial, odds are they had you make NAT adapter. You will not be using this because the two virtual machines will not be able to communicate. In order to fix this, we need to install a Bridge Adapter.

Right-click the Kali VirtualMachine > Settings

Click "Network" on the lefthand sidebar

Change the "Attached To:" dropdown to Bridged Adapter

Change the "Name:" dropdown to the name of your network adapter. If you do not know the name, then it's usually the first option.

<b>Making life easier with the .bashrc file</b>

What is .bashrc? Network security requires plenty of commands. Some of those commands in Kali are extremely long and it can be tedious typing them in over and over again. First, boot up Kali Linux in VB. The .bashrc file allows you to make command shortcuts. So start by opening .bashrc by opening a new terminal window. next type

{% highlight ruby %}
nano .bashrc
{% endhighlight %}

Note: If you choose to modify this file in something like Mousepad, then all you need to do is make it visible in the file manager. Click on the file manager, click on the toggle view drop-down in the upper right-hand corner, then click "Show hidden files." Once you have done that, the file labeled .bashrc should show up and you can edit it.

We will start by creating a series of variables at the top of the document. Commands often require us to type in the same set of commands over and over again. Make sure the first couple of lines include the following. I will use my own local IP's but you should use yours.

{% highlight ruby %}
targetIP="192.168.1.100"
myIP="192.168.1.4"
gatewayIP="192.168.1.1"
wifi="wlan0"
{% endhighlight %}

If you don't know how to find all this information, then we will go through it step by step.

<b>targetIP:</b> In your target computer (which I assume is windows), type in ipconfig. Look for the line that says IPv4 Address. If you are in a real world scenario, then you will need to obtain the IP by gathering more information on all computers on the network. For more information on how to do that, set up arp-scan and nmap using my [previous article.][first-hack-pt2]

<b>myIP:</b> This is the IP for your Kali Linux machine. Type ifconfig and look for the 192.168 address.

![image tooltip](/blog/images/wifi/ifconfig.JPG)

<b>gatewayIP:</b> Normally it is something like 192.168.1.1. If you are using your Windows machine, type {% highlight ruby %}ipconfig{% endhighlight %} again and look for the line titled Default Gateway. If you want to know how to do this in Linux, type {% highlight ruby %}ip r{% endhighlight %}

<b>eth:</b> Look at the ifconfig screenshot again. I highlighted eth0. This is my adapter name. The name is typically eth0 but you should double check.

<b>wifi:</b> Same as eth. The only difference is that this is the wireless adapter name when you are NOT using a virtual machine. Leave it wlan0 for now, but be sure to double check if you are using Kali as your main system.

Now that we have finished configuring the variables, we are going to add in some function commands. What are the functions? Let's break it down. The shortcut always will appear in the following format. function short() { long; }. The short is the shortened version of the actual command. The command itself is in between the { } space. Comments about the command referenced via #

On a new line, add the following

{% highlight ruby %}
function update() { sudo apt-get update && sudo apt-get -y upgrade; }
#This keeps your system up to date. Always run this before experimenting.
{% endhighlight %}

{% highlight ruby %}
function ip_forward_on() { sudo sysctl -w net.ipv4.ip_forward=1; }
#sysctl: used to modify built in system varibles.
#-w: used to specify a change in sysctl itself.
#net.ipv4.ip_forward=1: Enable IP forwarding which allows our network monitoring tool to forward all traffic to a different device.
{% endhighlight %}

{% highlight ruby %}
function ip_forward_off() { sudo sysctl -w net.ipv4.ip_forward=0; }
#Same as above, but =0 turns it off. Use this command when you are done.
{% endhighlight %}

{% highlight ruby %}
function poison () { sudo ettercap -T -M arp:remote /$gatewayIP// /$targetIP//; }
#ettercap: A tool that is used to monitor traffic
#-T: Text only interface. As opposed to graphical
#-M arp:remote: Tells ettercap to perform a "man in the middle attack" as discussed earlier in the article. We will tell it to use the arp attack to collect inbound and outbound traffic. As opposed to arp:oneway which only poisons the outbound traffic from the target.
#/$gatewayIP// The first target. Redirecting all traffic from target1 to target2
#/$gatewayIP// The second target.
{% endhighlight %}

{% highlight ruby %}
function poison_all () { sudo ettercap -T -M arp:remote /$gatewayIP// ///; }
#Same as above, but an unspecified target means, "collect all traffic going to and from the first targeted device"
{% endhighlight %}

The next function can not simply be be added with a copy paste. You will need to look at the second command and change myuser:myuser to whatever your active username is.

{% highlight ruby %}
function mv_pcap () 
{ 
sudo chmod 600 /tmp/*.pcap;
sudo chown myuser:myuser /tmp/*.pcap;
sudo mv /tmp/*.pcap ~;
}
#modifies the permissions for saved packet captures then moves them to the home folder
{% endhighlight %}

Save the file with Ctrl+O and confirm the changes. Press Ctrl+X to exit nano. You will need to close and reopen the terminal in order for the changes to take effect.

<b>Monitoring A Specific Individual</b>

Open a new terminal window and run the following commands

{% highlight ruby %}
ip_forward_on
{% endhighlight %}

{% highlight ruby %}
poison
{% endhighlight %}

You should a bunch of random data flying through. This means it is working. In the future, these will be the only two commands you will need to run in order to poison an ARP cache. If you ever need to change the IP addresses, then you simply need to modify the .bashrc variables at the beginning. While all that is running, open a new terminal tab with Ctrl+Alt+T.

<b>Capturing Traffic With Wireshark</b>

With the net terminal tab open, we are going to capture the traffic and store it for later viewing. Wireshark is a wonderful tool for this. Wireshark is normally used as a graphical application, but one can only collect so much traffic with the graphical tool before running out of RAM. The target might be doing something you as the attacker would consider interesting. He might simultaneously be watching Youtube, uploading files to the cloud or even listening to a podcast. This amount of data that will have passed to and from his device will add up to something tremendous. You will want to limit the file size. Much like RAM, hard drive space is also a limited resource. You can save a file with a maximum file size limit. Once that limit is reached, then a new file is created. You can specify the maximum number of files you want to create. Once the limit has been reached, Wireshark will start deleting older files in order to make room for newer ones. For security reasons, we want to save these files to the /tmp/ directory, then copy them to our home directory when we are done. Now run the following.

{% highlight ruby %}
sudo tshark -i $eth -b filesize:8192 -a files:10 -w /tmp/capture.pcap
#tshark: Wireshark capture tool for command interfaces.
#-i: Specify the interface. We are using our custom variable
#-b: Limit the capture either by duration, filesize, or number of files. We will be using filesize. Filesize in KB, so 8192KB is 8MB. You can make them larger if you wish.
#-a: Autostop. When to stop the process. In this case, after 10 files.
#-w: Write files to a directory. In this case, /tmp/. Make sure the file name ends with .pcap so we can open it later.
{% endhighlight %}

<b>Reading Captured Traffic</b>

While Wireshark is running, browse the internet with your target system. Take a break and do what you would normally do on the internet. Reframe from playing online games or streaming since this takes up a lot of space. I would suggest simply browsing the internet. After some time has passed, click the second terminal tab. Press Ctrl+C to stop the capture process if it hasn't stopped itself. Click on the first terminal tab. Press q in order to quit ettercap. Now run the following command to re-enable ip forwarding

{% highlight ruby %}
ip_forward_off
{% endhighlight %}

You will want to open the /tmp/ directory and move all the saved captures to your home directory. First edit the permissions, then use the move command

{% highlight ruby %}
mv_pcap
{% endhighlight %}

Double click any of the given capture files. You should see a long spreadsheet of information. Most of this information will be meaningless to you as an attacker. Why? If you simply want to see what website someone is visiting, then you don't need all the images, ads, and other stuff that comes with it. You just need to know the URL he is visiting. There are tons of ways to filter, analyze and parse information. Much of it will require more than a few simple blog post. But there are a few key things to look for.

<b>Filtering By Websites</b>

If you don't already have a collum named "port" then we will add it. Right click on any of the collum names  and click "collum preferences." We are going to add two new columns. The first will be a collum named "port". Once we do that, associate it's "type" with "destination port" from the drop-down menu. Also be sure to change "Time" to UTC Time from the dropdown, just to make things a bit easier.

Look for the collum named protocol. Click it in order to categorize all the captured packets by the protocol they were using. This should give you a good hint as to which services they were using. Each protocol should have an associated port. Protocols with corresponding Ports that should be of interest are...

{% highlight ruby %}
FTP	21
#File transfer 
SSH	22
#Remote login (Might be encrypted depending on configuration)
SMTP	25
#Email sending (Most likley encrypted)
DNS	53
#Website URL
HTTP	80
#Web traffic (unencrypted)
POP3	110
#Email receiving(Most likley encrypted)
SMB	445
#Network File Transfer
{% endhighlight %}

When we filter, we will need to be using the field's object name, followed by the operator, then the value. We will only be using "tcp.port" for today. We just want to know what website's people are visiting. We can do that regardless of whether or not the web traffic is encrypted or not. Go to filter and type "DNS".

You will be able to see plenty of "DNS" packets. While web traffic is encrypted, the domains the victim is visiting are not. This is because domains are resolved to their original IP via a separate service named DNS, or the accurately named "Domain Name Service." This should give you a clear picture of what websites the victim has been using.

<b>Save Only Filtered Traffic</b>

Now that we have an idea of what the victim is using, we can set Wireshark to capture traffic by ports exclusively. This makes the file size smaller and easier to read. We are going to go back into our .bashrc file and add a few more lines.

{% highlight ruby %}
function tshark_dns() { sudo tshark -i $eth -f "port 53" -b filesize:8192 -a files:10 -w /tmp/capture_dns.pcap; }
#same as the original tshark command but captures dns exclusivly
#-f: Specify port. you can say things like "port 53 and port 80" if you wish to save multiple protocols in the same packet capture.
#this particular function only captures dns traffic
function tshark_ftp() { sudo tshark -i $eth -f "port 21" -b filesize:8192 -a files:10 -w /tmp/capture_ftp.pcap; }
function tshark_ssh() { sudo tshark -i $eth -f "port 22" -b filesize:8192 -a files:10 -w /tmp/capture_ssh.pcap; }
function tshark_http() { sudo tshark -i $eth -f "port 80" -b filesize:8192 -a files:10 -w /tmp/capture_ssh.pcap; }
function tshark_pop3() { sudo tshark -i $eth -f "port 110" -b filesize:8192 -a files:10 -w /tmp/capture_pop3.pcap; }
function tshark_smb() { sudo tshark -i $eth -f "port 445" -b filesize:8192 -a files:10 -w /tmp/capture_smb.pcap; }
{% endhighlight %}

Save and exit the terminal in order for changes to take effect. When you run Wireshark, you can open as many extra terminal windows as you like and capture as much filtered traffic as you deem necessary.

[Pt2. Can Your Private Information Be Stolen?][part-2]

[lets-encrypt]: https://letsencrypt.org
[first-hack-pt2]: https://danielloosec.github.io/blog/jekyll/update/2018/04/16/MS08_067_Part_2.html
[part-2]: https://danielloosec.github.io/blog/jekyll/update/2018/08/29/PublicWifiMyths_Part_2.html