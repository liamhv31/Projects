## Perimeter Logs: Investigting the Breach

### Overview
Initech Corp, a mid-sized financial services company, has recently deployed a new firewall and intrusion detection system (IDS) to monitor its network perimeter. Over the past month, security analysts have noticed abnormal traffic patterns, but the SOC team has been overwhelmed and missed deeper analysis.

### Task
As a new security analyst, you have been tasked with reviewing one month of perimeter logs to determine what techniques the adversary used, and whether they succeeded in breaching the perimeter. You have been given three sets of logs from the time of the incident. The logs can be found in the Perimeter_logs/challenge directory on the Desktop.
- Firewall Logs: `firewall.log`
- WAF Logs: `ids_alerts.log`
- VPN Logs: `vpn_auth.log`

This lab allows us to use the command line for investigating raw log files, or a Splunk instance which will be much easier and simulate an actual SOC environment! In this lab however, I will show how to do it in both.

### Network Assets
The Network of Initech Corp contains the following assets. We can use that as a reference.

| IP       | Hostname | Role     | OS       | Team     | Criticality |
|----------|----------|----------|----------|----------|-------------|
| 10.0.0.20 | FINANCE-SRV1 | File/Finance Server (SMB) | Windows Server | Finance IT | High |
| 10.0.0.50 | VPN-GW | VPN Gateway | Linux | NetOps | Critical |
| 10.0.0.51 | APP-WEB-01 | Internal Web/App | Linux | Apps Team | High |
| 10.0.0.60 | WORKSTATION-60 | Employee Workstation | Windows 10 | Sales | Medium |
| 10.8.0.23 | VPN-CLIENT-ATTK | VPN Assigned Client (Ephemeral) | N/A | N/A | Critical |
| 10.0.1.10 | DMZ-WEB | DMZ Web Server | Linux | NetOps | Medium |

### Question 1 - Examine the firewall logs. What external IP performed the most reconnaissance?
#### Grep
This lab is very similar to [Network Perimeters: Monitoring and Protecting](https://github.com/liamhv31/Projects/blob/main/Network-Perimeters-Monitoring-and-Protecting.md). We can use `grep` to easily find this information. Since the question is asking for the most offending IP, we need to count and sort unique results:
```
grep 'BLOCK' firewall.log | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr
```
You can also achieve the same results with a slightly different query:
```
cat firewall.log | grep "BLOCK" | cut -d' ' -f5 | cut -d: -f1 | sort -nr | uniq -c
```
This shows the following IPs and how many BLOCK actions were performed against each:

<img width="231" height="86" alt="image" src="https://github.com/user-attachments/assets/a93498c7-42e6-4c9e-a836-85884f558df3" />

#### Splunk
If you're within a corporate environment, you'll hopefully have something like Splunk to use instead. These logs can be found uner the "**Search & Reporting**" blade and the "**network_logs**" **index**

<img width="1407" height="851" alt="image" src="https://github.com/user-attachments/assets/66d01be3-cc4d-4922-833e-02eefa3fe6cd" />

Now we want to select the "**firewall_log**" **sourcetype**. You can either select it from field options below (see above image) or type it into the query: `index="network_logs" sourcetype=firewall_logs`. To find the **BLOCKED** events, we can use the **action** field: `index="network_logs" sourcetype=firewall_logs action=BLOCK`. now if we click in the **src_ip** field, we can see the same results we got using `grep`.

<img width="796" height="397" alt="image" src="https://github.com/user-attachments/assets/95829812-c64a-405a-b7bf-b0413d02a533" />

Doing it with a Splunk query would look like this:
```
index="network_logs" sourcetype=firewall_logs action=BLOCK
| stats count AS block_count by src_ip
| sort - block_count
```

`stats count AS block_count by` says we want to count the following field and display those results in a column called `block_count`. `src_ip` is the field we're counting and it's going to be done for each unique value. Finally, we use `sort - block_count` to tell Splunk to sort our results using the `block_count` field in descending order (using the `-` character). the `|` operator is fundemantal here, as each line is treated as "do this, then this, then this." So, get results, then process, then process again, etc., until we get our desired results.

**Answer**: 203.0.113.45
