# KUBERNETES INGRESS

Kubernetes Ingress is an API object that manages external access to services within a Kubernetes cluster, typically 
HTTP and HTTPS traffic. It allows you to define how incoming traffic should be routed to different services based 
on the request's hostname or URL path. It helps manage external access to your applications in a more controlled 
and flexible way than exposing services individually.

- **Centralized Management:** You can manage the routing of all incoming traffic in a single resource.
- **Security:** You can define SSL certificates, authentication, and other security features within the Ingress resource.
- **Scalability:** As your application grows, Ingress allows you to easily add more routes or services

## Key Features of Ingress:

#### Path-based routing: 
- Route requests based on URL paths.
#### Host-based routing: 
- Route requests based on hostnames (e.g., ```app1.example.com``` and ```app2.example.com```).
#### TLS termination: 
- Ingress can handle SSL/TLS encryption, offloading it from the backend services.
#### Load balancing: 
- Provides load balancing for traffic routing to backend services.

## Key Concepts:

#### Ingress Controller: 
- A reverse proxy that processes the Ingress resource and routes the traffic based on its configuration. Popular 
Ingress controllers include NGINX, Traefik, and HAProxy.

#### Ingress Resource: 
- Defines how external HTTP/S traffic should be routed to services in the cluster. You can configure paths, 
hostnames, and even SSL termination in the Ingress resource.

## Example
Let's say you have two services, ```service1``` and ```service2```, and you want to route traffic to them based on 
the URL path. For example, traffic to ```/app1``` should go to ```service1```, and traffic to ```/app2``` should go 
to ```service2```.

1. **Create Services:**
First, create the services you want to expose. ```service.yaml```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: service1
spec:
  selector:
    app: app1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: service2
spec:
  selector:
    app: app2
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
```bash
kubectl apply -f service.yaml
```
2. **Create Ingress Resource:**
Now, create the Ingress resource ```ingress.yaml``` to route traffic to these services based on the path.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
        - path: /app1
          pathType: Prefix
          backend:
            service:
              name: service1
              port:
                number: 80
        - path: /app2
          pathType: Prefix
          backend:
            service:
              name: service2
              port:
                number: 80
```
```bash
kubectl apply -f ingress.yaml
```
3. **Deploy Ingress Controller:**
To use an Ingress resource, you need an Ingress Controller running in your cluster. For instance, you can install 
the NGINX Ingress Controller with the following command:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```
4. **Access Your Application:**
Once the Ingress resource and the Ingress Controller are set up, you can access the services through the defined 
hostname (```myapp.example.com```) and paths (```/app1 and /app2```).
<br>

## Challenges before Ingressand how Ingress solved these problems?

Before the introduction of Ingress in Kubernetes, managing external access to services was complicated and inefficient. 
Kubernetes had other methods to expose services, but they lacked flexibility, scalability, and fine-grained control 
over traffic routing.

1. **Exposing Services via NodePort:**
- Problem: If you wanted to expose a service outside the cluster, one option was to use a NodePort service type, 
where a port is opened on every node in the cluster.
- Ingress Solution: Ingress eliminates the need to expose multiple NodePorts for every service. It allows you to use 
a single entry point (via an Ingress Controller) for routing traffic to multiple services based on the request's 
hostname or URL path.

2. **Exposing Multiple Services:**
- Problem: To expose multiple services (e.g., service1 and service2), you would either need to use different 
NodePorts or create multiple LoadBalancer services. Each method required more configuration, leading to potential 
complexity and wasted resources. With LoadBalancer services, you would need an external load balancer for each service, which could result in 
increased costs and complexity.
- Ingress Solution: Ingress allows you to expose multiple services using a single external IP and a single entry 
point (Ingress Controller). It enables fine-grained routing rules based on the host or path 
(e.g., ```app1.example.com``` → ```service1```, ```app2.example.com``` → ```service2```), simplifying the routing 
and reducing the need for multiple external resources.

3. **Complex Load Balancing:**
- Problem: Kubernetes didn’t have a built-in way to perform sophisticated HTTP(S) load balancing across services. You
could configure services with external load balancers, but this often required additional setup and wasn’t native to 
Kubernetes.
- Ingress Solution: Ingress controllers, such as NGINX or Traefik, provide advanced load balancing capabilities out 
of the box. They distribute HTTP/HTTPS traffic across multiple backend services based on routing rules, helping to 
balance load and manage traffic more efficiently.

4. **TLS/SSL Management:**
- Problem: If you needed SSL termination (i.e., handling HTTPS traffic), you had to configure this manually for each 
service. You could use an external load balancer or a service with SSL, but managing certificates was difficult and 
inconsistent across services.
- Ingress Solution: Ingress makes SSL termination much easier by allowing you to define HTTPS routes and manage SSL 
certificates within the Ingress resource itself. You can specify which hosts should use SSL and configure SSL 
certificates for them.

5. **Lack of Path-Based Routing:**
- Problem: If you wanted to route traffic based on specific URL paths (e.g., ```/app1``` → 
```service1```, ```/app2``` → ```service2```), this was not easily possible with the older 
Kubernetes services like ClusterIP, NodePort, or LoadBalancer.
- Ingress Solution: Ingress supports path-based routing natively. You can define specific URL 
paths in the Ingress resource, and the Ingress controller will route traffic to the appropriate 
backend service based on those paths.









