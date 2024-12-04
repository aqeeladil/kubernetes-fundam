# Kubernetes Secrets

Kubernetes Secrets are a secure way to store and manage sensitive information such as passwords, OAuth tokens, and SSH keys in Kubernetes. Using Secrets helps keep this data out of your application code or pod configuration files, enhancing security.

Secrets are encoded in base64 and stored in the etcd database. Encryption can be enabled for additional security. They are tightly integrated with Kubernetes Role-Based Access Control (RBAC), ensuring only authorized entities can access them.

Secrets can be used:
- As Environment variables, 
- As Mounted volumes
- In Kubernetes resources like ConfigMaps, Ingress, and Deployment configurations.

________________________________________________________________________________________________

## Types of Secrets

- **Opaque:** A generic type for arbitrary key-value pairs.
- **docker-registry:** Stores credentials for Docker container registries.
- **tls:** Contains a certificate and its associated private key.
- **Basic Authentication:** Stores username and password.
- **SSH Authentication:** Stores SSH private keys.

____________________________________________________________________________________________

## Creating Secrets

1. **From Literal Values**
```bash
kubectl create secret generic my-secret --from-literal=username=myuser --from-literal=password=mypass
```

2. **From a File**
```bash
kubectl create secret generic my-secret --from-file=ssh-privatekey=/path/to/privatekey --from-file=ssh-publickey=/path/to/publickey
```

3. **From a YAML/JSON Manifest (```secret.yaml```)**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: bXl1c2Vy  # Base64 encoded "myuser"
  password: bXlwYXNzd29yZA==  # Base64 encoded "mypassword"
```

## Using Secrets

1. **As Environment Variables**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: my-container
    image: nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: password
```

2. **As Mounted Volumes**
```yaml
containers:
- name: my-container
  volumeMounts:
  - name: secret-volume
    mountPath: "/etc/secret"
    readOnly: true
volumes:
- name: secret-volume
  secret:
    secretName: my-secret
```

_______________________________________________________________________________________________

# Difference between ConfigMap & Secret ?

ConfigMaps and Secrets are Kubernetes resources designed to externalize and inject configuration data into applications in a containerized environment. But they serve slightly different purposes and handle data differently.

- ConfigMap is used to store non-sensitive configuration data such as environment variables, configuration files, or command-line arguments. Secret is designed to store sensitive information such as passwords, API tokens, SSH keys, or any other confidential data.
- ConfigMap stores data in plaintext, making it human-readable. Secret stores data in base64-encoded format, which is not encryption but provides obfuscation. For better security, you can use external tools (like KMS) or enable encryption at rest.
- ConfigMap is not encrypted and directly accessible to anyone with permissions to the Kubernetes cluster. Secret's Base64 encoding provides minimal security. However, Kubernetes allows encryption at rest, role-based access control (RBAC), and external secret management integration (e.g; HashiCorp Vault or AWS Secrets Manager) to enhance security.
- ConfigMap is ideal for data that doesn't need to be kept private, like application settings, configuration files, or feature flags. Secret is suitable for sensitive information where confidentiality is essential, such as database credentials or private keys.
- Both ConfigMap and Secret can be mounted into pods as: environment variables and files in a volume. However secrets require explicit permissions and are subject to stricter access controls.
- Both ConfigMaps and Secrets have a size limit of 1MB per object, but Secrets often contain smaller data sets.
- Use ConfigMap for general configurations that are safe to expose. Use Secret for sensitive data, and ensure you apply encryption and access control best practices.
- With ConfigMaps and Secrets, configurations are centrally managed. Updates to configuration files propagate automatically to the respective applications without manual intervention or code changes.

_____________________________________________________________________________________________

## Problems that Configmap and Secret solves:

1. Hardcoding configuration data within application code reduces reusability and creates rigid dependencies between environments. ConfigMaps allow you to inject configuration data at runtime, making your application portable and environment-agnostic.
2. Sensitive information (e.g., passwords, tokens) stored in application code or plaintext files can lead to security vulnerabilities. Secrets store sensitive data securely, and Kubernetes can encrypt Secrets at rest, reducing the risk of exposure.
3. Changing a single configuration value (e.g., log level) requires rebuilding and redeploying container images. ConfigMaps and Secrets allow on-the-fly updates to configurations without rebuilding container images, saving time and resources.
4. Managing multiple configuration files for different environments (dev, test, prod) can be cumbersome and error-prone. ConfigMaps and Secrets standardize configuration management, ensuring consistent deployments across environments.
5. Including sensitive data in logs or exposing them as plain environment variables increases the risk of accidental leaks. Kubernetes Secrets restrict access to sensitive data and provide RBAC (Role-Based Access Control) for better access control.

______________________________________________________________________________________________

## What are VolumeMounts ?

In Kubernetes, VolumeMounts are used to attach Volumes to a container, enabling containers to access and share data persistently, beyond the container's lifecycle. They are specified as part of a Pod or container definition.

**How VolumeMounts Work**
- Volume Defined: Define a volume at the Pod level.
- Volume Mounted: Attach the volume to the container's filesystem using volumeMounts.
- Access Path: The container accesses the volume content at the specified mountPath.

**Common Volume Types**
- EmptyDir: Temporary storage shared by all containers in a Pod.
- HostPath: Maps to a directory on the node's filesystem.
- PersistentVolumeClaim (PVC): Connects to a PersistentVolume for durable storage.
- ConfigMap: Mounts ConfigMap data as files.
- Secret: Mounts Secret data as files.
- Projected: Combines multiple sources like ConfigMaps and Secrets into one directory.



