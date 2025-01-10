# CHAT APPLICATION ON KUBERNETES WITH 3-TIER ARCHITECTURE

## Project Requirements
- **Frontend (web-tier):** React.js
- **Backend (app-tier):** Node.js
- **Database (db-tier):** MongoDB
- **Containers:** Docker
- **Orchestration:** Kubernetes (Pods, Deployments, Services)

### Overview
1. **Web-tier:** React.js --> Docker container --> Pods --> Deployment --> Services.
2. **App-tier:** Node.js --> Docker container --> Pods --> Deployment --> Services.
3. **Db-tier:** MongoDB --> Docker container --> Pods --> Deployment --> Services + Persistent Volumes (PV & PVC).

All tiers are integrated with Kubernetes ingress.

---

## Steps to Deploy

### Step 1: Create Project Folder and Clone Repository
```bash
mkdir chat-app
git clone https://github.com/Salilw1999/full-stack_chatApp.git
cd full-stack_chatApp
```

### Step 2: Login to Docker Hub
Generate a personal access token and log in:
```bash
docker login -u salilwadke
# Use your token when prompted: dckr_pat_ZqzqXumG_JKoELjLz9C-XiwKnh4
```

### Step 3: Build and Push Backend Docker Image
```bash
cd backend
docker build -t salilwadke/chatapp-backend .
docker images
docker push salilwadke/chatapp-backend
```

### Step 4: Build and Push Frontend Docker Image
```bash
cd ../frontend
docker build -t salilwadke/chatapp-frontend .
docker images
docker push salilwadke/chatapp-frontend
```

### Step 5: Set Up Kubernetes Manifests
Create a `k8s` folder for Kubernetes YAML files:
```bash
cd ..
mkdir k8s
cd k8s
```
Add the following YAML files:
- `namespace.yml`
- `backend-deployment.yml`
- `frontend-deployment.yml`
- `mongodb-deployment.yml`
- `mongodb-pv.yml`
- `mongodb-pvc.yml`

Create a directory for persistent volume:
```bash
mkdir mongodb-data
```

### Step 6: Apply MongoDB Persistent Volumes
```bash
kubectl apply -f mongodb-pv.yml
kubectl apply -f mongodb-pvc.yml
kubectl apply -f mongodb-deployment.yml
kubectl get po
kubectl get pv
kubectl get pvc
```

### Step 7: Create Frontend and Backend Services
Add the following YAML files:
- `frontend-service.yml`
- `backend-service.yml`

### Step 8: Apply Frontend and Backend Services
```bash
kubectl apply -f frontend-service.yml
kubectl apply -f backend-service.yml
```

### Step 9: Deploy Frontend Pods
```bash
kubectl apply -f frontend-deployment.yml
kubectl get po
```

### Step 10: Port Forward Services
```bash
kubectl port-forward service/frontend 8080:80 -n chat-app --address=0.0.0.0
kubectl port-forward service/backend 5001:5001 -n chat-app &
```
Access the application:
```
http://<ec2-ip>:8080
```

### Step 11: Configure Backend for MongoDB
Edit `backend-deployment.yml` and add:
```yaml
- name: MONGODB_URI
  value: "mongodb://mongoadmin:secret@mongodb:27017/dbname?authSource=admin"
```

### Step 12: Create MongoDB Service
Create `mongodb-service.yml` with the service name `mongodb` and apply it:
```bash
kubectl apply -f mongodb-service.yml
```

### Step 13: Create and Apply Secrets for JWT
Create a `secret.yml` file with the JWT secret key:
```yaml
- name: JWT_SECRET
  valueFrom:
    secretKeyRef:
      name: chatapp-secrets
      key: jwt
```
Apply the secret:
```bash
kubectl apply -f secret.yml
```
Update `backend-deployment.yml` with the secret reference:
```yaml
- name: JWT_SECRET
  valueFrom:
    secretKeyRef:
      name: chatapp-secrets
      key: jwt
```

### Step 14: Apply Backend Deployment
```bash
kubectl apply -f backend-deployment.yml
```

### Step 15: Verify MongoDB Connection
Connect to the MongoDB database:
```bash
kubectl exec -it mongodb-deployment-<pod-name> --namespace chat-app -- mongosh --host <mongo-service-ip> --port 27017 -u mongoadmin -p secret --authenticationDatabase admin
```

### Step 16: Access the Application
Open the application in your browser:
```
http://<ec2-ip>:8080
```
### Step 16: For running multiple services at a time (nginx and chat-app_
for ingrass applying :
```
kubectl apply -f ingrass.yml
```

---

This setup deploys the chat application with a 3-tier architecture on Kubernetes, integrating React.js for the frontend, Node.js for the backend, and MongoDB as the database. Use persistent volumes for data durability and configure Kubernetes secrets for secure JWT management.
