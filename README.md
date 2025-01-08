Ansible Realtime Project
Ansible project for provisioning and configuration management

Task 1
Create three(3) EC2 instances on AWS using Ansible loops
•	2 Instances with Ubuntu Distribution
•	1 Instance with Centos/Linux Distribution
Hint: Use connection: local on Ansible Control node.

Provisioning:
Control node is laptop where ansible is installed, 
Provisioning Resources on AWS, AWS is IAAS platform, we cannot connect Ansible control node with AWS using SSH, because AWS is not server , it is IAAS / PAAS platform , so we need to use ansible AWS collection, in ansible AWS Collection we have a module for EC2, this module will be run on control node, we are doing provisioning , u cannot SSH to aws , so we will use connection type as local and we will run the module on ansible control node, this module which is python module using boto3 will connect to AWS API and it will provision ec2 instance 

Task 2
Set up passwordless authentication between Ansible control node and newly created instances.
Prerequisites for task 3: use ssh based passwordless authentication using pem file or password

Task 3
Automate the shutdown of Ubuntu Instances only using Ansible Conditionals
Hint: Use when condition on ansible gather facts
Configuration management: to automate shutdown the ec2 instance , write tasks in ansible playbook or ansible role to shutdown ec2 instance that is created
Task1: 
We have to use IAM user if ansible has to talk to API of aws , we need authentication to connect to aws, b3 modules need to talk to aws api for that we will create IAM user and provide access token and secret key `
AWS -> IAM -> create user -> give permission ec2 full access
User->go to security credentials-> create access key-> select application running outside aws-> create access key

Project terminal :
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ pip install boto3
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ansible-galaxy collection install amazon.aws
protect with password base 64 encoded
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ openssl rand -base64 2048 > vault.pass
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ls
vault.pass
within ansible vault create file pass.yaml or password.yaml and protecting data in  password.yaml using encoded base 64 file that I have created

reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ls -l vault.pass
-rwxrwxrwx 1 reka reka 2775 Nov 25 16:02 vault.pass
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ mv vault.pass ~/vault.pass
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ chmod 600 ~/vault.pass
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ls -l ~/vault.pass
-rw------- 1 reka reka 2775 Nov 25 16:02 /home/reka/vault.pass
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ansible-vault create group_vars/all/pass.yml --vault-password-file ~/vault.pass
ansible  access key : AKISHVJDGFLG7E4CCCNW
ansible secret user : a81Wa232qvDJyrIxADbcvEYyGqLfq+nLjci76ibQ
[WARNING]: group_vars/all does not exist, creating...

https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_instance_module.html#ansible-collections-amazon-aws-ec2-instance-module
Create ec2_create.yaml and write tasks
---
- hosts: localhost
  connection: local

  tasks:
  - name: create EC2 instances
    amazon.aws.ec2_instance:
      name: "{{ item.name }}"
      key_name: "ansible-project"
      instance_type: t2.micro
      security_group: sg-0cbb27c5074e9b6c9  # Ensure this group belongs to the instance's VPC
      region: us-east-1
      aws_access_key: "{{ec2_access_key}}"  # From vault as defined
      aws_secret_key: "{{ec2_secret_key}}"  # From vault as defined      
      network_interfaces:
        - device_index: 0
          assign_public_ip: true
      image_id: "{{ item.image }}"
      # tags:
      # environment: "{{ item.name }}"
    loop:
      - { image: "ami-0453ec754f44f9a4a", name: "manage-node-1" } # Update AMI ID according 
      - { image: "ami-0866a3c8686eaeeba", name: "manage-node-2" } # to your account
      - { image: "ami-0866a3c8686eaeeba", name: "manage-node-3" }

Loop  to execute tasks repeatedly 
Now if u run the below cmd , only 2 instances will be created 1 for ubuntu and 1 for linux, because ansible is  idempotent , instance once created will not be created again, it will ignore the task, so since 2 task for ubuntu it will not consider 2 nd ubuntu instance , it will ignore the task.
ansible-playbook ec2_create.yaml --vault-password-file ~/vault.pass
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ls
ec2_create.yaml  group_vars
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ansible-playbook ec2_create.yaml --vault-password-file ~/vault.pass
PLAY [localhost] *****************************************************
TASK [Gathering Facts] *****************************************************
ok: [localhost]
TASK [create EC2 instances] *****************************************************
ok: [localhost] => (item={'image': 'ami-0453ec754f44f9a4a', 'name': 'manage-node-1'})
ok: [localhost] => (item={'image': 'ami-0866a3c8686eaeeba', 'name': 'manage-node-2'})
ok: [localhost] => (item={'image': 'ami-0866a3c8686eaeeba', 'name': 'manage-node-3'})
[DEPRECATION WARNING]: The network parameter has been deprecated, please use network_interfaces and/or network_interfaces_ids
instead. This feature will be removed from amazon.aws in a release after 2026-12-01. Deprecation warnings can be disabled by setting
 deprecation_warnings=False in ansible.cfg.
PLAY RECAP *****************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

Task2: 
-f will force copying of pem file
ssh-copy-id -f "-o IdentityFile ansible12.pem" ec2-user@ 3.239.230.123

reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ls -l ansible12.pem
-rwxrwxrwx 1 reka reka 1674 Nov 25 18:12 ansible12.pem
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ chmod 400 ansible12.pem
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ls -l ansible12.pem
-r-xr-xr-x 1 reka reka 1674 Nov 25 18:12 ansible12.pem
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ eval "$(ssh-agent -s)"
Agent pid 8687
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ssh-add ~/.ssh/ansible12.pem
Identity added: /home/reka/.ssh/ansible12.pem (/home/reka/.ssh/ansible12.pem)
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ls -l ~/.ssh/ansible12.pem
-rw------- 1 reka reka 1674 Nov 12 12:48 /home/reka/.ssh/ansible12.pem 
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ssh-copy-id -f "-o IdentityFile ~/.ssh/ansible12.pem" ec2-user@3.239.230.123
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/reka/.ssh/id_rsa.pub"

reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ssh-copy-id -f "-o IdentityFile ~/.ssh/ansible-project.pem" ec2-user@98.80.213.18
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/reka/.ssh/id_rsa.pub"
The authenticity of host '98.80.213.18 (98.80.213.18)' can't be established.
ED25519 key fingerprint is SHA256:uQL/DLjuxdXJbqw+VkLGfVvmrITO1fwvXkvunX+ELII.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Number of key(s) added: 1
Now try logging into the machine, with:   "ssh -o ' IdentityFile ~/.ssh/ansible-project.pem' 'ec2-user@98.80.213.18'"
and check to make sure that only the key(s) you wanted were added.
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ssh ec2-user@98.80.213.18
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-172-31-13-29 ~]$ exit
Similarly for other 2 ubuntu user 
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ssh-copy-id -f "-o IdentityFile ~/.ssh/ansible-project.pem" ubuntu@3.219.218.128
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ssh -o ' IdentityFile ~/.ssh/ansible-project.pem' 'ubuntu@3.219.218.128'
ubuntu@ip-172-31-1-32:~$

Task 3: Automate shutdown only ubuntu instance 
1.	Inventory file to connect to ec2 instance and run command
[all]
ec2-user@98.80.213.18
ubuntu@34.206.3.56
ubuntu@3.219.218.128
# u can group as ubuntu servers 

2.	Create ec2_stop.yaml 
---
- hosts: all      # u can group as ubuntu server and mention ubuntu in place of all , if u put all , use when condition to select only ubuntu
  become: true   # because it is administrative task , where I need elevated privileges, when become true to have permission to run as elevated permission and not with ec2 user permission
ansible.builtin.command: /sbin/shutdown -t now
# -t to shutdown immediately
Using ssh connect to ec2 instance and cmd gets executed , it will run on all 3 instances, ansible has when condition like if condition
  tasks:
    - name: shutdown ubuntu instances only
      ansible.builtin.command: /sbin/shutdown -t now
      when:
       ansible_facts['os_family'] == "Debian"

# if this condition is matched  ansible_facts['os.family'] == "Debian" it will execute ansible.builtin.command: /sbin/shutdown -t now
ansible_facts is coming from gathering facts, it collects all info from target machines which is manage nodes 

reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ansible-playbook -i inventory.ini ec2_stop.yaml
PLAY [all] *****************************************************
ERROR! Attempting to decrypt but no vault secrets found
failing because it is expected key
reka@Reka:/mnt/c/Users/nreka/vscodedevops/aws_provisioning_configuartation$ ansible-playbook -i inventory.ini ec2_stop.yaml --vault-password-file ~/vault.pass
TASK [shutdown ubuntu instances only] *****************************************************
skipping: [ec2-user@98.80.213.18]
changed: [ubuntu@3.219.218.128]
changed: [ubuntu@34.206.3.56]

PLAY RECAP *****************************************************
ec2-user@98.80.213.18      : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
ubuntu@3.219.218.128       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu@34.206.3.56         : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

U can see in aws console the ubuntu instance is shutdown , same for linux , for linux use RedHat instead of Debian in ec2_stop.yaml





