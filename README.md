# AWS DEPLOYER

Here are the instructions to setup Ansible to deploy multiple Edge2AI envs on AWS. 

## Ansible and AWS CLI installation and Configuration 

Ansible can be a little complicated to install on a local machine, especially if you run Windows. Here I just spin up a tiny VM with Ubuntu 19.04 on GCP.

Install ansible, awscli and required pacakges. Note that the VM has Python 3 isntalled already, and ansible runs on Python 3 too.
```
$ python3 -V
Python 3.7.3

$ apt-get update
$ apt-get install -y python3-pip ansible
$ pip3 install awscli boto3
$ ansible --version
ansible 2.7.8
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.7.3 (default, Apr  3 2019, 05:39:12) [GCC 8.3.0]
```

Configure AWS CLI with the access and secred access keys
```
$ aws configure
AWS Access Key ID [None]: xxxx
AWS Secret Access Key [None]: xxxxx
Default region name [None]: 
Default output format [None]: 
```

Create an Ansible Vault to secure your AWS keys for Ansible usage. 
```
$ ansible-vault create ansible_vault_for_aws_keys.yml
```

Enter the password for the vault, then insert these 2 lines and save.
```
aws_access_key: xxxx
aws_secret_key: xxxxx
```

Now you're ready to downlowd this repo:
```
$ git clone https://github.com/fabiog1901/AWS-Deployer.git
$ cd AWS-Deployer















Check the Product Code for CentOS 7 on https://wiki.centos.org/Cloud/AWS

Install aws cli and configure it with your aws access and secret access key


Then find the ami using awscli:

$ aws ec2 describe-images \
    --owners 'aws-marketplace' \
    --filters 'Name=product-code,Values=aw0evgkw8e5c1q413zgy5pjce' \
    --query 'sort_by(Images, &CreationDate)[-1].[ImageId]' \
    --output 'text'



Confirm the ami you get is the same as the ami in the playbook.

Then use Ansible to create the images

Setup ansible env


