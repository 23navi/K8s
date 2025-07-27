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

To delete the entire stack, we can do

```
helm uninstall monitoring
```

It creates a lot of service and pods in default namespace

```
[ec2-user@ip-172-31-2-18 ~]$ k get svc
```

| NAME                                    | TYPE         | CLUSTER-IP     | EXTERNAL-IP                                                                 | PORT(S)                         | AGE   |
| --------------------------------------- | ------------ | -------------- | --------------------------------------------------------------------------- | ------------------------------- | ----- |
| alertmanager-operated                   | ClusterIP    | None           | <none>                                                                      | 9093/TCP,9094/TCP,9094/UDP      | 92s   |
| jenkins                                 | LoadBalancer | 10.100.233.45  | aa35acb634b244c6fbbc7f5b15718bd7-739463129.ap-northeast-1.elb.amazonaws.com | 8080:31000/TCP, 50000:31988/TCP | 2d11h |
| kubernetes                              | ClusterIP    | 10.100.0.1     | <none>                                                                      | 443/TCP                         | 2d12h |
| monitoring-grafana                      | ClusterIP    | 10.100.248.6   | <none>                                                                      | 80/TCP 98s                      |
| monitoring-kube-prometheus-alertmanager | ClusterIP    | 10.100.105.230 | <none>                                                                      | 9093/TCP,8080/TCP               | 98s   |
| monitoring-kube-prometheus-operator     | ClusterIP    | 10.100.216.146 | <none>                                                                      | 443/TCP                         | 98s   |
| monitoring-kube-prometheus-prometheus   | ClusterIP    | 10.100.140.180 | <none>                                                                      | 9090/TCP,8080/TCP               | 98s   |
| monitoring-kube-state-metrics           | ClusterIP    | 10.100.106.164 | <none>                                                                      | 8080/TCP                        | 98s   |
| monitoring-prometheus-node-exporter     | ClusterIP    | 10.100.185.17  | <none>                                                                      | 9100/TCP                        | 98s   |
| prometheus-operated                     | ClusterIP    | None           | <none>                                                                      | 9090/TCP                        | 92s   |

---

```
[ec2-user@ip-172-31-2-18 ~]$ k get pods
```

| NAME                                                   | READY | STATUS  | RESTARTS | AGE   |
| ------------------------------------------------------ | ----- | ------- | -------- | ----- |
| alertmanager-monitoring-kube-prometheus-alertmanager-0 | 2/2   | Running | 0        | 86s   |
| jenkins-7db9587b4f-c78qs                               | 1/1   | Running | 0        | 2d11h |
| monitoring-grafana-6fbcccb5d7-49qkb                    | 3/3   | Running | 0        | 91s   |
| monitoring-kube-prometheus-operator-68bd67d4bf-s62wz   | 1/1   | Running | 0        | 91s   |
| monitoring-kube-state-metrics-585b45df98-xvvpv         | 1/1   | Running | 0        | 91s   |
| monitoring-prometheus-node-exporter-6clkf              | 1/1   | Running | 0        | 91s   |
| monitoring-prometheus-node-exporter-bccx4              | 1/1   | Running | 0        | 91s   |
| monitoring-prometheus-node-exporter-pzrmq              | 1/1   | Running | 0        | 91s   |
| prometheus-monitoring-kube-prometheus-prometheus-0     | 2/2   | Running | 0        | 86s   |


--------


To access the grafana dashboard, we can make the service `monitoring-grafana ` as LoadBalancer


```
k edit srv monitoring-grafana 
```


Now we can acess the grafana dashboard on port 80

eg: `a5bef4872992f4d4d925f0f70239670b-1156234268.ap-northeast-1.elb.amazonaws.com:80`


#### Updating the default values

So when we start our stack, we get grafana with default username and passowrd

```yaml
grafana:
 ....
  adminUser: admin
  adminPassword: prom-operator
```


Now to update these values after our package deployment, we can use `helm update` command


Step 1: Get the default `values.yaml` associated with the chart.

```
helm show values prometheus-community/kube-prometheus-stack > values.yaml
```

Step 2: Search for the config where the admin username and password for grafana is mentioned

```yaml
grafana:
  enabled: true
  namespaceOverride: ""
  ## ForceDeployDatasources Create datasource configmap even if grafana deployment has been disabled
  ##
  forceDeployDatasources: false
  ## ForceDeployDashboard Create dashboard configmap even if grafana deployment has been disabled
  ##
  forceDeployDashboards: false
  ## Deploy default dashboards
  ##
  defaultDashboardsEnabled: true
  operator:
    ## Enable references to ConfigMaps containing dashboards in GrafanaDashboard CRs
    ## Set to true to allow dashboards to be loaded from ConfigMap references
    dashboardsConfigMapRefEnabled: false
    ## Annotations for GrafanaDashboard Cr
    ##
    annotations: {}
    ## Labels that should be matched kind: Grafana instance
    ## Example: { app: grafana, category: dashboard }
    ##
    matchLabels: {}
    ## How frequently the operator should resync resources (in duration format)
    ## Controls how often dashboards are reconciled by the operator
    ##
    resyncPeriod: 10m
    ## Which folder all ddashboard in Grafana General means on Root level
    ##
    folder: General

  ## Timezone for the default dashboards
  ## Other options are: browser or a specific timezone, i.e. Europe/Luxembourg
  ##
  defaultDashboardsTimezone: utc

  ## Editable flag for the default dashboards
  ##
  defaultDashboardsEditable: true

  ## Default interval for Grafana dashboards
  ##
  defaultDashboardsInterval: 1m

  adminUser: admin
  adminPassword: prom-operator

```


Step 3: Update the values using `--set`

```
helm upgrade monitoring prometheus-community/kube-prometheus-stack --set grafana.adminPassword=Navi
```


Output:

```
[ec2-user@ip-172-31-2-18 ~]$ helm upgrade monitoring prometheus-community/kube-prometheus-stack --set grafana.adminPassword=Navi
Release "monitoring" has been upgraded. Happy Helming!
NAME: monitoring
LAST DEPLOYED: Sun Jul 27 04:38:59 2025
NAMESPACE: default
STATUS: deployed
REVISION: 2
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



#### Update the default config using `--values=`

```
helm upgrade monitoring prometheus-community/kube-prometheus-stack  --values=upgrade-values.yaml
```


output:

```
[ec2-user@ip-172-31-2-18 ~]$ helm upgrade monitoring prometheus-community/kube-prometheus-stack  --values=upgrade-values.yaml
Release "monitoring" has been upgraded. Happy Helming!
NAME: monitoring
LAST DEPLOYED: Sun Jul 27 05:00:50 2025
NAMESPACE: default
STATUS: deployed
REVISION: 3
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