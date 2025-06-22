To run our istio, I will be using the aws eks.

We will use the eksctl setup which we have already created in our cloud project.

The instance use is x86 so in `source -> _course_files -> x86_amd64 -> warmup-excercise`

Run the `1-istio-init.yaml` and `2-istio-minikube.yaml` to deploy the control plane components (7 containers including istiod)

Run the `3-kiali-secret.yaml` to set the default login and password for kiali ui

```
username: admin
password: admin
```

Now for istiod to inject sidecar containers to each pod, we will have to add label to the namespace we want sidecar injection

```
kubectl label namespace default istio-injection=enabled
```

Now we can run our initial application on default namespace (the current state of our application will have bugs which we will fix over time using the features of istio)

`k apply -f 4-application-full-stack.yaml`
