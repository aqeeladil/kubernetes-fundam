# Custom Resources in Kubernetes

In Kubernetes, Custom Resources allow you to extend the Kubernetes API to define and manage additional resources 
beyond the built-in ones, such as Pods, Deployments, and Services. By using Custom Resources, you can create configurations that are unique to your application, enabling you to manage more complex application configurations 
within Kubernetes.

### Key Components of Custom Resources

  1. **CustomResourceDefinition (CRD):** A Custom Resource Definition (CRD) is a Kubernetes resource that defines a 
  schema for your Custom Resource. Creating a CRD registers the new resource type within the Kubernetes API, 
  allowing users to create instances of this custom resource.
  - **For example**, if you want to introduce a ```Database``` object into Kubernetes, you would define a CRD with specifications 
  like ```spec```, ```status```, and other fields that describe the custom resource’s structure and behavior. Once the CRD is 
  applied, you can manage instances of the ```Database``` resource with standard ```kubectl``` commands.

  2. **Custom Resource (CR):** A Custom Resource is an instance of a CRD. It contains the configuration data and 
  settings you define and manage within Kubernetes. You interact with CRs just as you would with native Kubernetes resources.

### Why Use Custom Resources?

Custom Resources allow you to:

- Add specialized behavior or data models to Kubernetes that fit your application's needs.
- Manage configurations, lifecycle, and state information for applications that aren’t directly supported by native 
Kubernetes resources.
- Represent domain-specific configurations or custom objects, like a database or a monitoring setup.
- Use Kubernetes-native tools like ```kubectl``` to manage these new resources, making it easier to manage and automate 
infrastructure.

For instance, if you need to define and manage a custom workflow that is not supported by native resources, Custom 
Resources provide the means to build this into your Kubernetes cluster.

### Example: Defining and Using a Custom Resource

Suppose you want to define a new resource for managing a “```Book```” entity with attributes like ```title```, 
```author```, and ```publishedDate```.

1. **Create the CRD:**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: books.library.io
spec:
  group: library.io
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                title:
                  type: string
                author:
                  type: string
                publishedDate:
                  type: string
  scope: Namespaced
  names:
    plural: books
    singular: book
    kind: Book
    shortNames:
      - bk
```
```bash
kubectl apply -f book-crd.yaml
```
This registers the Book Custom Resource in Kubernetes, allowing you to create instances of ```Book```.
2. **Create a Custom Resource (CR):**
```yaml
apiVersion: library.io/v1
kind: Book
metadata:
  name: example-book
spec:
  title: "Kubernetes in Action"
  author: "Marko Luksa"
  publishedDate: "2018-01-09"
```
```bash
kubectl apply -f example-book.yaml
```
3. **Interacting with the Custom Resource:**
```bash
# List all books
kubectl get books

# Describe the details of a specific book
kubectl describe book example-book

# Delete a book
kubectl delete book example-book
```

## Custom Controllers

To manage the lifecycle of Custom Resources, you can develop Custom Controllers. A Custom Controller is a Kubernetes 
operator that watches for changes to Custom Resources and acts accordingly, automating processes and adding 
intelligence to your Kubernetes applications.

Custom Resources and CRDs are foundational for building more sophisticated applications and automation on 
Kubernetes, making them powerful for custom use cases.

**For example**, a Custom Controller for a ```Database``` resource could handle the creation, updating, or deletion 
of database instances in response to changes in the Custom Resource's definition.

### Why Use a Custom Controller?

Custom controllers extend Kubernetes functionalities, enabling you to manage resources and workflows specific to 
your application needs. Custom controllers offer flexibility and can be very powerful for implementing specific 
automation and orchestration workflows in Kubernetes. 

You might use a custom controller to automate tasks such as:

- Ensuring that a specific number of custom resources are always running.
- Responding to custom resource events (e.g., creation, update, deletion).
- Automating complex workflows like backups, scaling, or database clustering.
- The ```kubebuilder``` framework simplifies the process of creating and managing Custom Controllers in Kubernetes 
by providing tools to generate boilerplate code and setting up essential components.

### Key Concepts in a Custom Controller

- **Reconciliation Loop:** The core logic of a controller, which continuously watches the current state of resources in the cluster and compares it to the desired 
state, then takes corrective actions when they don't match. This is also known as the "watch and act" pattern.

- **Event Handlers:** Functions triggered by Kubernetes events (e.g., resource creation, update, deletion), prompting 
the controller to run its reconciliation logic.

### Creating a Custom Controller:

This is how you could set up a custom controller using the Kubernetes Python client or ```kubebuilder```, a popular 
tool for building custom controllers in Go.

1. **Define a Custom Resource Definition (CRD)**
Example YAML for a custom resource ```MyResource```:
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myresources.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: myresources
    singular: myresource
    kind: MyResource
    shortNames:
    - mr
```
2. **Create the Controller Logic:**
- In the controller, define the logic for managing ```MyResource``` instances.
- In Python, you can use the Kubernetes Python client to write your controller. In Go, ```kubebuilder``` provides a 
structured framework to set up the reconciliation loop and event handlers.
```python 
from kubernetes import client, config, watch

# Initialize Kubernetes client
config.load_kube_config()
v1 = client.CustomObjectsApi()

def reconcile(resource):
    # Check the current state and compare it with the desired state
    desired_replicas = resource['spec'].get('replicas', 1)
    current_replicas = get_current_replicas(resource)  # custom function to fetch current replicas

    if current_replicas != desired_replicas:
        scale_resource(resource, desired_replicas)  # custom function to scale resource

def watch_custom_resources():
    w = watch.Watch()
    for event in w.stream(v1.list_namespaced_custom_object,
                          group="example.com", version="v1", namespace="default",
                          plural="myresources"):
        resource = event['object']
        event_type = event['type']
        if event_type in ['ADDED', 'MODIFIED']:
            reconcile(resource)

watch_custom_resources()
```
3. **Deploy the Custom Controller:**
Package the controller code into a container, push it to a registry, and deploy it in your Kubernetes cluster as a 
Deployment.
```yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myresource-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myresource-controller
  template:
    metadata:
      labels:
        app: myresource-controller
    spec:
      containers:
      - name: controller
        image: mycontrollerimage:latest
```


You can also refer to : https://github.com/kubernetes/sample-controller


