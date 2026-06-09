## Packet Operations

### Question 1 - Investigate the resolved addresses. What is the IP address of the hostname starts with "bbc"?
The **Resolved Addresses** feature shows you which IP addresses Wireshark was able to resolve to a human-readable name. This can provide more context for the conversation. You can open this view by going to **Statistics** &rarr; **Resolved Addresses**. You can search for an entry by using a minimum of three characters. You can search across all entries or choose hosts, ethernet addresses, ethernet manufacturers, or ethernet well-known addresses. 

<img width="1162" height="727" alt="image" src="https://github.com/user-attachments/assets/7dd56343-c458-48da-99a8-e60ec4e667e0" />

If you do not see resolved addresses, then that means you likely either have the required settings disabled and/or no DNS traffic in your packet capture. Typically, you need to have **Resolve network (IP) addresses** enabled under **Edit** &rarr; **Preferences** &rarr; **Name Resolution**. This will let Wireshark translate certain IPs into human-readable names. I say "tyically" because even if you have this option disabled, Wireshark can still display hostnames that it learned from DNS, mDNS, LLMNR, or NetBIOS name-resolution traffic contained within the capture.

You can also select the **Use an external network name resolver** option under the same setting page which will tell Wireshark to use your DNS to resolve addresses. This will cause the machine running Wireshark to perform DNS lookups for addresses in the capture, generating additional network traffic.

There is also the **Resolve Physical Addresses** option, which allows Wireshark to use MAC addresses to display vendor/manufacturer names instead of raw MAC addresses.

**Answer**: 199.232.24.81

### Question 2 - What is the number of IPv4 conversations?
The **Conversations** feature shows you all of the traffic (packets) sent between two specific endpoints. Instead of looking at individual packets, it aggregates the communication and shows statistics about it. You can view this by going to **Statistics** &rarr; **Conversations**. Conversations are grouped into multiple categories:
- Ethernet
- IPv4
- IPv6
- TCP
- UDP

You can see the number of conversations for each group on the tab itself. E.g., there are 435 seperate IPv4 conversations.

<img width="1114" height="641" alt="image" src="https://github.com/user-attachments/assets/af7f9275-c995-4ce2-8249-5c16d1c4cd33" />

Here is some of the information shown within the **IPv4** tab:
- **Address A**: One endpoint involved in the conversation
- **Address B**: The other endpoint involved in the conversation
- **Packets**: The total number of packets sent between the endpoints
- **Bytes**: The total number of bytes sent between the endpoints
- **Packets A &rarr; B**: The number of packets sent from endpoint A to endpoint B
- **Bytes A &rarr; B**: The number of bytes sent from endpoint A to endpoint B
- **Packets B &rarr; A**: The number of packets sent from endpoint B to endpoint A
- **Bytes B &rarr; A**: The number of bytes sent from endpoint B to endpoint A

**Answer**: 435

### Question 3 - How many bytes (k) were transferred from the "Micro-St" MAC address?
The **Statistics** window will show you the conversations between two endpoints and even allow you to resolve the MAC address for ethernet conversations to see which hostnames are talking to each other. This will show you a conversation involving the "Micro-St" MAC address. However, this will only show you the data sent between the two endpoints for that specific conversation. We want to know how many total bytes were transferred from that MAC address.

We can find this by switching to the **Statistics** &rarr; **Endpoints** window. Select the **Ethernet** tab and check the **Name resolution** box at the bottom. Now we can see how many bytes were transferred from each MAC address. I should note that the accepted answer is 7474, however, the wording of the question implies "how many bytes were sent from the Micro-St MAC address". So, either the author coded the incorrect answer by mistake, or the question is poorly written. 7474 represents the total number of bytes transferred from and recieved to this endpoint. The actual number of transferred bytes (Tx Bytes) is 1083. So technically, the correct answer is 1083 bytes, but for the sake of completing this lab, the answer is 7474 bytes.

<img width="955" height="627" alt="image" src="https://github.com/user-attachments/assets/1e60bd00-4136-4664-aa0d-7e08a1ae1d81" />

**Ansswer**: 7474

