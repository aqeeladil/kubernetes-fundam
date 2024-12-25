# Key Kubernetes Security Measures

As DevSecOps engineers, one of the primary resposibilities is to maintain security of your Kubernetes clusters and the containers.

Here are some of the mandatory things to consider.
  1. Secure your API server
  2. RBAC
  3. Network Policies
  4. Encrypt data at rest
  5. Secure Container Images
  6. Cluster Monitoring
  7. Upgrades

## 1. Secure your API server
- The API server is the entry point for all requests to the Kubernetes cluster, making it a primary target for attacks.
  - Regularly rotate TLS certificates to reduce the risk of compromised keys.
  - Employ admission controllers like PodSecurityPolicy (deprecated but replaceable by OPA Gatekeeper) to enforce security policies.
  - Enable Mutual TLS (mTLS): Ensure communication between internal components uses mutual TLS for secure authentication.

1. **Enable TLS encryption:**
```bash
# In your Kubernetes API server configuration file (kube-apiserver.yaml), add the following lines to enable TLS encryption:

apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
spec:
  containers:
  - name: kube-apiserver
    image: k8s.gcr.io/kube-apiserver:v1.21.0
    command:
    - kube-apiserver
    - --tls-cert-file=/path/to/server.crt
    - --tls-private-key-file=/path/to/server.key
    - --client-ca-file=/path/to/ca.crt

# In this example, we are using server.crt and server.key files for the server certificate and private key respectively, and ca.crt for the client certificate authority.
```
2. **Use strong authentication:**
```bash
# In the same kube-apiserver.yaml file, configure the authentication method you prefer, such as client certificates or bearer tokens. Here is an example of how to use client certificates.

apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
spec:
  containers:
  - name: kube-apiserver
    image: k8s.gcr.io/kube-apiserver:v1.21.0
    command:
    - kube-apiserver
    - --tls-cert-file=/path/to/server.crt
    - --tls-private-key-file=/path/to/server.key
    - --client-ca-file=/path/to/ca.crt
    - --authentication-mode=x509
    - --requestheader-client-ca-file=/path/to/ca.crt
    - --requestheader-allowed-names=""
    - --requestheader-extra-headers-prefix="X-Remote-Extra-"

# In this example, we are using x509 authentication and configuring the request headers to include X-Remote-Extra- headers. This allows you to pass additional information about the client, such as the user's email or group membership.
```
3. **Restrict access:**
```bash
# Configure RBAC to restrict access to the Kubernetes API server based on roles and permissions. Here is an example of how to configure RBAC.

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: api-server-reader
rules:
- apiGroups: [""]
  resources: ["pods", "namespaces"]
  verbs: ["get", "watch", "list"]

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: api-server-reader-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: api-server-reader
subjects:
- kind: User
  name: alice

# In this example, we are creating a ClusterRole called api-server-reader that allows read-only access to pods and namespaces, and a RoleBinding that assigns this role to the user alice in the default namespace.
```
4. **Monitor and audit:**
```bash
# Use tools like Kubernetes Audit to monitor and audit the Kubernetes API server. Here is an example of how to configure Kubernetes Audit.

apiVersion: audit.k8s.io/v1beta1
kind: Policy
rules:
- level: Metadata
```
5. **Keep the API server up to date:**
  - Make sure to keep the Kubernetes API server up to date with the latest security patches and updates by regularly checking for updates and applying them as needed.

## 2. RBAC
- Use Role-Based Access Control to define who can access which resource in kubernetes. For example, not everyone should have access to kubernetes secrets.
  - Define roles and bind them to users or service accounts.
  - Regularly audit roles and bindings to identify unused or over-privileged roles.
  - Follow the Principle of Least Privilege (PoLP): Grant only the minimal necessary permissions required for roles.
  - Use specific resources and actions rather than broad wildcard permissions.
  - Use roles (namespace-scoped) and cluster roles (cluster-wide) effectively.

## 3. Network Policies
- Use network policies to restrict traffic within the cluster and to/from external sources. 
  - Define network policies at the namespace level to restrict access based on requirements.
  - Example: Allow only internal traffic for sensitive namespaces like those containing user databases.
  - Start with a default deny-all ingress and egress policy, and explicitly allow only required communication.
