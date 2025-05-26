# Using kops to manage cluster on aws

Step 1: Make sure to install kops on the local machine.

[kops install](https://kops.sigs.k8s.io/getting_started/install/)

```bash
brew update && brew install kops
```

Step 2: Make sure to set the aws cli profiles.

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

Step 3: Make sure the current profile has proper permissions to create new aws resources including IAM users, groups, permissions.

Step 4: Create a new user group and user for kops and give it required permissions

[kops on aws](https://kops.sigs.k8s.io/getting_started/aws/)

`AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
AmazonSQSFullAccess
AmazonEventBridgeFullAccess`



```bash
aws iam create-group --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonSQSFullAccess --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEventBridgeFullAccess --group-name kops

aws iam create-user --user-name kops

aws iam add-user-to-group --user-name kops --group-name kops

aws iam create-access-key --user-name kops
```


Step 5: Launch a new EC2 instance in that region and we will run all kops bootstrap commands from that ec2 instance

Note: By default ec2 instance won't have aws cli configured, so we can't make any calls from it.

We will take that kops user and add the cli configuration to this new bootstrap ec2 instance.

```bash
aws configure --profile kops
```
And add:

`AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: abc123...
Default region name [None]: ap-northeast-1


Make the kops profile as default profile

```bash
echo 'export AWS_PROFILE=kops' >> ~/.bashrc
source ~/.bashrc
```