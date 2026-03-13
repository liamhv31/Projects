## External vs Internal Scanning

### Question 1 - Which file contains logs that showcase internal scanning activity?
In this lab, we have three files that have been exported from a SIEM solution. One of the files contain sanitized logs while the others are raw. We need to identify which of them contains the internal scanning activity. Let's start by looking at a sample of each.

**log-session-0.csv**:
```
"@timestamp","source.ip","source.port","destination.ip","destination.port"
"Sep 7, 2025 @ 17:16:42.944","203.0.113.25",39120,"192.168.230.145",5922
"Sep 7, 2025 @ 17:16:42.941","203.0.113.25",39120,"192.168.230.145",5850
"Sep 7, 2025 @ 17:16:42.939","203.0.113.25",39120,"192.168.230.145",9500
"Sep 7, 2025 @ 17:16:42.937","203.0.113.25",39120,"192.168.230.145",57797
"Sep 7, 2025 @ 17:16:42.934","203.0.113.25",39120,"192.168.230.145",8333
"Sep 7, 2025 @ 17:16:42.932","203.0.113.25",39120,"192.168.230.145",6567
"Sep 7, 2025 @ 17:16:42.930","203.0.113.25",39120,"192.168.230.145",8100
"Sep 7, 2025 @ 17:16:42.927","203.0.113.25",39120,"192.168.230.145",416
"Sep 7, 2025 @ 17:16:42.925","203.0.113.25",39120,"192.168.230.145",2001
```

**log-session-1.csv**:
```
"@timestamp","source.ip","source.port","destination.ip","destination.port","rule.name","rule.category","rule.action","network.protocol",message,"event.dataset"
"Sep 7, 2025 @ 17:16:42.944","203.0.113.25",39120,"192.168.230.145",5922,"-","-","-","-","{""ts"":1757265402.944286,""uid"":""CjxnoPYeUTSU7IAo"",""id.orig_h"":""203.0.113.25"",""id.orig_p"":39120,""id.resp_h"":""192.168.230.145"",""id.resp_p"":5922,""proto"":""tcp"",""conn_state"":""S0"",""local_orig"":true,""local_resp"":true,""missed_bytes"":0,""history"":""S"",""orig_pkts"":1,""orig_ip_bytes"":44,""resp_pkts"":0,""resp_ip_bytes"":0,""community_id"":""1:JbLPvC3YPB/75Ez5qzk/uYmWvg4="",""orig_mac_oui"":""VMware, Inc.""}","zeek.conn"
"Sep 7, 2025 @ 17:16:42.941","203.0.113.25",39120,"192.168.230.145",5850,"-","-","-","-","{""ts"":1757265402.941845,""uid"":""CCQ2X65B10mAqnDk"",""id.orig_h"":""203.0.113.25"",""id.orig_p"":39120,""id.resp_h"":""192.168.230.145"",""id.resp_p"":5850,""proto"":""tcp"",""conn_state"":""S0"",""local_orig"":true,""local_resp"":true,""missed_bytes"":0,""history"":""S"",""orig_pkts"":1,""orig_ip_bytes"":44,""resp_pkts"":0,""resp_ip_bytes"":0,""community_id"":""1:W5L3G5RHb22jFf6he3Ny2tirmvY="",""orig_mac_oui"":""VMware, Inc.""}","zeek.conn"
"Sep 7, 2025 @ 17:16:42.939","203.0.113.25",39120,"192.168.230.145",9500,"-","-","-","-","{""ts"":1757265402.939535,""uid"":""CwFVNK30si35hu3IK3"",""id.orig_h"":""203.0.113.25"",""id.orig_p"":39120,""id.resp_h"":""192.168.230.145"",""id.resp_p"":9500,""proto"":""tcp"",""conn_state"":""S0"",""local_orig"":true,""local_resp"":true,""missed_bytes"":0,""history"":""S"",""orig_pkts"":1,""orig_ip_bytes"":44,""resp_pkts"":0,""resp_ip_bytes"":0,""community_id"":""1:Yk9/HosLqMnRfPdKAG+EcnUQUWY="",""orig_mac_oui"":""VMware, Inc.""}","zeek.conn"
"Sep 7, 2025 @ 17:16:42.937","203.0.113.25",39120,"192.168.230.145",57797,"-","-","-","-","{""ts"":1757265402.937283,""uid"":""CM5ES71L2iGyWBosQa"",""id.orig_h"":""203.0.113.25"",""id.orig_p"":39120,""id.resp_h"":""192.168.230.145"",""id.resp_p"":57797,""proto"":""tcp"",""conn_state"":""S0"",""local_orig"":true,""local_resp"":true,""missed_bytes"":0,""history"":""S"",""orig_pkts"":1,""orig_ip_bytes"":44,""resp_pkts"":0,""resp_ip_bytes"":0,""community_id"":""1:YypxgI+tttS4QoKoVCOjRCnJpwY="",""orig_mac_oui"":""VMware, Inc.""}","zeek.conn"
```

