## The Greenholt Phish

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

### Question 7 - What is the SPF record for the Return-Path domain?

### Question 8 - What is the DMARC record for the Return-Path domain?

### Question 9 - What is the name of the attachment?

### Question 10 - What is the SHA256 hash of the file attachment?

### Question 11 - What is the attachments file size? (Don't forget to add "KB" to your answer, NUM KB)

### Question 12 - What is the actual file extension of the attachment?
