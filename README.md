# Threat Detection with GuardDuty

![Image](http://learn.nextwork.org/calm_chartreuse_playful_saci_perere/uploads/aws-security-guardduty_v1w2x3y4)

### Tools and concepts

The services I used were CloudFormation, GuardDuty, CloudShell, and Systems Manager (Session Manager). 

Key concepts explored include; 
1. Automated deployment using CloudFormation
2. Troubleshooting deployment issues using session manager
3. Attack simulation on a vulnerable web server ; SQL injection, command injection anddata exfiltration
4. Analyzing GuadDuty findings

### Project reflection

This project took me approximately 6 hours 😅 (it shouldn't have - but there were some errors I had to troubleshoot when deployment with CloudFormation failed). The most challenging part was troubleshooting the errors (why some resources were failing to create.)

It was most rewarding to actually find the errors, solve them and jump on to the next error. Still an error, but closer to the solution😅.

Really, the most rewarding part was being able to simulate an attack and see the findings on GuardDuty.

I did this project to demonstrate the use of GuardDuty to detect malicious activities in your AWS environment. It was a win.

---

## Project Setup


To set up for this project, I deployed a CloudFormation template that launches:

- A VPC
- EC2 instance (the webserver) within the VPC
- GuardDuty for monitoring
- CloudFront
- An S3 bucket to store dummy sensitive data. This is the file we will simulate our attack on.
 
 The three main components are: 
1. The web app hosted on the EC2 instance
2. Networking components set up through the VPC
3. CloudFront Distribution to speed up delivery of the web app.


The web app deployed is called OWASP JuiceBox. To practice my GuardDuty skills, I will hack the juicebox app and analyze findings on GuardDuty.

GuardDuty is the "security guard" for AWS resources. It detects unusual activities in your AWS account. 

In this project, I will simulate an attack on the webserver and use GuardDuty to detect and investigate the attack.

![Image](http://learn.nextwork.org/calm_chartreuse_playful_saci_perere/uploads/aws-security-guardduty_n1o2p3q4)

---

## SQL Injection

The first attack I performed on the web app is SQL injection, which means inserting malicious SQL code into the form, for it to be executed by the application's database.  SQL injection is a security risk because' an attacker can use it to bypass authentication, gain unauthroized access to resources, or manipulate the database.

My SQL injection attack involved using an SQL statement that will always return true 'or 1=1;--. This means that the database will always take the email as correct even if it does not exist in the db. The -- ignores any other error message, hence enabling me to bypass authentication.

![Image](http://learn.nextwork.org/calm_chartreuse_playful_saci_perere/uploads/aws-security-guardduty_h1i2j3k4)

---

## Command Injection

Next, I used command injection, which is an exploit that tricks the server into executing a command entered by the enduser (attacker). The Juice Shop web app is vulnerable to this because it does not sanitize user input - it accepts anything the user enters including javascript code.

To run commandinjection, I ran a javascript script on the username field. The script will fetch the webserver's metadata, including IAM keys, and save it in a publicly accessible file.

![Image](http://learn.nextwork.org/calm_chartreuse_playful_saci_perere/uploads/aws-security-guardduty_t3u4v5w6)

---

## Attack Verification

To verify the attack's success, I navigated to the public url where the stolen credentials were stored. The credentials page showed me the webservers accessID and secret key.  These credentials are usually used for programmatic access.This means that they can be used by the web server to access other AWS resources. With these credentials, I can access and interact with other AWS resources as the web server would.

![Image](http://learn.nextwork.org/calm_chartreuse_playful_saci_perere/uploads/aws-security-guardduty_x7y8z9a0)

---

## Using CloudShell for Advanced Attacks

I continue the attack in CloudShell because I want to get access into the target AWS environment.

In CloudShell, I used wget to download the credentials.json file into the aws environment. Next, I viewed the downloaded file using cat command and jq to display the json data in a readable form

I then set up a profile called "stolen" to use with the stolen access credentials. I had to create a new profile because I want to simulate an attack scenario. Activities by this profile should be picked by GuardDuty.

![Image](http://learn.nextwork.org/calm_chartreuse_playful_saci_perere/uploads/aws-security-guardduty_j9k0l1m2)

---

## GuardDuty's Findings

After performing the attack, GuardDuty reported a finding within 2 minutes. The findings in GuardDuty are notifications that show something suspicious has happened in the AWS environment.

GuardDuty's finding was called UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.InsideAWS. This means that iour EC2 instance credentials were used for unauthorized access. Anomaly detection was used to detect unusual activity with our EC2 credentials.

GuardDuty's detailed finding reported that there was an API call that retrieved an object from an S3 bucket. A remote IP  was used to access the S3 bucket.

![Image](http://learn.nextwork.org/calm_chartreuse_playful_saci_perere/uploads/aws-security-guardduty_v1w2x3y4)

## Conclusion

GuardDuty is critical because it closes the gap between having cloud resources and actually knowing when they are being abused. In complex AWS environments, breaches rarely start with loud alarms. They begin with small, abnormal actions that look legitimate on the surface. GuardDuty continuously analyzes account behavior, API calls, network traffic, and credential usage to surface these hidden threats early. Without it, organizations often only discover incidents after data has been accessed or exfiltrated. Used properly, GuardDuty shifts security from reactive firefighting to proactive detection, giving teams the visibility they need to respond before minor misconfigurations turn into major breaches.
