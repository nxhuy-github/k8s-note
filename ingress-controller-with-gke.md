# Ingress Controller with GKE

## Terminology
First thing first, we need to clarify some terminologies: 
- **Ingress** or **Ingress resource** or **Ingress object** is a native K8s object in which we specify the DNS rules. We can deploy a bunch of **Ingress** rules, but nothing will happen unless we have a controller that can process them
- **Ingress Controller** in the other hand IS NOT a native K8s implementation. Meaning, It doesn’t come default in the cluster

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






