## Detection: Data Exfil through DNS Tunneling

### Question 1 - What is the suspicious domain receiving the DNS traffic?

#### Wireshark
In large packet captures, we need to start by narrowing down our search. First off, DNS tunneling involves encoding data in subdomains. Subdomains are a prefix added to the main domain name. E.g., in `app.foo.com`, `app` is the subdomain, `foo` is the domain name, and `.com` is the top-level domain (TLD). So for this question, we can use a filter to return only packets where `dns.qry.name` contains a subdomain. This can be done using regex.
```
dns.qry.name matches "^[^.]+\\..+\\..+"
```
This is just one indicator to filter by. In real scenarios you will likely need to filter by a few different indicators to remove false positives. You can further narrow it down by looking for high entropy or base32/base64-like patterns in the query name. This is where things get difficult in Wireshark, as it cannot calculate entropy. Additionally, if the subdomain doesn't follow conventional base32, base64, or something else, and you try filtering on those standard conventions, then you may miss malicious DNS queries. Outside of a SIEM or some custom scripting, your next best option is to filter based on subdomain length and prescense.
```
dns.qry.name matches "^[a-zA-Z0-9]{15,}\\..+\\..+"
```
This query returns us some high-entropy domain names. Another important thing to note is that in real DNS tunneling or beaconing scenarios, the destination may be a public DNS server like Google (8.8.8.8), Quad9 (9.9.9.9), or Cloudflare (1.1.1.1). Though I would only search for this directly if this aligns with a specific threat actor's TTPs. In the screenshot below, we can also see that the domain name remains the same, but the subdomain is always a different, high-entropy value. This is another strong DNS tunneling indicator.

<img width="1596" height="524" alt="image" src="https://github.com/user-attachments/assets/33b568dc-1ffc-464a-8cd7-3eb0416fea94" />

#### Splunk
This job is made easier in Splunk, though potentially less verbose in terms of logging depending on what is being ingested (likely not every packets detail). Ease of use will also largely depend on field mapping. In this case, the index and sourcetype will be: `index=data_exfil sourcetype="dns_logs"`. The full query will look like this:
```
index=data_exfil sourcetype="dns_logs"
| regex query="^[a-zA-Z0-9]{15,}\..+\..+"
```
Splunk uses standard regex, so there's no need to double escape like in Wireshark. This will return us the same results, but this is not the most efficient method to find our answer. What if there are tens of thousands of events like in a real enterprise environment? We want to return a count of all unique domains that match our criteria, and strip the subdomain so we don't return the same domain hundreds or thousands of times. This is what our new query will look like:
```
index=data_exfil sourcetype="dns_logs"
| regex query="^[a-zA-Z0-9]{15,}\..+\..+"
| eval base_domain=replace(query, "^[^.]+\.", "")
| stats count by base_domain
| sort -count
```
- `eval` is used to create or modify fields using expressions. So we're creating a new field (`base_domain`)
- `replace(query, "^[^.]+\.", "")` replaces the first domain from `query` and assigns what's left to our new variable. E.g., `ve3u2ryv8o23f387.tunnelcorp.net` will become `tunnelcorp.net`

This approach has one issue though, multi-subdomain queries will break this logic. If we have something like `foo.bar.website.com` we would get `bar.website.com`. Not terrible, but it could produce unintended results. Another version would be to account for multi TLDs. E.g., co.uk, com.au, com.mx, etc:
```
index=data_exfil sourcetype="dns_logs"
| regex query="^[a-zA-Z0-9]{15,}\..+\..+"
| rex field=query "(?i)(?<base_domain>(?:[^.]+\.(?:co\.uk|org\.uk|gov\.uk|ac\.uk|net\.uk|co\.jp|com\.au|net\.au|org\.au|co\.nz|com\.br|com\.mx|co\.in|com\.cn|com\.sg)|[^.]+\.[^.]+))$"
| stats count by base_domain
| sort -count
```
This will allow us to match both something like `3479fgwb8fgeq.tunnelcorp.net` and `3479fgwb8fgeq.tunnelcorp.net.uk`. This also has it's problems though. You need to now maintain a list of domain suffixes, which can cause you to miss events. This is when you would incorporate something like a lookup in Splunk to enrich your search (outside the scope of this lab).

**Answer**: tunnelcorp.net

### Question 2 - How many suspicious traffic/logs related to dns tunneling were observed?

#### Wireshark
You can see the total number of packets returned from the query at the bottom of Wireshark.

<img width="239" height="28" alt="image" src="https://github.com/user-attachments/assets/640dcc61-d716-4ab2-a273-94c7e0949e44" />

