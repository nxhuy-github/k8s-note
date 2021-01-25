# Helm 

## What is Helm?
This is a package manager for K8s, so it's a convenient way for packaging collections of K8s yaml files and distributing them in public or private repositories.

So let's say we have deployed our application in K8s cluster and we want deploy ElasticStack additionally in our cluster that our application will use to collect its logs. In order to deploy ElasticStack in our K8s cluster, we'll need a couple of K8s components, so we would need a **StatefulSet** which is for stateful application like databases, a **ConfigMap** for external configuration, a **Secret** for some credentials and a couple of **Services**. Now if we were to create all these files manually by searching for each one of them separately on Internet, it would be a tedious job and until we have all these yaml files collected and tested and tried out, it might take some time. And since ElasticStack deployment is pretty much standard across all cluster, other people will probably to go through the same. So it made perfect sense that someone created these yaml files once and packaged them up and made it available somewhere so that other people who also use the same kind of deployment could use them in their K8s cluster. And that bundle of yaml files is called **Helm Chart**.

So using **Helm** we can create our own **Helm Charts** or bundles of those yaml files and push them to some **Helm Repository** to make it available for the other or we can consume existing **Helm Charts** of the other people. Commonly used deployments like database applications (ElasticSearch, MongoDB, MySQL, etc), monitoring applications (Promotheus, etc) that all have this kind of complex setup all have charts available in some **Helm Repository**.

Features
- Sharing Helm Charts
- Templating Engine
    - Imagine that we have an application that is made up of multiple micro-services and we're deploying them in K8s cluster. **Deployments** and **Services** of those micro-services are pretty much the same with the difference like application name, version, docker image, tags, etc. So without **Helm**, we would write separate yml files configuration for each of those micro-services. So we would have multiple **Deployment**/**Service** files where each one has its own application name, version, etc. Since the only difference between those yaml files are just couple of lines or couple of values. Using **Helm** what we can do is
        - Define a common blueprint
        - Dynamic values are replaced by placeholders
            - Pratical for CI/CD: in our Build, we can replace the values on the fly before deploying them
- Deploy same application(s) across different environments
    - Consider usecase where we have our micro-service application that we want to deploy on **Development**, **Staging** and **Production** clusters. So instead of deploying the individual yaml files separately in each cluster, we can package them up to make our own application **Chart** that will have all the necessary yaml files that particular deployment needs and the we can use them to redeploy the same application in different K8s cluster environments.
- Release management
    - **Helm v2**
        - Two parts
            - Client (**Helm** CLI)
            - Server (**Tiller**)
                - live in K8s cluster
        - whenever we create or change deployment, **Tiller** will store a *copy of each configuration* send by client for future reference 
            - creata a history of **Chart** executions
            - changes are applied to existing deployment instead of creating a new one
            - and rollback if needed
        - Downsides
            - **Tiller** has too much power(permissions) in K8s
            - Security issues
    - **Helm v3**
        - **Tiller** got removed

## Helm Chart Structure
```
mychart/
|___Chart.yaml
|___values.yaml
|___charts/
|___templates/
|___...
```

- `mychart`: name of **Chart**
- `Chart.yaml`: meta info about **Chart** (like name, version, list of dependencies, etc)
- `values.yaml`: values for the template files
- `charts`: have **Chart** dependencies inside meaning that if this **Chart** depends on the other **Charts**, then those **Charts** dependencies will be stored here
- `templates`: the actual template files

So when we execute `helm install <chartname>` to actually deploy those yaml files into K8s. The template files from here will be filled with the values from `values.yaml` producing valid K8s manifest that can then be deployed into K8s.