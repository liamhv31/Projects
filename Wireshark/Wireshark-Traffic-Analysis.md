## Traffic Analysis

## Part 1 - Nmap Scans

### Question 1 - What is the total number of the "TCP Connect" scans?
First, let's understand a TCP connect scan. This type of scan uses the TCP three-way handshake against target ports to see if they're open. For each port, the scanner starts by sending a SYN (synchronize) packet. If the port is open, it will respond with a SYN-ACK (synchronize-acknowledge) packet. The scanner then returns an ACK (acknowledge) packet which establishes a connection. Immediately after the handshake completes, the scanner typically sends a TCP RST packet to terminate the connection. There are a few different ways we can identify TCP connect scans in Wireshark.

One way is by using the **Statistics** &rarr; **Conversations** &rarr; **TCP** menu. If you sort by the **Address A** column, you may see a single client sending TCP packets to the same host over hundreds or even thousands of different ports like we see here:

<img width="1028" height="528" alt="image" src="https://github.com/user-attachments/assets/e6179bdf-a4e0-4c28-9bb3-a6221cbfa361" />

You can also check the **Analyze** &rarr; **Expert Information** tab. In the screenshot below, we can see that there is an excessive number of "Warning - Connection reset (RST)" events. There are 2,000 of them. In a capture file of about 6500 packets, that's a big clue that points towards scanning activity.

<img width="804" height="111" alt="image" src="https://github.com/user-attachments/assets/b8221150-c0f2-4691-979b-bbcd36244631" />

You can also try and narrow it down with a filter. It can be difficult to identify scanning activity like this sometimes since you will also return legitimate TCP connection events. The first two ways are typically better. Assuming much of this file is scanning activity, we can narrow it down like so: `tcp.flags.syn==1 and tcp.flags.ack==0`. This only shows packets where the SYN flag is set (starting a TCP connection) and excluding SYN-ACK packets, essentially only showing client-to-server TCP SYN events. Again, this type of activity will also show for legitimate TCP connections, so the results will likely be unreliable (even though the lab provides this as an example). The lab's suggested filter also includes `tcp.window_size > 1024`. I see what they're trying to do here, but I believe this is a mistake, let me explain why.