#### Splunk
With the query we used above, you can see the count of the returned suspicious query. If there are multiple subdomains, you can modify the query like so:
```
index=data_exfil sourcetype="dns_logs"
| regex query="^[a-zA-Z0-9]{15,}\..+\..+"
| rex field=query "(?i)(?<base_domain>(?:[^.]+\.(?:co\.uk|org\.uk|gov\.uk|ac\.uk|net\.uk|co\.jp|com\.au|net\.au|org\.au|co\.nz|com\.br|com\.mx|co\.in|com\.cn|com\.sg)|[^.]+\.[^.]+))$"
| stats count as total
```
`| stats count as total` now counts each base_domain count returned and presents the total

**Answer**: 315

### Question 3 - Which local IP sent the maximum number of suspicious requests?

#### Wireshark
To find the answer to this question, we can use the **Statistics** --> **Endpoints** menu. Before navigating here, make sure the filter used before is still applied. Once open, click on the **IPv4** tab and click on the **Packets** columns header to sort in descending order.

<img width="1514" height="616" alt="image" src="https://github.com/user-attachments/assets/ac210f8a-6414-4147-a73d-821c6731080b" />

We can see that the top IP is the Google DNS server (8.8.8.8). We're looking for an internal, corporate, or pvivate IP, so this is not the answer. The next IP with the most packets (72) is our answer.

#### Splunk
In Splunk, there are two ways to do this - the easy way, or the better way. The easy way would be to just click on the `src_ip` field after running your query and looking at what the top IP is.

<img width="790" height="536" alt="image" src="https://github.com/user-attachments/assets/ee06f98c-fa4d-4a8c-b60d-561710917379" />

The better way, especially if creating something like a report, is to use a query.
```
index=data_exfil sourcetype="dns_logs"
| regex query="^[a-zA-Z0-9]{15,}\..+\..+"
| stats count by src_ip
| sort -count
```

<img width="1908" height="184" alt="image" src="https://github.com/user-attachments/assets/6cb0f44e-ae30-4f35-86da-68531006ca57" />

**Answer**: 192.168.1.103

## Detection: Data Exfil through FTP
**Note**: The FTP part of the lab will not involve Splunk since there is no FTP data in this Splunk instance

### Question 1 - How many connections were observed from the guest account?

#### Wireshark
Since this is FTP, we need to find FTP events where the `USER` is in cleartext. We can do this with the following query:
```
ftp.request.command == "USER" && ftp.request.arg == "guest"
```

<img width="796" height="125" alt="image" src="https://github.com/user-attachments/assets/5a7ecaf6-7c52-4012-be6b-fb0aee8e34a2" />

**Answer**: 5

### Question 2 - What is the name of the customer-related file exfiltrated from the root account?
We just need to modify the previous query to return events for the `root` account instead of `guest`.
```
ftp.request.command == "USER" && ftp.request.arg == "root"
```
If you select each packet and expand the FTP tree, you can see the `STOR` (upload) command and the name of the file transferred.

<img width="647" height="157" alt="image" src="https://github.com/user-attachments/assets/09a2c0c8-ea16-4446-bed0-d62dc1e85b45" />

There's also another file being transferred from `root` called `internal_passwords.csv` but the question is asking for the **customer-related file**. We can't see the contents but judging off name alone, this file would not be the answer.

**Answer**: customer_data.xlsx

### Question 3 - Which internal IP was found to be sending the largest payload to an external IP?
There are a couple different ways we can go about this. First, let's apply a filter where we only return packets where the traffic is FTP and only internal-to-external. Wireshark allows subnet searches.
```
ftp && (ip.src == 192.168.0.0/16 or ip.src == 10.0.0.0/8 or ip.src == 172.16.0.0/12) and !(ip.dst == 10.0.0.0/8 or ip.dst == 172.16.0.0/12 or ip.dst == 192.168.0.0/16)
```
Now we can use the **Statistics** &rarr; **Conversations** menu to find our answer. This feature provides details of traffic passed between two endpoints. Open the menu and click on the IPv4 tab. We can see all of the fields are nicely organized in a table format. Click on the **Bytes A &rarr; B** column to sort in descending order.

<img width="1634" height="786" alt="image" src="https://github.com/user-attachments/assets/0fc674bf-b01e-4670-9ab0-0713b27409dc" />

You can also use the **Statistics** &rarr; **Endpoints** feature, but this will only show you the total bytes sent, and you would need to cross-check if it was to an external IP. You can also just use the regular filtered view, but I find the **Conversations** menu has a cleaner look

