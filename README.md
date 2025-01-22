# Introduction to Kubernetes

- Kubernetes is the key player in DevOps**.

- Kubernetes, often abbreviated as K8s, is an open-source platform developed by Google to automate the deployment, scaling, and management of containerized applications. 

- It provides a robust and flexible system for managing workloads and services in clusters of containers, making it especially valuable in environments where scalability, resilience, and efficient resource use are essential.

- Kubernetes has become a cornerstone of modern cloud-native architecture due to its ability to manage and automate complex containerized applications at scale.

## Docker and Kubernetes difference?

- Docker is a platform for developing, shipping, and running containerized applications, focusing on managing individual containers on a single host. 

- Kubernetes (K8s) is an orchestration tool that automates the deployment, scaling, and management of containerized applications across a cluster of machines. 

- While Docker is ideal for building and running containers, Kubernetes provides advanced scheduling, networking, and storage options to manage complex applications.

- Docker runs standalone containers, whereas Kubernetes groups containers into "pods" for efficient management.

- Both can work together, with Docker handling containerization and Kubernetes orchestrating across clusters.

## Problem with Containers/Docker.

- **Single Host**: Containers are ephemeral in nature (short life) (can die & revive anytime). Because of single host nature of docker, a container can impact other containers. e.g. extra memory usage by a container can result in end-life for an other container.

- **No Auto-healing**: If a container get impacted/killed, the application running inside will be inaccesible, and you manually need to restart it again. Here kubernetes comes up with the concept of auto-healing (a container automatically get started without any manual intervention).

- **No Auto-scaling**: On Docker, you need to manually increase the container count and setup the load-balancer if there is increased incoming traffic on the application.

- **No Enterprise Support**: Docker doesn't provide any Enterprise level application support (load-balancer, firewall, auto-heealing, auto-scaling, API-gateways). Docker is excellent for packaging and running containers, it lacks features required for managing complex, scalable applications in a production environment.

## Benefits of Kubernetes.

- **Scalability:** Automatically scales applications up or down based on load.

- **High Availability:** Ensures resilience with self-healing, where failed pods are automatically replaced.

- **Portability:** Works across multiple environments (cloud, on-premises).

- **Resource Efficiency:** Optimizes hardware usage with intelligent scheduling and resource management.

- **Enterprise Support** Kubernetes enhances Docker by providing enterprise-level features making it ideal for managing complex, scalable applications in production.

    - Automated scaling
    - Self-healing
    - Declarative configuration
    - Rolling updates and rollbacks
    - Networking and load balancing
    - Storage orchestration
    - Multi-tenancy and security
    - Extensibility with a strong ecosystem
    - Multi-cloud support.

## K8's Architecture (Data Plane | Control Plane).

- **Containers:** Kubernetes is built to manage containers (e.g., Docker). Containers package applications with all dependencies, making them lightweight, consistent, and portable across environments. Kubernetes uses Docker containers but organizes them into pods for better scalability, resilience, and orchestration in large, distributed systems.

- **Pods:** The smallest deployable unit in Kubernetes, a pod, encapsulates one or more 
containers that share resources (like storage and network) and run on the same host. 
Kubernetes deploys, scales, and manages pods rather than individual containers.

- **Node:** A physical or virtual machine that runs containerized applications. Each node in 
Kubernetes has a runtime (e.g., Docker) and an agent (kubelet) to manage pod operations. Inside a 
node, Kubelet makes sure that pod/pods are always up and running.

- **Cluster:** A set of nodes managed by Kubernetes. The cluster is the fundamental structure 
where the orchestration occurs, enabling efficient workload distribution.

### Worker Node Components (Data Plane)

In Kubernetes, a worker node is a machine (physical or virtual) that runs the applications in 
pods and performs essential operations for the cluster. Each worker node has several key 
components that ensure it can host and manage containerized workloads effectively. 

Here are the main components:

1. **Kubelet:** is an agent that runs on each worker node and communicates with the 
Kubernetes control plane(api server). It ensures that containers are running as expected on the node.

2. **Container Runtime:** (e.g., Docker, container-D, or CRI-O) is responsible
for pulling container images from registries and running containers on the worker node.

