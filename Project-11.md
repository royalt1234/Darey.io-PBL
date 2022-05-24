# ANSIBLE – AUTOMATE PROJECT 7 TO 10

### STEP 1 - INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

1. Using the same server used for Jenkins as our ansible server, update the name to Jenkins-Ansible.

2. Created a new repo called ansible-config-mgt.

![](./images/p11/ScreenShot_5_7_2022_3_22_14_PM.png)

3. Install Ansible
```
sudo apt update

sudo apt install ansible
```

![](./images/p11/ScreenShot_5_7_2022_3_36_58_PM.png)

4. Setup Jenkins project to automatically build the ansible-config-mgt repo once there is an update to master branch

![](/images/p11/ScreenShot_5_7_2022_3_56_55_PM.png)

![](/images/p11/ScreenShot_5_7_2022_3_57_52_PM.png)

![](/images/p11/ScreenShot_5_7_2022_4_19_50_PM.png)


### STEP 2 - SETUP PROJECT ON VISUAL STUDIO CODE 

Install Visual Studio Code and configure it to connect to your newly created GitHub repository.

Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance
```
git clone <ansible-config-mgt repo link>
```

![](/images/p11/ScreenShot_5_7_2022_5_39_09_PM.png)

### STEP 3 - BEGIN ANSIBLE DEVELOPMENT

1. Create a new feature branch to develop the ansible playbook

2. Checkout the feature branch
3. Create a **playbook** directory - holds the ansible tasks and operations to be performed.
4. Create an **inventory** directory - holds the hosts file.
5. Create a common.yml file inside the playbook directory. This contains some of tasks to be run on all the hosts.
6. Create hosts file(dev.yml, staging.yml, uat.yml and prod.yml) inside the inventory directory. These files contains the list of hosts to be managed.

### STEP 4 – SETUP ANSIBLE INVENTORY

1. Update the inventory/dev.yml file with the following content
```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address>  ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```
2. We need to import our .pem file into the ansible server in order to execute the anisble playbook.<br>

We can acheive this by running the following command to add the .pem file to SSH Agent
```
eval `ssh-agent -s` #starts the ssh agent if its not running.
ssh-add <path-to-private-key> # adds the .pem to the ssh Agent
ssh-add -l # lists the keys in the ssh agent
```
![](/images/p11/ScreenShot_5_7_2022_4_54_00_PM.png)
3. SSH into the jenkins server;
```
ssh -A ubuntu@public-ip
```

![](/images/p11/ScreenShot_5_7_2022_5_06_54_PM.png)

### STEP 5 – CREATE ANSIBLE PLAYBOOK
1. Update the playbook/common.yml file with the following content:
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

    - name: create new directory
      file:
        path: /home/ec2-user/wales
        state: directory    

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest

    - name: create new directory
      file:
        path: /home/ubuntu/wales
        state: directory     
```
2. Save the project and push the branch to GitHub. Also create a PR to merge the branch to master.

![](/images/p11/ScreenShot_5_7_2022_5_22_58_PM.png)

3. Automatic build is triggered on jenkins and the files are copied to the jenkins server at ansible-playbook -i /var/lib/jenkins/jobs/ansible-config-mgt/builds/<build number>/archive

![](/images/p11/ScreenShot_5_7_2022_5_24_54_PM.png)

4. Run the ansible playbook using the following command:
```
ansible-playbook -i /var/lib/jenkins/jobs/ansible-config-mgt/builds/<build number>/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible-config-mgt/builds/<build number>/archive/playbook/common.yml --forks 1
```
![](/images/p11/ScreenShot_5_8_2022_12_59_49_AM.png)
![](/images/p11/ScreenShot_5_8_2022_1_00_26_AM.png)

5. Check the server to confirm the changes are reflected. Check if wireshark is installed and the directory is created.
```
wireshark --versions
```
![](/images/p11/ScreenShot_5_8_2022_1_04_35_AM.png)