### Question 4 - What is the number of IP addresses linked with "Kansas City"?
We can use the **Endpoints** window to get this answer too. Select the **Map** button on the bottom of the window, then choose **Open in browser**. Unfortunately, I wasn't able to open it myself in the provided lab machine at the time of this writeup, but you can see IPs grouped by city on a world map. Below is a reference screenshot.

<img width="1637" height="537" alt="image" src="https://github.com/user-attachments/assets/15414c97-cae6-4089-87c9-ce6f695725cf" />

An important note - This feature is not available in Wireshark by default. It requires supplemental geolcation data. Modern Wireshark versions supports the MaxMind GeoIP database, but it requires additional configuration.

**Answer**: 4

### Question 5 - Which IP address is linked with "Blicnet" AS Organisation?
Identifying to which ASN (Autonomous System Number) an IP address belongs to also requires GeoIP lookup. If it's configured, you can see this information in **Endpoints** window. Just select the **IPv4** tab and scroll to the right until you see **AS Organization**.

<img width="952" height="649" alt="image" src="https://github.com/user-attachments/assets/ebec834d-772a-4770-8cd7-4470fe506f54" />

**Answer**: 188.246.82.7

### Question 6 - What is the most used IPv4 destination address?
The best and easiest way to find out which IPv4 address was the most used _destination_ address, we can use the **Endpoints** window again. Go to the **IPv4** tab and sort by either **Rx Packets** or **Rx Bytes** in descending order. This will tell you which IP recieved the most packets and bytes.

<img width="1114" height="634" alt="image" src="https://github.com/user-attachments/assets/b16b13b0-60c1-4beb-b9c4-81b75d7ccda0" />

**Answer**: 10.100.1.33

### Question 7 - What is the max service request-response time of the DNS packets?
There are two queries that come to mine that can give you the same answer in pretty much the same way. You can either use `dns.flags.response==1` or `dns.time` to return DNS packets. Click on one of the packets and expand the **Domain Name System** tree, then **Additional records**. Right-click on the time field and select **Apply as Column**. This will add the response time value as a column for all visible DNS packets. You can then click on the column header to sort in descending order.

<img width="1918" height="849" alt="image" src="https://github.com/user-attachments/assets/a3142a82-5ed1-4fc8-bb6f-ebac84fd75f0" />

**Answer**: 0.467897

### Question 9 - What is the number of HTTP Requests accomplished by "rad[.]msn[.]com?
You can achieve this a couple differnet ways. Option 1, filter the packets where the HTTP host is the one we're looking for and only return HTTP requests:
```
http.host==rad.msn.com && http.request
```
You can then see the number of returned packets at the bottom. Option 2, go to the **Statistics** &rarr; **HTTP** &rarr; **Requests** page and look for the domain name. You can then see the number of total requests.

<img width="1919" height="870" alt="image" src="https://github.com/user-attachments/assets/64140362-eab1-43e1-ae15-de4844e9db79" />

**Answer**: 39

### Question 10 - What is the number of IP packets?
Simply type `ip` as a filter and then look at the number of displayed packets at the bottom of the window.

**Answer**: 81420

### Question 11 - What is the number of packets with a "TTL value less than 10"?
If you don't know the query, select any IP packet returned from the previous answer and expand the **Internet Protocol Version 4** tree (since TTL is part of the IP header). Right-click on the **Time to live** field and select **Apply as Filter** &rarr; **Selected**. This will produce: `ip.ttl==X` (X being the TTL of that specific packet). Then just adjust the query to return all packets with a TTL of less than 10: `ip.ttl<10`. You will now see the number of packets displayed at the bottom.

**Answer** 66

### Question 12 - What is the number of packets which uses "TCP port 4444"?
Use the following filter to return the number of packets: `tcp.port==4444`

**Answer**: 632

### Question 13 - What is the number of "HTTP GET" requests sent to port "80"?
Use the following filter to return the number of packets: `http.request.method=="GET" && tcp.dstport==80`. We use TCP as the transport layer protocol to return port 80 destination events since HTTP (HTTP/1.1 and HTTP/2) traditionally uses TCP. For HTTP/3 events, you may see **QUIC** (created by Google), which is a more advanced transport layer protocol built on top of UDP and essentially replaces the older TCP + TLS combination.

