# Kubernetes Configmaps

A Kubernetes ConfigMap is an API object used to store non-sensitive configuration data as key-value pairs for applications running in a Kubernetes cluster. Applications can use this data as configuration information. It allows applications to consume configuration data without hardcoding it into the code or container images. 

ConfigMaps can provide configuration data to Pods as:
- Environment variables.
- Command-line arguments.
- Mounted Volume files.

____________________________________________________________________________________________

## Why Use ConfigMaps?

- Easily manage different configurations for development, testing, and production environments.
- Updating a ConfigMap can automatically propagate changes to the applications without rebuilding or redeploying the container.
- Keeps configuration separate from the application codebase.

## Limitations:

- ConfigMaps are not encrypted by default and should not store sensitive data like passwords or API keys (use Kubernetes Secrets instead).
- ConfigMaps have a size limit of 1 MB.

_____________________________________________________________________________________________

## How a ConfigMap Works:

A ConfigMap consists of:
- Metadata (like name, namespace, and labels).
- Data: A collection of key-value pairs.

**Example:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  APP_ENV: production
  LOG_LEVEL: debug
```

1. **Using ConfigMap values as environment variables in a Pod.**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: example-config
          key: APP_ENV
```

2. **Mounting ConfigMap data as files inside a container.**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: example-config
```
This will create files in /etc/config with names as keys (APP_ENV, LOG_LEVEL) and their respective values as content.

____________________________________________________________________________________________

## Advantages of Volume Mounts over Environment Variables for ConfigMap

- Handles Large Data: Suitable for large configurations, as environment variables have size limits.
- Dynamic Updates: Reflects ConfigMap changes in real-time without restarting the pod (if the app supports it).
- Supports Structured Data: Ideal for JSON, YAML, or configuration files, avoiding extra parsing logic.
- Reduces Environment Pollution: Keeps environment variables clean and avoids conflicts.
- Allows File Permissions: Enables fine-grained access control for sensitive data.
- Application Compatibility: Works seamlessly with apps designed to read configurations from files.
- Facilitates Debugging: Files can be inspected directly inside the pod for troubleshooting.

#### Use Case:

**Use Volume Mounts:** 
- When you have large, structured, or sensitive data, need dynamic updates, or are working with applications that rely on configuration files.
**Use Environment Variables:** 
- For small, simple configurations that rarely change and must be directly accessible as key-value pairs.