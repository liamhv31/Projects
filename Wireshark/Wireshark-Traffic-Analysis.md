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

### Question 5 - What is the hostname of the available host in the Kerberos packets?

### Question 15 - Investigate the anomalous packets. Which protocol is used in ICMP tunnelling?

### Question 16 - Investigate the anomalous packets. What is the suspicious main domain address that receives anomalous DNS queries? (Enter the address in defanged format.)

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

