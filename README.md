## kubernetes-the-manual-setup
A manual bootstrap of Kubernetes in AWS infrastructure. - zero automations - just the hard way.

In this project, a kubernetes cluster will be set up manually. This may not be the choice approach in production environments.

This approach will show the different working parts of a kubernetes cluster, the configurations and the networking elements used for a sucessfull kubernetes cluster deployment. So if you're looking to widen your understanding of kubernetes cluster, you might find some relevance in this lab although its may have little support from the community.


Previously, we have highlighted the advantages of running applications in containers as opposed to running them in VMs - we had a POC job with an app, ``tooling app and its MySQL`` which was deployed as containers using ```docker compose``` in a single host machine. This is one of the limitations of docker compose and is considered as a single point of failure. To prevent such risk of loosing workload, a more scalable approach has to be employed. 

## Why Container Orchestration? 

 ``Container orchestration ``  is the  automated management, deployment, scaling, and networking of containers. It allows you to efficiently handle large-scale containerized applications across clusters of machines, ensuring high availability, load balancing, and smooth operations.

Two features of docker containers to keep in mind:

1. They are **Ephemeral in Nature**:  By design, Docker containers are ephemeral. By “ephemeral” it means that a container can be stopped and destroyed, and a new one can be built from the same Docker image and put in place with an absolute minimum set-up and configuration requirement.

2. **Scalability across multiple compute nodes**: To ensure that container workloads are highly scalable, they must be configured to run across multiple compute nodes. A compute node can be a physical server or a virtual machine with Docker engine installed to run your containers.

If we had two compute nodes to run a application like our ```tooling app``` and its database container, let us consider a following scenarios:

1. If containers are configured to run across 2 computer nodes, and a particular container, running on Node 1 dies, how will it know that it can spin up again on Node 2?

2. Let us imagine that Tooling website container is running on Node 1, and MySQL container is running on Node 2, how will both containers be able to communicate with each other? Remember we had to create a custom network on the same host and ensure that they can communicate through that network. But in the case of 2 separate hosts, this is natively not possible.

**Container orchestration** addresses these two scenarios, it provides automation of all the aspects of coordinating and managing containers. Container orchestration is focused on managing life cycle of containers and their dynamic environments.

**Automation**: Reduces manual effort by automating deployment, scaling, and updates.
**Scalability**: Dynamically scales services based on resource usage and demand.
**High Availability**: Ensures redundancy and distributes workloads across the cluster to prevent downtime.
**Resource Efficiency**: Optimizes resource utilization by scheduling containers effectively.
**Networking**: Provides service discovery and load balancing between containers.
**Monitoring and Logging**: Offers insights into application performance and container health.
**Self-Healing**: Restarts failed containers or reassigns workloads if nodes fail.

**Kubernetes** is one of the most popular orchestration tools and very efficient when correctly configured.
Another orchestration tool is **Docker Swarm**

### Kubernetes Architecture

![alt text](images/components-of-kubernetes.svg)


### Kubernetes manual set-up
Kubernetes (k8s) is  a very useful tool for orchestration. It is characterized by many moving parts and components that has to be configured correctly for its operation. 
There are k8s implementations that can easily be installed with ease, they spin up k8s clusters that can be used in test and development environments .g **minikube**.

Again, here, we will set up k8s from ground up.

To successfully implement "K8s From-Ground-Up", the following and even more will be done by you as a K8s administrator.

    1. Install and configure master (also known as control plane) components and worker nodes (or just nodes).
    2. Apply security settings across the entire cluster (i.e., encrypting the data in transit, and at rest)

        * In transit encryption means encrypting communications over the network using HTTPS
        * At rest encryption means encrypting the data stored on a disk**
    3. Plan the capacity for the backend data store etcd

    4. Configure network plugins for the containers to communicate
    5. Manage periodical upgrade of the cluster
    6. Configure observability and auditing.

**NB**: Unless there is a specific business or compliance restricition, **ALWAYS** consider to use managed versions of K8s - Platform as a Service offerings, such as [ Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/), [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/), or [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) as they usually have better default security settings, and the costs for maintaining the control plane are very low.

#### Tools to be used 

*  VM: AWS EC2
*   OS: Ubuntu 20.04 lts+
*    Docker Engine
* ```kubectl```  console utility
* ``cfssl`` and ``cfssljson`` utilities
* Kubernetes cluster

For the cluster, create 6 EC2 instances, which will have the following movings parts properly configured:

* 3 Kubernetes master nodes
* 3 kubernetes worker nodes
* Configured SSl/TLS certificates for kubernetes components to communicate securely
* Configured node network
* Configured Pod network.

####  Prerequisite client tools

* [awscli](https://aws.amazon.com/cli/) - is a unified tool to manage your AWS services.
* [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) - this command line utility will be your main control tool to manage your K8s cluster.
* [cfssl](https://blog.cloudflare.com/introducing-cfssl/) - an open source toolkit for everything TLS/SSL from ```Cloudflare```
* [cfssljson](https://github.com/cloudflare/cfssl) - a program, which takes the JSON output from the ```cfssl``` and writes certificates, keys, ```CSRs```, and bundles to disk.
 
 **Install AWS CLI** - 
 **Configure AWS CLI -** *An IAM profile with a programmatic access keys enabled should be created prior to the aws cli configuration,because these keys will be used to set up the aws environment in using the cli*

 ```
 aws configure
 ```
 *Set your the access key ID, Secret access key and region name. That should do for now*

 **Test that the CLI  is running**
 ```
 aws ec2 describe-vpcs
 ```

 and check if you can see VPC details.

**Install kubectl**
For install guide, click *[here](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)*
Kubernetes cluster has a Web API that can receive HTTP/HTTPS requests, but it is quite cumbersome to curl an API each and every time you need to send some command, so kubectl command tool was developed to ease a K8s administrator's life.
With this tool you can easily interact with Kubernetes to deploy applications, inspect and manage cluster resources, view logs and perform many more administrative operations.

**Install CFSSL and CFSSLJSON**

``cfssl`` is an open source tool by ``Cloudflare`` used to setup a Public Key Infrastructure (PKI Infrastructure) for generating, signing and bundling TLS certificates. In previous projects you have experienced the use of Letsencrypt for the similar use case. Here, cfssl will be configured as a Certificate Authority which will issue the certificates required to spin up a Kubernetes cluster.
Download, install and verify successful installation of cfssl and cfssljson:

**Mac OS X**
```
curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssl
curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssljson
```

```
chmod +x cfssl cfssljson
```

```
sudo mv cfssl cfssljson /usr/local/bin/
```

If you have issues using the binaries directly, you should consider using the package manager Homebrew and that might be a better option:
```
brew install cfssl
```


Verify that cfssl version 1.4.1 or higher is installed:
Output:
```
cfssl version
```  
```
Version: 1.4.1
Runtime: go1.12.12
```
```
cfssljson --version
```
```
Version: 1.4.1
Runtime: go1.12.12
```

**Linux Or Windows using Gitbash or similar tool** 
```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
```

```
chmod +x cfssl cfssljson
```

```
sudo mv cfssl cfssljson /usr/local/bin/
```