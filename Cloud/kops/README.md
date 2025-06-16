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

Step 11: Create cluster configuration

We need to specify all the az we want kops to create our nodes in, we should use multiple az for better DR

To find the azs in the give region, we can run the following command:

```bash
aws ec2 describe-availability-zones --region ap-northeast-1
```

We will set 3 az for `ap-northeast-1`

```bash
ap-northeast-1a
ap-northeast-1c
ap-northeast-1d
```

Run the cluster create command with multiple azs

```bash
kops create cluster \
    --name=${NAME} \
    --cloud=aws \
    --zones=ap-northeast-1a,ap-northeast-1c,ap-northeast-1d
```

STEP X: Deleting the cluster

Set the env variables

```bash
export NAME=myfirstcluster.k8s.local
export KOPS_STATE_STORE=s3://23navi-kops-bootstrap-configuration-bucket
```

To delete the cluster

```bash
kops delete cluster --name=${NAME} --yes
```

We can also mark our bootstrap ec2 instance to `stop` stage, it will keep the volume and we can restart it without losing any history or configs

Step 12: Starting the cluster

Note: Step 11 will just create the configurations, but it will not start the cluster.

Note: When I ran cluster create command, it failed with the error

```bash
Error: control-plane-ap-northeast-1a.spec.image: Invalid value: "099720109477/ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-20250502.1": specified image "099720109477/ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-20250502.1" is invalid: could not find Image for "099720109477/ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-20250502.1"
```

To solve, I manually changed the AMI for the control plane node

```bash
kops get ig --name ${NAME}
```

This will give me all the instance groups

| NAME                          | ROLE         | MACHINETYPE | MIN | MAX | ZONES           |
| ----------------------------- | ------------ | ----------- | --- | --- | --------------- |
| control-plane-ap-northeast-1a | ControlPlane | t3.medium   | 1   | 1   | ap-northeast-1a |
| nodes-ap-northeast-1a         | Node         | t3.medium   | 1   | 1   | ap-northeast-1a |
| nodes-ap-northeast-1c         | Node         | t3.medium   | 1   | 1   | ap-northeast-1c |
| nodes-ap-northeast-1d         | Node         | t3.medium   | 1   | 1   | ap-northeast-1d |

Then I used the following command to update the AMI used for `control-plane-ap-northeast-1a`

```bash
kops edit ig control-plane-ap-northeast-1a --name ${NAME}
```

```yaml
spec:
  image: ami-054400ced365b82a0
```

I have opend an issue [kubernetes/kops issue: Invalid default master node image for ap-northeast-1 #17440
](https://github.com/kubernetes/kops/issues/17440)

---

Finally to start the cluster

```bash
 kops update cluster --name ${NAME} --yes --admin
```

--admin is to make sure we have the admin privilages to the cluster

By default it gives admin permissions for 18hrs, to increase it, we can do

--admin=87600h

We can also set the kubectl admin access to our kops cluster using

```bash
kops export kubecfg --admin=87600h
```

Step 13: Validating our cluster startup

```bash
kops validate cluster
```

This command will try to connect to our cluster using NLB and to all the nodes, if any of them is not up, this command will show failure.

On successful validation , we will get something like

```
[ec2-user@ip-172-31-2-18 ~]$ kops validate cluster
Using cluster from kubectl context: myfirstcluster.k8s.local

Validating cluster myfirstcluster.k8s.local

```

INSTANCE GROUPS

| NAME                          | ROLE         | MACHINETYPE | MIN | MAX             | SUBNETS         |
| ----------------------------- | ------------ | ----------- | --- | --------------- | --------------- |
| control-plane-ap-northeast-1a | ControlPlane | 1           | 1   | ap-northeast-1a |
| nodes-ap-northeast-1a         | Node         | t3.medium   | 1   | 1               | ap-northeast-1a |
| nodes-ap-northeast-1c         | Node         | t3.medium   | 1   | 1               | ap-northeast-1c |
| nodes-ap-northeast-1d         | Node         | t3.medium   | 1   | 1               | ap-northeast-1d |

NODE STATUS
|NAME | ROLE | READY
| ----------------------------- | ------------ | -----------
|i-0052aec5b112719ed |control-plane |True
|i-030f96686a8fe086d| node |True
|i-0314c6279ab79b703 |node | True
|i-0fc1c273e1b152aa0 |node | True

```bash
Your cluster myfirstcluster.k8s.local is ready
```


Note: kops will automatically populate our

` ~/.kube/config `

So we can simply do 

`kubectl get nodes`

