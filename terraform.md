# Terraform 

## What is Terraform?
**Terraform** is a tool for insfrastructure provisioning which allows us to automate and manage 
- our infrastructure
- our platform 
- services that run on that platform

It's an open source and uses the **declarity language**, meaning we don't have to define every step of how these automation and management is done, we just clear what we want: the final result or end result. And **Terraform** will figure out how to execute. It versus imperative style where we specify how to execute each step.

What does *a tool for insfrastructure provisioning* mean exactly?

Let's say we just started a project where we create some application and we want **set up** an infrastructure **from scratch** where this application will run. How does our infrastructure look like? 
- Let's say we want spin up several servers where we'll deploy our 5 micro-service applications, which make up our application, as Docker containers and also we're going to deploy a DB container. 
    - We decide to use AWS/GCP platform to build your whole infrastructure on. 
        - So first step will be to go to AWS/GCP and prepare the setup so the applications can be deploy there. 
            - This means we create our private network space, 
            - we create an employee server in EC2 for ex server intances, 
            - we install Docker on each one of those plus any tools we might need, 
            - we set up security on our server like firewall, etc. 
        - And once our infrastructure is prepared, 
            - we can now deploy our Docker applications. 
    - So we can see that these are two different tasks or two separate steps of creating the whole setup: 
        - one is provisioning the infrastructure, preparing everything and 
        - the second one is deploying the applications on it. 
            - So we might need two different teams or two individuals who do these two separate tasks. 
- And **Terraform** is used for the first part where we provision the infrastructure

## What is Terraform used for?


## Terraform Architecture & Commands