**log-session-2.csv**
```
"@timestamp","source.ip","source.port","destination.ip","destination.port","rule.name","rule.category","rule.action","network.protocol",message,"event.dataset"
"Sep 7, 2025 @ 17:14:58.054","192.168.230.127",52424,"192.168.230.145",445,"-","-","-","-","{""ts"":1757265298.054489,""uid"":""C7n2be4tyyXjT0yml7"",""id.orig_h"":""192.168.230.127"",""id.orig_p"":52424,""id.resp_h"":""192.168.230.145"",""id.resp_p"":445,""proto"":""tcp"",""conn_state"":""S0"",""local_orig"":true,""local_resp"":true,""missed_bytes"":0,""history"":""S"",""orig_pkts"":1,""orig_ip_bytes"":44,""resp_pkts"":0,""resp_ip_bytes"":0,""community_id"":""1:DisMYr/alNGeSQTyiBpOeOI3uok="",""orig_mac_oui"":""VMware, Inc.""}","zeek.conn"
"Sep 7, 2025 @ 17:14:58.054","192.168.230.127",52424,"192.168.230.145",3389,"-","-","-","-","{""ts"":1757265298.054489,""uid"":""CB8RGb10QX0Gbeqyoi"",""id.orig_h"":""192.168.230.127"",""id.orig_p"":52424,""id.resp_h"":""192.168.230.145"",""id.resp_p"":3389,""proto"":""tcp"",""conn_state"":""S0"",""local_orig"":true,""local_resp"":true,""missed_bytes"":0,""history"":""S"",""orig_pkts"":1,""orig_ip_bytes"":44,""resp_pkts"":0,""resp_ip_bytes"":0,""community_id"":""1:DGFbQqklTLkGkEaHHI+ClFh6iRw="",""orig_mac_oui"":""VMware, Inc.""}","zeek.conn"
"Sep 7, 2025 @ 17:14:58.054","192.168.230.127",52424,"192.168.230.1",3389,"-","-","-","-","{""ts"":1757265298.054487,""uid"":""CO3Qlf4qql8OMQvzh8"",""id.orig_h"":""192.168.230.127"",""id.orig_p"":52424,""id.resp_h"":""192.168.230.1"",""id.resp_p"":3389,""proto"":""tcp"",""conn_state"":""S0"",""local_orig"":true,""local_resp"":true,""missed_bytes"":0,""history"":""S"",""orig_pkts"":1,""orig_ip_bytes"":44,""resp_pkts"":0,""resp_ip_bytes"":0,""community_id"":""1:IbuiX5K2jgyUoYxG14YmKXt542M="",""orig_mac_oui"":""VMware, Inc.""}","zeek.conn"
"Sep 7, 2025 @ 17:14:58.054","192.168.230.127",52424,"192.168.230.1",445,"-","-","-","-","{""ts"":1757265298.054489,""uid"":""CXyDkj2aEkFezjHlO8"",""id.orig_h"":""192.168.230.127"",""id.orig_p"":52424,""id.resp_h"":""192.168.230.1"",""id.resp_p"":445,""proto"":""tcp"",""conn_state"":""S0"",""local_orig"":true,""local_resp"":true,""missed_bytes"":0,""history"":""S"",""orig_pkts"":1,""orig_ip_bytes"":44,""resp_pkts"":0,""resp_ip_bytes"":0,""community_id"":""1:V8ZRxQVqxE6DPQOc6QUg0gQh+r8="",""orig_mac_oui"":""VMware, Inc.""}","zeek.conn"
```

