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
