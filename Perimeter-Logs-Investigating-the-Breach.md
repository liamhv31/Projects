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

- `stats count AS block_count by` says we want to count the following field and display those results in a column called `block_count`.
- `src_ip` is the field we're counting and it's going to be done for each unique value.
- Finally, we use `sort - block_count` to tell Splunk to sort our results using the `block_count` field in descending order (using the `-` character).
- The `|` operator is fundemantal here, as each line is treated as "do this, then this, then this." So, get results, then process, then process again, etc., until we get our desired results.

**Answer**: 203.0.113.45

### Question 2 - In the firewall log, Which internal host was targeted by scans?
#### Grep
First, let's see what the strcuture of a firewall event is: `2025-08-25 00:47:46 ALLOW TCP 198.51.100.77:60317 -> 10.0.0.50:443`. We have the date, time, action, protocol, source IP, directional arrow, and destination IP. We're looking for inbound scanning from a malicious/suspicious IP to internal devices, which means our target IP (destination) will be on the right had side of the directional arrow. Splitting by whitespace, that's the seventh value. Now we can craft our `grep` command accordingly:
```
grep 'BLOCK' firewall.log | awk '{print $7}' | cut -d: -f1 | sort | uniq -c | sort -nr
```
The logs actually show us that there are two internal IPs targeted equally, the File/Finance Server and VPN Gateway. The answer they're looking for though is the File/Finance Server.

<img width="123" height="78" alt="image" src="https://github.com/user-attachments/assets/3f862c31-3997-4962-ba8e-2f054688e77b" />

#### Splunk
The Splunk query is also essentially the exact same as it was in the previous question. Instead we'll just look at the **src_ip** field. You can even see that the percentage of each of the top two IPs being targeteed is the exact same.

<img width="778" height="294" alt="image" src="https://github.com/user-attachments/assets/c63096dc-e61f-420a-8c3e-456c9da50cd4" />

Then if we want to use a query to get the answer, it would look like this:
```
index="network_logs" sourcetype=firewall_logs action=BLOCK
| stats count AS block_count by dst_ip
| sort - block_count
```

<img width="1908" height="151" alt="image" src="https://github.com/user-attachments/assets/1acdc717-ea48-4797-b8d6-3b5b5c0ab25b" />

**Answer**: 10.0.0.20

### Question 3 - Which username was targeted in VPN logs?
#### Grep
Let's start again by looking at the VPN log format using the `head vpn_auth.log -n 100` command:
```
2025-09-13 07:59:18 203.0.113.100 bob SUCCESS assigned_ip=10.8.0.118
2025-09-03 02:11:50 203.0.113.45 svc_backup FAIL
```
We can see there are two different event types: `SUCCESS` and `FAIL`. We want to find the most targeted users in the VPN logs so the `FAIL` events are our focus. This is what the command would look like:
```
grep 'FAIL' vpn_auth.log | awk '{print $4}' | sort | uniq -c | sort -nr
```

<img width="133" height="60" alt="image" src="https://github.com/user-attachments/assets/2beb71f7-05de-4007-9ccc-a0356e84b13e" />

**Little fun fact**: Usernames starting with `svc` is a very commonly used naming convention in IT to identify service accounts!

#### Splunk
In Splunk, we still query against the **network_logs** **index**, but change the **sourcetype** to **vpn_logs**. Also instead of looking at the **action** field like we did with the **firewall_logs**, we need to look at the **result** field. Little side note, having data sources all map to different fields like what we see here can be quite inneficient for searching across vasts amounts of enterprise data. This is where something like a Unified Data Model (UDM) really shines. In a UDM structure, result and action from the VPN and firewall logs would just map to the same field. This is helpful, for example, if you want to search for failure events for a specific entity across disparate data sources. This is a topic for another time though.

Back to our question, if we use this query, `index="network_logs" sourcetype="vpn_logs" result=FAIL`, we can look at the **username** field to find the same answer:

<img width="784" height="264" alt="image" src="https://github.com/user-attachments/assets/fdf44043-dcfa-484b-aa42-9ea261ccd804" />

Again, if we want to use a query instead:
```
index="network_logs" sourcetype="vpn_logs" result=FAIL
| stats count AS fail_count by username
| sort - fail_count
```

<img width="1910" height="108" alt="image" src="https://github.com/user-attachments/assets/1f8109d4-7905-4be6-b90b-7b2a77ff792b" />

**Answer**: svc_backup

### Question 4 - What internal IP was assigned after successful VPN login?
#### grep
We need to find the IP assigned to the user for the first successful login after the failed attempts. We can simply do this with the command `grep 'svc_backup' vpn_auth.log`. We then just need to scroll top where there is the first successful authentication after the barage of failed attempts. We can also see the the source IP is the same for both the failed attempts and successful one.

<img width="658" height="36" alt="image" src="https://github.com/user-attachments/assets/5ccc7817-a3df-4dd3-985f-db16b55f03af" />