We can see that ****log-session-0.csv** contains the sanitized logs while the other two files contain the raw **Zeek** logs. You can tell because we can see all of the Zeek **conn.log** fields in the other log files, whereas in the sanitized file it's only the important data summarized. Just compare events one from both **log-session-0.csv** and **log-session-1.csv**. They are the same. Zeek conn.log events are great to identify scanning because they show us every connection attempt (TCP, UDP, ICMP).

We're looking for internal scanning, which means both the source and destination IPs will be private IPs. If we look at the sample events from **log-session-0.csv**, the source IP (`203.0.113.25`) is public (private IP ranges include 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16). This automatically eliminates **log-session-1.csv** because the events are the same, just the raw version. In **log-session-2.csv**, we can see that both the source and destination IPs are part of the 192.168.0.0/16 private IP range.

**Answer**: log-session-2.csv

### Question 2 - How many log entries are present for the internal IP performing internal scanning activity?
There are a few different ways you can achieve this using `grep`, `awk`, `cut`, etc. I use `awk`.
```
awk -F'"' 'NR>1 {print $4}' log-session-2.csv | wc -l
```
- `-F'"'` splits lines into fields using quotes as the delimiter
- `NR>1` skips the first row which is the header
- `{print $4}` returns the source IP on this case (`source.ip`). Because we are using quotes as the delimiter, the fields will look like this:
  - `$1` is empty because our line starts with a quote
  - `$2` is `@timestamp`
  - `$3` is `,`
  - `$4` is the `source.ip`
- `log-sessions-2.csv` is our log file
- `| wc -l` counts the number of lines returned
 
There are likely cleaner ways to do this, but this works. This returns 2276 events, which is "correct" as far as the accepted answer to the question goes, but in reality this is wrong. I modified the command to only return events where the `source.ip` and `destination.ip` were both private IPs to confirm all were actually internal-to-internal scanning and this returned 2020 events.
```
awk -F'"' 'NR>1 && $4 ~ /^(10\.|172\.16\.|192\.168\.)/ && $6 ~ /^(10\.|172\.16\.|192\.168\.)/ {print $4}' log-session-2.csv | wc -l
```
This is because there are many events where an internal IP is making ICMP requests to an external IP.
```
"Sep 7, 2025 @ 17:09:05.440","192.168.230.127",8,"203.0.113.2",0,"-","-","-","-","{""ts"":1757264945.440747,""uid"":""CbVQdA2SKCdRkGMIvg"",""id.orig_h"":""192.168.230.127"",""id.orig_p"":8,""id.resp_h"":""203.0.113.2"",""id.resp_p"":0,""proto"":""icmp"",""duration"":2.3759799003601074,""orig_bytes"":224,""resp_bytes"":0,""conn_state"":""OTH"",""local_orig"":true,""local_resp"":true,""missed_bytes"":0,""orig_pkts"":4,""orig_ip_bytes"":336,""resp_pkts"":0,""resp_ip_bytes"":0,""community_id"":""1:cxLZTyBY3XbE62SdWoAZw0AlApg="",""orig_mac_oui"":""VMware, Inc.""}","zeek.conn"
```
So it appears that whoever designed this lab made a mistake, either by accidentally including events in this log file that weren't internal-to-internal scans, or by incorrectly setting the correct answer. If this file was meant to only contain internal-to-internal scanning and was pulled from a SIEM, that would mean that either the query the person used to pull these events from the SIEM were flawed, or there is a mapping/parser problem with the SIEM (e.g., an event type specifically for internal-to-internal scanning). If the latter, this is where you would submit this finding to the team responsible for log management to have them fix it (in a real-world, enterprise environment).

**Answer**: 2276

### Question 3 - What is the external IP address that is performing external scanning activity?
Since **log-session-2.csv** contains (mostly) internal-to-internal scanning activity, and **log-session-0.csv** and **log-session-1.csv** are the same, we can just review the sanitized data. This shows just one IP with 2014 different connection attempts targeting the same internal IP (except one attempt) over hundreds of different ports. These events appear to all be milliseconds apart. All of these observations are highly indicative of automated scanning.

<img width="728" height="58" alt="image" src="https://github.com/user-attachments/assets/edf52ced-0f88-4933-aabf-22a0d37dc655" />

**Answer**: 203.0.113.25

## Horizontal vs Vertical Scanning

