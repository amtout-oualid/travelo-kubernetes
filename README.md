# Travelo on Kubernetes

Deployment of the 3-tier Travelo application (React, Spring Boot, MySQL) on a local Kubernetes cluster (Minikube).
This is the continuation of [ContainerizationofWebApplication](https://github.com/amtout-oualid/ContainerizationofWebApplication), where the Docker images were built.

## Structure
- `k8s/` Kubernetes manifests (namespace `travelo`): ResourceQuota and LimitRange, Secret, ConfigMap, MySQL StatefulSet, backend and frontend Deployments with their Services, RBAC, PodDisruptionBudgets, Ingress
- `helm/travelo/` Helm chart written from scratch (bonus C)
- `charts/` packaged chart and `index.yaml` (public Helm repository)
- `operator/` MySQL InnoDB Cluster created with the MySQL Operator (bonus A)
- `skaffold.yaml` build and deploy automation (bonus B)
- `app/` source code and Dockerfiles of the three images

## Deploy with kubectl
```
minikube start --driver=docker
minikube addons enable ingress
kubectl apply -f k8s/
minikube tunnel
```
Open http://travelo.localhost

## Deploy with Helm
```
helm repo add travelo https://raw.githubusercontent.com/amtout-oualid/travelo-kubernetes/main/charts
helm install travelo travelo/travelo --namespace helm --create-namespace
```
Open http://travelo-helm.localhost

## Develop with Skaffold
```
skaffold dev --cleanup=false
```

Author: Oualid AMTOUT - INPT, ASEDS
