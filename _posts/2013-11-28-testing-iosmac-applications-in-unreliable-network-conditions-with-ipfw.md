---
id: 7
title: Testing iOS and Mac applications with ipfw in unreliable network conditions
date: 2013-11-28T20:21:00+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=7
permalink: /ios/testing-iosmac-applications-in-unreliable-network-conditions-with-ipfw/
categories:
  - iOS
  - ipfw
  - network
tags:
  - ios
  - ipfw
  - mac
  - network testing
---
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><i>(This article is clearly targeted towards iOS development, but ipfw can be used for any type of network connection manipulation. Just disregard the iOS-specific portions)</i></span></span></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">How many apps you developed use networking? How many apps on your device use networking? Yeah, it's pretty clear that networking is a major consideration in today's software development, especially in mobile. In iOS, Apple always test applications submitted to the AppStore in environment with no internet access. Some special care has to be taken with testing your applications in such conditions. And yet, it is surprising how many programmers are not taking the time to setup a test environment. We are going to do just that.</span></span></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Nowadays, such functionality can be accessed right from the “Developer” section of your iPhone. It contains options for limiting bandwidth and introducing delays in the network connection. This is not some black magic created by Apple. In fact, the principles behind it are not anything that complicated. If those settings satisfy your needs, that's great, but what if you needed a little bit more control? Well, you can use ipfw!</span></span></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">So what IS ipfw? Well, ipfw is the standard firewall installed on Mac OS X and other BSD systems. It is the same thing iptables is for Linux machines. Stepping backwards a little bit for the unenlightened, a firewall inspects all incoming and outgoing network traffic on a system and controls what goes through and what gets blocked. You may not notice it, but it's there right now (looking at your browser history :) ) and protecting you from malicious traffic. Since it already observes and manipulates all connections, the developers have put in some handy tools for testing a configuring your network. You can block, delay, forward, modify and even say what percentage of packets you want the firewall to drop. This is what we are going to be using.</span></span></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">The first question we have to answer is – How do you use ipfw with a mobile device? It is easy for desktop applications – they are already on the same machine that has ipfw. If you hoped that there is some magical IPFW application in the AppStore, I have some bad news for you. If you hoped that there is some magical IPFW.app file for Mac OS X, I have some bad news for you. We will have to use the Terminal, but don't worry – it's just for a while. The Terminal can be accessed from Applications → Utilities → Terminal, just FYI.</span></span></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">First things first, we will have to share our internet connection with the mobile device in order to control all traffic it generates. This might cause some problems on a Windows machine, it might require some iptables work on Linux, but on a Mac it's a piece of cake. Just go to System Preferences → Sharing → Internet Sharing. We will be sharing the computer's internet connection coming from the cable (Ethernet) to the Wi-Fi network. Of course, this would require you to have an Ethernet cable connected to your machine. This might be a bit of a problem with the new MacBooks that don't have that port. In such case, you might need to use a USB ethernet adapter. To start internet sharing, select your cable network in the “Share your connection from” section and tick “Wi-Fi” in the “To computers using” combo box. You might want to set a password for your newly created hotspot in the “Wi-Fi Options” page in order to secure your network (In fact I would strongly recommend that). With all that completed, click on the check box next to the “Internet sharing” option to start your shared connection.</span></span></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">It this point the Wi-Fi icon on the top of the screen should start showing an arrow. This means that everything works as expected so far.</span></span></p>
<p align="LEFT"><a href="{{ site.url }}/images/2013/11/SharingSettings.png"><img class="size-medium wp-image-10 aligncenter" alt="SharingSettings" src="{{ site.url }}/images/2013/11/SharingSettings-300x185.png" width="300" height="185" /></a></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">You can now go ahead and connect your smartphone to the new hotspot. You should have a working internet connection.</span></span></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Additionally, you might have to check the IP address your device got so that you know which address to use in your ipfw rules. You can easily do that in Settings → Wi-Fi and tapping on the accessory button for your network.</span></span></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">In addition to being able to manage the internet access to your device from the computer, this setup can be very convenient for sniffing the traffic. Sniffing is the process of eavesdropping on a connection without modifying the content. Meaning that you read all data the two (or more) parties exchange. This can be extremely useful for debugging communication issues. The most popular tool for sniffing is without a doubt, Wireshark (http://www.wireshark.org). But let's not get off track - I wont be talking about Wireshark right now.</span></span></p>
<p align="LEFT"><a href="{{ site.url }}/images/2013/11/wireshark.png"><img class="aligncenter" alt="wireshark" src="{{ site.url }}/images/2013/11/wireshark-300x182.png" width="300" height="182" /></a></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Finally, we are ready to start tampering with the network connection. So let's get on with using ipfw <i>(I'll try to be gentle)</i></span></span></p>
<p align="LEFT"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">The ipfw command line tool can be accessed from the terminal by typing... ipfw.</span></span></p>
<p align="LEFT"><span style="color: #000000;"><span style="font-family: Menlo-Regular;"><i><code>$ ipfw</code></i></span></span></p>
<span style="color: #000000;"><span style="font-family: Menlo-Regular;"><i><code>usage: ipfw [options]</code></i></span></span>

