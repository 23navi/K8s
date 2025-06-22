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

Note: The application is setup such a way that it exposes node port at port 30080
The problem is that the ec2 instances we launched with eks does not allow inbound traffic on this port. So for a hack I have opend all tcp ports.

Now the application is accessible on (<ip_of_any_of_my_ec2_instances>:30080)

We also have:

Kiali UI: `http://35.74.5.126:31000/`

Jaeger UI: `http://35.74.5.126:31001/` (Traces, similar to Grafana Tempo)

---

### Telemetry with Istio

To start the project, we must delete all the things we created `source -> _course_files -> x86_amd64 -> warmup-excercise` , we can do it by running `kubectl delete -f .`

Now we can go to `source -> _course_files -> x86_amd64 -> 1-Telemetry` folder and run all the yaml by doing `kubectl apply -f <file.yaml>`

Note: We no longer need to set username:password for kiali console, as we set it behind our gateway. So we don't require to run 3-kiali-secret.yaml

Note: What happens if we deploy our application without adding the lable to our namespace?

eg: We forgot to do `kubectl label namespace default istio-injection=enabled`

and we ran `kubectl apply -f 5-application-no-istio.yaml`

In this case, even after we add the label, the proxies won't be applied to pods until we restart each pod.

We can restart all the pods by doing `kubectl delete po --all` (It is like a bounce, as deployment we create all the pods)