**Answer**: 192.168.1.105

### Question 4 - What is the flag hidden inside the ftp stream transferring the CSV file to the suspicious IP?
The first thing we need to do is identify the suspicious IP. We can find this by applying the same filter from question 1 or 2 where the customer data and internal passwords were being transmitted. Since this internal data is going from an internal to external IP, this is a strong indicator of malicious activity. The malicious IP is **185.203.119.12**. So far, out filter only looks like this: `ftp && ip.dst == 185.203.119.12`.

The question mentions a CSV file. We saw a CSV file from our previous question (**internal_passwords.csv**), but if we don't know the name of the file, than all we can search for is the CSV extension. Now there's one problem with this Wireshark packet capture - it seems that only the USER command was parsed as an actual **FTP** command. That's why when we try to apply **PASS** or **STOR** as a column, the column gets created like so:

<img width="84" height="309" alt="image" src="https://github.com/user-attachments/assets/e0eeba8f-49a1-45b0-ba99-f5bbd7cfdbb2" />

That just means that text does exist there. We also can't filter on **PASS** or **STOR**. Instead, it applies a raw byte comparison at a fixed offset inside the frame.

<img width="1914" height="803" alt="image" src="https://github.com/user-attachments/assets/5e8f5fc0-c57a-4ac5-a63e-15d10c761b50" />

Parsed commands should look like this:

<img width="207" height="49" alt="image" src="https://github.com/user-attachments/assets/6d935100-bf27-4b0f-baa3-55c1c17e1bd9" />

So what can we do? We can actually do a keyword search against the whole FTP part of the packet itself: `ip.dst == 185.203.119.12 && ftp contains "STOR" && ftp contains ".csv"`. This will return all **CSV** file transfers to the malicious IP. Then we just add our flag filter to it `ip.dst == 185.203.119.12 && ftp contains "STOR" && ftp contains ".csv" && ftp contains "THM"`. This will reveal the flag. You could also just simply use `ftp contains "THM"` as the filter to begin with, but in a real environment, you often go through a process of elimination, especially if you don't know the exact values you're looking for. If you wanted to go about this in an even more relistic way, you would use a regex match instead based on the structure of the flag and number of characters (if known).
```
ip.dst == 185.203.119.12 && ftp contains "STOR" && ftp contains ".csv" && ftp matches "[A-Za-z0-9]{3}\\{[A-Za-z0-9_]{21}\\}"
```

**Answer**: THM{ftp_exfil_hidden_flag}

## Detection: Data Exfil through HTTP

### Question 1 - Which internal compromised host was used to exfiltrate this sensitive data?

#### Wireshark
The problem with just diving into Wireshark to identify the compromised host is that the host is usually already identified via the detection/alert that was triggered on the SIEM/security platform beforehand. Wireshark is not for detection, it's for deeper investigation after the alert (and if it warrants further investigation). This makes it difficult to identify the answer to this question because now we need to make some assumptions as to _what could have_ the detection been that alerted to potential data exfiltration. There are several potential indicators for data exfiltration via HTTP, and Wireshark can't be used to effectively detect (not identify) all, so we will need to start with the basics.

_Typically_, HTTP data exfiltration is performed using the POST method (sending data to an external server). Attackers can also hide encoded data within GET requests, but POST is more common. So our filter will start like this:
```
http.request.method == "POST"
```
This returns about 50% of tha packet capture, which is still a bit too large to look through. The next thing we can do is try filtering all packets with a frame langth below a certain number. If we sort packets by frame length in descending order, we can see that they range steadily from about 400 to 565, with one outlier with a frame length of 782. That's quite the outlier actually, so we can just look at this packet to see why.

If we expand the **Hypertext Transfer Protocol** tree and click on **Data**, we can see that there was **654 bytes** of uploaded data in this request, and we can see the transferred, cleartext data on the right hand side.

<img width="1568" height="388" alt="image" src="https://github.com/user-attachments/assets/8684dcbc-586c-47d5-a7e3-e27ddd803b6f" />

The data shows things like "Internal Access Credentials - Finance Department", a username, password, and some incident repsonse notes for an HTTP POST alert. We can actually even see the flag for the next question if we scroll down a little bit. This is definitely the compromised host sending data.

#### Splunk
In this part of the lab we have data in Splunk under `index=data_exfil sourcetype="http_logs"`. Next we want to return **POST** requests only.
```
index=data_exfil sourcetype="http_logs" method=POST
```
We can follow the same pattern we used with Wireshark - sorting by the number of bytes sent from most to least.
```
index=data_exfil sourcetype="http_logs" method=POST
| table _time src_ip dest_ip method bytes_sent
| sort - bytes_sent
```
We can see the outlier at the top, and the compromised host.

