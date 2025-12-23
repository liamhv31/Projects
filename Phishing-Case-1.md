### Scenario
You are a Level 1 SOC Analyst. Several suspicious emails have been forwarded to you from other coworkers. You must obtain details from each email for your team to implement the appropriate rules to prevent colleagues from receiving additional spam/phishing emails.
### Task
Use the tools discussed throughout this room (or use your own resources) to help you analyze each email header and email body.

### Question 1 - What brand was this email tailored to impersonate?
This one was very straight forward. I just opened up the Phish3Case1.eml file in the virutal machine. This will open up the original email in the Outlook client installed on the system. From there we can see the name of the company that the email is trying to impersonate plastered everywhere.

<img width="941" height="844" alt="image" src="https://github.com/user-attachments/assets/f76a0086-feb7-4016-bf66-8ab154eda4e7" />

**Answer**: Netflix

### Question 2 - What is the From email address?
Another easy one. You can see sender address at the top of the email.

<img width="938" height="840" alt="image" src="https://github.com/user-attachments/assets/a8889b1a-00ac-497a-8528-e7203313d789" />

**Answer**: JGQ47wazXe1xYVBrkeDg-JOg7ODDQwWdR@JOg7ODDQwWdR-yVkCaBkTNp[.]gogolecloud[.]com

**Note**: The answer is not in defanged format, but I did that here to sanitize the domain.

### Question 3 - What is the originating IP? Defang the IP address
For this one you'll need to open the raw email format. In this Outlook client, that's done by going to **View** &rarr; **Message Source**

<img width="941" height="842" alt="image" src="https://github.com/user-attachments/assets/28a830a9-109a-4b9c-913f-d0693ee95dff" />

Then on line four, you will see the **X-Originating-Ip**. This is your answer.

<img width="638" height="510" alt="image" src="https://github.com/user-attachments/assets/79364678-cee3-4372-ba78-df76536f29ce" />

**Answer**: 209[.]85[.]167[.]226

### Question 4 - From what you can gather, what do you think will be a domain of interest? Defang the domain
This one can either be done by manually looking through the raw email data (not recommended, even though you don't have to search far :smiley:), or by using CyberChef to extract all of the relevant domains. The key thing to note here is the word **domains**, not URLs. Make sure to use the right CyberChef recipe.

1. I selected the **Extract domains** operation. You can search for it then just drag it into the **Recipe** box.

<img width="946" height="844" alt="image" src="https://github.com/user-attachments/assets/96d70516-c774-400e-89fb-6fac667ceeb7" />

2. I copied and pasted the raw email in the **Input** section. This outputs 56 domains, quite a lot.

<img width="472" height="754" alt="{E0C4DF94-B302-4C4C-A229-52DEBF377CF0}" src="https://github.com/user-attachments/assets/9015217f-9474-4c14-b2b2-7be24f494774" />

3. Now it's just a matter of going through them. I skipped all of the commonly known domains to start (yahoo.com, google.com. gmail.com, etc.). The first domain of interest is **etekno[.]xyz**. Scanning briefly through the raw email data, we can see that this is the domain of the senders email. Very much a domain of interest I would say. Next we have **gogolecloud[.]com**. This looks like a potential typo squat of Google Cloud (which is actually cloud.google.com). **DMARC** is also showing as **unknown**, and **SPF** as **none**, meaning nothing verified who this email was claiming to be. All of these in combination point towards classic phishing infrastructure. Next up is **gstatic[.]com**. Doing some Google searching and OSINT lookups we can see that this appears to be a domain owned by Google and is used as a Content Delivery Network (CDN) for serving static web content. <img width="1185" height="695" alt="{8F48AC9F-0B82-4795-BD0E-1A700EB421C5}" src="https://github.com/user-attachments/assets/42c7a769-30f1-441e-b6b0-da2f59ddd2e0" /> Nothing to be concerned about.

I think the first two domains we found are a good start. If we look at the answer, the domain name is 6 characters long (not including the square bracket part of the defanged format). This elimnates **gogolecloud[.]com** as the answer, but **etekno[.]xyz** fits perfectly, and that's our answer.

**Answer**: etekno[.]xyz

### Question 5 - What is the shortened URL? Defang the URL
For this answer, we need to change the CyberChef operation to **Extract URLs**, since it's asking for the shortened **URL**. This is what we see:

<img width="944" height="845" alt="image" src="https://github.com/user-attachments/assets/d2d9ac68-68f3-4411-8cfc-c932e8a8a7fb" />

About a dozen or so URLs, so which is it? We also need to enter it in defanged format. So we can add the **Defang URL** operation to our recipe:

<img width="953" height="843" alt="image" src="https://github.com/user-attachments/assets/99a395a2-2941-41a1-8e0a-438dc3f2b284" />

This shows us all of the URLs in defanged format. Make sure you have all of the defang options selected since the answer requires it. We can kind of identify a shortened URL manually by just looking at the length of it. E.g., `t.co`. Also if you look at the path section of the URL, you can see what looks like a token. When in combination with the URL shortened domain, you can safely assume that the token (`yuxfZm8KPg`) is the identifier for the actual destination of the shortened URL. Now there are a few options, but the answer is 34 characters long, so we can just go by process of elimination. But wait a second, all of the options in our URL output are 35 characters long, what gives? Well it technically is the right answer, and if you submit one of the options as the answer, it will auto correct it to the correct one. The difference is the end of the URL, so then how do we find the exact answer?

From what I've been able to research, apparently the characters `3D` at the end of of the shortened URLs is actually a deconding artifact from email encoding. So `amp=3D1` is actually `amp=1`. If this is applied to our URLs in CyberChef, we get the following (in defanged format):
1. hxxps[://]t[.]co/yuxfZm8KPg?amp=1
2. hxxps[://]t[.]co/yuxfZm8KPg?amp==
3. hxxps[://]t[.]co/yuxfZm8KPg?amp=1

The actual answer is `hxxps[://]t[.]co/yuxfZm8KPg?amp==1` (option two from above with `1` appended at the end). I have a suspicion though that you could paste any one of those though and TryHacKMe will just auto correct the answer, since the important part is likely just everything up until the `?` character. Credit to **HelloFriend** though, in his blog at [TryHackMe | Phishing Analysis Tools](https://medium.com/@mikmrc/tryhackme-phishing-analysis-tools-4164ce6a59bd), he confirms that you can use **PhishTool Community** to find the answer. I'll be honest, I was too lazy to make an account and do that myself (that's for another time), but you can see in the screenshot from the blog that it shows the message URL and you just need to put the domain, path, and query all together to get your answer:

<img width="456" height="229" alt="{DDDF92D9-5342-471A-8768-AC788E8706DE}" src="https://github.com/user-attachments/assets/341b533d-0b5a-4c9b-a45e-7ec51b645875" />

So...

**Answer**: hxxps[://]t[.]co/yuxfZm8KPg?amp==1

**Fun fact**: t.co is the official URL shortening service used by X (Twitter).
