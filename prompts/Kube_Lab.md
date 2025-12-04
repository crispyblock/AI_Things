Absolutely — here is a **fully detailed, step-by-step weekly lab manual** covering **Docker, Podman, Kubernetes, AWS ECR, ECS, and EKS**.
This is written like a real training manual used in DevOps/Cloud bootcamps — every week includes:

✔ Objectives
✔ Required tools
✔ Lab steps with explicit commands
✔ Validation checks
✔ Deliverables

You can put each week into its own GitHub folder when you complete the labs.

---

# 📘 **WEEK 1 – Containers Foundations (Docker + Podman)**

### **Objectives**

* Understand how containers work
* Build and run container images using Docker & Podman
* Use volumes, networks, multi-container setups
* Publish an image to a remote registry

### **Required Tools**

* Docker Desktop or Docker Engine
* Podman
* Git
* VS Code (recommended)

---

## **Lab 1.1 — Install & Validate Docker**

**Commands**

```bash
docker --version
docker run hello-world
```

**Validation**

* Output shows Docker version
* “Hello from Docker!” prints successfully

---

## **Lab 1.2 — Install & Validate Podman**

**Commands**

```bash
podman --version
podman run hello-world
```

**Validation**

* Podman runs containers rootless

---

## **Lab 1.3 — Create a Simple Application**

Create a simple Flask or Node.js API.

**Example (Python Flask `app.py`)**

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from the container!"

app.run(host="0.0.0.0", port=5000)
```

---

## **Lab 1.4 — Write a Dockerfile**

`Dockerfile`:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install flask
EXPOSE 5000
CMD ["python", "app.py"]
```

---

## **Lab 1.5 — Build and Run with Docker**

```bash
docker build -t myapp:1 .
docker run -p 5000:5000 myapp:1
```

