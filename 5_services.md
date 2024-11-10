# KUBERNETES SERVICES (svc) | DISCOVERY | LOAD BALANCING | NETWORKING

Kubernetes Services are a foundational element for ensuring stable, scalable, and accessible 
applications within and outside the cluster. 

In Kubernetes (K8s), Services are used to expose applications running on a set of Pods, 
ensuring that they are accessible to other parts of the cluster or even external clients. 
Here’s a breakdown of K8s Services and the types available:

## Types of Kubernetes Services:

1. **ClusterIP (default)**
- Purpose: Exposes the Service only within the Kubernetes cluster.
- Use Case: Ideal for internal communication between applications within the cluster.
- Example: Microservices that need to talk to each other within the cluster.

2. **NodePort**
- Purpose: Exposes the Service on a specific port of each Node's IP address.
- Use Case: Used for exposing an application to external traffic by exposing it through a 
high-numbered port on each node.
- Example: Useful for quick testing from outside the cluster (not generally recommended for 
production).

3. **LoadBalancer**
- Purpose: Exposes the Service externally using a cloud provider's load balancer.
- Use Case: Ideal for production applications on cloud providers like AWS, Google Cloud, or 
Azure where managed load balancers are available.
- Example: Running a web application accessible to users outside the cluster.

4. **ExternalName**
- Purpose: Maps a Service to an external DNS name.
- Use Case: Used to proxy a service within the cluster to an external service.
- Example: Redirecting traffic to a third-party service without creating an endpoint within the 
cluster.

## Key Components of a Service:

1. **Selector:** 
- Defines which Pods will be targeted by the Service based on labels.
2. **Endpoints:** 
- Dynamically tracks the IP addresses of the selected Pods.
3. **Cluster IP:** 
- The internal IP assigned to the Service (only for ClusterIP, NodePort, and LoadBalancer types).

## Creating a Basic Service

Here’s an example YAML file to create a ClusterIP Service: ```service.yml```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
This configuration selects Pods labeled with ```app: my-app```, exposing port 8080 on the Pods 
to port 80 on the Service.

## Service Discovery in Kubernetes:

K8s automatically provides DNS entries for Services. For example, if you create a Service named 
```my-service``` in a namespace default, it can be accessed by ```my-service.default.svc.cluster.local```.

## Workflow:

- **Purpose:** Services provide a stable IP address and DNS name to connect to a dynamic set of 
Pods that may change over time (due to scaling, updates, etc.). They enable decoupling between 
client and server, making it easy to connect clients to Pods without worrying about individual 
Pod IP addresses.
- **How it Works:** Services select a set of Pods using selectors (labels and selectors 
mechanism). K8s then manages load balancing across these Pods to distribute traffic.

## Why We Need a Service in Kubernetes?

- **Stable Network Endpoint:** Pods in Kubernetes are ephemeral; they can be created and 
destroyed as needed, often receiving new IP addresses. A Service provides a stable, unchanging 
IP (or DNS name) that abstracts the constantly changing set of Pods. This makes it easier to 
access Pods reliably.
- **Load Balancing:** Services automatically distribute incoming traffic across multiple Pods, 
balancing the load and helping to prevent a single Pod from becoming a bottleneck.
- **Decoupling Components:** Services allow you to define how different components of your 
application communicate with each other without directly referencing individual Pods. This 
promotes better scaling and isolation within your microservices architecture.
- **External Access:** If you want your application to be accessible from outside the Kubernetes 
cluster, a Service (like a NodePort, LoadBalancer, or Ingress) is essential for exposing Pods 
to external traffic.

##  What If There Is No Service in Kubernetes?

- **Pod IP Changes:** Without a Service, you’d need to track each Pod's IP manually, which 
would change every time the Pod restarts. This would be highly unreliable and operationally 
impractical.
- **No Load Balancing:** Requests would hit individual Pods rather than being distributed 
across replicas, increasing the risk of overload on certain Pods and decreasing reliability.
- **Lack of Scalability:** If your application grows, you would need to manually update 
configurations whenever new Pods are created or removed, instead of relying on a Service to 
automatically route traffic to available Pods.
- **Limited External Access:** You would lack a straightforward way to expose Pods to the 
outside world, complicating the process of making your application accessible.