3. **Kube-proxy:** manages network communication within the cluster, enabling 
service discovery and load balancing for pods on the worker node.

4. **cAdvisor:** is an open-source monitoring agent integrated into the Kubelet, which collects 
performance data for containers running on the node.

### Kube-Proxy | vs | Bridge-Nertwork:

- **Kube-proxy** Kube-proxy is a network component in Kubernetes responsible for routing and load balancing network traffic within a Kubernetes cluster. Like bridge network in docker, we had kube-proxy in Kubernetes.

- Kube-proxy is central to the Kubernetes Service abstraction, which provides stable endpoints 
to clients and distributes traffic across pods for high availability and fault tolerance.

- When a client sends a request to a service, the request is intercepted by kube-proxy on the node where it was received. Kube-proxy then looks up the destination endpoint for the service and routes the request accordingly.

- Kube-proxy is Kubernetes's way of handling service-to-pod traffic across the 
cluster, while bridge networking is Docker’s way of enabling container-to-container 
communication on a single host.

### Control Plane: 

Coordinates the cluster, handling tasks like scheduling workloads, maintaining the desired 
state, and scaling. It includes components such as:

- **API Server:** The interface for users and developers to interact with Kubernetes.

- **Scheduler:** Places containers on appropriate nodes.

- **Controller Manager:** Ensures the cluster’s desired state matches the actual state.

- **etcd:** Stores cluster data and is crucial for configuration consistency.

### Why we need *Control Plane* in kubernetes architecture?

- The control plane is the "brain" of the Kubernetes cluster, responsible for managing, 
scheduling, monitoring, and securing workloads, ensuring that the cluster maintains stability, 
reliability, and resilience at scale(multiple worker nodes).

- The **Cloud Controller Manager (CCM)** is a Kubernetes component that integrates the cluster 
with underlying cloud provider infrastructure. It enables Kubernetes to manage resources in a 
cloud environment, ensuring seamless operation and interaction between the cluster and the 
cloud provider(e.g., AWS, Google Cloud, Azure).

## Kubernetes Distributions

**Here are some Kubernetes distributions commonly used in production:**

1. Red Hat OpenShift

2. Rancher

3. VMware Tanzu

4. Google Kubernetes Engine (GKE)

5. Amazon Elastic Kubernetes Service (EKS)

6. Azure Kubernetes Service (AKS)

7. DigitalOcean Kubernetes

8. IBM Cloud Kubernetes Service.

**Here are some Kubernetes distributions commonly used for local development and testing:**

  1. Minikube

  2. IND (Kubernetes IN Docker)

  3. k3s

  4. MicroK8s

  5. Docker Desktop (Kubernetes)

  6. Rancher Desktop.

**Kubernetes vs EKS**

- Kubernetes requires self-management, while EKS is fully managed by AWS.

- Kubernetes can run anywhere, but EKS is designed for use in AWS environments.

- EKS has additional costs for the managed control plane (charged by AWS), whereas Kubernetes running on your own infrastructure might only incur hardware or cloud instance costs.

## Why Kubernetes prefer YAML over other languages?