### Question 1 - One of the log files contains evidence of a horizontal scan. Which IP range was scanned? Format X.X.X.X/X
We need to identify a single IP range being scanned across multiple ports. Let's start with **log-session-0.csv**. Counting the number of unique IPs shows us that there is only a couple IPs in this log file.
```
2013 192.168.230.145
1 239.255.255.250
```
This could mean that this file contains vertical scanning activity, but not horizontal. Since **log-session-1.csv** is the same, we can skip to **log-session-2.csv**. We can see in this log file that there is a large number of unique IPs. Scrolling through the output, we can see very clearly that there is a single IP range showing up. It appears that the entire 203.0.113.0/24 subnet is being scanned.
```
1 203.0.113.254
1 203.0.113.253
1 203.0.113.252
1 203.0.113.251
1 203.0.113.250
1 203.0.113.249
1 203.0.113.248
1 203.0.113.247
1 203.0.113.246
1 203.0.113.245
1 203.0.113.244
1 203.0.113.243
...
```
It doesn't tell us what the subnet is though, so how do you find that out? The subnet number tells us how many bits of an IP address reprersent the network portion. Whatever is remaining belongs to the host. For `203.0.113.0/24`, the binary representation would look like this: `11111111.11111111.11111111.00000000`. So the first three octets of the IP rnage represent the network. The part for the host can range from 0-255 (203.0.113.0 - 203.0.113.255). How do we know it's `/24` though? We just count the number of bits for the part of the IP rangenthat is always the same. In the logs, the first three octets are always `203.0.113`. That's `8 x 3 = 24`. Since the entire last octet is available for the host, we would represent the range starting at the first available number for the fourth octet, which is `0`.

**Answer**: 203.0.113.0/24

### Question 2 - In the same log file, there is one IP address on which a vertical scan is performed. Which IP address is this?
In the same output for question 1, we can see that IP `192.168.230.145` shows up 2007 times. Let's see what the activity looks like.
```
...
1 ,3030,
1 ,33354,
1 ,524,
1 ,1717,
1 ,1007,
1 ,7106,
1 ,3031,
1 ,2301,
1 ,9898,
1 ,999,
...
```
The way I printed the data is a bit crude but it still tells us what we need to know - this IP is being scanned across thousands of different ports. This is a clear sign of vertical scanning.

**Answer**: 192.168.230.145

### Question 3 - On one of the IP addresses, only a few ports are scanned which host common services. Which are the ports that are scanned on this IP address? Format: port1, port2, port3 in ascending order.
We need to identify which log file contains an IP scanned only a few times. **log-session-0.csv** only has one other IP with one connection attempt.
```
2013 192.168.230.145
1 239.255.255.250
```
Again, this means **log-session-1.csv** will be the same, so our answer is in the final log file. This is the file that had the horizontal scanning, so we can ignore that entire IP range from before
```
>$ awk -F'"' 'NR>1 && $6 !~ /^(203\.0\.113)/ {print $6}' log-session-2.csv | sort | uniq -c

2 -
7 192.168.230.1
2013 192.168.230.145
1 239.255.255.250
```
We can also ignore `192.168.230.145` since there are thousands of events for it. Conversely, `239.255.255.250` only has one event so it's likely not that one either. `192.168.230.1` only has a few connection events so let's take a look at what the unique ports are.
```
>$ awk -F'"' 'NR>1 && $6 ~ /^(192\.168\.230\.1)$/ {print $7}' log-session-2.csv | sort| uniq -c

2 ,0,
2 ,3389,
2 ,445,
1 ,80,
```
Three ports here stand out: `3389`, `445`, and `80`. These ports are **RDP**, **SMB**, and **HTTP** respectively. These are definitely common services!

**Answer**: 80, 445, 3389

## The Mechanics of Scanning
In this part of the lab, we will use Elastic (a.k.a the ELK stack), to answer the questions below. Elastic Stack is an open-source platform used for centralized logging, data analysis, and visualization. While it is not a SIEM, you can configure it to act like one.

Searching for logs in Elastic is done within the **Analytics** &rarr; **Discover** page.

<img width="218" height="703" alt="image" src="https://github.com/user-attachments/assets/d3e9d047-292a-426e-8033-162ccf2901a9" />

Data is typically (and should be) seperated in different **Data views** of **indexes**. For this lab, our data is under the **All logs** index.

<img width="363" height="380" alt="image" src="https://github.com/user-attachments/assets/9debf17c-0707-4527-8593-b342495c55da" />

