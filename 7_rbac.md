# Introduction to K8s RBAC

In Kubernetes, Role-Based Access Control (RBAC) is a system for regulating access to resources based on a user’s role within the cluster. RBAC is essential for securing a Kubernetes environment, as it allows you to control which users or service accounts can perform certain actions on specific resources.

Kubernetes RBAC provides a structured, scalable, and secure way to manage permissions, helping protect clusters against unauthorized access, enforcing the least privilege principle, supporting multi-tenancy, and simplifying auditing and compliance.

- **Fine-Grained Access Control:** RBAC lets you set specific permissions per user or group.
- **Enhanced Security:** By limiting actions, you minimize risks associated with unauthorized access.
- **Easy Auditing:** Clear and structured access rules allow for easier tracking of who can do what in the cluster.

## Main components in Kubernetes RBAC:

### 1. Roles and ClusterRoles:

- **Role:** Defines permissions within a specific namespace. A Role is a set of rules that determine which actions are allowed or denied on resources (like pods, services, etc.).
- **ClusterRole:** Similar to Role but can define permissions across the entire cluster, not limited to a single namespace. ClusterRoles are used when permissions need to be cluster-wide (e.g., for nodes, persistent volumes).


### 2. RoleBindings and ClusterRoleBindingsL

- **RoleBinding:** Binds a Role to a user, group, or service account within a specific namespace, granting them the permissions defined in the Role.
- **ClusterRoleBinding:** Binds a ClusterRole to a user, group, or service account across the entire cluster.

### 3. Verbs and Resources:

- **Verbs:** Actions allowed on resources, such as get, list, create, update, delete, etc.
- **Resources:** Specify the Kubernetes resources (like pods, services, deployments, etc.) that the Role or ClusterRole will apply to.

## Use Case

**For example**, if you want to give a developer ```read-only``` access to pods in the dev namespace, you could:

- Define a Role (e.g., pod-reader) that allows get and list permissions on pods.
- Create a RoleBinding to bind this Role to the developer’s user account in the dev namespace.

## Example-1: Role and RoleBindings:

To demonstrate Kubernetes RBAC, let's walk through a practical example where we set up a read-only role for viewing pods in a namespace. In this demo, we’ll create a Role, a ServiceAccount, and a RoleBinding to connect them. Finally, we'll verify the permissions granted by the Role.

Prerequisites
- A running Kubernetes cluster (local or cloud-based).
- kubectl configured to access your Kubernetes cluster.

1. **Create a Namespace for Isolation**
First, let's create a namespace so that we can keep our RBAC rules scoped within it.
```bash
kubectl create namespace rbac-demo
```
2. **Create a Role with Limited Permissions**
- In this step, we'll define a Role that only grants ```get```, ```list```, and ```watch``` permissions for pods in the rbac-demo namespace.

Create a file named ```pod-reader-role.yaml``` with the following content:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: rbac-demo
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```
```bash
kubectl apply -f pod-reader-role.yaml
```
3. **Create a ServiceAccount**
```bash
kubectl create serviceaccount pod-reader-sa -n rbac-demo
```
4. **Bind the Role to the ServiceAccount**
- Now, we need to create a RoleBinding that assigns the ```pod-reader``` Role to the ```pod-reader-sa``` ServiceAccount.

Create a file named ```pod-reader-binding.yaml```:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: rbac-demo
subjects:
- kind: ServiceAccount
  name: pod-reader-sa
  namespace: rbac-demo
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
```bash
kubectl apply -f pod-reader-binding.yaml
```
5. **Test the Permissions**
- To test the permissions, we’ll impersonate the ```pod-reader-sa``` ServiceAccount to verify that it has read-only access to pods.

  1. First, create a pod in the ```rbac-demo``` namespace for testing purposes:
  ```bash
  kubectl run test-pod --image=nginx -n rbac-demo
  ```
  2. Use ```kubectl``` to impersonate the ```pod-reader-sa``` ServiceAccount and try to list the pods:
  ```bash
  kubectl get pods -n rbac-demo --as=system:serviceaccount:rbac-demo:pod-reader-sa
  ```
  This command should work, showing the ```test-pod``` in the output.
  3. Now, try to perform an action that’s not allowed, such as deleting the pod:
  ```bash
  kubectl delete pod test-pod -n rbac-demo --as=system:serviceaccount:rbac-demo:pod-reader-sa
  ```
  This command should fail, showing an error message like ```Error from server (Forbidden)```:.
6. **Cleanup**
```bash
kubectl delete namespace rbac-demo
```
<br>

## Example-2: ClusterRole and ClusterRoleBindings:

In this example, we’ll create a ClusterRole with cluster-wide permissions to list nodes. Then, we’ll create a ClusterRoleBinding to assign this role to a ServiceAccount. Finally, we’ll test that the ServiceAccount has the expected permissions.

Prerequisites
- A running Kubernetes cluster.
- kubectl configured to access your cluster.

1. **Create a ClusterRole with Node Access Permissions:**
- A ClusterRole defines permissions that can apply cluster-wide, not limited to a single namespace. In this case, we’ll create a ClusterRole that grants ```read-only``` access to nodes.
Create ```node-reader-clusterrole.yaml```:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
```
```bash
kubectl apply -f node-reader-clusterrole.yaml
```
This ClusterRole (```node-reader```) now has permissions to list and get information about nodes across the entire cluster.
2. **Create a ServiceAccount:**
```bash
kubectl create serviceaccount node-reader-sa -n default
```
3. **Bind the ClusterRole to the ServiceAccount:**
Create a file named ```node-reader-clusterrolebinding.yaml```:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-reader-binding
subjects:
- kind: ServiceAccount
  name: node-reader-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```
```bash
kubectl apply -f node-reader-clusterrolebinding.yaml
```
This binds the ```node-reader``` ClusterRole to the ```node-reader-sa``` ServiceAccount, granting it cluster-wide ```read-only``` access to nodes.
4. **Test the Permissions**
  1. List nodes
  ```bash
  kubectl get nodes --as=system:serviceaccount:default:node-reader-sa
  ```
  This command should succeed, displaying the nodes in the cluster. This shows that the ServiceAccount has read-only access to nodes as defined by the ```node-reader``` ClusterRole.
  2. Try to perform an unauthorized action, such as deleting a node:
  ```bash
  kubectl delete node <node-name> --as=system:serviceaccount:default:node-reader-sa
  ```
  This command should fail with an error message like ```Error from server (Forbidden)```:, demonstrating that the ServiceAccount lacks delete permissions on nodes.
5. **Cleanup:**
```bash
kubectl delete clusterrole node-reader
kubectl delete clusterrolebinding node-reader-binding
kubectl delete serviceaccount node-reader-sa -n default
```