- Use firewalls and security groups to control traffic to and from the cluster.

## 4. Encrypt Data At Rest
- Even if RBAC is configured, sensitive data (like secrets) stored in etcd can be accessed if not encrypted.
- To encrypt data at rest in Kubernetes, you can use the Kubernetes Encryption Provider feature, which encrypts sensitive data stored in etcd (a consistent and highly-available key-value store used as Kubernetes' backing store for all cluster data.). 
  - The Encryption Provider uses a key management system to manage and store encryption keys.
  - Implement a regular key rotation policy in the Key Management System (KMS).
  - Continuously monitor access to encryption keys and ensure strict access controls.

1. **Enable the Encryption Provider feature by configuring the Kubernetes API server.**
```bash
# You can enable the Encryption Provider feature by adding the --encryption-provider-config option to the Kubernetes API server command-line arguments or to the API server manifest file. This option points to a configuration file that specifies the encryption provider and its settings.

# Example: A simple encryption provider configuration file:

apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources: ["secrets"]
    providers:
      - identity: {}

# In this example, we are enabling the Encryption Provider for Secrets resources using the "identity" provider, which uses a default encryption algorithm and key size.
```
2. **Configure the key management system to store and manage encryption keys.**
```bash
# The Encryption Provider requires a key management system to store and manage encryption keys. You can use a cloud-based key management system like Google Cloud KMS or Amazon Web Services KMS, or a self-hosted key management system like HashiCorp Vault.

# You must configure the key management system to generate a key and give the Kubernetes API server access to the key. The specific steps to do this depend on the key management system you are using.
```
3. **Create a Kubernetes Secret object with the encryption key.**
```bash
# Once you have a key from your key management system, you can create a Kubernetes Secret object that stores the key. You can create the Secret object using kubectl:

`kubectl create secret generic encryption-key --from-literal=encryption-key=<base64-encoded-key>`

# In this example, we are creating a Secret object called "encryption-key" and storing the key as a base64-encoded literal.
```
4. **Configure Kubernetes resources to use the Encryption Provider.**
```bash
# To use the Encryption Provider to encrypt data at rest, you need to configure Kubernetes resources to use the feature. You can do this by setting the encryptionConfig field in the Kubernetes API server manifest file or by setting the metadata.annotations field in the Kubernetes resource definition.

# Here is an example of a Kubernetes Secret definition that uses the Encryption Provider:

apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  annotations:
    encryptionConfig: secrets
type: Opaque
data:
  username: <base64-encoded-username>
  password: <base64-encoded-password>

# In this example, we are defining a Secret object called "my-secret" that contains sensitive data. We are setting the metadata.annotations field to specify that the Encryption Provider should be used to encrypt the data. The data field contains the sensitive data, which is base64-encoded.
```

## 5. Secure Container Images
- Vulnerable container images can compromise the cluster.
- Use container images from trusted sources and scan them for vulnerabilities before deployment.
  - Use specific, secure base images instead of generic ones like ubuntu:latest.
  - Scan container images for vulnerabilities using tools like Snyk, Trivy, or Clair.
  - To scan images for vulnerabilities you can use simple commands like `docker scan --severity high <docker-image-location>`.
- Integrate image security scanning into CI/CD pipelines.
- Implement image signing with tools like Cosign to verify image integrity before deployment.
- Continuously rescan images in use to identify new vulnerabilities.

## 6. Cluster Monitoring
- Use tools like Kubernetes Audit Logs and security monitoring solutions to detect and respond to malicious activities, misconfigurations, and potential security breaches in real-time.
  - Connect Kubernetes audit logs and security events to a Security Information and Event Management (SIEM) system.
- Use tools like Sysdig for runtime monitoring.
  - Monitor for suspicious activities, such as containers running as root or accessing sensitive files (e.g., `/etc/shadow`).
  - Employ behavior analysis tools like Falco to monitor and detect unusual container or host activities.

## 7. Upgrades
- Older versions may contain vulnerabilities fixed in newer releases.
- Regularly update Kubernetes and its components to the latest stable version.
- Apply security patches promptly to other DevOps tools (e.g., Jenkins, Terraform).
- Schedule upgrades during planned maintenance windows to minimize disruption.
- Always back up etcd and critical resources before applying upgrades.