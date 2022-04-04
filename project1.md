## WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

This is a step by step process on how I created a LAMP(Linux, Apache, MySql, PHP) stack.

Perequisite:
- An AWS account(free tier would be preferred as you basically pay for nothing to get through to the end of this project).
- A linux Terminal
- Some basic understanding of Linux, AWS EC2, Web Technology stack and VIM editor.

First things before the core Steps include:
- Spinning up an EC2 instance on your AWS account
![EC2 Instance](./images/ScreenShot_4_3_2022_9_45_53_PM.png)
- SSH into your instance via your Linux Terminal -
You first have to change permissions for the private key using the command `sudo chmod 0400 <private-key-name>.pem` 
Next you connect to your instance via this command `ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>`
![shh-instance](./images/ScreenShot_4_3_2022_11_39_43_PM.png)

### STEP 1 — INSTALLING APACHE AND UPDATING THE FIREWALL
