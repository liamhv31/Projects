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