<span style="color: #000000;"><span style="font-family: Menlo-Regular;"><i><code>do "ipfw -h" or see ipfw manpage for details</code></i></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Not that helpful, is it? Well, you can always access the help page for it if you type “ipfw -h”. </span></span></span><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">If you tried typing this, you have noticed that it is also not very helpful – it's too confusing. It is just a reference of all options, not a tutorial.</span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">First of all, I would like to note that ipfw is a super user tool, meaning that it requires root privileges. So most of the time typing a command with it will result in a "<code>ipfw: socket: Operation not permitted</code>" error or the terminal will just say that it cannot find such command. That's because it's not looking in the sbin directory. So, to prevent that you should use sudo (“ipfw” turns into “sudo ipfw”).</span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Since I don't want you to suffer, I'm not going to go into details with the internals of the firewall (It also might have something to do with the fact that I don't know them either). I'm going to only show you how to do four things:</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Blocking all incoming and outgoing connections from and to a given address</span></span></span></li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Drop packets in order to simulate unstable network</span></span></span></li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Limit the bandwidth for a connection to simulate a slow network</span></span></span></li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Listing all active firewall rules</span></span></span></li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Last but certainly not least – remove all previously set rules.</span></span></span></li>
</ol>
<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><i><span style="text-decoration: underline;">Block all incoming and outgoing connections from and to a given address</span></i></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">In order to achieve that, we are going to add two rules in our firewall. One to drop all incoming traffic from a certain address or network and a second one to drop outgoing traffic with a destination in a certain network. You will be using the following command:</span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><code><i>ipfw add deny ip from 10.0.2.0/8 to 66.147.244.153/8</i></code></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><code><i>ipfw add deny ip from 66.147.244.153/8 to 10.0.2.0/8</i></code></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">So what does that do exactly? Let break down the two commands:</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">ipfw</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">We invoke the ipfw command. Remember to put “sudo” before it if needed</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">add</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Add specifies that we will be adding a new rule in our firewall</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">deny</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Deny will make the firewall drop all rackets that fit the description of the rule that follows</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">ip</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">The following parameters are IP addresses (not ports for example)</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">from</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Specifies the originator of the traffic</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><i>10.0.2.0/8</i></span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">The originating IP address</span></span></span></li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">The “/8” notation is a network submask. It corresponds to 255.0.0.0. Intuitively, it means that all IP address starting with “10.” will match this rule</span></span></span></li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Most often will you know exactly what IP you will want to block, and you can skip the submask.</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">To</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Specifies that the next parameter is the destination address for the rule</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">66.147.244.153/8</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">The destination address or network that will be blocked. In this case we will be blocking the traffic going to my blog. But be careful – specifying a network mask 8 means that this rule will not only block this address – it will also block every IP address that starts with 66.</span></span></span></li>
</ol>
</li>
</ol>
<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">The second command does the same thing, only in reverse. If we don't execute the second command, address 10.0.2.0 WILL NOT be able to send messages to my website, but my website WILL be able to send messages to 10.0.2.0. This kind of “one way” communication can certainly be useful, but it is important to note that we need two rules in order to stop all traffic between two hosts.</span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><i><span style="text-decoration: underline;">Drop packets in order to simulate unstable network</span></i></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">In some cases, your application might not work properly in un unstable network. It is unlikely to happen with protocols such as HTTP and TCP in general, but it can be useful to test this scenario. Especially with mobile applications, your users are bound to sometimes have bad connectivity. Using ipfw, you can easily configure our Mac to drop a certain portion of traffic. Let try this:</span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><code><i>ipfw add prob 0.5 deny ip from 192.168.1.0/24 to 10.0.0.0/8 in</i></code></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">This is a bit similar to the commands we discussed above, but with some exception:</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">prob</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">The prob options specifies that the following rule will match traffic (that also matches the following conditions) 50% of the time.</span></span></span></li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Intuitively, this means that 50% of the traffic for this rule will be handled (in our case dropped – as denoted by the deny option)</span></span></span></li>
</ol>
</li>
</ol>
<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">You should already be familiar with the rest of the command. It is the conditions that packets have to meet in order to match the rule. In this case, all traffic originating from the 192.168.1.0 network with a destination in the 10.0.0.0 network.</span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><i><span style="text-decoration: underline;">Limit the bandwidth for a certain connection to simulate a slow network</span></i></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">It is always useful to test your application in slow network condition. There's always something that can go wrong in such case. Here's how to simulate that:</span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><code><i># setup a pipe restricting all traffic to a speed specified by $ipAddress</i></code></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><code><i>ipfw pipe 1 config bw 50</i></code></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><code><i># add all traffic to the specified address through the pipe</i></code></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><code><i>ipfw add 11 pipe 1 tcp from any to 192.168.0.1</i></code></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><code><i>ipfw add 11 pipe 1 tcp from 192.168.0.1 to any</i></code></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Yikes! That was a little more complicated than expected. However, it gives me a chance to introduce a new concept – pipes. But what are those pipes?</span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">I'm glad you asked. Pipes, just like their real life counterparts, are objects that allow… other object to pass through them. In this case, these objects are network packets. We create a pipe with certain qualities and make parts of the traffic pass though it. In the previous example, the pipe's “special ability” is delaying the connection. You might have a Gigabit network, but ipfw will make sure information going through the pipe maintains the speed that you provided. With that said, let's dissect the command:</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">ipfw pipe 1 config bw 50</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">pipe 1</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Surprisingly enough, this tells ipfw to create a pipe</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">config</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Indicates that what follows are the configuration options for the pipe</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">bw</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">short for bandwidth</span></span></span></li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Specifies that this pipe will hava a limitation on it's bandwidth</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">50</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">the pipe's bandwidth</span></span></span></li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Written like this, the value is in Kbit/s</span></span></span></li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">You can also use values other than Kbit/s with the right option</span></span></span></li>
</ol>
</li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">ipfw add pipe 1 tcp from any to 192.168.0.1</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">pipe 1</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Instead of writing prob or deny like previously, we put our own, custom made pipe, where traffic matching the following condition, will go through</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">tcp</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">This time, instead of looking at the IP addresses alone, we put another restriction – the matching traffic has to be using the TCP transport protocol.</span></span></span></li>
</ol>
</li>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">any</span></span></span>
<ol>
	<li><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Matches all IP addresses, all transport protocols, all port etc.</span></span></span></li>
</ol>
</li>
</ol>
</li>
</ol>
<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><span style="text-decoration: underline;">Listing all active firewall rules</span></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Any time you need a little bit of debugging or just need to see what rules are there in the firewall, you can do that like this:</span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><code><i>ipfw show</i></code></span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">If you try this right now, you will probably get something like “</span></span></span><span style="color: #000000;"><span style="font-family: Menlo-Regular, serif;">65535 6167 4504002 allow ip from any to any</span></span><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">”. This is the default set of rules. They allow all traffic to pass undisturbed.</span></span></span>

<i><span style="text-decoration: underline;"><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Last but certainly not least – remove all previously set rules.</span></span></span></span></i>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">You will, no doubt, end up in a situation where you mess up your ipfw settings and need to revert them. Normally, a reboot will be sufficient – firewall rules are not persistent unless you specifically configure them that way. But don't worry, you don't need to restart – there is a command for that:</span></span></span>

<i><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;"><code>ipfw -f flush</code></span></span></span></i>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Here, “flush” tells ipfw to remove all rules. The “-f” option stands for “force”. All it does is suppress a confirmation message saying “</span></span></span><span style="color: #000000;"><span style="font-family: Menlo-Regular, sans-serif;">Are you sure? [yn]</span></span><span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">”.</span></span></span>

<span style="color: #000000;"><span style="font-family: Verdana, sans-serif;"><span style="font-size: medium;">Overall, ipfw is a great tool for debugging network code, especially if you are not afraid of the terminal (if you are, there are some GUI tools for it, but I haven't used them and cannot verify that they work properly).</span></span></span>