In Kubernetes, YAML (YAML Ain't Markup Language) files are used extensively for defining configurations, resources, and desired states of the infrastructure. Here’s why YAML is the preferred choice for Kubernetes:

1. **Human Readability:** Simple, indentation-based syntax makes it easy to read and write.

2. **Declarative Syntax:** Ideal for describing the desired end state of resources, aligning with   Kubernetes' declarative management.

3. **Widely Supported:** Standard format in the Kubernetes ecosystem with extensive tool and   community support.

4. **Complex Data Structuring:** Handles nested configurations well, enabling clear resource   definitions.
  
5. **Less Verbose than JSON:** More concise, reducing file size and making it easier to manage.

6. **Version Control Friendly:** Line-based format works well with Git for tracking and reviewing   changes.

## `kubectl` command

kubectl is the command-line tool for interacting with Kubernetes clusters. It lets you deploy and manage applications, inspect and manipulate cluster resources, and view logs on the Kubernetes cluster. Using kubectl, you can manage Kubernetes objects and configurations.

Here’s a quick overview of how kubectl works and some common commands:

### Configuration: 

- To connect kubectl to a Kubernetes cluster, you usually need a configuration file (often stored in ~/.kube/config), which contains information like cluster server URLs and access credentials.

```bash
kubectl config view               # View your current Kubernetes config
kubectl config use-context <name> # Switch to a different cluster context
```

### Basic Commands

```bash
kubectl get nodes          # List all nodes in the cluster
kubectl get pods           # List all pods in the current namespace
kubectl get services       # List all services in the current namespace
kubectl get namespaces     # List all namespaces
kubectl get pods --all-namespaces       # Get Pods in All Namespaces
kubectl get deployments    # List all deployments

kubectl describe pod <pod_name>          # View detailed information about a specific pod
kubectl describe node <node_name>        # View node details
kubectl logs <pod_name>                  # View logs from a pod
kubectl logs <pod_name> -c <container>   # View logs from a specific container in a pod

kubectl create deployment nginx --image=nginx  # Create a deployment named "nginx" using the nginx image
kubectl scale deployment nginx --replicas=3   # Scale the "nginx" deployment to 3 replicas
kubectl set image deployment/nginx nginx=nginx:1.16.1  # Update the nginx deployment to a different image version

kubectl delete pod <pod_name>            # Delete a specific pod
kubectl delete deployment <deployment>   # Delete a specific deployment
kubectl delete service <service_name>    # Delete a service

kubectl exec -it <pod_name> -- /bin/bash  # Access shell in a pod
kubectl exec <pod_name> -- <command>      # Run a command in a pod, like `kubectl exec <pod> -- ls /app`
kubectl port-forward <pod_name> 8080:80  # Forward local port 8080 to pod's port 80
kubectl apply -f <file.yaml>   # Create or update resources from a YAML file
```

## kubectl Port Forward

Port forwarding is a networking concept that maps a port on one machine to a port on another machine, creating a secure tunnel for communication. In Kubernetes, this allows you to:

The **`kubectl port-forward`** command allows you to map a local port on your machine to a port on a Kubernetes pod or service. It is useful for local testing and debugging, especially when you want to access applications running inside a Kubernetes cluster without exposing them externally.

### Use Cases

```bash
# Local Development
# When developing applications that need to interact with services running in a Kubernetes cluster.
# This maps your local port 8080 to port 80 of the service in the cluster.
kubectl port-forward service/my-service 8080:80

# Remote Cluster Access
# When accessing services running in a remote cluster (e.g., on cloud providers):
# The `--address 0.0.0.0` flag allows connections from any IP address.
kubectl port-forward service/my-service 8080:80 --address 0.0.0.0


# Debugging and Testing
# For temporary access to internal services without modifying service types or ingress rules.
kubectl port-forward pod/my-pod 8080:80
```

### Best Practices

#### 1. Dynamic Port Assignment
If you're unsure about port availability, let Kubernetes choose an available port:
```bash
kubectl port-forward service/my-service :80
```
This will automatically select an available local port.

#### 2. Namespace Specification
Always specify the namespace when working with non-default namespaces:
```bash
kubectl port-forward service/my-service 8080:80 -n my-namespace
```

#### 3. Security Considerations
- Port forwarding should be used primarily for development and debugging
- Avoid using port forwarding in production environments
- Consider using proper ingress controllers for production access
- Be cautious when using `--address 0.0.0.0` as it allows connections from any IP

### Common Use Case: Accessing Internal Services

```bash
# Accessing a Database
# Forward local port 5432 to PostgreSQL running in the cluster
kubectl port-forward service/postgres 5432:5432

# Accessing Web Applications
# Access a web application running in the cluster
kubectl port-forward service/web-app 8080:80
```

### Real-World Example: Accessing Argo CD UI

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 9000:80

# After running these commands, you can access the Argo CD UI at `localhost:9000`
```

## Docker Swarm | vs | Kubernetes:

- Docker Swarm and Kubernetes (K8s) are both orchestration tools for managing containerized applications, but they differ in complexity and functionality. 

- Docker Swarm is easier to set up and manage, ideal for smaller projects needing quick, simpler deployment. Kubernetes, however, provides advanced features like automated scaling, self-healing, and a vast ecosystem, making it suitable for complex, large-scale deployments. 

- Swarm has tighter Docker integration, whereas Kubernetes supports various container runtimes. - For long-term, production-level management, Kubernetes is often preferred due to its flexibility and extensive support.

## Day-to-day Kubernetes tasks:

1. **Managing Cluster Health:** Monitoring the Kubernetes clusters, nodes, and Pods for performance, stability, and availability. This includes using tools like Prometheus, Grafana, or ELK stack for observability.

2. **Deploying Applications:** Utilizing CI/CD pipelines to deploy and roll out applications in Kubernetes environments, ensuring zero downtime and smooth updates through rolling updates or canary deployments.
  
3. **Optimizing Resource Allocation:** Configuring and fine-tuning resource requests and limits for containers, scaling services up or down based on load to maintain cost efficiency and performance.

4. **Troubleshooting and Incident Response:** Diagnosing and resolving issues with applications or cluster resources, including debugging failed Pods, addressing node issues, or resolving network conflicts.

5. **Maintaining Security and Compliance:** Enforcing RBAC (Role-Based Access Control), setting up network policies, and ensuring that cluster security configurations meet organizational compliance requirements.

6. **Automation and Scripting:** Writing and updating scripts (often in Bash, Python, or Go) to automate Kubernetes-related tasks, such as deployments, monitoring setups, or resource management.
  
7. **Documentation and Collaboration:** Documenting Kubernetes configurations, best practices, and troubleshooting procedures, while collaborating with developers to ensure applications are “Kubernetes-ready.”

## Kubeshark

- KubeShark is a tool designed for Kubernetes cluster observability, providing deep network visibility for debugging, monitoring, and securing your Kubernetes environments. 

- It offers real-time, protocol-level visibility, allowing you to capture and monitor traffic between containers, pods, nodes, and across the entire cluster.

- Kubeshark also integrates seamlessly with tools like InfluxDB and Grafana, allowing you to visualize network traffic and create real-time dashboards for further analysis​. 

- Its filtering capabilities let you focus on specific pods or traffic types, which is crucial for optimizing Kubernetes resource management or investigating network anomalies​.

With Kubeshark, you can:

- **Capture and analyze traffic:** Track communications in real time and perform deep packet inspection across your Kubernetes resources.

- **Identify security issues:** Detect suspicious traffic patterns and monitor network behaviors, which is critical for security teams.

- **Debug network issues:** Quickly troubleshoot connectivity issues, including network errors, latency, and packet loss​.

- **Record traffic for offline analysis:** Store and review traffic from past periods, which is useful for auditing, compliance, or detailed post-incident analysis.


## How uers, groups or service-accounts are created in Kubernetes?

In Kubernetes, users, groups, and service accounts are treated differently, and each has its own creation and management methods:

1. **Users:**

    - Kubernetes doesn’t manage users directly; it relies on external systems for user authentication, such as certificates, OAuth providers (e.g., Google, GitHub), or identity providers (e.g., LDAP).
    
    - Typically, you manage users by creating client certificates or using an Identity Provider (IDP) integrated with Kubernetes.
    
    - Identity Providers (IDP) are used for enterprise setups where user management is centralized.

2. **Groups:**

    - Groups in Kubernetes are not explicitly created in the cluster.
    
    - Groups are typically defined within the identity provider or set during the client certificate creation using the O (organization) field in the subject.
    
    - When using an IDP, groups can be assigned at the IDP level, and users are placed into groups based on roles defined in the provider.

3. **Service Accounts:**

    - Service accounts are Kubernetes-native accounts designed for workloads running within the cluster (like pods).

    - Service accounts are managed directly by Kubernetes, and they are created within specific namespaces.

    - By default, each namespace has a default service account, but you can create additional service accounts as needed.

Creating a Service Account: `kubectl create serviceaccount <service-account-name> -n <namespace>`

Once created, service accounts are assigned to pods in your deployment or pod spec, allowing the pods to authenticate with the Kubernetes API.

