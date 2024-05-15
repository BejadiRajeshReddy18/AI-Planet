# GitOps Pipeline with ArgoCD

This repository contains the implementation of a GitOps pipeline using Argo CD and Argo Rollouts for the continuous deployment of a simple web application on a Kubernetes cluster.


## Steps


1. Setup and Configuration

--> Create a GitHub repository    
--> Install Argo CD on the Kubernetes cluster  
--> Install Argo Rollouts on the Kubernetes cluster


2. Creating the GitOps Pipeline

-->Dockerize the web application        
-->Deploy the application using Argo CD


3. Implementing a Canary Release with Argo Rollouts

--> Define a canary release strategy using Argo Rollouts
Trigger a rollout with a new version of the application
Monitor the canary release process

## Prerequisites:


Install Minikube (https://minikube.sigs.k8s.io/docs/start/)  
Install kubectl (Kubernetes command-line tool)   
Install Git  
Install Docker

## Procedure

### Step 1: Create a GitHub Repository

Go to GitHub.com and create a new repository (like planet)
Clone the repository to your local machine

```bash
  git clone https://github.com/your-username/planet.git
  cd planet
```

### Step 2: Create a Simple Python Web Application

Create a new file app.py with the following content:
```bash
    from flask import Flask

    app = Flask(__name__)

    @app.route('/')
    def hello():
    return 'Hello, World!'

    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=5000)
```

Create a requirements.txt file with the following content:
```bash
    flask
```
### Step 3: Dockerize the Application

Create a Dockerfile with the following content:
```bash
    FROM python:3.9-slim

    WORKDIR /app

    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt

    COPY . .

    CMD ["python", "app.py"]
```
Build the Docker image:

```bash
docker build -t planet .
```

Run the container locally to test the application:

```bash
docker run -p 5000:5000 planet
```

### Step 4: Push the Docker Image to a Registry

Create an account on a public container registry (e.g., Docker Hub, GitHub Container Registry).
Tag the Docker image with your registry URL:

```bash
docker tag planet your-registry-url/planet:v1
```

Push the image to the registry:

```bash
docker push your-registry-url/planet:v1
```

### Step 5: Set Up Minikube and Install ArgoCD

Start Minikube:

```bash
minikube start
```

Install ArgoCD:

```bash 
 kubectl create namespace argocd 

 kubectl apply -n argocd -f  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml 
 ``` 

Access the ArgoCD UI:

```bash 
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}' 

kubectl get svc -n argocd # Get the external IP of the argocd-server
```
Use the default username admin and get the password with:
```bash 
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" 

[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("ak5qZVVJRENFUG9tWUQ3WA=="))
```
With first command encoded password is shown to encode the password use the second command

### Step 6: Deploy the Application Using ArgoCD (Task 2)

Create a Manifests directory in your repository and add the following files:

Manifests/deployment.yaml:
```bash 
apiVersion: apps/v1
kind: Deployment
metadata:
    name: planet
spec:
  replicas: 1
  selector:
    matchLabels:
      app: planet
  template:
    metadata:
      labels:
        app: planet
    spec:
      containers:
      - name: planet
        image: your-registry-url/planet:v1
        ports:
        - containerPort: 5000
```
Manifests/service.yaml:

```bash 
    apiVersion: v1
    kind: Service
    metadata:
        name: planet
    spec:
        type: NodePort
    ports:
      - port: 80
        targetPort: 5000
    selector:
        app: planet
```
2. Commit and push the changes to your GitHub repository.
3. In the ArgoCD UI, connect your GitHub repository as a new application:


--> Click "New App"   
--> Enter the repository URL, path to the manifests, and select the cluster.  
--> Click "Create"


4. ArgoCD will automatically deploy the application to your Minikube cluster. You can verify the deployment by running:
```     
kubectl get pods
```
5. Get the external IP of the service:
``` 
minikube service planet --url
```
6. Visit the URL to see the application running.

### Step 7: Implementing a Canary Release with Argo Rollouts (Task 3)

1. Install Argo Rollouts:
```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```
2. Update the Manifests/deployment.yaml file to use Argo Rollouts:
```bash
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: planet
spec:
  replicas: 1
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {}
  selector:
    matchLabels:
      app: planet
  template:
    metadata:
      labels:
        app: planet
    spec:
      containers:
      - name: planet
        image: your-registry-url/planet:v1
        ports:
        - containerPort: 5000
```

3. Commit and push the changes to your GitHub repository.
4. ArgoCD will automatically sync the changes, and Argo Rollouts will manage the canary release.
5. To simulate a new version, update the app.py file to return a different message:
```bash
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, Canary World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
6. Build a new Docker image with a different tag (e.g., v2):
```bash 
docker build -t planet:v2 .
docker push your-registry-url/planet:v2
```
7. Update the k8s/deployment.yaml file to use the new image tag:
```bash
# ...
spec:
  # ...
  template:
    # ...
    spec:
      containers:
      - name: my-gitops-app
        image: your-registry-url/planet:v2
        # ...
```
8. Commit and push the changes to your GitHub repository.
9. ArgoCD will sync the changes, and Argo Rollouts will manage the canary release, gradually shifting traffic from the old version to the new version.
10. You can monitor the rollout progress in the ArgoCD UI or by running:
```bash
kubectl argo rollouts get rollout planet --watch
```
11. Once the rollout is complete, you should see the new message when visiting the application URL.         

#### Python web application is deployed using GitOps principles with ArgoCD and Argo Rollouts, implementing a canary release strategy

## Screenshots
## Screenshots


## Commands

![Screenshot 2024-05-15 223236](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/8ae909c9-9266-4b02-b7de-ed7cdc919957)

![Screenshot 2024-05-15 123852](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/673ff247-d455-44d6-b9e5-f1c2a17e2f47) 


![Screenshot 2024-05-15 213434](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/e1613f8f-8a37-4e25-a334-a67926534b1e)





![Screenshot 2024-05-15 123920](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/f12958b0-6702-48a5-b3ec-f2dc7bff57cb)


## Docker Operations

![Screenshot 2024-05-15 214500](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/d90f0727-b985-4dac-bfb7-8def51b271de)


![Screenshot 2024-05-15 214526](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/9aeebcaa-46d7-4cbe-9a24-8da708b3a504)

![Screenshot 2024-05-15 214736](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/7e95ae05-a67a-4041-a88b-d2eef8d4f436)

![Screenshot 2024-05-15 214754](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/bb5d741d-640c-4f8d-8d7a-be3530901db2)

## 


##  

## ArgoCD Interface

![Screenshot 2024-05-15 125652](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/bee7d56d-b701-4a64-9db1-588a58354a48)

![Screenshot 2024-05-15 130355](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/7abe67aa-f727-4a1c-a197-800a06ab4f05)

![Screenshot 2024-05-15 214239](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/b865990e-ce62-4f48-841a-d0e7ff3416b9)

![Screenshot 2024-05-15 214224](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/b77748a3-2461-4a88-92d0-a33fdcd429b2)

![Screenshot 2024-05-15 214133](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/bac1ed46-b7ac-4c4c-9080-f71c0ff2aaf5)

## 
## Version 1

![Screenshot 2024-05-15 213407](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/a1b01c22-c561-40ec-b7d2-2c150e83980d)

## 
## Version 2
![Screenshot 2024-05-15 215035](https://github.com/BejadiRajeshReddy18/AI-Planet/assets/64033035/c2658192-0548-4cfc-9789-9470c43eb9f6)
