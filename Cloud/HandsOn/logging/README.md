## Running the elastic stack

We have /logging with stack and fluentd config, we just have to apply them in our cluster and we will have elastic stack in our culster under (kube-system namespace)

Make sure we have our storage class in up as the stack uses the pvc that refer to our storage class

```yaml
volumeClaimTemplates:
  - metadata:
      name: elasticsearch-logging
    spec:
      storageClassName: cloud-ssd
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 31Gi
```

Run fluentd-config.yaml followed by elastic-stack.yaml

`k apply -f fluentd-config.yaml`

`k apply -f elastic-stack.yaml`

Our setup will create a load balancer for kibana dashboard, we can view it at lb_domain:5601

`k get svc -n kube-system `

| NAME                      | TYPE         | CLUSTER-IP     | EXTERNAL-IP                                                                 | PORT(S)        | AGE  |
| ------------------------- | ------------ | -------------- | --------------------------------------------------------------------------- | -------------- | ---- |
| eks-extension-metrics-api | ClusterIP    | 10.100.104.231 | <none>                                                                      | 443/TCP        | 170m |
| elasticsearch-logging     | ClusterIP    | 10.100.153.88  | <none>                                                                      | 9200/TCP       | 51s  |
| kibana-logging            | LoadBalancer | 10.100.37.128  | a0bf184c5271c406299ac1fa845c77d3-794990380.ap-northeast-1.elb.amazonaws.com | 5601:32532/TCP | 51s  |
