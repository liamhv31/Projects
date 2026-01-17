<img width="476" height="92" alt="image" src="https://github.com/user-attachments/assets/11a593a9-4159-467e-9505-4e578f980c11" />## The Greenholt Phish

### Overview
A Sales Executive at Greenholt PLC received an email that he didn't expect to receive from a customer. He claims that the customer never uses generic greetings such as "Good day" and didn't expect any amount of money to be transferred to his account. The email also contains an attachment that he never requested. He forwarded the email to the SOC (Security Operations Center) department for further investigation.

### Task
Investigate the email sample to determine if it is legitimate.

### Question 1 - What is the Transfer Reference Number listed in the email's Subject?
Just look at the email subject.

<img width="959" height="422" alt="image" src="https://github.com/user-attachments/assets/56a759f8-1038-4cef-b8a5-1687699f40af" />

**Answer**: 09674321

### Question 2 - Who is the email from?
This question is asking for the name of the person that sent the email.

<img width="959" height="425" alt="image" src="https://github.com/user-attachments/assets/b9abb0dd-6cc4-4dfb-b4f6-693fb3b3d750" />

**Answer**: Mr. James Jackson

### Question 3 - What is his email address?
This would be the email of Mr. James Jackson.

<img width="959" height="425" alt="image" src="https://github.com/user-attachments/assets/1baf3162-61d8-43fd-ad80-c4e8126260ee" />

**Answer**:

### Question 4 - What email address will receive a reply to this email?
In the email client I'm using (**Thunderbird Mail**), you can see the **Reply to** email at the top of the email. However, if you're using a client that doesn't show this, you may need to look at the raw email data. In Thunderbird, I did this by going to **View** &rarr; **Message Source** (or **Ctrl+U**).

<img width="958" height="421" alt="image" src="https://github.com/user-attachments/assets/0b2f75b9-2c89-4a61-ab57-3afa10873a59" />

Now just **Ctrl+F** for the **Reply-To** field.

<img width="478" height="377" alt="image" src="https://github.com/user-attachments/assets/ea7a52ba-1ff5-4ea4-a2dc-c0bb6e52c4f5" />

**Answer**: info.mutawamarine@mail.com

### Question 5 - What is the Originating IP?
The first few answers have been very surface level and easy to find. Now we will need to start digging a little deeper. All you need to do is look for the **X-Originating-IP** field and... there's nothing there. It almost looks redacted: `X-Originating-IP: [x.x.x.x]`. What gives? This field _can_ show the original sender of the email. However, the **X-Originating-IP** field is optional and not enforced by SMTP. Many modern email clients will hide the IP here for privacy.

A more trusting source to look at is the **Recieved** field directly below it.
```
Received: from 10.197.41.148  (EHLO sub.redacted.com) (x.x.x.x)
  by mta4212.mail.bf1.yahoo.com with SMTP; Wed, 10 Jun 2020 05:58:54 +0000
Received: from hwsrv-737338.hostwindsdns.com ([192.119.71.157]:51810 helo=mutawamarine.com)
	by sub.redacted.com with esmtp (Exim 4.80)
	(envelope-from <info@mutawamarine.com>)
	id 1jissD-0004g5-Ts
	for webmaster@redacted.org; Wed, 10 Jun 2020 01:02:04 -0400
```

This will show the **mail-routing trace**, showing every destination the email hit before getting to you (and where it came from). The destinations, or hops, are in **ascedning** order, meaning that the first **Recieved** entry at the bottom is where is first entered the mailing system, and the top entry is the last destination before reaching you. So with that in mind, if we want to find where the email came from, we look at the bottom. There's alot of interesting information there, but what we're looking for is the IP, which is also there!

<img width="959" height="445" alt="image" src="https://github.com/user-attachments/assets/9c500e19-f877-466a-b196-2700dac35fc1" />

**Answer**: 192.119.71.157

### Question 6 - Who is the owner of the Originating IP? (Do not include the "." in your answer.)
This admittedly took me some time to figure out. At first, I thought the answer was `mutawamarine.com`, as that is who the sending server is claiming to be, but this was not correct. After a bit of pondering I decided to try and lookup that domain in VirusTotal, WHOIS (DomainTools), and Cisco Talos. I found out more about the owners of that domain but none of that was the answer I needed. I then did the same thing for the server itself, to see if I could find out who actually owned it. VirusTotal didn't provide me much besides the fact that it was a Hostwinds domain (obvious given the domain name). I then tried a WHOIS lookup and same thing. It only kept showing Hostwinds, so I tried that as an answer but it didn't match the pattern it was looking for. After going through the WHOIS results again I noticed this:

<img width="387" height="349" alt="image" src="https://github.com/user-attachments/assets/28fa4bce-12c3-4b72-9836-048cc794204a" />

The question says not to include the "." in the answer, and without it, this matches the pattern of the answer it's looking for. Sure enough, it was correct. I'm a bit surprised as I interpreted the question as "who owns the domain which is using Hostwinds to host it". I doubt that Hostwinds _actually_ owns that domain themselves, and it is likely someone using Hostwind's services to host their mail VPS (virtual private server).

**Asnwer**: Hostwinds LLC

### Question 7 - What is the SPF record for the Return-Path domain?
You can find the the `Return-Path` field in the email source, just **Ctrl+F** for it. 

<img width="476" height="92" alt="image" src="https://github.com/user-attachments/assets/b3b430de-01c7-4ad9-a69e-763125eda6f6" />

Once you have that, use any SPF record lookup tool for this. I used [MX Toolbox SuperTool - SPF Record Checker](https://mxtoolbox.com/SuperTool.aspx?action=spf%3amutawamarine.com&run=toolpage).

**Answer**: v=spf1 include:spf.protection.outlook.com -all

### Question 8 - What is the DMARC record for the Return-Path domain?
Now just use a DMARC record checker. I used MX Toolbox SuperTool again but this time their [DMARC Lookup](https://mxtoolbox.com/dmarc.aspx) tool of course.

**Answer**: v=DMARC1; p=quarantine; fo=1

### Question 9 - What is the name of the attachment?

### Question 10 - What is the SHA256 hash of the file attachment?

### Question 11 - What is the attachments file size? (Don't forget to add "KB" to your answer, NUM KB)

### Question 12 - What is the actual file extension of the attachment?
