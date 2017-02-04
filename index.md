## Setup AWS IAM USERS and permissions using ansible and AWS-CLI

### Introduction

It is no secret that managing users and permissions to any software resource could be a daunting one. There are so many factors to be considered. Despite best efforts in the past, so many people still get it wrong. The things that could go wrong include but not limited to assigning inappropriate permissions to a user, assigning insufficient privilege to a user.

Actually AWS [advises](http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) that it is better to assign insufficient privilege to a user rather than over enabling a user.
>It's more secure to start with a minimum set of permissions and grant additional permissions as necessary, >rather than starting with permissions that are too lenient and then trying to tighten them later.

Today, we'll go through how to create AWS group, users and assign permissions to the users/group created using AWS-CLi with ansible module for AWS.
