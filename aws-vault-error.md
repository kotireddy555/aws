# AWS-VAULT - Unable to login using aws-vault to child accounts where MFA enabled 

`url : https://github.com/99designs/aws-vault`


Error: 

  Error: aws-vault: error: exec: Failed to get credentials for admin: AccessDenied: User: arn:aws:iam::<accountnumber>:user/my_user is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::<accountnumber>:role/admin
```
   
π ~ ❯ aws-vault --version
            v5.4.4
```

Source profile something similar in `.aws/config` and All are in same region and managed under single master account
```
[profile base-profile]
mfa_serial=arn:aws:iam::11111111111:mfa/my_user

[profile account-dev]
role_arn=arn:aws:iam::222222222222:role/<AssumeRoleName>
source_profile=base-profile
region=eu-west-1

[profile account-qa]
role_arn=arn:aws:iam::222222222222:role/<AssumeRoleName>
source_profile=base-profile
region=eu-west-1

[profile account-prod]
role_arn=arn:aws:iam::3333333333333:role/<AssumeRoleName>
source_profile=base-profile
region=eu-west-1

```

We were able to login to `account-dev` and `account-qa` but not `account-prod` the reason was we want to force team to login via MFA for prod by applying below policy @group level in master account
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1559651192000",
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Resource": [
                "arn:aws:iam::<childaccountnumber>:role/OrganizationAccountAccessRole"
            ],
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": true
                }
            }
        }
    ]
}
```

This is the condition which forcing users to use MFA 

```
 "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": true
                }
            }
```
## Solution 

Though we have enabled mfa at base-profile level `mfa_serial=arn:aws:iam::11111111111:mfa/my_user`

For some reason `aws-vault` won't inherit for child accounts probably overwriting to mfa level none :(

To fix this problem we need to pass `mfa_serial` at profile level something like below 

```
[profile base-profile]
mfa_serial=arn:aws:iam::11111111111:mfa/my_user

[profile account-dev]
source_profile=base-profile
role_arn=arn:aws:iam::222222222222:role/<AssumeRoleName>
region=eu-west-1

[profile account-qa]
source_profile=base-profile
role_arn=arn:aws:iam::4444444444444:role/<AssumeRoleName>
region=eu-west-1

[profile account-prod]
source_profile=base-profile
mfa_serial=arn:aws:iam::11111111111:mfa/my_user
role_arn=arn:aws:iam::3333333333333:role/<AssumeRoleName>
region=eu-west-1
```
You might raise the question like What is the use of base profile :D


### Hope it helps someone 

