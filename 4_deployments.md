
# K8s Deployments

## Container | Pod | Deployments

**Containers** are lightweight, standalone, and executable packages that include an application 
and its dependencies (libraries, binaries, and configuration files). Containers isolate 
applications from their environment, making them highly portable.

A **Pod** is the smallest deployable unit in Kubernetes and typically represents a single 
instance of a running process in the cluster. Pods can contain one or more containers that 
share the same network namespace, IP address, and storage volumes.

A **Deployment** in Kubernetes manages Pods by defining the desired state for an application, 
like the number of replicas (instances), the container image version, and other configurations. 

- The **Deployment controller** handles rolling updates, scaling, and self-healing to maintain the 
desired state.
- Kubernetes Deployments are essential for managing and scaling applications in a Kubernetes 
cluster. They provide a declarative way to define the desired state of your applications, 
automating the creation, update, and rollback of Pods to ensure the system reaches the 
desired state.
- Example: A Deployment for a web app might specify to maintain five replicas of a Pod, 
automatically scaling up if demand increases and rolling back to a previous version if the 
latest update fails.

## Controllers In Kubernetes:

**Deployment Controller:**
- Manages stateless apps with rolling updates.
- Web servers, APIs

**ReplicaSet Controller:**
- Ensures desired replica count	
- Standalone replicas (or via Deployments)

**StatefulSet Controller:**
- Manages stateful apps needing stable identity	
- Databases, message queues

**DaemonSet Controller:**	
- Runs a Pod on each node	
- Logging, monitoring, node utilities

**Job Controller:**	
- Runs tasks to completion	
- Batch processing

**CronJob Controller:**	
- Schedules Jobs at fixed intervals	
- Backups, maintenance tasks

## Key Concepts of Kubernetes Deployments

  1. **Declarative Approach:** In a Deployment, you define the desired state of your application, including the number of replicas, container images, and update strategies. Kubernetes then automatically maintains that state.

  2. **Replica Management:** Deployments manage the desired number of replicas. If any Pods crash or go offline, Kubernetes will recreate them to match the replica count.

  3. **Rolling Updates and Rollbacks:** Deployments support rolling updates, allowing you to update your application without downtime. If something goes wrong, you can easily roll back to a previous version.

  4. **Self-Healing:** Kubernetes continuously monitors Pods and replaces them if they fail, keeping your application available.

## Deployment Use Cases:

- **Scaling** web applications by adjusting replicas
- **Zero-downtime** updates using rolling updates
- **Continuous delivery and deployment** by integrating with CI/CD tools like Jenkins or GitLab CI.
<br>

## Creating a Basic Deployment

Here’s a simple example to create a Deployment for an Nginx web server.

#### Deployment YAML File
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.10
        ports:
        - containerPort: 80
```
#### Applying the Deployment:
```bash
kubectl apply -f nginx-deployment.yaml
```
#### Verifying the Deployment:
```bash
kubectl get deployments
kubectl get pods
kubectl get rs
kubectl get all
kubectl get all -A
kubectl get pods -o wide
kubectl get pods -w
```
You can use any of the above commands accordingly.
#### Scaling the Deployment:
To adjust the number of replicas (for example, increasing to 5), run:
```bash
kubectl scale deployment nginx-deployment --replicas=5
```
#### Updating the Deployment:
To update the image, edit the YAML file (e.g., to nginx:1.21.1) and reapply it:
```bash
kubectl apply -f nginx-deployment.yaml
```
This triggers a rolling update, ensuring smooth transitioning of Pods to the new version.

#### Rolling Back:
If the update causes issues, you can roll back to the previous version using:
```bash
kubectl rollout undo deployment nginx-deployment
```

## What is a namespace in k8s?

In Kubernetes, a namespace is a way to partition a single Kubernetes cluster into multiple virtual clusters, allowing resources to be organized and isolated. Namespaces are useful in multi-tenant environments or when working with multiple teams or projects in the same cluster. They provide a scope for resources, so resources in different namespaces can have the same name without conflict.

For example, within the same cluster, you could have two ```pod``` resources named ```frontend``` in two different namespaces, ```dev``` and ```prod```.

