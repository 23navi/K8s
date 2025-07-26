### Installing Helm on our ec2 bootstrap server


```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```


### Adding prometheus stack on our eks cluster


Got to `https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack`


Add repo locally

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Installing chart with default settings

```
helm install  monitoring prometheus-community/kube-prometheus-stack
```

output:

```
NAME: monitoring
LAST DEPLOYED: Sat Jul 26 16:05:20 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace default get pods -l "release=monitoring"

Get Grafana 'admin' user password by running:

  kubectl --namespace default get secrets monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

Access Grafana local instance:

  export POD_NAME=$(kubectl --namespace default get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=monitoring" -oname)
  kubectl --namespace default port-forward $POD_NAME 3000

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```


Note: When running on EKS, some of the pod will fail bec it requires PV which by default we didn't set on our EKS.


To delete the entire stack, we can do 

```
helm uninstall monitoring
```

