## Network Perimeters: Monitoring and Protecting

### Task
Review the perimeter logs (traditional firewall, VPN, and WAF) to answer the questions below.

### Question 1 - Examine the firewall logs. Which IP address is performing the port scan?
You can use `grep` to find only the blocked firewall events. This will show you the malicious IP for this task.

<img width="749" height="322" alt="image" src="https://github.com/user-attachments/assets/3f628ccb-2818-43e5-ab65-144cc92f2c5d" />

This works well enough for this exercise, but in a much larger log file, there will likely be 10s of thousands of lines to look through at least. A cleaner way to do this is with the following `grep` query:
```
grep 'BLOCK' firewall_logs.txt | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr
```
Let's break it down:
- `grep 'BLOCK' firewall_logs.txt` gets us only the BLOCKED firewall events
- `awk '{print $5}'` sepeartes each string on a line by whitespace by default. Since this is a structured log, the IP and port number will always be at the 5th index (or 4th for you prgrammers)
- `cut -d: -f1` strips the colon and port number away, leaving only the IP. the `-d` flag tells `cut` where to split the string (the colon is the delimiter), meaning the IP is field one and the port is field two. The `-f1` flag tells vut to return field one (the IP)
- `sort | uniq -c` counts occurences per grouped (unique) IP
- `sort -nr` orders the list in descending order based on the count of each IP

There may be even better ways to do this, but this way is a good start. Ideally, you have a SIEM you can use to query all of your logs with ease!

**Answer**: 203.0.113.10

### Question 2 - In the WAF Logs, which single source IP is responsible for all the blocked web attacks?
We can apply the same logic as we did in question 1 with some slight alterations based on the WAF log format.
```
timestamp=2025-09-22T09:14:10Z src_ip=198.51.100.12 action=BLOCK request="GET /products.php?id=12%20UNION%20SELECT%20user,pass%20FROM%20users" rule_id=942100 attack_type="SQL Injection"
timestamp=2025-09-22T09:14:46Z src_ip=198.51.100.12 action=BLOCK request="GET /search.php?q=<script>alert('XSS')</script>" rule_id=941100 attack_type="XSS"
timestamp=2025-09-22T09:15:42Z src_ip=198.51.100.12 action=BLOCK request="GET /../../../../etc/passwd" rule_id=930120 attack_type="Directory Traversal"
```

Query: `grep 'action=BLOCK' waf_logs.txt | awk '{print $2}' | cut -d= -f2 | sort | uniq -c | sort -nr`

<img width="1283" height="40" alt="image" src="https://github.com/user-attachments/assets/ad791d4d-767e-4c6a-8ce5-35acd04d32ec" />

**Answer**: 198.51.100.12

### Question 3 - In the VPN logs, how many brute-force attempts failed?
This is what the VPN logs look like:
```
2025-09-22 10:12:01 SUCCESS_AUTH TCP 198.51.100.88:41233 -> 10.0.0.1:443 (user 'b.jones')
2025-09-22 10:12:02 FAILED_AUTH TCP 45.137.22.13:31245 -> 10.0.0.1:443 (user 'admin')
2025-09-22 10:12:03 FAILED_AUTH TCP 45.137.22.13:31246 -> 10.0.0.1:443 (user 'administrator')
2025-09-22 10:12:04 SUCCESS_AUTH TCP 203.0.113.55:51234 -> 10.0.0.1:443 (user 's.smith')
```

We can use grep again to count only the FAILED_AUTH events: `grep -c 'FAILED_AUTH' vpn_logs.txt`

<img width="749" height="36" alt="image" src="https://github.com/user-attachments/assets/27c352ed-027a-47f1-ab10-5b2d1ca7e652" />

If you wanted to be a bit more technical about this attack classification, based on MITRE techniques and what we see in the VPN logs, this would be a Brute Force via Password Spraying (T1110.003) attack. This is because the brute force is spread across multiple different users

**Answer**: 90

### Question 4 - Which suspicious IP address was found attempting the brute-force attack against the VPN gateway?
Same `grep` logic as the previous questions: `grep 'FAILED_AUTH' vpn_logs.txt | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr`

<img width="1268" height="34" alt="image" src="https://github.com/user-attachments/assets/be0e9efe-eda0-4757-80f2-7058ab2d293f" />

**Answer**: 45.137.22.13
