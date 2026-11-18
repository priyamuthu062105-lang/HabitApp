
# HabitApp – CI/CD Deployment on Kubernetes

## 1. Project Overview

HabitApp is a Habit Tracker web application built using Java 21 and Spring Boot.

This project demonstrates an end-to-end CI/CD and GitOps workflow using Jenkins, Docker, SonarCloud, Helm, Kubernetes, and Argo CD.

## 2. Technologies Used

- Java 21
- Spring Boot 3.3.4
- Maven
- HTML, CSS, JavaScript
- Docker
- Docker Hub
- Jenkins
- SonarCloud
- Kubernetes (k3d / K3s)
- Helm
- Argo CD
- Traefik Ingress
- GitHub

## 3. Application Features

- Add new habits
- Select daily or weekly frequency
- Mark habits as completed
- Track current streak
- Track longest streak
- View habit statistics
- Delete habits

Note: The current application stores habit data in server memory. Data is not persistent across application restarts.

## 4. CI/CD Pipeline

The Jenkins pipeline performs the following stages:

1. GitHub source code checkout
2. Maven build
3. Unit testing
4. SonarCloud code analysis
5. Quality Gate verification
6. Docker image build
7. Docker image push to Docker Hub
8. Helm chart validation
9. GitOps deployment configuration update

## 5. Docker Configuration

- Docker image: gokulapriyam/habitapp
- Image tag: 7
- Application port: 8080
- Java runtime: Eclipse Temurin 21
- Container runs as a non-root user.

## 6. Kubernetes Deployment

The application is deployed on a local k3d Kubernetes cluster.

Namespace: habitapp

Deployment: habitapp-habitapp-chart

Replicas: 2

Service type: ClusterIP

Service port: 8080

Health checks:
- Readiness probe
- Liveness probe

Resource requests and limits are configured in the Helm values file.

## 7. Helm Configuration

Helm chart location:

habitapp-chart/

Important configuration:
- Replica count: 2
- Image tag: 7
- Service port: 8080
- Ingress enabled: true
- Ingress class: traefik
- Host: habitapp.local

## 8. Argo CD GitOps

Argo CD continuously monitors the GitHub repository.

Repository:
https://github.com/priyamuthu062105-lang/HabitApp

Branch: main

Application: habitapp

Namespace: argocd

Deployment namespace: habitapp

Automated synchronization and self-healing are enabled.

## 9. Application Access

Local browser access through Kubernetes port-forward:

http://localhost:8080

Ingress hostname:

http://habitapp.local

The Ingress resource is managed through the Helm chart and Argo CD.

## 10. Verification Results

The following checks have been completed:

- Jenkins Build #7 successful
- SonarCloud Quality Gate passed
- Docker image successfully pushed
- Helm lint validation passed
- Helm template rendering passed
- Kubernetes nodes Ready
- HabitApp deployment: 2/2 replicas available
- Internal Kubernetes Service returned HTTP 200
- Traefik Ingress resource created
- Argo CD application Synced and Healthy
- Habit Tracker opened successfully in Firefox
- Add Habit functionality tested
- Mark Today Done functionality tested
- Habit statistics updated successfully

## 11. Useful Kubernetes Commands

Check nodes:

kubectl get nodes

Check application pods:

kubectl get pods -n habitapp

Check deployment:

kubectl get deployment -n habitapp

Check service:

kubectl get svc -n habitapp

Check Ingress:

kubectl get ingress -n habitapp

Check Argo CD:

kubectl get application habitapp -n argocd

## 12. Project Outcome

This project demonstrates an automated CI/CD and GitOps deployment workflow for a Java Spring Boot application using Jenkins, Docker, SonarCloud, Helm, Kubernetes, Argo CD, and Traefik.

The application was successfully deployed and accessed through a browser in a local Kubernetes environment.

