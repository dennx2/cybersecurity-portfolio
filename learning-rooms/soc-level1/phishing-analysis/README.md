# Phishing Analysis

## Practice Questions

**Q: In the attached virtual machine, view the information in email2.txt and reconstruct the PDF using the base64 data. What is the text within the PDF?**

Check the line count.

![phishing-q1-1](./screenshots/phishing-q1-1.png)

Check the file content.

![phishing-q1-2](./screenshots/phishing-q1-2.png)
...
![phishing-q1-3](./screenshots/phishing-q1-3.png)

Grab the base64 part to parse from. The part starts from line 6 and ends on line 624.

![phishing-q1-4](./screenshots/phishing-q1-4.png)

Open the reconstructed PDF to view the flag.

![phishing-q1-5](./screenshots/phishing-q1-5.png)

Answer: THM{not disclosing here}

**Q: What is the website for the - CLICK HERE URL in a defanged format? (e.g. https://website.thm)**

Grab the link

![phishing-q2-1](./screenshots/phishing-q2-1.png)

Use **Cyber Chef** (https://gchq.github.io/CyberChef/) to obtain the defanged URL.

![phishing-q2-2](./screenshots/phishing-q2-2.png)

Answer: hxxp[://]t[.]teckbe[.]com
