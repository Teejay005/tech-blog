## Setup AWS IAM USERS and permissions using ansible and AWS-CLI

### Introduction

It is no secret that managing users and permissions to any software resource could be a daunting one. There are so many factors to be considered. Despite best efforts in the past, so many people still get it wrong. The things that could go wrong include but not limited to assigning inappropriate permissions to a user, assigning insufficient privilege to a user.

Actually AWS [advises](http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) that it is better to assign insufficient privilege to a user rather than over enabling a user.
>It's more secure to start with a minimum set of permissions and grant additional permissions as necessary, rather than starting with permissions that are too lenient and then trying to tighten them later.

Today, we'll go through how to create AWS group, users and assign permissions to the users/group created using AWS-CLI with ansible module for AWS.

### Requirements

To run this exercise, the user needs to install the following:

1. [python](https://www.python.org/)

2. [pip](https://pypi.python.org/pypi/pip)

3. [boto](https://pypi.python.org/pypi/boto/)

4. [ansible](https://www.ansible.com/)

5. [awscli](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)

It is important for the user to have an account with [aws](https://aws.amazon.com).

### Configure AWS-CLI

Run the command ```aws configure``` from your terminal. This should prompt you to supply your access key id,secret access key and the default region.
```

 AWS Access Key ID [None]: xxxxxxxxxxxxxxx
 AWS Secret Access Key [None]: xxxxxxxxxxxxxxxxxxxxxxx
 Default region name [None]: ap-southeast-2
 Default output format [None]:

```

The user with the credential used to configure the aws-cli should have permission to create users and groups.

To be sure things are configured correctly, run the command from your ```ls ~/.aws``` terminal. You should have these two files: **config** and **credentials**.

The content of the two files are:
**_config_**
>[default]
region = ap-southeast-2

**_credentials_**
>[default]
aws_access_key_id = xxxxxxxxxxxxxxx
aws_secret_access_key = xxxxxxxxxxxxxxxxxxxxxxx

### Create a user

1. Create a file named create-user.yml

copy the instructions below to the file(create-user.yml)
```

- hosts: localhost
  connection: local
  gather_facts: False
  vars:
   temp_pass: "c42wCDT1ETPc"

  tasks:
  - name: Create two AWS IAM users
    iam:
      iam_type: user
      name: "{{ item }}"
      state: present
      password: "{{ temp_pass }}"
      access_key_state: create
    with_items:
       - my_user_1
       - my_user_2

```
Looking at the instructions above, ofcourse we are using iam module of the type user. Possible options are: user, group and role. The two users i want to create are my_user_1 and my_user_2. I am also specify temporary password. This will be changed by each user on first login.

In the aws console, you should see this after the users are created.

![alt text](https://github.com/Teejay005/tech-blog/blob/master/images/04022017/aws-user-created.png "aws user created")

Run the command from the terminal to create the two users.```ansible-playbook create-user.yml```.

```

[WARNING]: provided hosts list is empty, only localhost is available
PLAY [localhost] ***************************************************************

TASK [Create two AWS IAM users] ************************************************
changed: [localhost] => (item=my_user_1)
changed: [localhost] => (item=my_user_2)

PLAY RECAP *********************************************************************
localhost                  : ok=1    changed=1    unreachable=0    failed=0

```
Checking each user further will reveal that it has not been added to any group nor has permission to do anything.

![alt text](https://github.com/Teejay005/tech-blog/blob/master/images/04022017/user_1_page.png "aws user created").


### Add users to group

In this section we'll create two groups and add one the user (my_user1) created above.

So, we add another file, create-group.yml with the following content:

```

- hosts: localhost
  connection: local
  gather_facts: False

  tasks:
    - name: Create Two Groups
      iam:
        iam_type: group
        name: "{{ item }}"
        state: present
      with_items:
         - group_1
         - group_2
      register: new_groups

    - name:
      iam:
        iam_type: user
        name: my_user_1
        state: update
        groups: "{{ item }}"
      with_items:
        - group_1
        - group_2

```


```

[WARNING]: provided hosts list is empty, only localhost is available


PLAY [localhost] ***************************************************************

TASK [Create Two Groups] *******************************************************
changed: [localhost] => (item=group_1)
changed: [localhost] => (item=group_2)

TASK [iam] *********************************************************************
changed: [localhost] => (item=group_1)
changed: [localhost] => (item=group_2)

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0

[root@manager vagrant]#

```

Checking the console again, we should have the following information.

![alt text](https://github.com/Teejay005/tech-blog/blob/master/images/04022017/groups_page.png "aws user created")

![alt text](https://github.com/Teejay005/tech-blog/blob/master/images/04022017/user_1_group.png "aws user created")

![alt text](https://github.com/Teejay005/tech-blog/blob/master/images/04022017/user_2_group.png "aws user created")

As can be seen from the images above, the groups created listed and also the user added to a group also shows that compared to the other user(my_user_2) not added to any group.

The next step is to add permission to the users to perform some specific tasks.

### Issues
```

failed: [localhost] (item=my_user_1) => {"failed": true, "item": "my_user_1", "msg": "Signature expired: 20170204T090827Z is now earlier than 20170204T100848Z (20170204T102348Z - 15 min.)"}
failed: [localhost] (item=my_user_2) => {"failed": true, "item": "my_user_2", "msg": "Signature expired: 20170204T090830Z is now earlier than 20170204T100850Z (20170204T102350Z - 15 min.)"}

```

This issue is caused by discrepancy between my local time and aws time. I solved this issue by running this command to update the local time.

Run command ```$ date ; sudo service ntp stop ; sudo ntpdate -s time.nist.gov ; sudo service ntp start ; date```


Ofcourse, install ntp first.
