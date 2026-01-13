### Analyzing SMTP Responses

### Task
In this task, I will use Wireshark to analyze a PCAP file with SMTP traffic.

Helpful resources:
- [Display Filter Reference: Simple Mail Transfer Protocol](https://www.wireshark.org/docs/dfref/s/smtp.html)
- [What are SMTP codes and how to troubleshoot them](https://www.mailersend.com/blog/smtp-codes)

### Question 1 - Which Wireshark filter can you use to narrow down your results based on SMTP response codes?
**Answer**: smtp.response.code

### Question 2 - How many packets in the capture contain the SMTP response code 220 Service ready?
The above answer tells us what the filter for SMTP response codes is, while this question give us the response code itself (220). Now we just add those two together:

<img width="479" height="424" alt="image" src="https://github.com/user-attachments/assets/91f4e1fd-1859-4112-8fc2-a51fb3fe6277" />

**Answer**: 19

### Question 3 - One SMTP response indicates that an email was blocked by spamhaus.org. What response code did the server return?
There could be a few reasons why this email was blocked. Let's see which response codes we can see that are related to activity with spamhaus.org. First, I did a search for "spamhaus" in the packet details pane:

<img width="690" height="840" alt="image" src="https://github.com/user-attachments/assets/5e0671a8-e867-448c-80d3-617070bd013f" />

I set the search type to "String" and hit "Find"

<img width="1919" height="47" alt="image" src="https://github.com/user-attachments/assets/934d62fb-3f59-4dbc-82ec-5a8090e65b95" />

We can see that packet 156 shows in the info tab "Email blocked uing spamhaus.org". You can see the response there too, but also in the packet details pane if you expand the "Simple Mail Transfer Protocol" tree. The response code is "Requested action not taken: mailbox name not allowed (553)".

<img width="959" height="314" alt="image" src="https://github.com/user-attachments/assets/672a7736-3f3c-4d4c-81b9-7c459502a02d" />

This means that the recipient cannot be found due to errors in the email address. This is a "permanent SMTP error code" type, with this one in particular more related to spam or suspicious activity.

**Answer**: 553

### Question 4 - Based on the packet from the previous question, what is the full Response code: message?
We answered this one in the analysis of the previous question.

**Answer**: Requested action not taken: mailbox name not allowed (553)

### Question 5 - Search for response code 552. How many messages were blocked for presenting potential security issues?
Like in question 1, we filter by the response code: `smtp.response.code == 552`

**Answer**: 6
