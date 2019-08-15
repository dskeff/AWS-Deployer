# AWS DEPLOYER

Here are the instructions to setup Ansible to deploy multiple Edge2AI envs on AWS. 

## Ansible and AWS CLI installation and Configuration 

Ansible can be a little complicated to install on a local machine, especially if you run Windows. Here I just spin up a tiny VM with Ubuntu 19.04 on GCP.

Install ansible, awscli and required packages. Ensure Python3 is installed.
```
$ sudo apt-get update
$ sudo apt-get install -y python3-pip ansible 
$ ln -s /usr/bin/python3 /usr/bin/python

$ python -V
Python 3.7.3

$ pip3 install awscli boto boto3

$ ansible --version
ansible 2.7.8
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.7.3 (default, Apr  3 2019, 05:39:12) [GCC 8.3.0]
```

At this point, on my VM I need to logout and log back in, but you should be able to reload the PATH by doing
```
. ~/.profile
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
$ ansible-vault create aws-keys.vault
```

Enter the password for the vault, then insert these 2 lines and save.
```
aws_access_key: xxxx
aws_secret_key: xxxxx
```

## Use

Now you're ready to downlowd this repo:
```
$ git clone https://github.com/fabiog1901/AWS-Deployer.git
$ cd AWS-Deployer
```

Update Ansible `hosts` file with a local group. This is required as Ansible always asks for a list of servers to connect to. To create EC2 instances you don't really connect to anything, so we just tell Ansible to connect to localhost...
Edit the file using vi/nano and add these 2 lines:
```
[local]
localhost
```

### Review Ansible Playbook
Now check the `edge2ai.yml` file, particularly the `vars` section at the top. You might want to update some values specific for your AWS environment.

1. **ami** - Make sure the AMI ID hasn't changed. Verify that as follows:
    - Check the Product Code for the Centos 7 image at https://wiki.centos.org/Cloud/AWS
    - Find the latest Centos 7 AMI using awscli:
      ```
      $ aws ec2 describe-images \
          --owners 'aws-marketplace' \
          --region 'us-east-1' \
          --filters 'Name=product-code,Values=aw0evgkw8e5c1q413zgy5pjce' \
          --query 'sort_by(Images, &CreationDate)[-1].[ImageId]' \
          --output 'text'
      ami-02eac2c0129f6376b
      ```
    - Confirm the ami is the same as the ami in the playbook. If not, update with new ami id.

2. **sg** - Security group, ensure you use a security group that is valid for your region.
3. **keypair** - You might want to create a new key for the workshop as you'll have to share the private key with all the students.
4. **region** - easy..
5. **count**: - self-explanatory..
6.  **subnet** - again, ensure the subnet is valid in your region
7. **onwer, enddate, project** - ensure you set the EC2 Tags, specifically ensure the **project** tag is unique (eg: *yourname-todaysdate-edge2ai*) as you will search for that tag when it's time to delete the instances, and you don't want to delete someone else's instances!

### Run the Playbook
At this point you're ready to run the playbook
```
$ ansible-playbook --ask-vault-pass edge2ai.yml
```

You should see at the end of the run a list of all IP addresses, which you can distribute among your students. You also need to include the private key file you created along with it. 
```
PLAY RECAP ***************************************************************************************
100.26.242.72              : ok=1    changed=0    unreachable=0    failed=0   
100.26.145.15              : ok=1    changed=0    unreachable=0    failed=0 
100.26.212.36              : ok=1    changed=0    unreachable=0    failed=0 
localhost                  : ok=6    changed=4    unreachable=0    failed=0   
```


### Destroy the instances
Once finished with your workshop, stop all instances - make sure you update the value of `tag:project` in file `stop_edge2ai.yml` with your tag before running it:
```
$ ansible-playbook --ask-vault-pass stop_edge2ai.yml
```
Alternatively, just terminate the instances from the AWS console.


