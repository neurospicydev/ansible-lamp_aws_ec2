# Automated LAMP Stack Deployment with Ansible on AWS

This project demonstrates the use of Ansible to automate the deployment of a LAMP (Linux, Apache, MySQL, PHP) stack on AWS EC2 instances. The project covers setting up the necessary AWS infrastructure, writing Ansible playbooks, and deploying a fully functional web server.

## Table of Contents

2. [Technologies Used](#technologies-used)
4. [Prerequisites](#prerequisites)
5. [Setup](#setup)
6. [Usage](#usage)
7. [Verification](#verification)
8. [Contributing](#contributing)
9. [License](#license)


## Technologies Used

- **Ansible**: Automation tool for configuration management.
- **AWS EC2**: Amazon Web Services Elastic Compute Cloud for scalable computing capacity.
- **AWS IAM**: Identity and Access Management for secure access control.
- **Security Groups**: Virtual firewall for controlling inbound and outbound traffic.
- **Bash**: Unix shell and scripting language.

## Prerequisites

- An AWS account
- AWS CLI installed and configured
- Ansible installed on your local machine
- An SSH key pair for accessing your EC2 instances

## Setup

### 1. Set Up Your AWS Environment

1. **Create an IAM User**:
   - Go to the IAM service in the AWS Management Console.
   - Create a new user with programmatic access and attach the following policies: `AmazonEC2FullAccess`, `AmazonVPCFullAccess`, `IAMFullAccess`.
   - Download the user's access key and secret key.

2. **Install and Configure AWS CLI**:
   - Follow the installation instructions [here](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
   - Configure the AWS CLI with `aws configure` using the IAM user's credentials.

### 2. Set Up Ansible Environment

1. **Install Ansible**:
   - On Ubuntu: `sudo apt update && sudo apt install ansible`
   - On macOS: `brew install ansible`
   - On Windows: Use WSL or a virtual machine with Linux.

2. **Configure EC2 Key Pair**:
   - In the AWS Management Console, go to the EC2 service.
   - Create a new key pair and download the `.pem` file.
   - Change the permission of the key file: `chmod 400 your-key.pem`.

### 3. Create Security Groups

**Create a Security Group**:
   - Go to the EC2 Dashboard in the AWS Management Console.
   - Create a new security group with rules allowing SSH (port 22) and HTTP (port 80) access, with your IP as source.

## Usage

### 1. Write Ansible Playbooks 

**Create Project Directory**:
   
   ```sh
   mkdir ansible-aws-lamp && cd ansible-aws-lamp
   ```
### 2. Create Inventory File

**Create a file named `hosts.ini`**:

   ```ini
   [webservers]
   your-ec2-public-ip
   ```
### 3. Create a Playbook for LAMP stack in a YAML File

**Create a file named `lamp_playbook.yaml`**:

   ```
- name: Install and configure LAMP stack
  hosts: webservers
  become: yes
  vars:
    mysql_root_password: 'yourpassword'  # Replace 'yourpassword' with a secure password
  tasks:
    - name: Update and upgrade apt packages
      apt:
        update_cache: yes
        upgrade: dist

    - name: Install Apache
      apt:
        name: apache2
        state: present

    - name: Start and enable Apache service
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Preconfigure MySQL password
      debconf:
        name: 'mysql-server'
        question: 'mysql-server/root_password'
        value: "{{ mysql_root_password }}"
        vtype: 'password'

    - name: Preconfigure MySQL password again
      debconf:
        name: 'mysql-server'
        question: 'mysql-server/root_password_again'
        value: "{{ mysql_root_password }}"
        vtype: 'password'

    - name: Install MySQL
      apt:
        name: mysql-server
        state: present

    - name: Start and enable MySQL service
      service:
        name: mysql
        state: started
        enabled: yes

    - name: Install PHP
      apt:
        name: php
        state: present

    - name: Install PHP MySQL module
      apt:
        name: php-mysql
        state: present

    - name: Create a simple PHP info file
      copy:
        dest: /var/www/html/info.php
        content: "<?php phpinfo(); ?>"

```

### 4. Create User Data Script
**Create file named `user.data.sh`**:
   ```
   #!/bin/bash
   apt-get update
   apt-get install -y apache2
   systemctl start apache2
   systemctl enable apache2
   echo "<h1>Welcome to your web server</h1>" > /var/www/html/index.html
  ```
## 2. Launch AWS EC2 Instance

**Use the AWS Management Console or AWS CLI to launch an instance with the following configuration**:

- **AMI**: Amazon Linux 2023 AMI
- **Instance Type**: t2.micro (free-tier eligible)
- **Key Pair**: Your created key pair
- **Security Group**: Your created security group
- **User Data**: Use the `user_data.sh` script

## 3. Run Ansible Playbook

**Run the Playbook**:

```
 ansible-playbook -i hosts.ini --private-key your-key.pem lamp_playbook.yml
```
## Verification

### Access Your Web Server:
Open a browser and go to `http://your-ec2-public-ip` to see the Apache welcome page.

## Contributing
Contributions are welcome! Please fork this repository and submit a pull request with your changes.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