**Answer**: 527

### Question 14 - What is the number of type A DNS Queries?
There are two parts to this query. The first is filtering by DNS queries (reqeusts). The second is scoping it the DNS A records. We can do this with the following query:
```
dns.flags.response == 0 && dns.qry.type == 1
```

- `dns.flags.response == 0` satisfies our first condition. This may seem counterintuitive since we're looking for DNS queries (requests), but we're using dns.flags.**response**. This is because there is no "request" flag in the DNS header. Think of it as a question. `dns.flags.response` is basically asking 'is this packet a response?'. If yes, then the flag is set to `1`. If no, the flag is set to `0`. The DNS header contains a Query/Response (QR) bit.

<img width="170" height="105" alt="image" src="https://github.com/user-attachments/assets/a0da60a8-fa19-433f-b9e4-fcf13cd5bdd1" />

- `dns.qry.type == 1` satisfies the second condition for type A records only. Type A DNS records are the most fundamental type of DNS record, and is what is responsible for mapping domain names to IP addresses.

You will notice that this doesn't just return DNS packets, but also packets using the LLMNR protocol. LLMNR (Link-Local Multicast Name Resolution) is a Microsoft name resolution protocol that looks very similar to DNS. If a host can't resolve a name through normal DNS, it uses LLMNR. Many of the same DNS fields also exist in LLMNR packets. For example, if you expand the **Link-local Multicast Name Resolution** trees, then **Queries** and all sub-trees, you can see that the LLMNR record type is A. If you right-click on the **Type** field and apply it as a filter, you will see the same filter used for DNS packets.

<img width="1917" height="852" alt="image" src="https://github.com/user-attachments/assets/5904fd1f-441f-49e0-a961-0e5c4058f89b" />

This is why Wireshark may return additional packets that aren't DNS. The question is specifically asking for **type A DNS queries**, which means we need to exclude the LLMNR packets. There are two ways you can do this.

Specify only DNS packets (best if you _just_ want DNS):
```
dns.flags.response == 0 && dns.qry.type == 1 && dns
```

Filter out LLMNR packets:
```
dns.flags.response == 0 && dns.qry.type == 1 && !llmnr
```

**Answer**: 51

### Question 15 - Find all Microsoft IIS servers. What is the number of packets that did not originate from "port 80"?
To find the answer to this question, we first must have a basic understanding of what Microsoft IIS is. Microsoft IIS (Internet Information Services) is a web server created by Microsoft. That is the very high-level definition, and really all we need to know.

Now that we know what this service is, how do we find it amongst the packets? Microsoft IIS is a **web server**, which means it will communicate using HTTP/S. There's actually a **server** field in HTTP headers, so we can use that in Wireshark to return HTTP packets that only involves a Microsoft IIS web server. We likely don't know the right format of the **server** header field value, so to ensure we find these packets, we can either do a **contains** search for **Microsoft** or **IIS**.
```
http.server contains "IIS"
```

This will return all HTTP packets where the **server** header field contains **IIS** in the name.

<img width="1919" height="846" alt="image" src="https://github.com/user-attachments/assets/7686e45a-e38a-45d2-b8a8-def961c4b9cc" />

Now that we know the format for the server field and how **Microsoft-IIS** appears in it, we can refine our search to make sure we don't pick up extra packets that match our contain filter, but aren't Microsoft IIS servers.
```
http.server contains "Microsoft-IIS"
```
To answer the port part of the question, we need to look at TCP since that is the transport layer protocol that HTTP primarily uses. We want packets that _didn't_ come from port 80.
```
http.server contains "IIS" && not tcp.srcport==80
```
This will return the correct number of packets. You can use the `!=` operator, but this is deprecated. It will still return the correct number of packets, but using it may return unexpected results so this is why we use the `not` operator.

**Answer**: 21

