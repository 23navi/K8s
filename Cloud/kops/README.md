# Using kops to manage cluster on aws

Step 1: Make sure to install kops on the local machine.

[kops install](https://kops.sigs.k8s.io/getting_started/install/)

```bash
brew update && brew install kops
```

Step 2: Make sure to set the aws cli profiles.

Say I am using some other profile, then I will have to set that context.

