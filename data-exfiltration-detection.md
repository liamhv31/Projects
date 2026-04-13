## Detection: Data Exfil Through DNS Tunneling

### Question 1 - What is the suspicious domain receiving the DNS traffic?
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

**Answer**: tunnelcorp.net

### Question 2 - How many suspicious traffic/logs related to dns tunneling were observed?
You can see the total number of packets returned from the query at the bottom of Wireshark.

<img width="239" height="28" alt="image" src="https://github.com/user-attachments/assets/640dcc61-d716-4ab2-a273-94c7e0949e44" />

**Answer**: 315

### Question 3 - Which local IP sent the maximum number of suspicious requests?
