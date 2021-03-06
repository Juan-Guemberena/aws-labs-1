# LAB_1_Lambda_Privesc_via_bad_NotAction

This lab covers the following scenario:

* An attacker discovers user credentials for marketing-dave which allows lambda management.
* Due to a flaw in the policy intended to restrict marketing-dave's ability to modify IAM privileges, the attacker can priv-esc to admin by launching a lambda with high privileges.
* The attacker discovers a secret in an S3 bucket after granting themselves S3 privileges to modify bucket policies

## Part I: Diagnosing and Exploiting the Problem

This lab examines a real-word issue discovered on a cloud pen test.
Devops had wanted to restrict the marketing team from privilege escalation by disallowing `iam:*` actions.

The policy they used was as follows:

        {
            "Arn": "arn:aws:iam::111111111111:group/PowerUsers-marketing-group",
            "AttachedManagedPolicies": [
                {
                    "PolicyArn": "arn:aws:iam::aws:policy/AWSLambdaFullAccess",
                    "PolicyName": "AWSLambdaFullAccess"
                }
            ],
            "CreateDate": "2012-12-20T16:20:10+00:00",
            "GroupId": "AGPAJNZACMXXXXXXXXXX",
            "GroupName": "owerUsers-marketing-group",
            "GroupPolicyList": [
                {
                    "PolicyDocument": {
                        "Statement": [
                            {
                                "Effect": "Allow",
                                "NotAction": "iam:*",
                                "Resource": "*"
                            }
                        ]
                    },
                    "PolicyName": "PowerUserAccess-marketing-2012122000000"
                }
            ],
            "Path": "/"
        }

Notice the NotAction in the PolicyDocument. 
* What is this trying to do? 
* What does it actually do?

See the answer in [Answers/Readme_Theory.md](Answers/Readme_Theory.md)


We will explore this with a scenario where an attacker has obtained the credentials to a user `marketing-dave`.
who is part of the PowerUsers-marketing-group. We will demonstrate how the policy above can be abused to
create a lambda function with a high-privileged role to elevate dave's own privileges.


1. Using admin creds, Create the roles and policies to simulate discovered high-privilege .

    ./setup.sh

2. Perform reconnaissance. Run with --admin if you want to simulate discovering high-privilege creds (capable of IAM introspection).
   More often, you won't be able to list the discocered credentials IAM privileges, but you can run enumerate-iam.py linked in the script.
   ./recon.sh [--amin]

3. Using dave's credentials, create the lambda function using the `discovered_role_with_iam_privs` role.

   ./kingme.sh

4. Once you are king (IAM boss), you can give yourself S3 Full Access privileges. Then you can list buckets.
You will need to [change the bucket IAM policy](https://aws.amazon.com/blogs/security/how-to-restrict-amazon-s3-bucket-access-to-a-specific-iam-role/) 
to grant yourself access to the bucket.



## Part II: Fixing the Problem (WIP)

Now comes the hard part. How do we accomplish what the Devops team had intended? Allow the marketing power users as
much freedom as possible (at least as much control over Lambda as possible) without giving them a path to privilege escalate to Admin?

### Options

* Permissions Boundary
* Explicit Deny
* Whitelist Passrole Resources
* Separate AWS account where marketing is admin + CI/CD to publish into existing account



## TODO
- [x] Add cleanup instrutions.
- [x] Replace all hard-coded random sequences with $RANDOM or similar. 
- [] Change the group privileges to only allow assume-role and push all privs into that group role, following best practices.
- [] Design a version of the Lab so that the priv-esc is only to read a CTF hash in S3, not become admin.
- [] Develop the "How to Fix" options.
