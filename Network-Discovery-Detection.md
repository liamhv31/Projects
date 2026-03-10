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

## The Mechanics of Scanning
