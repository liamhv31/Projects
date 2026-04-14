## Detection: Data Exfil Through DNS Tunneling

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
