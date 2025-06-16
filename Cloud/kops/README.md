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

```bash
AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: abc123...
Default region name [None]: ap-northeast-1
```

Make the kops profile as default profile

```bash
echo 'export AWS_PROFILE=kops' >> ~/.bashrc
source ~/.bashrc
```

Step 6: Install kops and kubectl on the bootstrap server

[kops and kubectl on linux](https://kops.sigs.k8s.io/install/)

To verify the succesful installation of kops and kubectl run:

```bash
kops
kubectl
```

Step 7: Export the aws secrets as env variables

```bash
# Because "aws configure" doesn't export these vars for kops to use, we export them now
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
```

Step 8: Setup the s3 configuration source bucket

In order to store the state of your cluster, and the representation of your cluster, we need to create a dedicated S3 bucket for kops to use. This bucket will become the source of truth for our cluster configuration.

```bash
aws s3api create-bucket \
    --bucket 23navi_kops_bootstrap_configuation_bucket \
    --region ap-northeast-1
```

Note: Above command will not work as it only creates bucket for `us-east-1`

To create bucket in any reason other than `us-east-1`

Use the following endpoint:

```bash
aws s3api create-bucket \
  --bucket 23navi-kops-bootstrap-configuration-bucket \
  --region ap-northeast-1 \
  --create-bucket-configuration LocationConstraint=ap-northeast-1
```

Step 9: Configure DNS

Note: In the kops doc, we have a big section for configuration of DNS, but we will be skipping it as we will be using something called [gossip-based DNS](https://kops.sigs.k8s.io/gossip/)

Step 10: Prepare local environment for cluster creation

As we are using gossip-based DNS, we will suffix our dns as k8s.local

```bash
export NAME=myfirstcluster.k8s.local
export KOPS_STATE_STORE=s3://23navi-kops-bootstrap-configuration-bucket
```

