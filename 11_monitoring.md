# Introduction to Kubernetes Monitoring

Monitoring is essential in DevOps, especially when working with Kubernetes clusters, to ensure the health and performance of workloads.

Prometheus and Grafana are popular tools used for monitoring and visualization:

- Prometheus: An open-source monitoring and alerting system with a time-series database.
- Grafana: A visualization tool that retrieves data from Prometheus (and other sources) to display it on customizable dashboards.
- Prometheus fetches data from Kubernetes API server (default metrics). Grafana pulls data from Prometheus and displays it.

[Prometheus Docs]("https://prometheus.io/docs/introduction/overview/")

[Grafana Docs]("https://grafana.com/docs/grafana/latest/")

## Pre-Requisite

- Kubernetes Cluster (e.g., minikube)
- Helm 

If you don't have them installed. Follow the below links:

[Install Minikube]("https://minikube.sigs.k8s.io/docs/start/")

[Install Helm]("https://helm.sh/docs/intro/install/")

<br>

## Why Monitoring Matters

Monitoring your Kubernetes cluster is essential for ensuring the health and performance of your applications and infrastructure. Single Kubernetes clusters are manageable, but monitoring becomes critical in environments with multiple clusters (e.g., development, staging, production). Here are some reasons why monitoring your Kubernetes cluster is important:

DevOps engineers need metrics to:
Identify issues in deployments, services, or clusters.
Generate alerts for flaky behavior or outages.

- **Identify issues and troubleshoot:** By monitoring your Kubernetes cluster, you can quickly identify issues in deployments, services, or clusters (such as application crashes, resource bottlenecks, and network problems). With real-time monitoring, you can troubleshoot issues before they escalate and impact your users.

- **Optimize performance and capacity:** Monitoring allows you to track the performance of your applications and infrastructure over time, and identify opportunities to optimize performance and capacity. By understanding usage patterns and resource consumption, you can make informed decisions about scaling your infrastructure and improving the efficiency of your applications.

- **Ensure high availability:** Kubernetes is designed to provide high availability for your applications, but this requires careful monitoring and management. By monitoring your cluster and setting up alerts, you can ensure that your applications remain available even in the event of failures or unexpected events.

- **Security and compliance:** Monitoring your Kubernetes cluster can help you identify potential security risks and ensure compliance with regulations and policies. By tracking access logs and other security-related metrics, you can quickly detect and respond to potential security threats.

<br>

## Why Prometheus over other monitoring tools ?

Prometheus is a popular choice for Kubernetes monitoring for several reasons:

- **Open-source:** Prometheus is an open-source monitoring and alerting system that helps you collect and store metrics about your software systems and infrastructure, and analyze that data to gain insights into their health and performance. It is free to use and has a large community of contributors. This means that you can benefit from ongoing development, bug fixes, and feature enhancements without paying for a commercial monitoring solution.

- **Native Kubernetes support:** Prometheus is designed to work seamlessly with Kubernetes, making it easy to deploy and integrate with your Kubernetes environment. It provides pre-configured Kubernetes dashboards and supports auto-discovery of Kubernetes services and pods.

- **Powerful query language:** Prometheus provides a powerful query language that allows you to easily retrieve and analyze metrics data. This allows you to create custom dashboards and alerts, and to troubleshoot issues more easily.

- **Scalability:** Prometheus is designed to be highly scalable, allowing you to monitor large and complex Kubernetes environments with ease. It supports multi-node architectures and can handle large volumes of data without significant performance degradation.

- **Integrations:** Prometheus integrates with a wide range of other tools and systems, including Grafana for visualization, Alertmanager for alerting, and Kubernetes API server for metadata discovery. With Prometheus, you can easily monitor metrics such as CPU usage, memory usage, network traffic, and application-specific metrics, and use that data to troubleshoot issues, optimize performance, and create alerts to notify you when things go wrong.

<br>

## Prometheus Architecture

