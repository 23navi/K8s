We've supplied two verisons of the yaml for monitoring - for EKS and Kops.

What's the difference between the two?
 
If we are planning to deploy to a production *Kops* cluster (EKS is fine), please note that the version shipped here has removed several control plane alerts. I needed to remove these alerts because Kops users would get spurious control plane (master) alerts which can only be silenced via an awkward workaround (see below).

So we could install my kops-monitoring.yaml to wer cluster, but there's a chance we might be missing out on some valuable alerts. So I recommend for a production cluster, get the latest version of the monitoring stack (with all alerts enabled) from https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack


Then we would need to make a change to wer cluster as follows:

kops edit cluster

Then insert the following two lines, as a child element of "spec:"

spec:
.
.
  kubeProxy:
    metricsBindAddress: 0.0.0.0

After doing this, we will need to run "kops update cluster --yes" and then run "kops rolling-update cluster". This will terminate all the nodes in wer cluster one by one (so the only one node is unavailable at any one time), and will make the changes needed.

Full details of the reason behind this (it is quite obscure) here: https://github.com/helm/charts/issues/16476

Without doing this, we will get lots of spurious alerts about the control plane.