We also want to search the entire time range.

<img width="1112" height="703" alt="image" src="https://github.com/user-attachments/assets/c03cee12-f243-450c-9ab0-ee27fedc3c21" />

Data!

<img width="1917" height="707" alt="image" src="https://github.com/user-attachments/assets/57558fb4-a755-4296-bad3-fc7f86d7b8fb" />

### Question 1 - Which source IP performs a ping sweep attack across a whole subnet?
A ping sweep is an ICMP based scanning technique, which is one of the most basic types of scanning. I use **Kibana Query Language (KQL)**, but it looks like there is also a new piped language structure called **[ES|QL](https://www.elastic.co/docs/reference/query-languages/esql)**, which almost looks like a child between **Splunk Query Language (SPL)** and **Structured Query Language (SQL)** at first glance.

I digress, in order to find our answer we need to identify ICMP traffic. Looking at the field options, `network.protocol` looks like a safe bet: `network.protocol: icmp`. This gives us 256 **documents** (which is like a record or event in Elastic speak, represented as JSON).

<img width="1914" height="703" alt="image" src="https://github.com/user-attachments/assets/fd231cbe-616c-4594-a62d-7a52a3fbe5b8" />

We should also add the `source.ip` and `destination.ip` fields as columns in the output. You can do this by clicking the :heavy_plus_sign: icon beside the field in the field list.

<img width="627" height="697" alt="image" src="https://github.com/user-attachments/assets/d34b659b-eca3-4a1b-82bd-b08cd5aa1c11" />

You can also drag and drop columns to re-arrange them in an order you find best.

<img width="1633" height="418" alt="image" src="https://github.com/user-attachments/assets/bfdd7b5e-2b2b-4c9c-828b-2721a94ae568" />

Now you can't create an aggregation output of unique values in the **Discover** tab, but you can click on the fields on the field list like in Splunk to see the top values. We can see that there is only one source IP making ICMP requests.

<img width="490" height="267" alt="image" src="https://github.com/user-attachments/assets/34852f37-328e-40d9-9938-f7fb30f510ec" />

We can see that the `destination.ip` has a pattern of IPs that look like they belong in the same subnet. In fact, this looks like the same data in the previous parts of this lab where we identified the `203.0.113.0/24` subnet.

<img width="503" height="566" alt="image" src="https://github.com/user-attachments/assets/c6ce33c9-5682-435d-a59f-59b7afd6c1b8" />

I would show a better way to do this is by creating a **Lens** visualization, but it looks like this instance doesn't haven't it configured correctly.

**Answer**: 192.168.230.127

### Question 2 - The zeek.conn.conn_state value shows the connection state. Using the information provided by this value, identify the type of scan being performed by 203.0.113.25 against 192.168.230.145
If we click on the `zeek.conn.conn_state` field, we can see that there is only one connection state which is `S0`.

<Insert image>
  
This indicates that a connection attempt was made, but there was no response from the destination. I was looking at the different `conn_state` values in the **[base/protocols/conn/main.zeek](https://docs.zeek.org/en/current/scripts/base/protocols/conn/main.zeek.html#field-Conn::Info$conn_state)** documentation, but it didn't really indicate what the type of scanning activity could be.

<Insert image>

Some more Googling brought be to the official Zeek GitHub repository, specifically **[zeek/scripts/base/protocols/conn/main.zeek](https://github.com/zeek/zeek/blob/master/scripts/base/protocols/conn/main.zeek)**. Doing a Ctrl + F brought me to this code snippet
```
...
else if ( rs == TCP_CLOSED && os == TCP_CLOSED )
  return "SF";
else if ( os == TCP_CLOSED )
  return r_inactive ? "SH" : "S2";
else if ( rs == TCP_CLOSED )
  return o_inactive ? "SHR" : "S3";
else if ( os == TCP_SYN_SENT && rs == TCP_INACTIVE )
  return "S0";                                             <----LOOK HERE
else if ( os == TCP_ESTABLISHED && rs == TCP_ESTABLISHED )
  return "S1";
else
  return "OTH";
...
```
You can see on the line where I added `<----LOOK HERE`, that there is the same conn_state value. The condition for it to be returned looks to be related tp **TCP SYN**. If we look back at our logs in Elastic, we can actually see that the `network.protocol` value is also `tcp`.

<Insert image>