![Alt text](https://prometheus.io/assets/architecture.png)

- Prometheus Server: Scrapes metrics (data) from Kubernetes objects exposed by the Kubernetes API server or other metric sources. It stores metrics in a time-series database.
- Pushgateway: For non-scrapable jobs.
- Alertmanager: Handles alerts and notifications (send alerts via platforms like Slack or email).
- Data Exporters: For custom metrics.
- Visualization Tools: Often paired with Grafana for enhanced visualization.

**Cube State Metrics:** 
- This component provides metrics about Kubernetes objects (e.g., deployments, replicas, services) that are not exposed by default by the Kubernetes API server.
- Expose Cube State Metrics as a NodePort and configure Prometheus to scrape it.

**Custom Application Metrics:** 
- Developers can expose application-specific metrics via /metrics endpoints using Prometheus libraries.
- Add these endpoints to Prometheus configurations to collect custom metrics.

**PromQL:**
- Prometheus Query Language (PromQL) is used to query metrics from the time-series database.
- Predefined queries are included in Grafana dashboards for ease of use.

<br>

## What is Grafana ?

Grafana is a popular open-source data visualization and analytics platform that allows you to create custom dashboards and visualizations based on a variety of data sources. Grafana is often used for monitoring and analyzing metrics and logs in real-time, making it an ideal tool for monitoring systems and applications, including Kubernetes environments.

Grafana supports a wide range of data sources, including databases, time-series databases, and other data storage systems. It provides a powerful query language that allows you to retrieve and analyze data from these sources, and to create custom dashboards and alerts based on that data.

In addition to its powerful data visualization and analysis capabilities, Grafana is also highly extensible. It supports a wide range of plugins and integrations, including integrations with popular monitoring and logging tools like Prometheus, Elasticsearch, and InfluxDB.

While Prometheus can display raw data, Grafana converts it into user-friendly charts and graphs, ideal for dashboards.
You can use predefined dashboards from the Grafana repository (like ID 3662) to quickly visualize common Kubernetes metrics.

<br>

## Installation Guide (Prometheus & Grafana)

### Install Prometheus using Helm

#### 1. Add the Helm Repository

`helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`

#### 2. Update Helm repo

`helm repo update`

#### 3. Install Prometheus

`helm install prometheus prometheus-community/prometheus`

- The Helm chart will install the Prometheus stack, including various components like the Prometheus server, alertmanager, and pushgateway.

#### 4. Expose Prometheus Service

The provided command will expose the Prometheus server via a NodePort, which allows external access to Prometheus on your browser. Here's the corrected version:

`kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext`

- Access Prometheus in your browser using `<Node_IP>:<NodePort>`.
- `--type=NodePort`: Exposes the service on a random high port on your node.
- By default, the Prometheus chart doesn’t set up any authentication or password. 

```bash
# Check the port assigned to the service

kubectl get svc prometheus-server-ext
```

**Note:** For production-grade setups, using an Ingress controller (like NGINX) is recommended instead of NodePort for better flexibility and security.

<br>

### Install Grafana using Helm

#### 1. Add the Helm Repository

`helm repo add grafana https://grafana.github.io/helm-charts`

#### 2. Update Helm repo

`helm repo update`

#### 3. Install Grafana

`helm install grafana grafana/grafana`

- Note: If you want to customize Grafana configurations (e.g., admin username/password, persistence, etc.), provide a custom `values.yaml` file
- `helm install grafana grafana/grafana -f values.yaml`

#### 4. Expose Grafana Service

```bash
kubectl expose service grafana \
    --type=NodePort \
    --target-port=3000 \
    --name=grafana-ext
```

#### 5. Access Grafana

`kubectl get svc grafana-ext`

By default, Grafana uses the username `admin` and a randomly generated password. Retrieve the password with:
`kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo`

Navigate to `http://<Node-IP>:<NodePort>` to access the Grafana UI.

#### 6. Port Forwarding (If not exposing the service)

`kubectl port-forward service/grafana 3000:80`

This will allow access to Grafana at `http://localhost:3000`.

<br>

## Kubernetes monitoring setup on an EC2 Ubuntu machine

### 1. Launch the EC2 Instance:

- Use a t2.medium instance or higher (ensure at least 4GB RAM).
- Open the following ports in the security group: `30000-32767` (for NodePort services like Prometheus and Grafana), `22` (for SSH)

```bash
# SSH into the EC2 Instance
ssh -i <your-key.pem> ubuntu@<ec2-public-ip>

# Update the System
sudo apt update && sudo apt upgrade -y
```

### 2. Install Prerequisites

```bash
# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu

# Install Kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 3. Set Up the Kubernetes Cluster

`minikube start --driver=docker --memory=4096`
`kubectl get nodes`

### 4. Install Prometheus Using Helm

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack
kubectl get pods
kubectl expose service prometheus-kube-prometheus-prometheus --type=NodePort --name=prometheus-ext

# Get the Prometheus Port
kubectl get svc prometheus-ext
```
Access Prometheus at `http://<ec2-public-ip>:<prometheus-nodeport>`

### 5. Install Grafana Using Helm

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana
kubectl expose service grafana --type=NodePort --name=grafana-ext

kubectl get svc grafana-ext

# Get Grafana Admin Password
kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```
Access Grafana at `http://<ec2-public-ip>:<grafana-nodeport>`

### 6. Configure Prometheus as a Data Source in Grafana

- Log in to Grafana using the `admin` credentials.
- Navigate to `Configuration` > `Data Sources` > `Add Data Source`.
- Select Prometheus and enter URL: `http://<ec2-public-ip>:<prometheus-nodeport>`
- Save and test the connection.

### 7. Import a Pre-Built Kubernetes Dashboard in Grafana

- Go to Dashboards > Import in Grafana.
- Enter Dashboard ID: `3662`(or any predefined ID for Kubernetes monitoring).
- Click Load, select Prometheus as the data source and click Import.

A complete Kubernetes dashboard is now displayed, showing:
- Node metrics
- API server health
- Pod statuses
- Deployment replicas

### 8. Expose and Use Cube State Metrics

```bash
kubectl get pods
kubectl expose service kube-state-metrics --type=NodePort --name=cube-state-metrics-ext
kubectl get svc cube-state-metrics-ext
```
Access Cube State Metrics at `http://<ec2-public-ip>:<cube-state-metrics-nodeport>/metrics`

**Configure Cube State Metrics in Prometheus:**
```
kubectl get cm
kubectl edit configmap prometheus-server
```

Add the following scrape config:
```yaml
- job_name: 'kube-state-metrics'
  static_configs:
    - targets: ['<ec2-public-ip>:<cube-state-metrics-nodeport>']
```

**Restart Prometheus:**
`kubectl rollout restart deployment prometheus-server`

### 9. View the Dashboard

Refresh the imported Grafana dashboard to see real-time metrics from Kubernetes, including:
- Node health
- Deployment replicas
- Pod statuses
- API server performance





