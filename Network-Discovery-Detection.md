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

## Horizontal vs Vertical Scanning

## The Mechanics of Scanning
