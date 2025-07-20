## VPA based on CPU utilization

We have a python webapp, we did not define any request or limit for this container

`k apply -f vpa-cpu-testing.yaml`

We can see the current utilization of the webapp

`k top pods`

Note: To run `k top`, we need to have metrics server installed on our k8s cluster.

| NAME                         | CPU(cores) | MEMORY(bytes) |
| ---------------------------- | ---------- | ------------- |
| flask-app-4-7dcd9549fc-8zpzd | 1m         | 19Mi          |
| flask-app-4-7dcd9549fc-mt7js | 1m         | 19Mi          |


We have vpa-cpu.yaml which creates a VPA object to create scaling based on memory

```yaml
containerPolicies:
      - containerName: '*'
        minAllowed:
          cpu: 100m
        maxAllowed:
          cpu: 1000m
        controlledResources: ["cpu"]
```


`k apply -f vpa-cpu.yml `

`kubectl get vpa`


Initiate the load on the flask-app-4 deployment by executing the script located at `load.sh`.


`/root/load.sh`


Now to see the recommendation 

`kubectl get vpa flask-app -o jsonpath="{.status.recommendation.containerRecommendations[*].resources}"
`

or simply run `kubectl get vpa flask-app -o yaml`



Output:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"autoscaling.k8s.io/v1","kind":"VerticalPodAutoscaler","metadata":{"annotations":{},"name":"flask-app","namespace":"default"},"spec":{"resourcePolicy":{"containerPolicies":[{"containerName":"*","controlledResources":["cpu"],"maxAllowed":{"cpu":"1000m"},"minAllowed":{"cpu":"100m"}}]},"targetRef":{"apiVersion":"apps/v1","kind":"Deployment","name":"flask-app-4"},"updatePolicy":{"updateMode":"Off"}}}
  creationTimestamp: "2025-07-20T16:44:49Z"
  generation: 1
  name: flask-app
  namespace: default
  resourceVersion: "5121"
  uid: 01f025e4-4697-4b80-9c19-053841e095e2
spec:
  resourcePolicy:
    containerPolicies:
    - containerName: '*'
      controlledResources:
      - cpu
      maxAllowed:
        cpu: 1000m
      minAllowed:
        cpu: 100m
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: flask-app-4
  updatePolicy:
    updateMode: "Off"
status:
  conditions:
  - lastTransitionTime: "2025-07-20T16:45:42Z"
    status: "True"
    type: RecommendationProvided
  recommendation:
    containerRecommendations:
    - containerName: flask-app-4
      lowerBound:
        cpu: 100m
      target:
        cpu: 920m
      uncappedTarget:
        cpu: 920m
      upperBound:
        cpu: "1"
```