First, what even is the TCP window size for? The TCP receive window advertises how much additional data (in bytes) the receiver is currently willing to accept without further acknowledgment. The TCP window size can be used as a **fingerprinting** characteristic. This is a technique used to identify an entity by looking at one or more unique characteristics associated with that entity. In this case, that entity is **nmap**. The lab uses `nmap -sT` as the example for TCP connect scans. The pcap for this lab is also located in the "nmap" directory. The biggest clue though is in how nmap handles setting the TCP window size. In [this](https://github.com/nmap/nmap/issues/2491) now-closed issue in the nmap GitHub repo, the user points out how nmap sets the TCP window size.

(In [nmap/tcpip.cc, line 602](https://github.com/nmap/nmap/blob/806c0af5ee008ace06dbf1623765ff97edb89e33/tcpip.cc#L602) at the time of this writeup):
```
  if (window)
    tcp->th_win = htons(window);
  else
    tcp->th_win = htons(1024); /* Who cares */
```

Essentially, this code is saying: If no TCP window size is explicitly specified, Nmap sets it to 1024. The "Who cares" comment kind of tells you what the developer was thinking, the window size doesn't really matter. Since nmap isn't expecting to receive any application data (only attempting to see if the port is open), the window size doesn't matter. However, this does somewhat matter though because now we have a unique characteristic that can be used to detect potential nmap traffic (if the person initiating the scanning is lazy). The final query will look like this:
```
ip.src == 10.10.60.7 and tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size==1024
```
You can choose to include or not include the scanner IP, the key here is the default TCP window size of 1024 used by Nmap when no custom window size is specified.

**Answer**: 1000

### Question 2 - Which scan type is used to scan the TCP port 80?
To identify the type of scan performed against a specific port, we need to analyze the packet sequence for the traffic targeting that port. Since we know the scanner IP, we can do this with the following filter: `ip.addr==10.10.60.6 and tcp.port==80`. We can see what appears to be two different conversation by looking at the packet numbers.

<img width="1469" height="200" alt="image" src="https://github.com/user-attachments/assets/f3252e46-7ee2-4ee1-8447-913033620ec5" />

The first four packets show the following sequence:
```
SYN
SYN, ACK
ACK
RST, ACK
```
This is indicative of a TCP connect scan, since that type of scan is performed by completing the full TCP three-way handshake. The last three packets show:
```
SYN
SYN, ACK
RST, ACK
```
This sequence is indicative of a SYN scan since the scanner immediately terminates the connection with a `RST, ACK` after receiving an `ACK`. So this packet capture actually shows two scan types from what we can see. This question expects one answer, yet there are two. Based on the accepted answer string length, and the fact that the previous question was about TCP connect scans, we can correctly assume the answer.

**Answer**: TCP Connect

### Question 3 - How many "UDP close port" messages are there?
UDP port connections don't require a three-way handshake for connection, nor is there a prompt for open ports. Closed UDP ports will _almost_ always return an **ICMP Destination Unreachable** and/or **Port Unreachable** message. I say "_almost_ always" because it depends on the device and network configuration of the receiving device:
- **Firewalls**: Some firewalls are configured to silently **DROP** ICMP packets, instead of rejecting them (which prompts a response message). This will show an ICMP timeout for the sender.
- **ICMP rate limiting**: Many devices introduce rate limiting for ICMP messages to prevent DDoS attacks, like those caused by **ICMP (Ping) Flood** attacks
- **Network blocking/filtering**: Some people, like ISPs, may block ICMP packets from traversing the network to prevent network mapping or reduce overhead. E.g., Apple has been known to send packets with a very large Maximum Transmission Unit (MTU), which helps find the largest allowed packet size to reduce fragmentation, and thus connectivity latency. This is a double edged sword though, as the routers reciving these packets are possibly receving hundreds of millions of packets per second if it's a large scale telecommunications network. This means that to prevent network performance issues, some routers may have to drop these packets (even if they are legitimate), or there blocked via the firewall from reaching the network entirely.

I digress. To find closed UDP ports, we can use the following filter:
```
icmp.type==3 and icmp.code==3
```
- `icmp.type==3` means destination unreachable
- `icmp.code==3` means port unreachable (closed UDP port)

This will return your answer, which is the number of packets on the bottom of the page.

<img width="1275" height="1204" alt="image" src="https://github.com/user-attachments/assets/07f11f06-213b-4c5a-afe5-d851b51d7c74" />

**Answer**: 1083

### Question 4 - Which UDP port in the 55-70 port range is open?
This question is easy to answer if we understand how UDP works. UDP is **connectionless**, meaning it doesn't wait to establish a connection before you can send data. It is a "fire and forget" protocol. You do not get notified that packets have been receieved and lost ones are not resent. What that means is that we are looking for any UDP packet that did _not_ return **ICMP - destination unreachable (port unreachable)**. We can use the greater than `>` and less than `<` operators to return only packets where the destination UPD port is equal to or between ports 55 and 70.
```
udp.dstport>=55 and udp.dstport<=70
```

<img width="1118" height="284" alt="image" src="https://github.com/user-attachments/assets/69d71583-a14f-4fdf-9673-ed376340766c" />

In the image above, we can see that there are three ports in the target range for UDP traffic. We can see that ports 67 and 69 returned an **ICMP - destination unreachable (port unreachable)** response. Port 68 did not.

**Answer**: 68

## Part 2 - ARP Poisoning/Spoofing (A.K.A. Man-in-the-Middle Attack)

### Question 1 - What is the number of ARP requests crafted by the attacker?
First, a quick and high-level description on ARP. The Address Resolution Protocol is used to map an IP address to a physical MAC address within a local network. Essentially, the device sending the data sends out an **ARP request** asking "Who has this IP address? Tell IP X". The receiver then returns an **ARP reply** saying "IP X is at MAC address Y". That answer is then temporarily stored in the device's local ARP cache/table so it doesn't have to keep asking in the future.

Before we do anything, we need to identify the threat actor. It starts to get suspicious at packet 287. One way we can do this is by looking for potential ARP spoofing attempts. This is when two MAC addresses claim the same IP address. This is suspicious because normally an IPv4 address should map to exactly one MAC address at any time. Since we just want to look at the MAC addresses claiming the IPv4 addresses, then we need to only return ARP replys. We can do this with the following query: `arp.opcode==2`.

<img width="1091" height="634" alt="image" src="https://github.com/user-attachments/assets/a5b4c9e6-f69e-4bf3-93e1-ab98c876a2fc" />

The first few packets look normal. Devices are claiming ownership of specific IPv4 addresses. It only starts to get suspicious at packet `287` when we see that `VMware_e2:18:b4 (00:0c:29:e2:18:b4)` claims that the IP `192.168.1.1` belongs to it. That IP was already claimed by `zte_f3:cd:f4 (50:78:b3:f3:cd:f4)` in packet `4`. We then see the same device claim another previously claimed IPv4 address (`192.168.1.12`). We can see this pattern throughout, which is a strong indication that the threat actor is `VMware_e2:18:b4`. Now that we've identified the attacker, we need to know how many ARP requests they've made. We can use the following filter:
```
eth.src==00:0c:29:e2:18:b4 and arp.opcode==1
```
When the opcode is set to `1`, this only shows us ARP reqeusts. This will give us the answer.

<img width="957" height="842" alt="image" src="https://github.com/user-attachments/assets/149b9456-30b3-45c2-8a2f-c4163f798a98" />

**Answer**: 284

### Question 2 - What is the number of HTTP packets received by the attacker?
Now that we know the attacker MAC address, we can answer this with ease.
```
eth.dst==00:0c:29:e2:18:b4 and http
```
Since we need to know how many HTTP packets the attacker recieved, we set the ethernet destination address to the attacker's MAC address, and filter on HTTP packets only.

<img width="951" height="848" alt="image" src="https://github.com/user-attachments/assets/5e61f777-2abe-452d-9f29-220093c4626f" />

**Answer**: 90

### Question 3 - What is the number of sniffed username&password entries?
After the attacker performs the man-in-the-middle attack, we can see the attacker machine is sniffing HTTP traffic between the user and the web server (seen in the previous screenshot). Before we get into this question, I'll quickly explain how this worked and allowed the threat actor to sniff traffic. In this scenario, the victim wants to visit **testphp[.]vulnweb[.]com**. This means that it needs to send ethernet frames to the routers MAC address. However, the router may not know which IPv4 address belongs to which MAC address, so it sends out an ARP request. Other routers and devices start sending back ARP replies, claiming ownership of the various IPv4 addresses. This is when the attacker asserts themselves into the network. They begin sending out forged ARP replys that were crafted to trick the router into believing it owns those IPs (see examples in the previous question). ARP has no authentication, so it just simply trusts the ARP replies. When it sees the attackers ARP reply, the router just updates its ARP cache that essentially says if the packets are destined to IPv4 X, send the frames to `00:0c:29:e2:18:b4` (the attackers MAC).

Let's use the same query as before but just slightly modify it to look at HTTP POST requests.
```
eth.dst==00:0c:29:e2:18:b4 and http.request.method=="POST"
```
This only returns a handful of packets. Let's look through each of them.

<img width="984" height="179" alt="image" src="https://github.com/user-attachments/assets/a726e7c1-92f1-4150-87fa-bfb249d01a18" />

We can see that packets 1226, 1446, 1561, 1599, 1668, and 1791 all appear to be submissions to a web form by looking at the info column which contains **POST**, **userinfo.php**, and **application/x-www-form-urlencoded**. This gives us six unique username and password captures. This is our answer.

<img width="1954" height="1131" alt="image" src="https://github.com/user-attachments/assets/122339bc-223c-425a-a617-f1fafef8c168" />

**Answer**: 6

### Question 4 - What is the password of the "Client986"?
Since there are so few events, you could just look through each of the packets to try and find the login event from client986. However, if this is a much larger packet capture with hundreds of login events, it would be easier to apply a filter. Since we now know the format of these login event packets, we can select any one of them, expand the **HTML Form URL Encoded** tree, then the form item for the username. Right-click on the value (username), and select **Apply as Filter** &rarr; **Selected**.

<img width="988" height="1024" alt="image" src="https://github.com/user-attachments/assets/82704622-c863-4de3-a75c-fc6a51c5b27f" />

Then just replace the username with the one we are looking for. This will show just one event.

<img width="1949" height="1210" alt="image" src="https://github.com/user-attachments/assets/3c050547-509b-449f-bd53-b209135b1c74" />

You could also just do a string search across all packets with **Edit** &rarr; **Find packet...**.

<img width="2545" height="1206" alt="image" src="https://github.com/user-attachments/assets/29f59181-6247-4673-b6f9-d62c3b246b18" />

**Answer**: clientnothere!

### Question 5 - What is the comment provided by the "Client354"?
We can just modify the previous username search to find the comment:
```
urlencoded-form.value=="client354"
```
Expand the **HTML Form URL Encoded** tree to find the comment.

<img width="304" height="343" alt="image" src="https://github.com/user-attachments/assets/166ba291-d1a1-4d84-bbd3-4e3e807d6925" />

**Answer**: Nice work!

## Part 3 - Identifying Hosts: DHCP, NetBIOS, and Kerberos

### Question 1 - What is the MAC address of the host "Galaxy A30"?
For these first three questions, we're going to be looking at DHCP and NetBIOS traffic to find our answers. For this question, we're going to use DHCP to identify the MAC address. DHCP (Dynamic Host Configuration Protocol) allows IP addresses to be automatically assigned to devices on a network. It also assigns other important information like subnet masks, default gateways, and DNS servers. Without this, you would have to manually configure these settings for every network device.

We can find lots of information about the device and request by looking at **DHCP Request** packets. This will contain important details like the hostname, requested IP address, IP address lease time, and the client's MAC address. A **DHCP Request** is the third step in the DHCP **DORA (Discover, Offer, Request, Acknowledge)** process where the client sends a broadcast message acknowleding the offered DHCP configuration and requesting to officially use it.

Our query will be two parts:
```
dhcp.option.dhcp==3 and dhcp.option.hostname contains "Galaxy A30"
```
- `dhcp.option.dhcp==3` filters by DHCP Requests
- `dhcp.option.hostname contains "Galaxy A30"` performs a keyword search for our hostname string in the DHCP options hostname (**Option 12**) field

If we first try searching by `Galaxy A30`, nothing will be returned. The format of the string is slightly different. If we search by just `Galaxy` instead, we get two results. If we search for both `Galaxy` _and_ `A30` as seperate strings, we will return the correct packet. There are multiple ways of writing this query, but this is what I chose:
```
dhcp.option.dhcp==3 and (dhcp.option.hostname contains "Galaxy" and dhcp.option.hostname contains "A30")
```
Expand **Option: (12) Host Name** and **Option: (61) Client identifier** under the **Dynamic Host Configuration Protocol (Request)** tree to see the full hostname and client MAC address respectively.

<img width="958" height="424" alt="image" src="https://github.com/user-attachments/assets/c135d8a5-4626-4f31-a972-d3cf9e7fb6ae" />

**Answer**: 9a:81:41:cb:96:6c

### Question 2 - How many NetBIOS registration requests does the "LIVALJM" workstation have?
NetBIOS stands for **Network Basic Input/Output System**. It is a legacy programming interface that allows applications on different computers to communicate over a local network. NetBIOs has largely been replaced by more modern solutions like DNS, and mDNS (Multicast DNS) and LLMNR for local network discovery.

In Wireshark, we can find NetBIOS packets with the `nbns` filter. This returns to many packets to sift through. If we click on one of the packets though and expand the **NetBIOS Name Service** tree &rarr; **Queries**, you will see a workstation name branch that can be expanded. Open that, and then we can right-click on the name field and apply it as a filter.

<img width="957" height="848" alt="image" src="https://github.com/user-attachments/assets/671341ce-c33b-4ce1-8df1-65dd3b663898" />

Now we just need to adjust the query to find the value and packet types we're looking for.
```
nbns.name contains "LIVALJM" and nbns.flags.opcode == 5 and nbns.flags.response == 0
```
- `nbns.flags.opcode == 5` filters for NetBIOS registration events
- `nbns.flags.response == 0` filters for for requests

**Answer**: 16

### Question 3 - Which host requested the IP address "172.16.13.85"?
Since this question is asking about a requested IP address, we need to look at DHCP packets since this protocol is what automatically assigns IP addresses to requesting devices. This question can be answered with a simple filter.
```
dhcp.option.requested_ip_address==172.16.13.85
```
This returns just one packet, and you can find the hostname under **Dynamic Host Configuration Protocol (Request)** &rarr; **Option: (12) Host Name** &rarr; **Host Name**

<img width="954" height="851" alt="image" src="https://github.com/user-attachments/assets/3669718e-75e2-49c3-8b59-b88dc881eec1" />

**Answer**: Galaxy-A12

### Question 4 - What is the IP address of the user "u5"? (Enter the address in defanged format.)
For these next two questions, we will switch to Kerberos traffic. First, a quick explanation on this protocol. Kerberos allows authentication on a network. It's a secure protocol that allows users and services to prove their identity using encrypted "tickets" instead of sending passwords over the network. With this in mind, to answer this question, we are essentially looking for kerberos authentication packets for user "u5". We can start with the folllowing filter.
```
kerberos.CNameString=="u5"
```
`CNameString` stands for "Client Name String". This will return a few different types of kerberos packets (AS-REP, AS-REQ, and TGS-REP). AS-REQ and AS-REP are used for the initial login to get a master ticket. TGS-REP is the reply that grants the requester access to a specific service. The AS-REQ packets are where the user or service first starts the kerberos authentication by making a request to the **Key Distribution Center (KDC)** for a master ticket. This means that the packets source IP belongs to whoever is making that request (in this case, "u5"), and the destination is the **KDC**/**domain controller** (The domain controller being the server, and the KDC being the authentication service running on it). Make sure the IP is defanged.

<img width="464" height="160" alt="image" src="https://github.com/user-attachments/assets/2b31a5de-f246-4a15-93b5-2f953bb94499" />

**Answer**: 10[.]1[.]12[.]2

### Question 5 - What is the hostname of the available host in the Kerberos packets?
This is honestly a pretty poorly worded question and threw me off for a minute. As far as I know, there is no concept in Kerberos for "available host". What the author almost certainly meant (and this is confirmed by the answer), is "what is the client host/computer name requesting the Kerberos ticket?". Hostnames in Kerberos can be identified by looking for a `$` appended to the `CNameString` in Kerberos packets. Essentially, `CNameString`'s with a `$` are hostnames, ones without are usernames. So to find our hostname, we just need to search for `CNameString`'s that contain a `$`.
```
kerberos.CNameString contains "$"
```
This returns one packet.

<img width="476" height="389" alt="image" src="https://github.com/user-attachments/assets/cccd1dd4-8309-4d1c-908b-743db891522b" />

**Answer**: xp1$

## Part 4 - Tunneling Traffic: DNS and ICMP

### Question 1 - Investigate the anomalous packets. Which protocol is used in ICMP tunnelling?
Wireshark is not an NIDS or NIPS, so identifying tunneling activity needs to be done entirely manually. ICMP can be a popular choice for data tunneling since it's generally trusted by firewalls for things like network diagnostics. There are a few clues we can start with to help find the packets of interest:
- Large ICMP echo requests/replies
- Continuous streams of echo requests/replies
- Unusual data within ICMP payloads
- Traffic that is abnormal to send between specific hosts (better if you know the network)

We can start by simply just returning ICMP packets with the `icmp` filter. This doesn't remove much, but it's about the process of elimination. Sorting the packets by frame length shows many packets over 1000 bytes long.

<img width="843" height="469" alt="image" src="https://github.com/user-attachments/assets/481399a2-49ce-48ab-ac08-bca5d810948b" />

Now it's a matter of going through the ICMP packet data to see if we spot anything out of the ordinary. It's also important to keep an eye out for breaks in the pattern. This could take a while. As I was scrolling through, I intuitevly start to recognize a pattern emerge. Humans are exceptionally good when it comes to pattern recognition. This skill was hardwired in our brains as it was necesssary a long, long time ago for our survival. So, if you find a a pattern, and then spot a deviation in said pattern, trust your instincts.

<img width="640" height="369" alt="image" src="https://github.com/user-attachments/assets/4ab5e0a3-7d1d-485d-9d1d-b31bfa079ccb" />

Do you see it too? Request, reply, request, reply... This continues for a few dozen packets, and then I stumbled upon this.

<img width="636" height="374" alt="image" src="https://github.com/user-attachments/assets/a9bd7ce0-7d6d-400b-858c-a63ab0b8bd74" />

This is where I start to really look closely at what's in the ICMP packet data. It looks like our intuition paid off. Packet 46, 48, and 52 start to show some intersting stuff.

<img width="674" height="346" alt="image" src="https://github.com/user-attachments/assets/303e1c00-1018-4177-b735-f1d79ede4b26" />

We can see several strings related to encryption algorithms. Let's take a depper look at this packet to see what we can find.
```
0000   45 00 03 4c 39 6c 40 00 40 06 e7 7f 0a 5f 01 01   E..L9l@.@...._..
0010   0a 5f 01 02 c8 8b 00 16 31 7d 53 ca 0e 1b bf cb   ._......1}S.....
0020   80 18 01 c9 ac 10 00 00 01 01 08 0a 00 6d 6a eb   .............mj.
0030   00 6d 86 43 00 00 03 14 08 14 1e e1 33 4c 41 dc   .m.C........3LA.
0040   10 c2 8f cb c2 72 49 d7 72 3c 00 00 00 7e 64 69   .....rI.r<...~di
0050   66 66 69 65 2d 68 65 6c 6c 6d 61 6e 2d 67 72 6f   ffie-hellman-gro
0060   75 70 2d 65 78 63 68 61 6e 67 65 2d 73 68 61 32   up-exchange-sha2
0070   35 36 2c 64 69 66 66 69 65 2d 68 65 6c 6c 6d 61   56,diffie-hellma
0080   6e 2d 67 72 6f 75 70 2d 65 78 63 68 61 6e 67 65   n-group-exchange
0090   2d 73 68 61 31 2c 64 69 66 66 69 65 2d 68 65 6c   -sha1,diffie-hel
00a0   6c 6d 61 6e 2d 67 72 6f 75 70 31 34 2d 73 68 61   lman-group14-sha
00b0   31 2c 64 69 66 66 69 65 2d 68 65 6c 6c 6d 61 6e   1,diffie-hellman
00c0   2d 67 72 6f 75 70 31 2d 73 68 61 31 00 00 00 0f   -group1-sha1....
00d0   73 73 68 2d 72 73 61 2c 73 73 68 2d 64 73 73 00   ssh-rsa,ssh-dss.
00e0   00 00 9d 61 65 73 31 32 38 2d 63 74 72 2c 61 65   ...aes128-ctr,ae
00f0   73 31 39 32 2d 63 74 72 2c 61 65 73 32 35 36 2d   s192-ctr,aes256-
0100   63 74 72 2c 61 72 63 66 6f 75 72 32 35 36 2c 61   ctr,arcfour256,a
0110   72 63 66 6f 75 72 31 32 38 2c 61 65 73 31 32 38   rcfour128,aes128
0120   2d 63 62 63 2c 33 64 65 73 2d 63 62 63 2c 62 6c   -cbc,3des-cbc,bl
0130   6f 77 66 69 73 68 2d 63 62 63 2c 63 61 73 74 31   owfish-cbc,cast1
0140   32 38 2d 63 62 63 2c 61 65 73 31 39 32 2d 63 62   28-cbc,aes192-cb
0150   63 2c 61 65 73 32 35 36 2d 63 62 63 2c 61 72 63   c,aes256-cbc,arc
0160   66 6f 75 72 2c 72 69 6a 6e 64 61 65 6c 2d 63 62   four,rijndael-cb
0170   63 40 6c 79 73 61 74 6f 72 2e 6c 69 75 2e 73 65   c@lysator.liu.se
0180   00 00 00 9d 61 65 73 31 32 38 2d 63 74 72 2c 61   ....aes128-ctr,a
0190   65 73 31 39 32 2d 63 74 72 2c 61 65 73 32 35 36   es192-ctr,aes256
01a0   2d 63 74 72 2c 61 72 63 66 6f 75 72 32 35 36 2c   -ctr,arcfour256,
01b0   61 72 63 66 6f 75 72 31 32 38 2c 61 65 73 31 32   arcfour128,aes12
01c0   38 2d 63 62 63 2c 33 64 65 73 2d 63 62 63 2c 62   8-cbc,3des-cbc,b
01d0   6c 6f 77 66 69 73 68 2d 63 62 63 2c 63 61 73 74   lowfish-cbc,cast
01e0   31 32 38 2d 63 62 63 2c 61 65 73 31 39 32 2d 63   128-cbc,aes192-c
01f0   62 63 2c 61 65 73 32 35 36 2d 63 62 63 2c 61 72   bc,aes256-cbc,ar
0200   63 66 6f 75 72 2c 72 69 6a 6e 64 61 65 6c 2d 63   cfour,rijndael-c
0210   62 63 40 6c 79 73 61 74 6f 72 2e 6c 69 75 2e 73   bc@lysator.liu.s
0220   65 00 00 00 69 68 6d 61 63 2d 6d 64 35 2c 68 6d   e...ihmac-md5,hm
0230   61 63 2d 73 68 61 31 2c 75 6d 61 63 2d 36 34 40   ac-sha1,umac-64@
0240   6f 70 65 6e 73 73 68 2e 63 6f 6d 2c 68 6d 61 63   openssh.com,hmac
0250   2d 72 69 70 65 6d 64 31 36 30 2c 68 6d 61 63 2d   -ripemd160,hmac-
0260   72 69 70 65 6d 64 31 36 30 40 6f 70 65 6e 73 73   ripemd160@openss
0270   68 2e 63 6f 6d 2c 68 6d 61 63 2d 73 68 61 31 2d   h.com,hmac-sha1-
0280   39 36 2c 68 6d 61 63 2d 6d 64 35 2d 39 36 00 00   96,hmac-md5-96..
0290   00 69 68 6d 61 63 2d 6d 64 35 2c 68 6d 61 63 2d   .ihmac-md5,hmac-
02a0   73 68 61 31 2c 75 6d 61 63 2d 36 34 40 6f 70 65   sha1,umac-64@ope
02b0   6e 73 73 68 2e 63 6f 6d 2c 68 6d 61 63 2d 72 69   nssh.com,hmac-ri
02c0   70 65 6d 64 31 36 30 2c 68 6d 61 63 2d 72 69 70   pemd160,hmac-rip
02d0   65 6d 64 31 36 30 40 6f 70 65 6e 73 73 68 2e 63   emd160@openssh.c
02e0   6f 6d 2c 68 6d 61 63 2d 73 68 61 31 2d 39 36 2c   om,hmac-sha1-96,
02f0   68 6d 61 63 2d 6d 64 35 2d 39 36 00 00 00 1a 6e   hmac-md5-96....n
0300   6f 6e 65 2c 7a 6c 69 62 40 6f 70 65 6e 73 73 68   one,zlib@openssh
0310   2e 63 6f 6d 2c 7a 6c 69 62 00 00 00 1a 6e 6f 6e   .com,zlib....non
0320   65 2c 7a 6c 69 62 40 6f 70 65 6e 73 73 68 2e 63   e,zlib@openssh.c
0330   6f 6d 2c 7a 6c 69 62 00 00 00 00 00 00 00 00 00   om,zlib.........
0340   00 00 00 00 00 00 00 00 00 00 00 00               ............
```
Above is the hex and ASCII dump of the data payload. While the first clue was the cryptographic data payload, another key finding is that the data payload starts with an IP header. Let me explain. The data hex begins with the values `45 00 03 4c ...`. Keep in mind that every IPv4 header is structured like so:
- **Byte 1**: Version/IHL
- **Byte 2**: DSCP/ECN
- **Byte 3-4**: Total length

<img width="1273" height="576" alt="image" src="https://github.com/user-attachments/assets/db65354e-e1df-4b06-ab16-307d41e4e2cc" />

With that said, let's convert each of these values. Since `45` is a hex value, we evaluate each number on it's own (`4` and `5`). We don't really need to do any math since the hex values `4` and `5` equal the decimal values `4` and `5` respectively.

<img width="691" height="583" alt="image" src="https://github.com/user-attachments/assets/9fe2efe9-3558-4db6-b888-6d7f970f6b31" />

For the **Version** field of an IP header, the only possible values are `4` and `6`. This is `4`, meaning it's an IPv4 packet. The **IHL (Internet Header Length)** is `5`. This will tell us the header size of this packet, which is done by multiplying the **IHL** by `4`, so `5 x 4 = 20 bytes`. This is a standard IPv4 header size.

So why is it 20 _bytes_ and why multiply by `4`? The IHL field contains the number of 32-bit words in the header. `32 bits / 8 = 4 bytes` (because 8 bits is 1 byte). So one word is 4 bytes. So if the IHL is `5`. that means that `5 words x 4 bytes/word = 20 bytes`.

Next is `00`. Before we see how this tells us it's part of an IP header, let's first understand, just a bit, what the second byte of an IPv4 header is. This represents the **Type of Service (TOS)**. This is how it's labeled in the screenshot above. Directly from **RFC 791** from 1981 - "the TOS is for internet service quality selection". It tells routers and other intermediary devices how to handle packets (prioritization, delay, throughput, and reliability). **RFC 2474** was published in 1998 which divided the TOS into two parts - the **Differentiated Services Codepoint (DSCP)** and **Currently Unused (CU)**. The first six bits belonged to the DSCP, and the last two belonged to the CU. Together, they redefine the TOS byte as the **Differentiated Services (DS) Field**. DSCP is the modern standard for prioritizing data across Layer 3 IP networks. CU, as the name suggests, were unused and therefore not really defined. DS-compliant interfaces would simply ignore them when determining how to forward a packet. Finally, **RFC 3168** is published in 2001. This repurposes the two CU bits for **Explicit Congestion Notification (ECN)**. This is an an optional extension that allows network routers and end-nodes to detect and signal network congestion without dropping packets. This is the current format still used today. To recap, the DSCP makes up the first 6 bits, and the ECN makes up the last two.

So, technically the above image is a _bit_ outdated, but it serves the purpose of demonstrating the IPv4 header byte breakdown.

DSCP has 64 possible values that can be set for it's bits. The DSCP bits in this packet are `000000`, which is just `0` in decimal. That value means **Default (best-effort)**. The ECN field has four possible values. Again, in this case it's just `0`, which means **Not-ECT**, or "The packet sender or receiver does not support ECN". We can actually confirm this in the packet itself. The final two bytes are `03 4c`. We can treat this as a single number because the last part of the header is defined as one, 16-bit value. First we need to convert each to decimal. `03` is simply `3`, and `4c` can be treated as such:
- `4` = `4`
- `c` = `12`
Now to actually convert it to decimal: `4c = (4 x 16) + 12 = 64 + 12 = 76`. So now `03 = 3` and `4c = 76`. Let me explain the conversion math for `4c` first. Hexadecimal is base 16, which is why we do `(4 x 16)`. So why isn't the entire hex value calculated by 16? Well look at a decimal number like 45 as an example. We don't calculate it by doing `4 x 10` and `5 x 10`. Instead, we do: `(4 x 10) + 5` because the `4` is in the _tens_ place, while the `5` is in the _ones_ place. Hex works the same way. `4` is in the _16s_ place, while `c` is in the _ones_ place. This is why it's `(4 x 16) + 12`. Next we need to understand why we need to multiply by `256`. Each byte (e.g., `03`) contains `8` bits. That means an 8-bit number can represent 256 different values (`2^8 = 256`). So when you have two bytes, the left bytes represents how many groups of `256` you have, just like how the left most two digit decimal number represents the number of _tens_ it has. With that being said, the math after converting each byte to decimal would be: `3 x 256 + 76 = 844`. Another way to think about it is that the left byte tells us how many full blocks of 256 there are, while the right byte is the remainder. So the nested IPv4 packet is 844 bytes.

Let's tie this all together now. The **total** size of this packet is 886 bytes. Assuming a standard Ethernet II header, that would be 14 bytes. This leaves 872 bytes remaining. The outer IPv4 packet is also a standard 20 byte header, which brings us to 852 bytes. The last 8 bytes outside of the nested IPv4 packet belong to the ICMP header. An ICMP Echo Request (Type 8) always has an 8-byte header before the data begins. So, that's `886 total packet bytes - 14 byte Ethernet II header - 20 byte outer IPv4 header - 8 byte ICMP Echo Request header = 844 bytes for the encapsulated IPv4 packet`!

<img width="955" height="845" alt="image" src="https://github.com/user-attachments/assets/a2215289-d825-4c57-b654-b17c475ed932" />

That was a lot to explain just to demonstrate that this was likely an ICMP tunneling event, but now for the nail in the coffin. The payload data contains unmistakable SSH key-exchange data:
```
diffie-hellman-group-exchange-sha256
ssh-rsa
ssh-dss
aes128-ctr
hmac-sha1
zlib@openssh.com
```
With all of this evidence, we can be absolutely certain about the answer.

**Answer**: SSH

<img width="726" height="285" alt="image" src="https://github.com/user-attachments/assets/711dd5eb-5086-48fc-b80e-c81f81257512" />

With just this information alone, it is an abundance of evidence that points towards this being an IPv4 packet encapsualted in ICMP

### Question 2 - Investigate the anomalous packets. What is the suspicious main domain address that receives anomalous DNS queries? (Enter the address in defanged format.)

### Question 17 - How many incorrect login attempts are there?

### Question 18 - What is the size of the file accessed by the "ftp" account?

### Question 19 - The adversary uploaded a document to the FTP server. What is the filename?

### Question 20 - The adversary tried to assign special flags to change the executing permissions of the uploaded file. What is the command used by the adversary?

### Question 21 - Investigate the user agents. What is the number of anomalous  "user-agent" types?

### Question 22 - What is the packet number with a subtle spelling difference in the user agent field?

### Question 23 - Locate the "Log4j" attack starting phase. What is the packet number?

### Question 24 - Locate the "Log4j" attack starting phase and decode the base64 command. What is the IP address contacted by the adversary? (Enter the address in defanged format and exclude "{}".)

### Question 25 - What is the frame number of the "Client Hello" message sent to "accounts.google.com"?

### Question 26 - Decrypt the traffic with the "KeysLogFile.txt" file. What is the number of HTTP2 packets?

### Question 27 - Go to Frame 322. What is the authority header of the HTTP2 packet? (Enter the address in defanged format.)

### Question 28 - Investigate the decrypted packets and find the flag! What is the flag?

### Question 29 - What is the packet number of the credentials using "HTTP Basic Auth"?

### Question 30 - What is the packet number where "empty password" was submitted?

### Question 31 - Select packet number 99. Create a rule for "IPFirewall (ipfw)". What is the rule for "denying source IPv4 address"?

### Question 32 - Select packet number 231. Create "IPFirewall" rules. What is the rule for "allowing destination MAC address"?