<img width="943" height="446" alt="image" src="https://github.com/user-attachments/assets/5c4f147b-f120-40d9-830d-372cc8f9139f" />

If you're creating a detection that uses bytes sent as an indicator, start by finding the 95th percentiale of bytes sent and trigger only when the number of bytes sent is above that threshold.
```
index=data_exfil sourcetype="http_logs" method=POST
| eventstats perc95(bytes_sent) as p95_bytes_sent
| where bytes_sent >= p95_bytes_sent
| table _time src_ip dest_ip method bytes_sent p95_bytes_sent
| sort - bytes_sent
```

<img width="952" height="584" alt="image" src="https://github.com/user-attachments/assets/bbc69f22-1000-4d7c-9e63-87e73788c887" />

Splunk actually has a function called `perc95()`. The `eventstats` keyword allows us to use that function and also adds the new field to every event. We can see that the number of bytes sent in the 95th percentile is greater than ~441. This returns a lot of hits, so you can tweak it further to reduce noise/false positives. In this case, I would probably start with any events with 500 or more bytes transferred.

**Answer**: 192.168.1.103

### Question 2 - What's the flag hidden inside the exfiltrated data?
We can see the flag in the data of our previous answer. Let's assume we don't though. You can use regex like we did in question four in the previous section of this lab to find it if it's been parsed (which it has). You can also use the **Edit** &rarr; **Find Packet** search for regex matching or string searching if you know enough of the thing you're looking for (in this case we're looking for a string that starts with "THM").

It looks like there's actually a second flag! This one isn't the right one since it 1) Isn't coming from the compromised khost we identified previously; 2) The flag is too short for the flag the question is expecting; and 3) This packet doesn't show as strong of data exfiltration signals as the other packet. It's only a frame length of 120, it uses the GET method, and there's no data in the HTTP part of the packet being exfiltrated that we can see.

<img width="1579" height="687" alt="image" src="https://github.com/user-attachments/assets/bd55867a-1267-4a58-99d0-d97a2a013cd1" />

Real answer:

<img width="1579" height="838" alt="image" src="https://github.com/user-attachments/assets/3b1ad907-44e7-41f4-a15d-b1aaa565cbe7" />

**Answer**: THM{http_raw_3xf1ltr4t10n_succ3ss}

## Detection: Data Exfil through ICMP

### Question 1 - What is the flag found in the exfiltrated data through ICMP?
Using Wireshark, we need to find ICMP packets that contain exfiltrated data. One technique we can look for is encoded data inside ICMP echo requests. ICMP echo requests can have up to 64 bytes (depedning on the operating system), so we can start by looking for anything above that.
```
icmp.type==8 && frame.len > 64
```
We can see 6 packets, most of which being far higher than 64 bytes. If we look at the first one, it has 148 bytes. The IP is a private IP range, so the request isn't being sent externally, rather locally on the same network. Clicking on **Data** under the **Internet Control Message Protocol** tree shows some encoded data (in hex) that was attached to this echo request. This shows the following:
```
db01.internal.corp
Port: 5432
User: db_reader
Password: R3@d0nly!2025
```
The name of the server and username suggests that this is a database of some sort, and the port being 5432 *the official default TCP port for PostgreSQL) confirms that.

<img width="1576" height="608" alt="image" src="https://github.com/user-attachments/assets/34cb3658-56f9-4404-8c4a-e0ba57ec8160" />

The next packet contains data too. This time it looks like an SSH key fringerprint, and possibly the start of a datase connection string, though it is not actually present. There's also another host present.
```
rpnet.local
SSH Key Fingerprint: SHA256:9f:3a:2b:7c:1d:8e:4f:aa:bb:cc:dd:ee:ff:00:11:22
Database Connection:
```

<img width="1572" height="607" alt="image" src="https://github.com/user-attachments/assets/af267555-0586-44d1-8f94-0ef56e90efb1" />

If we continue to look at each packet, all of them have data encoded into the echo request. We need to find a flag with the format: `THM{.*}` We can search for `THM by doing a packet search against the packet bytes.

<img width="1575" height="627" alt="image" src="https://github.com/user-attachments/assets/c10f8fc2-5543-4b83-b360-aab0611d8d7e" />

**Answer**: THM{1cmp_3ch0_3xf1ltr4t10n_succ3sss}
