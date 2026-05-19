## The Basics

### Question 1 - Read the "capture file comments". What is the flag?
The packet capture file comments can be used to add metadata notes about the pcap. It can used to writeup anything noteworthy about the pcap file, including capture purpose, network details, discovered hosts and IPs, etc. You can find the comments by going to **Statistics** &rarr; **Capture File Properties**. The comments are at the bottom.

<img width="575" height="709" alt="image" src="https://github.com/user-attachments/assets/100de862-4cf1-487b-9ecf-d5d41c517a2e" />

Scroll down in the comments to see the flag.

**Answer**: TryHackMe_Wireshark_Demo

### Question 2 - What is the total number of packets?
Every packet capture in Wireshark will tell you the number of packets that are inclulded at the bottom of the page. As you filter and query for the data you want, that number will change to show you the number returned from your query, as well as the percentage of the total pcap it makes up.

It also shows you how many comments there are for the capture!

<img width="364" height="27" alt="image" src="https://github.com/user-attachments/assets/d1c7fedc-c90e-4c7b-a9a6-29e37dffb49a" />

**Answer**: 58620

### Question 3 - What is the SHA256 hash value of the capture file?
You can actually find the hash values of the packet capture in Wireshark itself (which is the purpose of this exercise), though I will also demonstrate two other ways to find the has value (Linux specific). First you can find the hash in the same place you find the comments in **Statistics** &rarr; **Capture File Properties**. The **Details** window shows you both the SHA256 and SHA1 hashes.

<img width="574" height="235" alt="image" src="https://github.com/user-attachments/assets/8f9b8733-f2a9-4ed5-90da-e1e03237e890" />

The next way you can find the has, which applies to any file on Linux, is by right-clicking the file, selecting **Properties**, and then page over to the **Digests** tab. There you will see a list of hash functions. Select the ones you want to see the **Digest** for, and click the **⚙️Hash** button.

<img width="609" height="510" alt="image" src="https://github.com/user-attachments/assets/c9dc77b7-08bc-4137-8cbb-943439188bc2" />

The final method is by going to the terminal and using the `sha256sum` command:
```
ubuntu@ip-10-66-189-142:~/Desktop$ sha256sum Exercise.pcapng 
f446de335565fb0b0ee5e5a3266703c778b2f3dfad7efeaeccb2da5641a6d6eb  Exercise.pcapng
```

The PowerShell equivalent to this on Windows would be `Get-FileHash .\Exercise.pcapng`.

**Answer**: f446de335565fb0b0ee5e5a3266703c778b2f3dfad7efeaeccb2da5641a6d6eb

### Question 4 - View packet number 38. Which markup language is used under the HTTP protocol?
You can filter by packet number with the following query: `frame.number == 38`. You can also filter by packet numbers in the following ways:
- Return multiple packets: `frame.number in {1 2 3}`
- Return a range of packets: `frame.number >= 1 && frame.number <= 10`
- Return everything except a specific packet(s): `frame.number != 2`

Once the packet is returned, select it, then there should be a portion of the packet that says **eXtensible Markup Language**, or more commonly referred to - **XML**. This is an HTTP packet that contains XML body/content. You may also notice this in the packet details pane: `[14 Reassembled TCP Segments (18364 bytes)]`. That suggests that the XML content was likely too large to fit into a single TCP packet, so it had to be reassmebled.

<img width="954" height="793" alt="image" src="https://github.com/user-attachments/assets/6d4c353e-5aed-4964-8d80-5c200739d55d" />

**Answer**: eXtensible Markup Language

### Question 5 - What is the arrival date of the packet?
The arrival date and time is stored in the frame metadata of the packet. Look for the `Arrival Time` field. This represents when the packet was captured by the device running Wireshark. There are a few other date/time related fields within the frame metadata:
- `[Time shift for this packet: 0.000000000 seconds]` - This will show if a manual timestamp adjustment has been appied to the packet. Wireshark will actually let you shift packet timestamps forward or backwords. This can be helpful for these like synchronizing multiple captures, correcting clock drift, or compensating for incorrect system times
- `Epoch Time: 1084443432.158193000 seconds` - Wireshark also shows the time in **Epoch**, which is a way of tracking time as a single, continuous number (seconds) since January 1, 1970, at 00:00:00.
- `[Time delta from previous captured frame: 0.070101000 seconds]` - This is the time passed since the previous packet (number 37) was captured
- `[Time delta from previous displayed frame: 0.000000000 seconds]` - This is the time difference between the currently selected packet, and the last visible or displayed packet shown after applying a filter
- `[Time since reference or first frame: 4.846969000 seconds]` - This is the time passed since the first packet in the capture (by default)

<img width="953" height="849" alt="image" src="https://github.com/user-attachments/assets/2b8feb55-ba3b-4978-9106-bcdc9e3dec55" />

**Answer**: 05/13/2004

### Question 6 - What is the TTL value?
**TTL** (Time-to-Live), is a limit on how long a packet can traverse the network before being discarded. TTL is an IP header field, and is measured in "hops". A hop is when one device forwards the packet to another device, that is one hop. To find the TTL of a packet in Wireshark, simply expand the **Internet Protocol Version 4** (IPv4) tree header in the packet details pane. You will see the "Time to live" field.

<img width="1916" height="783" alt="image" src="https://github.com/user-attachments/assets/7533afc3-d595-4e93-80bf-d58d7878e3dd" />

So this packet can be forwarded to 47 devices before it is discarded. This prevents packets from traversing the network indefinitely.

**Answer**: 47

### Question 7 - What is the TCP payload size?
**TCP (Transmission Control Protocol)** is part of **Transport Layer** (Layer 4) of the **OSI model**, so the payload size will be under a different part of the packet. Expand the "Transmission Control Protocol" tree header in the packet details pane, and scroll until you see the "TCP payload" field.

<img width="1918" height="780" alt="image" src="https://github.com/user-attachments/assets/64588be8-e225-4445-93e4-3467ee154662" />

TCP has two main parts - the header and the payload. The header will contain networking information and metadata, while the payload is the actual application data being transmitted.

**Answer**: 424

### Question 8 - What is the e-tag value?
An **ETag** (Entity Tag) is an HTTP response header that identifies a specific version of a resource. Think of it like a digitial fingerprint, version number, or hash. You will find the ETag under the "Hypertext Transfer Protocol" tree.

<img width="1917" height="784" alt="image" src="https://github.com/user-attachments/assets/7e92192f-bf61-4a6c-9e2f-66307de68350" />

**Answer**: 9a01a-4696-7e354b00

### Question 9 - Search the "r4w" string in packet details. What is the name of artist 1?
Wireshark allows you to search packet lists, packet details, and packet bytes for a specific value. Depending on what you're looking for, you can search using a display filter, hex value, string, or regular expression. To do this, you go to **Edit** &rarr; **Find Packet**. For this question, make sure the search criteria is set to **Packet details** and **String**. This will find the specified string in every packet it appears in.

<img width="1919" height="845" alt="image" src="https://github.com/user-attachments/assets/f0f9a55f-1d8f-428d-838c-31b49359b65f" />

We can see the answer embedded in the HTML data (which contains the "r4w" string)

**Answer**: r4w8173

### Question 10 - Go to packet 12 and read the packet comments. What is the answer?

### Question 11 - There is a ".txt" file inside the capture file. Find the file and read it; what is the alien's name?

### Question 12 - Look at the expert info section. What is the number of warnings?

## Packet Operations

## Traffic Analysis
