## Traffic Analysis

### Question 1 - What is the total number of the "TCP Connect" scans?
First, let's understand a TCP connect scan. This type of scan uses the TCP three-way handshake against target ports to see if they're open. For each port, the scanner starts by sending a SYN (synchronize) packet. If the port is open, it will respond with a SYN-ACK (synchronize-acknowledge) packet. The scanner then returns an ACK (acknowledge) packet which establishes a connection. Immediately after the handshake is complete, the scanner sends a RST-ACK packet to close the connection. There are a few different ways we can identify TCP connect scans in Wireshark.

One way is by using the **Statistics** &rarr; **Conversations** &rarr; **TCP** menu. If you sort by the **Address A** column, you may see a single client sending TCP packets to the same host over hundreds or even thousands of different ports like we see here:

<img width="1028" height="528" alt="image" src="https://github.com/user-attachments/assets/e6179bdf-a4e0-4c28-9bb3-a6221cbfa361" />

You can also check the **Analyze** &rarr; **Expert Information** tab. In the screenshot below, we can see that there is an excessive number of "Warning - Connection reset (RST)" events. 2000 of them. In a capture file of about 6500 packets, that's a big clue that points towards scanning activity.

<img width="804" height="111" alt="image" src="https://github.com/user-attachments/assets/b8221150-c0f2-4691-979b-bbcd36244631" />

You can also try and narrow it down with a filter. It can be difficult to identify scanning activity like this sometimes since you will also return legitimate TCP connection events. The first two ways are typically better. Assuming much of this file is scanning activity, we can narrow it down like so: `tcp.flags.syn==1 and tcp.flags.ack==0`. This only shows packets where the SYN flag is set (starting a TCP connection) and excluding SYN-ACK packets, essentially only showing client to server TCP SYN events. Again, this type of activity will also show for legit TCP connections, so the results will likely be unreliable (even though the lab provides this as an example). The lab query also includes `tcp.window_size > 1024`. I see what they're trying to do here, but I believe this is a mistake, let me explain why.

First, what even is the TCP window size for? This is a field used by the recevier to tell the sender how much unacknowledged data (in bytes) to transmit before expecting an ACK. Specifying the window size is a **fingerprinting** technique. This is a technique used to identify an entity by looking at one or more unique characteristics associated with that entity. In this case, that entity is **nmap**. The lab uses `nmap -sT` as the exmaple for TCP connect scans. The pcap for this lab is also located in the "nmap" directory. The biggest clue though is in how nmap handles setting the TCP window size. In [this](https://github.com/nmap/nmap/issues/2491) now closed issue in the nmap GitHub repo, the user points out how nmap sets the TCP window size.

(In [nmap/tcpip.cc, line 602](https://github.com/nmap/nmap/blob/806c0af5ee008ace06dbf1623765ff97edb89e33/tcpip.cc#L602) at the time of this writeup):
```
  if (window)
    tcp->th_win = htons(window);
  else
    tcp->th_win = htons(1024); /* Who cares */
```

Essentially this code block is saying - if the TCP window size is not specified, set it to 1024. The "Whoe cares" comment kind of tells you what the developer was thinking, the window size doesn't really matter. Since nmap isn't expecting to receive any application data (only attempting to see if the port is open), the window size doesn't matter. This does somewhat matter though because now we have a unique caharteristic that can be used to identify potential nmap traffic (if the person initiating the scanning is lazy).

### Question 2 - Which scan type is used to scan the TCP port 80?

### Question 3 - How many "UDP close port" messages are there?

### Question 4 - Which UDP port in the 55-70 port range is open?

### Question 5 - What is the number of ARP requests crafted by the attacker?

### Question 6 - What is the number of HTTP packets received by the attacker?

### Question 7 - What is the number of sniffed username&password entries?

### Question 8 - What is the password of the "Client986"?

### Question 9 - What is the comment provided by the "Client354"?

### Question 10 - What is the MAC address of the host "Galaxy A30"?

### Question 11 - How many NetBIOS registration requests does the "LIVALJM" workstation have?

### Question 12 - Which host requested the IP address "172.16.13.85"?

### Question 13 - What is the IP address of the user "u5"? (Enter the address in defanged format.)

### Question 14 - What is the hostname of the available host in the Kerberos packets?

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

