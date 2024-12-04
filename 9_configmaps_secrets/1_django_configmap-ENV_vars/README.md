# Using ConfigMap data as environment variables inside a Django container

1. **Apply a ConfigMap**
```bash
kubectl apply -f configmap.yaml
```

2. **Reference ConfigMap in the Deployment YAML**
```bash
kubectl apply -f deployment.yaml
```

3. **Access the Environment Variables in Django**
Example: ```settings.py```
```python
import os

DEBUG = os.getenv('DEBUG', 'False') == 'True'
DATABASE_URL = os.getenv('DATABASE_URL')
SECRET_KEY = os.getenv('SECRET_KEY')

# Use DATABASE_URL with a library like dj-database-url for database configuration
import dj_database_url
DATABASES = {
    'default': dj_database_url.parse(DATABASE_URL)
}
```

4. **Verify the Setup**
```bash
# Ensure the ConfigMap is successfully created:
kubectl get configmap django-env-config -o yaml

# Check if the environment variables are correctly injected into the container
kubectl exec -it <django-pod-name> -- env

# Test the application by accessing the service endpoint to ensure it picks up the environment variables correctly.
```