This could be easy to do given the size of the file, or very difficult. You can also simply look at which source IP is responsible for the majority of the failed attempts (I would say minimum 3-5 consecutive fails from the same IP before success), then use that to find the success events.
```
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ grep 'FAIL' vpn_auth.log | awk '{print $3}' | sort | uniq -c | sort -nr
    118 203.0.113.45
    1 203.0.113.100
    1 198.51.100.92
    1 198.51.100.45
```
We see that most are from **203.0.113.45**. Now let's see the successful events that follow.

<img width="668" height="151" alt="image" src="https://github.com/user-attachments/assets/0a27e64b-4493-4dc4-ab7c-b42e37d3b060" />

So we can see that right after the many failed attempts from that IP against the **svc_backup** user, there was a success event where they were assigned an internal IP. If we wanted to be even more precise, and lean more towards building out an actual detection, we can create a classic **Multiple Failed Logon Attempts Followed by a Success** detection. We'll do this in bash using the `awk` command. This is not ideal, and has it's **blind spots** as a detection, but in a real environment, you'll hopefully have a very capable enterprise SIEM.
```
awk '
{
  # Define which element in the log event is the IP and status
  ip=$3
  status=$5

  if (status=="FAIL") {
    # Consecutive FAIL streak only continues if same IP as previous line
    if (ip==prev_ip) fail_streak++
    else fail_streak=1

    flagged = (fail_streak>=3) ? 1 : 0
  }
  else if (status=="SUCCESS") {
    # Print only the first SUCCESS immediately after a 3+ FAIL streak
    if (flagged && ip==prev_ip) {
      print
      flagged=0
    }
    # SUCCESS breaks the FAIL streak
    fail_streak=0
  }
  else {
    # Unknown status breaks streak
    fail_streak=0
    flagged=0
  }

  prev_ip=ip
}
' vpn_auth.log
```
This is how it works:
1. This command is ran against each line sequentially in the log file
2. First the variables are assigned (strongly relies on consistent event format)
3. `prev_ip`, `fail_streak`, and `flagged`, persist between lines
4. If we assume the first scanned line is a **SUCCESS** event, the `fail_streak` will remain zero
5. Once the first **FAIL** event is reached, `fail_streak` will be set to 1, and that source IP will be assigned to `prev_ip`
6. If the next event is also a **FAIL** from the same source IP, the `fail_streak` will be incremented by one, then that `ip` is set to `prev_ip`
7. `flagged = (fail_streak>=3) ? 1 : 0` checks each iteration if the `fail_streak` is three or more
8. This continues on and on. Let's assume that each subsequent event is a **FAIL** from the same source IP (you can start to see the crack in the logic), the `fail_streak` will rise above three and that will trigger the `flagged` variable to be set to one
9. Once the next **SUCCESS** event is reached, assuming it's from the same source IP and it is immediately after the **FAIL** events, the script will print the event to the terminal
10. `2025-09-03 02:19:40 203.0.113.45 svc_backup SUCCESS assigned_ip=10.8.0.23`

#### Splunk
In Splunk, we can do this with less lines of code and per each source IP.
```
index=network_logs sourcetype=vpn_logs
| sort 0 _time
| streamstats last(result) as prev_result by src_ip
| streamstats count(eval(result="FAIL")) as fail_count by src_ip reset_on_change=true
| where result="SUCCESS" AND fail_count>=3
| table _time src_ip username assigned_ip fail_count
```

This is how it works:
1. Searches **vpn_logs** in the **network_logs** index
2. `sort 0 _time` sorts all events in chronological order
3. `streamstats last(result) as prev_status by src_ip` creates the `prev_result` field which contains the status from the previous event for that same IP. This gives us event sequence per IP
4. `streamstats count(eval(result="FAIL")) as fail_count by src_ip reset_on_change=true` is where most of the leg work happens
   - `count(eval(result="FAIL"))` counts only events where the **result** is **FAIL**
   - `as fail_count` creates a new column to track this value
   - `by src_ip` does this for every IP
   - `reset_on_change=true` resets the counter when the IP changes or the sequence of events breaks
5. `where result="SUCCESS" AND fail_count>=3` is the condition for the detection to trigger. So the condition is a **SUCCESS** event with three or more **FAIL** events immediately before it
6. `table _time src_ip username assigned_ip fail_count` formats the results into a table
The final results will show us two hits since there are sequential **SUCCESS** events for the same IP after the **FAIL** events

<img width="1904" height="92" alt="image" src="https://github.com/user-attachments/assets/8170f9b2-214b-49b3-b363-0f8fa93898b0" />

You could implement deduplication logic in the rule or the playbook to fix this in a real environment, but that's outside the scope of this lab.

**Answer**: 10.8.0.23

### Question 5 - Which port was used for lateral SMB attempts?
