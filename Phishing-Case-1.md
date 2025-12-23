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
