### Known limitations with VPA

* Whenever VPA updates the pod resources, the pod is recreated, which causes all
running containers to be recreated. The pod may be recreated on a different
node.
* VPA cannot guarantee that pods it evicts or deletes to apply recommendations
(when configured in Auto and Recreate modes) will be successfully
recreated. This can be partly
addressed by using VPA together with Cluster Autoscaler.
* VPA does not update resources of pods which are not run under a controller.
* Vertical Pod Autoscaler should not be used with the Horizontal Pod Autoscaler (HPA) on CPU or memory at this moment.
However, you can use VPA with HPA on custom and external metrics.
* The VPA admission controller is an admission webhook. If you add other admission webhooks
to your cluster, it is important to analyze how they interact and whether they may conflict
with each other. The order of admission controllers is defined by a flag on API server.
* VPA reacts to most out-of-memory events, but not in all situations.
* VPA performance has not been tested in large clusters.
* VPA recommendation might exceed available resources (e.g. Node size, available
size, available quota) and cause pods to go pending. This can be partly
addressed by using VPA together with Cluster Autoscaler.
* Multiple VPA resources matching the same pod have undefined behavior.