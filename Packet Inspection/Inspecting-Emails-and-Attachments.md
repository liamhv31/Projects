## Inspecting Emails and Attachements

### Task
In this task, we will use Wireshark and move beyond SMTP status codes and responses, and begin analyzing SMTP traffic using the same traffic capture from the [Analyzing SMTP Resonses](https://github.com/liamhv31/Projects/blob/main/Analyzing-SMTP-Responses.md) lab. We will utilize the Internet Message Format (IMF) to examine the inner details of emails, such as sender and recipient fields, content type, and attachments.

Helpful resources:
- [Display Filter Reference: Simple Mail Transfer Protocol](https://www.wireshark.org/docs/dfref/s/smtp.html)
- [What are SMTP codes and how to troubleshoot them](https://www.mailersend.com/blog/smtp-codes)

### Question 1 - How many SMTP packets are available for analysis?
You can find this by using the query: `smtp`.

<img width="959" height="424" alt="image" src="https://github.com/user-attachments/assets/e220ce38-af9f-4cc9-80d3-805b9345a9dc" />

**Answer**: 512

### Question 2 - What is the name of the attachment in packet 270?
To get that specific frame number, you can use `frame.number==270`

<img width="1919" height="412" alt="image" src="https://github.com/user-attachments/assets/b4673c86-59b5-4a17-b4a1-a970a1b67a9d" />

Next, expand the "Simple Mail Transfer Protocol" portion. Scroll down until you see the content part of the email within tha payload

<img width="566" height="182" alt="image" src="https://github.com/user-attachments/assets/69147f67-a718-4068-add5-6bfbc8716c5f" />

**Answer**: document.zip

### Question 3 - According to the message in packet 270, which Host IP address is not responding, making the message undeliverable?
This is also found under the SMTP part of the payload. If you scroll just a bit, you'll find the section talking about why the message was undeliverable, and which host IP it was

<img width="959" height="311" alt="image" src="https://github.com/user-attachments/assets/6b08bd5d-d09c-4276-a6de-372060c8ad14" />

**Answer**: 212.253.25.152

### Question 4 - By filtering for imf, which email client was used to send the message containing the attachment attachment.scr?
As seen in the example below, attachment/file names can be found in the following headers of the email body: `Content-Type` or `Content-Disposition`.

<img width="849" height="367" alt="image" src="https://github.com/user-attachments/assets/2025a2e7-94a6-4956-b4c3-2ef805a04f36" />

The question also wants us to find the answer by first filtering using `imf`. This gives us a few hits, which means we just have to search for the correct packet by analyzing the **Image Message Format** portion. The attachment we're looking for will be found under the **MIME Multipart Media Encapsulation** portion within IMF, since this is what allows our emails to handle diverse content such as attachements. Now we just need to find the right headers to confirm we have the right packet.

<img width="959" height="425" alt="image" src="https://github.com/user-attachments/assets/9ea45c37-1391-434d-be14-7dde56feaac5" />

Nice! Now the question is asking which email client was used to send this message. The header we're looking for under IMF is `X-Mailer`, and our answer is there.

<img width="959" height="422" alt="image" src="https://github.com/user-attachments/assets/d68782be-cc00-43fe-90fa-128e96143a8a" />

**Answer**: Microsoft Outlook Express 6.00.2600.0000

A quick tip: If you want to find what you're looking for faster, you can just search for the attachment using regex by way of the `matches` filter. Much faster especially if there are hundreds of packets.

<img width="1919" height="842" alt="image" src="https://github.com/user-attachments/assets/28b6ae9b-6917-48fc-9ac2-3f5d0ec130ea" />

### Question 5 - Which type of encoding is used for this potentially malicious attachment?
The header we're looking for is `Content-Transfer-Encoding` and can be found in the same part of the packet as our headers containing the attachment name.

<img width="959" height="419" alt="image" src="https://github.com/user-attachments/assets/c3da7233-16a8-43cf-bb37-5a0a3b603483" />

**Answer**: base64
