# Kubernetes-Workshop-Tohar
## Project Overview
This project deploys a WordPress application on Amazon's Elastic Kubernetes Service (EKS), leveraging various tools to ensure the application is both highly available and easily manageable. The primary objective is to create a scalable, reliable, and well-monitored WordPress environment, running on Kubernetes with integrated observability and traffic management.

## Architecture
The architecture of the project involves the following:

1. *EKS Cluster:* A managed Kubernetes cluster on AWS.
2. *WordPress:*
   - Deployment: The WordPress application is deployed as a Deployment with multiple replicas.
   - Service: A Kubernetes ClusterIP service exposes the WordPress app internally.
   - Ingress: Traffic is routed to WordPress through the NGINX Ingress Controller.
4. *MariaDB:*
   - StatefulSet: wordpress-db is deployed as a StatefulSet, providing stable persistent storage.
   - Service: A ClusterIP service is exposed for MariaDB on port 3306.
5. *Prometheus:*
   - StatefulSet: Prometheus is deployed as a StatefulSet for high availability.
   - Service: A ClusterIP service exposes Prometheus for scraping metrics.
   - Monitoring: Prometheus scrapes metrics from WordPress and Kubernetes components.
6. *Grafana:* Collecting and visualizing application and system metrics.
   - Deployment: Grafana is deployed as a deployment and visualizes data from Prometheus.
   - Service: Grafana is exposed via a ClusterIP service, available on port 80.
7. *Alertmanager:* Handling alerts based on defined Prometheus rules.
   - StatefulSet: Alertmanager is deployed as a StatefulSet.
   - Service: A ClusterIP service exposes Alertmanager to handle alerts based on metrics.
8. *NGINX Ingress Controller:*
   - Deployment: The NGINX Ingress controller is deployed to route traffic.
   - Service: The service is of type LoadBalancer, exposing the ingress controller externally.
   or
   *ALB Ingress Controller:*
   - Deployment: The AWS ALB Ingress Controller is used to route traffic to the WordPress application.
   - Service: The ALB routes traffic externally via the Load Balancer exposed on port 80.


# Setup Instructions
  Follow these steps to deploy the project:

## Step 1: Set up your EKS Cluster
  1. Create your EKS cluster through the AWS Console or CLI.
  2. Ensure kubectl is configured for the EKS cluster:

      aws eks --region <region> update-kubeconfig --name <cluster_name>
  
## Step 2: Deploy NGINX Ingress Controller
 Deploy the NGINX Ingress Controller using Helm:
     - to install in cluster (match to one person using cluster):

     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

     helm repo update

     helm install tohar-ingress ingress-nginx/ingress-nginx --namespace tohar --set controller.ingressClassResource.name=tohar

  or
     
  1. Deploy the ALB Ingress Controller using Helm:
    helm repo add eks https://aws.github.io/eks-charts
    helm repo update
    helm install <dist-name> eks/aws-alb-ingress-controller --namespace tohar \
    --set clusterName=<cluster_name> \
    --set ingressClass=alb

    *to verify installation:*
    kubectl get all -n tohar
    - verify there is running alb-ingress-controller deployment
    - verify there alb-ingress-controller service, which his type is load balancer and has external ip
    - verify there is alb-ingress-controller-admission service, which his type is cluster ip and has internal ip


      
## Step 3: Deploy Prometheus and Grafana
 Deploy Prometheus and Grafana using Helm:

     helm install prometheus-tohar prometheus-community/kube-prometheus-stack -n tohar -f pro-values
    
## Step 4: Deploy WordPress and MariaDB
 1. Deploy WordPress:

     kubectl apply -f wordpress-deployment

     kubectl apply -f wordpress-service

 2. Deploy storageClass and PVC:

     kubectl apply -f ebs-tohar

     kubectl apply -f db-pvc
 
 4. Deploy MariaDB:

     kubectl apply -f db-statefulset

     kubectl apply -f db-service

## Step 5: Configure Ingress
 1. Deploy Ingress resource to route traffic to WordPress through the NGINX Ingress controller:

     kubectl apply -f wordpress-ingress

 2. Ingress Setup
    Once the NGINX Ingress Controller is deployed, access WordPress via the external IP or domain that the load balancer is exposes in port 80 and to the grafana via the external IP or domain that the load balancer is exposes /grafana.