### Question 16 - Find all Microsoft IIS servers. What is the number of packets that have "version 7.5"?
If you havent noticed already, the server version is included in the **server** header field. We can use the following filter to find only version 7.5 Microsoft IIS servers.
```
http.server contains "Microsoft-IIS" && http.server contains "7.5"
```
It's a bit cleaner to use regex with the `matches` keyword instead.
```
http.server matches "^Microsoft-IIS\/7\.5"
```

**Answer**: 71

### Question 17 - What is the total number of packets that use ports 3333, 4444 or 9999?
Two queries come to mind that helps answer this question. First, by just using the `||` operator.
```
tcp.port == 3333 || tcp.port == 4444 || tcp.port == 9999 || udp.port == 3333 || udp.port == 4444 || udp.port == 9999
```
The better way would be to use the `in` operator, which is a bit cleaner. This allows us to search for multiple, space separated values.
```
tcp.port in {3333 4444 9999} || udp.port in {3333 4444 9999}
```

**Answer**: 2235

### Question 18 - What is the number of packets with "even TTL numbers"?
Wireshark actually allows you to use arithmetic functions within filters. However, you need to be running at least version 4.0. This lab only had Wireshark 3.2.3 installed. This would have allowed us to do something like: `ip.ttl % 2 == 0`. This expression uses modulo (%), which is a common arithmetic operator in programming used to see what the remainder of a division operation is. E.g., `4 % 2 = 0` since `4` can be divided by `2` evenly so there is no remainder.

Since we can't use arithmetic though, we'll need to get creative. We need to find even number TTLs. Even numbers always end with a 0, 2, 4, 6, or 8. We can use this knowledge to do a regex match for any TTL value ending in one of those numbers. `ip.ttl` is an unsigned integer field type though, so we will need to cast it to a string first before performing the regex match.
```
string(ip.ttl) matches "[02468]$"
```
- `string()` is the Wireshark function that casts values to strings
- `"[02468]$"` is the pattern that says match either of these numbers at the end (`$`) of the string

**Answer**: 77289

### Question 19 - Change the profile to "Checksum Control". What is the number of "Bad TCP Checksum" packets?
For this question, we need to change to the **"Checksum Control" Configuration Profile**. Configuration profiles are saved Wireshark configurations that let you quickly change your Wireshark settings without having to change multiple settings each time. E.g., certain configuration profiles may have certain packet coloring rules depending on the type of analysis you want to do.

There is a built-in configuration profile called **Default**, but any others are custom. To see these profiles, go to **Edit** &rarr; **Configuration Profiles...** (or **Ctrl+Shift+A**). We can see multiple custom profiles.

<img width="635" height="465" alt="image" src="https://github.com/user-attachments/assets/c2f87ccf-bb18-4d26-ac6e-301b4fdeb952" />

Some are _global_ (saved in the `usr/share/wireshark/profiles` directory, and some may be _personal_ (saved in your own home directory). Select **Checksum Control** and then **Ok**. Now we need to find "Bad TCP checksums". Now there are a couple ways we can find these bad TCP checksum packets. First is by using a display filter to check the status of a TCP checksum.
```
tcp.checksum.status == Bad
```
We can also use `tcp.checksum_bad.expert`, which checks to see if there was an expert info warning or error generated for this. We can see that all of the bad packets are colored black.

<img width="1916" height="334" alt="image" src="https://github.com/user-attachments/assets/b984cc70-b105-47dd-b295-17e77c2e023a" />

The second approach is by looking at the **Analyze** &rarr; **Expert Information** window itself. There we can see multiple **Bad checksum** errors for TCP, IPv4, and UDP, along with the number of packets that match.

<img width="571" height="433" alt="image" src="https://github.com/user-attachments/assets/7dd335f8-7452-498a-9533-e8746c923cf8" />

The reason this wouldn't work in the **Default** profile is because the **Validate the TCP checksum if possible** option is disabled.

<img width="416" height="272" alt="image" src="https://github.com/user-attachments/assets/1a651e63-0b1d-4c9d-91fa-88d84d1c2e31" />

**Answer**: 34185

### Question 20 - Use the existing filtering button to filter the traffic. What is the number of displayed packets?
