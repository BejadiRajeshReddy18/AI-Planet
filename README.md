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
