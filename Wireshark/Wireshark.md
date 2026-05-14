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

### Question 5 - What is the arrival date of the packet?

### Question 6 - What is the TTL value?

### Question 7 - What is the TTL value?

### Question 8 - What is the e-tag value?

## Packet Operations

## Traffic Analysis
