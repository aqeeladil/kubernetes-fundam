# Nginx Pod Deployment

1. **Apply the files:**
NOTE: In a CI/CD pipeline, these files get executed automatically during deployment:
```bash
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f pod.yaml
kubectl apply -f service.yaml
```

2. **Access and verify the Application**
```bash
# Find the NodePort assigned to the service
kubectl get service nginx-service

# Open the application in browser using the cluster node's IP and the NodePort
http://<node-ip>:<node-port>

# Check the logs of the Pod to verify environment variables and volumes
kubectl logs nginx-app

# Verify that the application reads configuration and credentials correctly without exposing sensitive data in logs.
```



