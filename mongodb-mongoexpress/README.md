# Kubernetes MongoDB + Mongo Express Deployment

This project demonstrates how to deploy MongoDB and Mongo Express on a Kubernetes cluster using:

- Deployments
- Services
- Secrets
- ConfigMaps

MongoDB acts as the database backend, while Mongo Express provides a web-based UI to manage MongoDB.

---

# Project Architecture

```text
Mongo Express UI  --->  MongoDB Service  --->  MongoDB Pod
        |
     LoadBalancer
```

---

# Step 1 - Create Kubernetes Secret

File:

```text
mongodb-secret.yaml
```

Purpose:
- Stores sensitive information securely
- Avoids hardcoding usernames and passwords directly inside deployment files

We store:
- MongoDB root username
- MongoDB root password

Base64 Encoded Values:

```text
admin     -> YWRtaW4=
password  -> cGFzc3dvcmQ=
```

Why Base64 Encoding?

Kubernetes Secrets require values to be stored in Base64 encoded format.

Encode values using:

```bash
echo -n admin | base64
echo -n password | base64
```

Apply Secret:

```bash
kubectl apply -f mongodb-secret.yaml
```

Verify:

```bash
kubectl get secret
```

---

# Step 2 - Deploy MongoDB

File:

```text
mongodb-deployment.yaml
```

Purpose:
- Creates MongoDB Pod
- Pulls MongoDB Docker image
- Uses credentials from Kubernetes Secret

MongoDB container reads:
- Username from Secret
- Password from Secret

Apply Deployment:

```bash
kubectl apply -f mongodb-deployment.yaml
```

Verify:

```bash
kubectl get deployment
kubectl get pods
```

---

# Step 3 - Create MongoDB Service

File:

```text
mongodb-service.yaml
```

Purpose:
- Exposes MongoDB internally inside the Kubernetes cluster
- Allows other applications to connect using service name

Service Name:

```text
mongodb-service
```

Mongo Express uses this service name to connect to MongoDB.

Apply Service:

```bash
kubectl apply -f mongodb-service.yaml
```

Verify:

```bash
kubectl get svc
```

---

# Step 4 - Create ConfigMap

File:

```text
mongoexpress-configmap.yaml
```

Purpose:
- Stores non-sensitive configuration data
- Keeps configuration separated from application code

We store:
- MongoDB service URL

ConfigMap provides:
- Database connection information
- Application configuration

Apply ConfigMap:

```bash
kubectl apply -f mongoexpress-configmap.yaml
```

Verify:

```bash
kubectl get configmap
```

---

# Step 5 - Deploy Mongo Express

File:

```text
mongoexpress-deployment.yaml
```

Purpose:
- Deploys Mongo Express UI
- Connects to MongoDB using:
  - Secret
  - ConfigMap

Mongo Express uses:
- Username from Secret
- Password from Secret
- MongoDB server address from ConfigMap

Apply Deployment:

```bash
kubectl apply -f mongoexpress-deployment.yaml
```

Verify:

```bash
kubectl get deployment
kubectl get pods
```

---

# Step 6 - Create Mongo Express Service

File:

```text
mongoexpress-service.yaml
```

Purpose:
- Exposes Mongo Express externally
- Allows browser access to Mongo Express UI

We use:

```text
type: LoadBalancer
```

This creates external access to the application.

Apply Service:

```bash
kubectl apply -f mongoexpress-service.yaml
```

Verify:

```bash
kubectl get svc
```

---

# Apply All Files Together

```bash
kubectl apply -f mongodb-secret.yaml
kubectl apply -f mongodb-deployment.yaml
kubectl apply -f mongodb-service.yaml

kubectl apply -f mongoexpress-configmap.yaml
kubectl apply -f mongoexpress-deployment.yaml
kubectl apply -f mongoexpress-service.yaml
```

---

# Verify All Kubernetes Resources

```bash
kubectl get all
```

Check specific resources:

```bash
kubectl get pods
kubectl get deployment
kubectl get svc
kubectl get secret
kubectl get configmap
```

---

# Access Mongo Express

If using Minikube:

```bash
minikube service mongoexpress-service
```

Or check external IP:

```bash
kubectl get svc
```

Open in browser:

```text
http://<EXTERNAL-IP>:8081
```

---

# Important Kubernetes Concepts Used

## Deployment

Used to:
- Create Pods
- Manage replicas
- Restart failed Pods automatically

---

## Service

Used to:
- Expose applications
- Enable Pod communication
- Provide stable networking

---

## Secret

Used to:
- Store sensitive data securely
- Avoid hardcoding passwords

---

## ConfigMap

Used to:
- Store non-sensitive configuration
- Separate configuration from application code

---

# Final Outcome

After deployment:

- MongoDB runs inside Kubernetes
- Mongo Express connects to MongoDB
- Mongo Express UI becomes accessible from browser
- Kubernetes manages Pods automatically
- Sensitive data stays secure using Secrets

---

# Pro Tip

In real-world Kubernetes projects, multiple resources can be written inside a single YAML file using:

```yaml
---
```

separator.

For example:
- Deployment + Service
- ConfigMap + Deployment
- Secret + Deployment

can all exist inside one file.

This reduces:
- Number of files
- Management overhead
- Repetitive commands

Example:

```text
mongodb.yaml
```

can contain:
- MongoDB Deployment
- MongoDB Service

Similarly:

```text
mongoexpress.yaml
```

can contain:
- Mongo Express Deployment
- Mongo Express Service

---

In this project, all resources were intentionally separated into different files for:
- Better learning
- Easier understanding
- Clear separation of Kubernetes components
- Beginner-friendly structure

This approach helps understand:
- What each Kubernetes resource does
- How resources interact with each other
- Real purpose of Deployments, Services, Secrets, and ConfigMaps independently
