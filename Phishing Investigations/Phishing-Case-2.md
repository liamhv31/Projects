### Scenario
You are a Level 1 SOC Analyst. Several suspicious emails have been forwarded to you from other coworkers. You must obtain details from each email for your team to implement the appropriate rules to prevent colleagues from receiving additional spam/phishing emails.

### Task
Investigate the analysis and answer the questions below.

**Link**: https://app.any.run/tasks/8bfd4c58-ec0d-4371-bfeb-52a334b69f59

### Question 1 - What does AnyRun classify this email as?
All of the questions in this lab involve using the AnyRun analysis form the above link. To see how AnyRun classified the activity, look at the top right of the analysis page:

<img width="1920" height="910" alt="image" src="https://github.com/user-attachments/assets/da702e39-e886-455d-8d0c-a1646a48480e" />

**Answer**: Suspicious activity

### Question 2 - What is the name of the PDF file?
The answer to this question can be found just below the classification:

<img width="1919" height="909" alt="image" src="https://github.com/user-attachments/assets/7bd7ca62-2c05-4568-adf3-cfadb124a509" />

**Answer**: Payment-updateid.pdf

### Question 3 - What is the SHA 256 hash for the PDF file?
You can only see the MD5 hash below the PDF file by default. To see more details (including the SHA256 hash), click on the name of the fileA

<img width="1919" height="909" alt="image" src="https://github.com/user-attachments/assets/08e47808-fd1c-4044-86ec-d6c997eb78f3" />

**Answer**: cc6f1a04b10bcb168aeec8d870b97bd7c20fc161e8310b5bce1af8ed420e2c24

### Question 4 - What two IP addresses are classified as malicious? Defang the IP addresses. (answer: IP_ADDR,IP_ADDR)
This one took a bit to find but after clicking around, you can find the malcious IPs by viewing the **Text report**:


<img width="1914" height="908" alt="image" src="https://github.com/user-attachments/assets/6d6191bd-4a2c-433d-a0c1-d7104cf3541e" />


Scroll down to the **Connections** table under **Network activity** and you will see the first malicious IP on page one, and the second on page two.


<img width="1095" height="531" alt="image" src="https://github.com/user-attachments/assets/cbf6122b-8ad6-47eb-81ef-997f16b47207" />


<img width="1088" height="510" alt="image" src="https://github.com/user-attachments/assets/db36d94f-9b29-4fca-a018-ff826a340e5e" />


**Answer**: 2[.]16[.]107[.]24,2[.]16[.]107[.]83

### Question 5 - What Windows process was flagged as Potentially Bad Traffic?
You can find the answer to this one by clicking on the **Network Threats** tab


<img width="1916" height="909" alt="image" src="https://github.com/user-attachments/assets/0767ec4e-8dc3-40c6-90d1-5a04b300b3c2" />


**Answer**: svchost.exe
