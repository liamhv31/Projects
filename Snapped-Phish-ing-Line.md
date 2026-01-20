## Snapped Phish-ing Line

### Overview
As an IT department personnel of SwiftSpend Financial, one of your responsibilities is to support your fellow employees with their technical concerns. While everything seemed ordinary and mundane, this gradually changed when several employees from various departments started reporting an unusual email they had received. Unfortunately, some had already submitted their credentials and could no longer log in.

### Task
You now proceeded to investigate what is going on by:
1. Analysing the email samples provided by your colleagues
2. Analysing the phishing URL(s) by browsing it using Firefox
3. Retrieving the phishing kit used by the adversary
4. Using CTI-related tooling to gather more information about the adversary
5. Analysing the phishing kit to gather more information about the adversary

### Question 1 - Who is the individual who received an email attachment containing a PDF?
There are five different phishing email samples. I opened up each one to look at the attachment types. As mentioned in the previous lab, almost all email clients will show the attachment either at the top or bottom of the email. You can always find the attached content by looking at the `Content-Type` and `Content-Disposition` fields in the email source. The email with the PDF is the one titled **Quote for Services Rendered: processed on June 29, 2020, 10:01:32 AM**

<img width="885" height="823" alt="image" src="https://github.com/user-attachments/assets/1604170f-a341-4a11-b059-75110d673edf" />

**Answer**: William McClean

### Question 2 - What email address was used by the adversary to send the phishing emails?
You should be able to see this at the top of your email client. It is the same email for every sample since this is a single campaign.

<img width="885" height="822" alt="image" src="https://github.com/user-attachments/assets/fcff7493-83e2-4a59-9cd8-140a30e0bcfb" />

**Answer**: Accounts.Payable@groupmarketingonline.icu

### Question 3 - What is the redirection URL to the phishing page for the individual Zoe Duncan? (defanged format)
Zoe Duncan's email is the one titled **Group Marketing Online Direct Credit Advice**. If you look at the HTML attachment, you can find the answer. I saved it to the **Downloads** folder and then used `cat` to dump the contents to the command line. It's not a big file so need to `grep` anything.

<img width="879" height="250" alt="image" src="https://github.com/user-attachments/assets/2614c7e0-7f51-425b-be32-aca66db871b6" />

Then we just need to defang it using **CyberChef**. Make sure you select all of the **Defang URL** options (escape dots, escape http, and escape ://)

<img width="884" height="663" alt="image" src="https://github.com/user-attachments/assets/b2fdf2d9-078f-4ddd-9338-e2d6c15db66c" />

**Answer**: hxxp[://]kennaroads[.]buzz/data/Update365/office365/40e7baa2f826a57fcf04e5202526f8bd/?email=zoe[.]duncan@swiftspend[.]finance&error

### Question 4 - What is the URL to the .zip archive of the phishing kit? (defanged format)

### Question 5 - What is the SHA256 hash of the phishing kit archive?

### Question 6 - When was the phishing kit archive first submitted? (format: YYYY-MM-DD HH:MM:SS UTC)

### Question 7 - When was the SSL certificate the phishing domain used to host the phishing kit archive first logged? (format: YYYY-MM-DD)

### Question 8 - What was the email address of the user who submitted their password twice?

### Question 9 - What was the email address used by the adversary to collect compromised credentials?

### Question 10 - The adversary used other email addresses in the obtained phishing kit. What is the email address that ends in "@gmail.com"?

### Question 11 - What is the hidden flag?
