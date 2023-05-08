# How resources are allocated in cluster nodes
**TL;DR**: In K8s, [not all](https://learnk8s.io/allocatable-resources) CPU and memory in our Kubernetes nodes can be used to run Pods. In fact, there are other available resources that consume the cluster's resource too
- Operating system and system daemons such as SSH, systemd, etc
- Kubernetes agents such as the `kubelet`, the container runtime, node problem detector, etc
- Pods
- Resources reserved to the [eviction threshold](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/#eviction-thresholds)

More information 👉 [here](https://learnk8s.io/allocatable-resources)

# Container resources: Limits & Requests
A simple example
```
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
- `requetst`: request resources that container absolutely needs.
- `limits`: limit consumption of resources to the maximum that container might ever need

🎗️ _Remind_
- `Mi` refers to Mebibytes, which is a binary prefix equal to 1024^2 bytes, approximately 0.000977 `GB` (gigabytes) or 0.000954 `MB` (megabytes).
- `m`  refers to millicores, which is a unit of CPU resources equal to 1/1000th of a CPU core


Each pod has one of those [Quality of Service (QoS)](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/) classes class assigned. We can not assign this class directly. Instead, Kubernetes does it for us, based on the following principle:
- Pods without any `resources` set get `BestEffort` class
- Pods with `limits` higher than `requests` get `Burstable` class
- Pods with `limits` equal to `requests` get `Guaranteed` class

In theory, Kubernetes scheduler will always give priority to pods with `Guaranteed` class, followed by `Burstable`, and only then by `BestEffort`.

If a single node has a mix of pods with all 3 classes, the ones with `BestEffort` class should get terminated if it tries to consume too many resources. And because that, the [first lesson](https://mkdev.me/posts/kubernetes-capacity-and-resource-management-it-s-not-what-you-think-it-is) is:
```diff
+ ℹ️ We have to set requests and limits for every single container in our cluster 🤓 including not only our applications, but any third party software that we run
```

✍️ _Tip_: configure [LimitRanges](https://kubernetes.io/docs/concepts/policy/limit-range/) for each namespaces in the cluster - this way, each pod will get default values for requests, that can be later on overwritten by each workload when needed. 

# Limits & Requests - Memory
First of all, memory is a non-compressible resource. Once we give a pod memory, we can only take it away by 🔪 killing the pod !!

✍️ [Best practice](https://home.robusta.dev/blog/kubernetes-memory-limit) here for `memory` is set `limit = request`

👀 Setting the memory limit equal to the memory request does not guarantee that a container will never be OOMKilled. However, it helps ensure that a container has enough memory to function properly, which can reduce the likelihood of it being killed due to insufficient memory

More information 👉 [here](https://home.robusta.dev/blog/kubernetes-memory-limit)

# Limits & Requests - CPU
CPU is fundamentally different than memory. CPU is a compressible resource and memory is not. In simpler terms, we can give someone spare CPU in one moment when it's free, but that **does not** obligate us to continue giving them CPU in the next moment when another pod needs it. There is no downside to giving away idle CPU, because it's easy and non-violent to reclaim it.

According to this [doc](https://github.com/kubernetes/design-proposals-archive/blob/8da1442ea29adccea40693357d04727127e045ed/node/resource-qos.md#compressible-resource-guaranteess), pods always get the CPU requested by their CPU request! (They can take advantage of excess CPU too if they have no limit.)

✍️ [Best practice](https://home.robusta.dev/blog/stop-using-cpu-limits) here for `CPU` is "**Do Not use CPU `limits`**"

More information 👉 [here](https://home.robusta.dev/blog/stop-using-cpu-limits)

Terminology
- CPU throttling: is a mechanism used by operating systems or hypervisors to limit the amount of CPU resources allocated to a process or virtual machine. This can be used to prevent a process or VM from consuming too much CPU, which could cause system instability or affect the performance of other processes or VMs running on the same host. 
