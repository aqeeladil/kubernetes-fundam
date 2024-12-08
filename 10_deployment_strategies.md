# Kubernetes Deployment Strategies

Deployment strategies are essential for minimizing downtime during application upgrades and ensuring reliability. 

Without a strategy, upgrading applications can lead to significant downtime, potentially affecting user experience and incurring revenue loss for organizations like Amazon or Instagram.

____________________________________________________________________________________________

## 1. Rolling Update Strategy

- Rolling update is default strategy in kubernetes.
- Best for less critical updates with near-zero downtime.

#### How It Works:

- Adds a new replica (e.g., V2) incrementally, while gradually removing old replicas (e.g., V1).
- By default, updates 25% of replicas at a time (configurable using `maxUnavailable` and `maxSurge` parameters).
- Maintains service availability by ensuring that only a fraction of replicas are replaced simultaneously.

#### Advantages:

- Near-zero downtime.
- Retains older versions until new versions are confirmed stable.
- Handles failures gracefully: if a new version fails, older replicas continue serving traffic.

#### Disadvantages:

- Limited control over traffic distribution during transitions.
- Does not allow for thorough testing of new versions with production data.

#### Example:

- Deploy version V1 with four replicas and upgrade to V2 incrementally.
- Outcome: Pods are replaced one by one, ensuring continuous service availability.

```bash
# Deploy version 1 of your application:
kubectl create deployment myapp --image=nginx:1.19
kubectl scale deployment myapp --replicas=4

# Verify the deployment
kubectl get pods

# Get the Minikube IP and the NodePort
# Access the application in your browser at `http://<MINIKUBE-IP>:<NODEPORT>`.
minikube ip
kubectl get svc myapp

# Update the deployment to version 2
kubectl set image deployment/myapp nginx-container=nginx:1.21

# Monitor the rolling update
# Kubernetes updates the pods incrementally (e.g., 1 pod at a time with 25% surge)
kubectl rollout status deployment/myapp

# Refresh the application URL to confirm the updated version

# Roll back to the previous version if needed
kubectl rollout undo deployment/myapp
```
**Troubleshooting:**
```bash
# If pods fail to start, inspect logs
kubectl logs <pod-name>

# Check deployment events
kubectl describe deployment myapp
```

______________________________________________________________________________________________

## 2. Canary Deployment Strategy

- Ideal for gradual testing and user feedback in production.

#### How It Works:

- New versions (e.g., V2) are introduced alongside the old version (e.g., V1).
- A load balancer distributes a small percentage of traffic (e.g., 10%) to the new version.
- Traffic percentages are adjusted incrementally after confirming stability.

#### Advantages:

- Allows controlled, real-time testing with actual production data.
- Limits user impact during failures (e.g., only 10% of users experience issues).
- Provides flexibility to revert to the older version if necessary.

#### Disadvantages:

- Requires configuration of ingress controllers and traffic weights.
- Testing may extend deployment timelines.

#### Example:

- Deploy V1 and V2 alongside each other.
- Configure ingress annotations to control traffic distribution.
- Outcome: 10% of traffic is routed to V2 while 90% continues to V1. Traffic can be adjusted incrementally.

```bash
# Deploy version 1 (current production version):
kubectl create deployment myapp-v1 --image=nginx:1.19
kubectl scale deployment myapp-v1 --replicas=4

# Deploy version 2 (canary version):
kubectl create deployment myapp-v2 --image=nginx:1.21
kubectl scale deployment myapp-v2 --replicas=1

# Expose both versions using services:
kubectl expose deployment myapp-v1 --port=80 --target-port=80 --type=ClusterIP --name=myapp-v1-svc
kubectl expose deployment myapp-v2 --port=80 --target-port=80 --type=ClusterIP --name=myapp-v2-svc

# Install and enable an ingress controller (e.g., NGINX):
minikube addons enable ingress
```
Create an ingress resource `ingress-canary.yaml` for traffic splitting:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-v1-svc
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-v2-svc
            port:
              number: 80
```
```bash
# Apply the ingress resource
kubectl apply -f ingress-canary.yaml

# Verify traffic distribution
# Map the host in `/etc/hosts`
minikube ip
<MINIKUBE-IP> myapp.local
curl -H "Host: myapp.local" http://<MINIKUBE-IP>

# Gradually increase the traffic to version 2
# Change `nginx.ingress.kubernetes.io/canary-weight` to a higher value (e.g., 30, 50, 100).
kubectl edit ingress myapp-ingress
```
**Troubleshooting:**
```bash
# If traffic doesn’t split correctly, check ingress logs
kubectl logs <nginx-ingress-pod-name>

# Check deployment events
kubectl describe deployment myapp-v1
kubectl describe deployment myapp-v2

# Inspect events for failures
kubectl get events
```

_______________________________________________________________________________________________

## 3. Blue-Green Deployment Strategy

- The Safest But Most Expensive Option.
- Ideal for high-stakes environments (e.g., banking) where downtime is unacceptable.

#### How It Works:

- Deploys a duplicate environment (green) with the new version, while maintaining the existing environment (blue).
- Traffic is switched between environments by updating the load balancer configuration.

#### Advantages:

- Instant rollback: revert traffic to the old version by switching the load balancer.
- Zero impact during deployment failures.

#### Disadvantages:

- High resource consumption: requires maintaining duplicate environments.
- Expensive for organizations with multiple services.

#### Example:

- Deploy separate services for V1 (blue) and V2 (green).
- Point the load balancer to the green service after deployment.
- Outcome: Traffic is instantly switched between environments, ensuring zero downtime.

```bash
# Deploy version 1 (blue environment)
kubectl create deployment myapp-blue --image=nginx:1.19
kubectl scale deployment myapp-blue --replicas=4
kubectl expose deployment myapp-blue --port=80 --target-port=80 --type=ClusterIP --name=myapp-blue-svc

# Deploy version 2 (green environment)
kubectl create deployment myapp-green --image=nginx:1.21
kubectl scale deployment myapp-green --replicas=4
kubectl expose deployment myapp-green --port=80 --target-port=80 --type=ClusterIP --name=myapp-green-svc

# Install and enable an ingress controller (e.g., NGINX):
minikube addons enable ingress
```
Create an ingress `ingress-blue.yaml` pointing to the blue service.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-blue-svc
            port:
              number: 80
```
```bash
# Apply the ingress resource:
kubectl apply -f ingress-blue.yaml

# Switch to the green service:
# Change name: `myapp-blue-svc` to name: `myapp-green-svc`.
kubectl edit ingress myapp-ingress

# Roll back to blue if needed by reverting the service name.

# Verify traffic distribution
# Map the host in `/etc/hosts`
minikube ip
<MINIKUBE-IP> myapp.local
curl -H "Host: myapp.local" http://<MINIKUBE-IP>
```
**Troubleshooting:**
```bash
# Check pods
kubectl get pods
kubectl logs <pod-name>

# Validate deployment and service health
kubectl describe deployment myapp-blue
kubectl describe svc myapp-blue-svc

kubectl describe deployment myapp-green
kubectl describe svc myapp-green-svc

# Inspect events for failures
kubectl get events
```
