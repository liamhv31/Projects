### Scenario
You are a Level 1 SOC Analyst. Several suspicious emails have been forwarded to you from other coworkers. You must obtain details from each email for your team to implement the appropriate rules to prevent colleagues from receiving additional spam/phishing emails.

### Task
Investigate the analysis and answer the questions below.

**Link**: https://app.any.run/tasks/82d8adc9-38a0-4f0e-a160-48a5e09a6e83

### Question 1 - What is this analysis classified as?
This answer is found in the same place as the previous lab - in the upper right hand corner of the analysis:


<img width="1917" height="910" alt="image" src="https://github.com/user-attachments/assets/6fa76498-29c5-45d6-904d-0f068b3cadf7" />


**Answer**: Malicious activity

### Question 2 - What is the name of the Excel file?
The name of the excel file is just below the classification:


<img width="1917" height="910" alt="image" src="https://github.com/user-attachments/assets/31928e80-701e-46c6-b9ab-b696e2149578" />


**Answer**: CBJ200620039539.xlsx

### Question 3 - What is the SHA 256 hash for the file?
You can find this answer by clicking on the name of the Excel file:


<img width="1920" height="910" alt="image" src="https://github.com/user-attachments/assets/d6157791-cbba-4ed6-bae2-55fb3c33ef4d" />


**Answer**: 5f94a66e0ce78d17afc2dd27fc17b44b3ffc13ac5f42d3ad6a5dcfb36715f3eb

### Question 4 - What domains are listed as malicious? Defang the URLs & submit answers in alphabetical order. (answer: URL1,URL2,URL3)
1. Open the **Text report**.


<img width="1916" height="902" alt="image" src="https://github.com/user-attachments/assets/f5ead52d-9e1c-4e44-a04b-e20ff3fe995e" />


2. Scroll down to **Connections** under **Network activity** and you'll see the two malicious domains and one suspicious one. To defang, just wrap the period (.) with square brackets ([]).


<img width="1142" height="824" alt="image" src="https://github.com/user-attachments/assets/76c8e813-7458-4fa3-84ca-f4a6b1143eb9" />


**Answer**: biz9holdings[.]com,findresults[.]site,ww38[.]findresults[.]site

### Question 5 - What IP addresses are listed as malicious? Defang the IP addresses & submit answers from lowest to highest. (answer: IP1,IP2,IP3)
This can be found in the **Text report** as well. You'll see the IPs in all of the **Network activity** tables.


<img width="1145" height="826" alt="image" src="https://github.com/user-attachments/assets/c2fed260-6889-4fdf-b9ac-b89adec7dd16" />


**Answer**: 75[.]2[.]11[.]242,103[.]224[.]182[.]251,204[.]11[.]56[.]48

### Question 6 - What vulnerability does this malicious attachment attempt to exploit?
This can also be found in the **Text report**, either at the top under the **General Info** section listed as a tag, or in the **Behavior activities** section under the **Malicious** behaviors.


<img width="1142" height="747" alt="image" src="https://github.com/user-attachments/assets/58419271-495e-44b1-a465-4d611f25b8ee" />
<img width="1144" height="375" alt="image" src="https://github.com/user-attachments/assets/b29158f5-a753-402d-94fa-1432b8689fca" />


**Answer**: CVE-2017-11882