**Validation**
Visit:
[http://localhost:5000](http://localhost:5000) → Should show message

---

## **Lab 1.6 — Run the Same Image With Podman**

```bash
podman build -t myapp:1 .
podman run -p 5000:5000 myapp:1
```

---

## **Lab 1.7 — Using Volumes & Networks**

Create a network:

```bash
docker network create appnet
```

Run DB:

```bash
docker run -d --name db --network appnet -e POSTGRES_PASSWORD=pass postgres
```

Run API:

```bash
docker run -d --network appnet -p 5000:5000 myapp:1
```

---

## **Lab 1.8 — Push Image to Docker Hub**

Tag:

```bash
docker tag myapp:1 <your-dockerhub-username>/myapp:1
```

Push:

```bash
docker push <your-dockerhub-username>/myapp:1
```

---

## **WEEK 1 Deliverables**

* API code
* Dockerfile (Docker + Podman verified)
* Screenshot: local run + Docker Hub push
* GitHub repo: `week1-containers`

---

---

# 📘 **WEEK 2 – Kubernetes Fundamentals**

### **Objectives**

* Deploy workloads using Pods & Deployments
* Create Services, ConfigMaps, and Secrets
* Scale pods
* Use Ingress

### **Required Tools**

* kind **or** Minikube
* kubectl

---

## **Lab 2.1 — Install Kubernetes Locally**

Using kind:

```bash
kind create cluster --name lab
kubectl get nodes
```

---

## **Lab 2.2 — Create Kubernetes Manifests**

### **deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: <your-dockerhub-username>/myapp:1
        ports:
        - containerPort: 5000
```

### **service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 5000
  type: NodePort
```

---

## **Lab 2.3 — Apply Manifests**

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

## **Lab 2.4 — Validate Deployment**

```bash
kubectl get pods
kubectl get svc
```

Visit your NodePort:
`http://localhost:<nodeport>`

---

## **Lab 2.5 — ConfigMaps & Secrets**

### **ConfigMap**

```bash
kubectl create configmap app-config --from-literal=ENV=dev
```

### **Secret**

```bash
kubectl create secret generic app-secret --from-literal=PASSWORD=12345
```

Patch Deployment to use them:

```yaml
env:
- name: ENV
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: ENV
- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: PASSWORD
```

---

## **Lab 2.6 — Scaling**

```bash
kubectl scale deployment myapp --replicas=5
kubectl get pods
```

---

## **WEEK 2 Deliverables**

* Kubernetes manifest folder
* Screenshot of Deployment + Service working
* GitHub repo: `week2-k8s-basics`

---

---

# 📘 **WEEK 3 – Advanced Kubernetes + Helm**

### **Objectives**

* Convert YAML manifests to Helm chart
* Add health probes
* Add resource limits
* Implement HPA
* Perform rolling updates

### **Required Tools**

* Helm

---

## **Lab 3.1 — Create Helm Chart**

```bash
helm create myapp
```

This generates structure:

```
myapp/
  templates/
  values.yaml
  Chart.yaml
```

Copy your Deployment logic into `templates/deployment.yaml`.

---

## **Lab 3.2 — Add Liveness/Readiness Probes**

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 5000
  initialDelaySeconds: 10
readinessProbe:
  httpGet:
    path: /
    port: 5000
  initialDelaySeconds: 5
```

---

## **Lab 3.3 — Add Resource Limits**

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
```

---

## **Lab 3.4 — Install Chart**

```bash
helm install myapp ./myapp
kubectl get pods
```

---

## **Lab 3.5 — Enable HPA**

```bash
kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=50
kubectl get hpa
```

---

## **Lab 3.6 — Rolling Update**

Update image tag in `values.yaml`
Then:

```bash
helm upgrade myapp ./myapp
kubectl rollout status deployment/myapp
```

---

## **WEEK 3 Deliverables**

* Helm chart
* Screenshots of HPA + rollout
* GitHub repo: `week3-k8s-helm`

---

---

# 📘 **WEEK 4 – AWS ECR + ECS (Fargate)**

### **Objectives**

* Push container images to ECR
* Deploy an application on ECS (Fargate)
* Attach an ALB

### **Required Tools**

* AWS CLI
* IAM user with ECR/ECS permissions

---

## **Lab 4.1 — Create ECR Repository**

```bash
aws ecr create-repository --repository-name myapp
```

Get login:

```bash
aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

---

## **Lab 4.2 — Push Image to ECR**

```bash
docker tag myapp:1 <aws_account_id>.dkr.ecr.<region>.amazonaws.com/myapp:1
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/myapp:1
```

---

## **Lab 4.3 — Create ECS Cluster (Fargate)**

```bash
aws ecs create-cluster --cluster-name myapp-cluster
```

---

## **Lab 4.4 — Create Task Definition**

Use AWS Console (GUI) or JSON:

`taskdef.json`:

```json
{
  "family": "myapp-task",
  "requiresCompatibilities": ["FARGATE"],
  "networkMode": "awsvpc",
  "memory": "512",
  "cpu": "256",
  "containerDefinitions": [
    {
      "name": "myapp",
      "image": "<aws_account_id>.dkr.ecr.<region>.amazonaws.com/myapp:1",
      "portMappings": [
        { "containerPort": 5000, "protocol": "tcp" }
      ]
    }
  ]
}
```

Register:

```bash
aws ecs register-task-definition --cli-input-json file://taskdef.json
```

---

## **Lab 4.5 — Create Service**

```bash
aws ecs create-service \
  --cluster myapp-cluster \
  --service-name myapp-service \
  --task-definition myapp-task \
  --launch-type FARGATE \
  --desired-count 1 \
  --network-configuration "awsvpcConfiguration={subnets=[<subnet>],securityGroups=[<sg>],assignPublicIp=ENABLED}"
```

---

## **WEEK 4 Deliverables**

* Screenshot of ECS service running
* Public URL of ALB
* GitHub: `week4-aws-ecr-ecs`

---

---

# 📘 **WEEK 5 – Kubernetes on AWS (EKS)**

### **Objectives**

* Create EKS cluster with eksctl
* Deploy your Helm chart to AWS
* Use ECR images with EKS
* Configure autoscaling

---

## **Lab 5.1 — Create EKS Cluster**

```bash
eksctl create cluster --name myapp-eks --region us-east-1 --nodes 2
```

---

## **Lab 5.2 — Push Image to ECR (reuse Week 4)**

Ensure image in ECR is up to date.

---

## **Lab 5.3 — Deploy Helm Chart**

Update Helm values:

```yaml
image:
  repository: <aws_account_id>.dkr.ecr.<region>.amazonaws.com/myapp
  tag: "1"
```

Deploy:

```bash
helm install myapp ./myapp --namespace prod --create-namespace
```

---

## **Lab 5.4 — Create LoadBalancer Service**

```yaml
type: LoadBalancer
```

```bash
kubectl get svc -n prod
```

---

## **Lab 5.5 — Configure Cluster Autoscaler**

Apply AWS Cluster Autoscaler manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

---

## **WEEK 5 Deliverables**

* LoadBalancer URL
* EKS screenshots
* GitHub repo: `week5-aws-eks`

---

---

# 📘 **WEEK 6 — CAPSTONE PROJECT**

### **Objective**

Create a full end-to-end cloud-native system deployed using:

* Docker
* Podman
* Kubernetes (local + EKS)
* AWS ECR/ECS
* Helm

---

## **Capstone Requirements**

* Multi-service app (API + DB + frontend)
* Local containerization (Docker/Podman)
* Local Kubernetes deployment
* AWS ECS deployment for one service
* AWS EKS deployment for entire stack
* Documentation + architecture diagram

---

## **Capstone Checklist**

* [ ] `/docker/` folder → Dockerfiles for each service
* [ ] `/k8s/` folder → manifests
* [ ] `/helm/` chart → production setup
* [ ] `/aws/ecs/` → task definitions
* [ ] `/aws/eks/` → deployment instructions
* [ ] `/docs/architecture.png`

---

# ✅ If you want, I can also generate:

### ✔ A printable PDF version

### ✔ A GitHub-ready folder structure (with README templates)

### ✔ A version tailored for *cybersecurity-focused cloud roles*

### ✔ Automated CI/CD labs (GitHub Actions)

Would you like any of these added?
