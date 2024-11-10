# KUBERNETES PODS

Kubernetes Pods are the smallest deployable units in Kubernetes, encapsulating containers (typically one or a small set) that run together, sharing resources and a network namespace. 

- A Pod is a group of one or more containers that share storage, network, and specifications on how to run.
- Containers in a Pod can communicate with each other using localhost as they share the same IP and port space.
- Containers in the same Pod share storage volumes and configurations like secrets and environment variables.

### Use Pods?

- Pods group containers that need to work closely together.
- They provide isolation within a Kubernetes cluster by abstracting application components that should be managed together, making them resilient and scalable.

### Example: Web Server and Sidecar Logging Container:

Imagine you have a web server that logs activity. To handle logs better, you add a sidecar container that collects and ships logs to a central logging system.

- The main container runs an NGINX server, serving web content.
- A sidecar container in the same Pod watches the log files and forwards them to a monitoring service.

These containers:

- Share the same storage volume (so the sidecar container can access the logs).
- Share network space, allowing the sidecar to operate in close sync with the main container.

## Creating a Simple NGINX Pod using Minikube and Aws-Ec2

Here we will create a basic Kubernetes Pod with a single NGINX container and deploy it to a Kubernetes cluster (Minikube). You’ll need kubectl and access to a Kubernetes cluster (like Minikube or a cloud provider). Here we will use Minikube on a Ec2-instance.

#### Step 1: Launch an EC2 Instance

Launch an EC2 instance with at least 2 vCPUs and 2 GB of RAM (t2.medium is a good choice for testing).
- Access the instance using SSH.
```bash
ssh -i your-key.pem ec2-user@your-ec2-instance-public-ip
```

#### Step 2: Install Dependencies on EC2

Run the following commands to install Docker, Minikube, and kubectl:

1. **Install Docker:**
```bash
sudo yum update -y
sudo yum install -y docker
sudo service docker start
sudo usermod -aG docker ec2-user
```
Log out and log back in for Docker permissions to apply.
2. **Install Minikube:**
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
sudo mv minikube /usr/local/bin/
```
3. **Install Kubectl:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

#### Step 3: Start Minikube on the EC2 Instance

Start Minikube with Docker as the driver
```bash
minikube start --driver=docker --cpus=2 --memory=2048
```

#### Step 4: Create a YAML Configuration for the Pod

Here’s the ```nginx-pod.yaml``` file, defining a Pod with a single NGINX container:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

#### Step 5: Deploy the Pod

Apply the configuration file:
```bash
kubectl apply -f nginx-pod.yaml
```
This command instructs Kubernetes to deploy the Pod with an NGINX container.

#### Step 6: Verify the Pod is Running

Check if the Pod is up and running on EC2:
```bash
kubectl get pods
```
You should see your Pod nginx-pod with a status of Running.

#### Step 7: Access the NGINX Web Server

To access the NGINX server, you can either use port forwarding or expose it via a service.

- *Port Forwarding:*
```bash
kubectl port-forward pod/nginx-pod 8080:80
```
Now, you can access the server at ```http://your-ec2-instance-public-ip:8080```.
You should see the NGINX welcome page if the Pod is running successfully.

<br>

#### Step 8: Scaling with Replicas
If you want multiple instances of the NGINX Pod, you can define a Deployment, which manages replicas and allows scaling.

Here’s a Deployment configuration for multiple replicas:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
          image: nginx:latest
          ports:
            - containerPort: 80
```

To apply the deployment, run:
```bash
kubectl apply -f nginx-deployment.yaml
```

#### Step 9: Cleaning Up
When you’re done, you can delete the Pod and Deployment with:
```bash
kubectl delete pod nginx-pod
kubectl delete deployment nginx-deployment
```