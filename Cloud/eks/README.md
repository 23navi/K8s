# Using EKS to manage cluster on aws

Step 1: Make sure to install eks cli (ekscli)

It is a cli based tool to manage our eks server on aws and is managed by AWS itself

[eksctl docs](https://eksctl.io/)

We will be installing it on our aws bootstrap server we already created during kops

```bash
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
```

Step 2: Create a new IAM group and user for eks setup

For permissions, we can use this list of inline policies

[Minimum IAM policies] (https://eksctl.io/usage/minimum-iam-policies/)

| AWS Service      | Access Level                                       |
| ---------------- | -------------------------------------------------- |
| CloudFormation   | Full Access                                        |
| EC2              | Full: Tagging Limited: List, Read, Write           |
| EC2 Auto Scaling | Limited: List, Write                               |
| EKS              | Full Access                                        |
| IAM              | Limited: List, Read, Write, Permissions Management |
| Systems Manager  | Limited: List, Read                                |

Create group : eks_group
Create user: eks_user and attach it to eks_group



Step 3: Add this new user profile in aws cli on the bootstrap ec2 server

```bash
aws configure --profile eks_user
```

And add:

```bash
AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: abc123...
Default region name [None]: ap-northeast-1
```

Make the kops profile as default profile

```bash
echo 'export AWS_PROFILE=eks_user' >> ~/.bashrc
source ~/.bashrc
```

Step 4: Make sure to set the aws cli profiles.

Say I am using some other profile, then I will have to set that context.

(Check all available profiles)

```bash
less ~/.aws/credentials
```

To set the current session aws cli profile and region

```bash
export AWS_PROFILE=nc
export AWS_REGION=ap-northeast-1
```

To verify the current cli session profile and region

```bash
aws configure list
```

Run aws commands to test

```bash
aws eks list-clusters
```


Step 6: Install kubectl on the bootstrap server

[kops and kubectl on linux](https://kops.sigs.k8s.io/install/)

Note: Make sure the kubectl client version is available on eks kubernetes version.

To verify the succesful installation of kubectl run:

```bash
kubectl version --client
```

Step 7: Export the aws secrets as env variables

```bash
# Because "aws configure" doesn't export these vars for kops to use, we export them now
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
```


Step 8: Create the cluster using ekscluster

Note: eksctl in the background just calls aws commands, eksctl is just a high level wrapper.

```bash
eksctl create cluster --name mycluster --nodes-min=3 --node-type=t3.medium
```

by default --node-type=m5.large (which is very expensive)

The above create command will take around 5-10 mins to create the cluster.

