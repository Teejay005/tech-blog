## Setup AWS IAM USERS and permissions using ansible and AWS-CLI

### Introduction

It is no secret that managing users and permissions to any software resource could be a daunting one. There are so many factors to be considered. Despite best efforts in the past, so many people still get it wrong. The things that could go wrong include but not limited to assigning inappropriate permissions to a user, assigning insufficient privilege to a user.

Actually AWS [advises](http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) that it is better to assign insufficient privilege to a user rather than over enabling a user.
>It's more secure to start with a minimum set of permissions and grant additional permissions as necessary, >rather than starting with permissions that are too lenient and then trying to tighten them later.

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

To be sure things are configured correctly, run the command from your ```ls ~/.aws``` terminal. You should have these two files: config and credentials.

The content of the two files are:
_config_**
>[default]
>region = ap-southeast-2

_credentials_**
>[default]
>aws_access_key_id = xxxxxxxxxxxxxxx
>aws_secret_access_key = xxxxxxxxxxxxxxxxxxxxxxx

### Create a user
