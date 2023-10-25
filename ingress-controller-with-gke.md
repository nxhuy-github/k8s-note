# Ingress Controller with GKE

## Terminology
First thing first, we need to clarify some terminologies: 
- **Ingress** or **Ingress resource** or **Ingress object** is a native K8s object in which we specify the DNS **rules**. We can deploy a bunch of **Ingress** rules, but nothing will happen unless we have a controller that can process them
- **Ingress Controller** in the other hand IS NOT a native K8s implementation. Meaning, It doesn’t come default in the cluster

While **Ingress Controller** can be deployed in **any namespace** it is usually deployed in a namespace separate from your app services. It can see **Ingress rules** in all other namespaces and pick them up. However, each of the **[Ingress rules must reside in the namespace where the app that they configure reside](https://stackoverflow.com/questions/59844622/ingress-configuration-for-k8s-in-different-namespaces)** (i.e Ingress can only route to Services within the same namespace).

## What is Ingress Controller
An **Ingress Controller** is typically a [reverse web proxy server](https://www.cloudflare.com/en-gb/learning/cdn/glossary/reverse-proxy/) implementation in the cluster. In K8s terms, it is a reverse proxy server deployed as K8s [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) exposed to a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) type `LoadBalancer`.

Technically, **Ingress Controller** is implemented <u>as a</u> Layer 7 Load Balancer that dynamically configures itself based on **Ingress** objects, when it receives traffic from the Cloud Load Balancer (normally an L4 Load Balancer). So we could somehow say that **Ingress Controller** is really just a Layer 7 Load Balancer that exists inside of a K8s cluster.

And because **Ingress Controller** acts a L7 Load Balancer (which works at the highest OSI model layer: the application layer FYI) it can route HTTP and HTTPS traffic (it will know how to terminate HTTPS, distinguish each website’s traffic since it understands url names and paths, and know which backend server to forward traffic to) - **but not** for TCP and UDP traffic using <u>standard K8s **Ingress**</u> (it could be seen as its limitation). 

We've talked a lot about L7 Load Balancer and how it can handle traffic but in reality, to obtain a scalable Load Balancer architecture, we usually combine **L7 LB** and **L4 LB** together to get all their features. For more detail, although **L7 LB** offers more features than **L4 LB**, it can’t handle nearly as much traffic as **L4 LB**. In fact, within **L4 LB**, messages can be forwarded efficiently, quickly, and securely because they are neither decrypted nor inspected whereas the features offered by **L7 LB** could make it more costly in terms of required computing power and time than **L4 LB**.

Following are the commonly used ingress controllers available for Kubernetes
- Nginx Ingress Controller (Community & From Nginx Inc)
- Traefik
- HAproxy
- Contour
- GKE Ingress Controller for GKE
- AWS ALB Ingress Controller for AKS
- Azure Application Gateway Ingress Controller
- etc

## Exposing Services
Like we know, in K8s, if we want to expose a **Service** to the outside word, we could use [Service](https://kubernetes.io/docs/concepts/services-networking/service/) type `LoadBalancer` to do that. As a reminder, a K8s [Service](https://kubernetes.io/docs/concepts/services-networking/service/) type `LoadBalancer` is externally exposed by using a Cloud provider’s load balancer. So it could be very expensive as it requires each [Service](https://kubernetes.io/docs/concepts/services-networking/service/) to have its own IP address and cloud provider’s load balancer: `N` [Service](https://kubernetes.io/docs/concepts/services-networking/service/) type `LoadBalancer` => `N` different IPs address + `N` cloud provider’s load balancer.

That could be why it is ideal to use **Ingress** and **Ingress Controller** over [Service](https://kubernetes.io/docs/concepts/services-networking/service/) type `LoadBalancer`.  As **Ingress Controller** collect **Ingress** resources behind **a common IP**, they are then less expensive. Beside that, **Ingress Controller** can also provide many features such as ACME , middleware, and load balancing, TLS configurations - TLS security, observability of a platform, etc. When used correctly, **Ingress Controller** can dramatically simplify the operations of K8s clusters while improving security, observability, performance, and resilience.

## GKE Ingress Controller built-in
In GKE, the good thing is that it comes up with an **inbuilt** GKE **Ingress Controller**. So we **DON'T** have to do any extra configurations to set up an Ingress Controller. Of course, GKE also lets us set up other Ingress Controllers (like the Nginx ingress controller for ex) - Choosing a **GKE Ingress Controller** vs. another **Ingress Controller** really depends on the project requirements and features required in the Ingress layer. 

By default, when we create an **Ingress** object in GKE cluster, the **GKE Ingress Controller** will automatically spin up a HTTP(S) Load Balancer (in fact, we can choose the type of LB - an `external` or an `internal` HTTP(S) LB by using an `annotation` on the **Ingress** object) for us. And usually, the backend [Services](https://kubernetes.io/docs/concepts/services-networking/service/) referenced in this **Ingress** object should be of type `NodePort`.

For now, :warning: GKE [doesn't support](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress#limitations) combining multiple **Ingress** resources into **a single** Google Cloud Load Balancer. Meaning that for each **Ingress** object created, a separate HTTP(S) Load Balancer will be automatically provisioned (so it could be very expensive in some cases). We could circumvent this problem by passing all our ingress rules into a single **Ingress** resource (like a big **Ingress** yaml file that will contain all of our rules).

## Traefik on GKE
Like we know, GKE lets us set up other Ingress Controller if needed and [Traefik](https://doc.traefik.io/traefik/) is also the case. In fact, we can simply [install](https://github.com/traefik/traefik-helm-chart/tree/master/traefik) **Traefik** via [Helm](https://helm.sh/docs/), which works totally fine. Once we install **Traefik** in GKE cluster, GCP will automatically spin up a **Network (TCP protocol) Load Balancer** (an L4 LB) for us. So this is the case that we mentionned above - combining L4 LB and L7 LB: here, we have a GCP Network Load Balancer which lives **outside** the cluster and a **Traefik Ingress Controller** acts as a HTTP(S) Load Balancer which resides **inside** the cluster.

To inform our **Ingress** objects that **Traefik Ingress Controller** is the thing that we want to use, in the **Ingress** configuration yaml file, we configure
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    annotations:
        # kubernetes.io/ingress.class: <traefik-fullname> <- it seems like this annotations was deprecated or what, it doesn't work well with Traefik at least, use `ingressClassName` instead
...
spec:
    ingressClassName: <traefik-fullname>
...
``` 
By using Helm, the `<traefik-fullname>` here will be either
- by default, the Chart Name -> `ingressClassName: traefik`
- or the [Release name](https://stackoverflow.com/questions/51718202/helm-how-to-define-release-name-value) if its value contains Chart name :joy:
- or the value of `fullnameOverride` in our **override** Traefik configuration [values.yaml](https://github.com/traefik/traefik-helm-chart/blob/master/traefik/values.yaml) if we use one
    - yah, <u>in our override value.yaml</u> since the value.yaml default didn't show this option at the first glance :smiling_face_with_tear:

:warning: by installing Traefik via helm, we should always read the [repo](https://github.com/traefik/traefik-helm-chart/tree/master/traefik) for more understanding

This `<traefik-fullname>` value (or whatever we call it) is very important because if we use a wrong one, **Traefik Ingress Controller** won't work as we expected. For example, in the case where we want to deploy one distinct **Traefik Ingress Controller** for each [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) in our cluster, by setting/deploying different `fullnameOverride` could be helpful.



