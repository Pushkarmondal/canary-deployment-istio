# Canary Releases with Istio on Kubernetes

This project demonstrates how to implement **canary releases** in Kubernetes using **Istio**. Canary releases allow you to roll out new versions of your application to a small subset of users, monitor their performance, and gradually shift traffic to the new version if everything looks good.
![Diagram](./Screenshot%202025-01-26%20at%2017.44.18.png)
![Deployment](./Screenshot%202025-01-26%20at%2017.21.12.png)
![Monitoring](./Screenshot%202025-01-26%20at%2017.18.40.png)
## [Main Blog Link](https://www.aviator.co/blog/implementing-canary-releases-in-kubernetes-with-istio/#)
---

## Table of Contents
1. [What is a Canary Release?](#what-is-a-canary-release)
2. [Why Use Istio for Canary Releases?](#why-use-istio-for-canary-releases)
3. [Prerequisites](#prerequisites)
4. [Steps to Implement Canary Releases](#steps-to-implement-canary-releases)
5. [Traffic Shifting with Istio](#traffic-shifting-with-istio)
6. [Monitoring and Rollback](#monitoring-and-rollback)
7. [Conclusion](#conclusion)

---

## What is a Canary Release?
A **canary release** is a deployment strategy where a new version of an application is rolled out to a small percentage of users. This allows you to test the new version in production with minimal risk. If the new version performs well, traffic is gradually shifted to it. If issues arise, you can roll back to the stable version.

---

## Why Use Istio for Canary Releases?
Istio is a **service mesh** that provides advanced traffic management, security, and observability for microservices. It is ideal for canary releases because:
- It allows **fine-grained traffic control** (e.g., route 10% of traffic to the new version).
- It provides **automatic metrics collection** (e.g., error rates, latency).
- It supports **automatic retries, timeouts, and circuit breakers** for resilience.

---

## Prerequisites
Before starting, ensure you have the following:
1. A **Kubernetes cluster** (e.g., Minikube, Kind, or a cloud provider like GKE/EKS/AKS).
2. **Istio** installed in your cluster.
3. **kubectl** configured to access your cluster.
4. A sample application (e.g., a simple web app with two versions: `v1` and `v2`).

---

## Steps to Implement Canary Releases

### 1. Install Istio
If Istio is not already installed, follow these steps:
```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH

# Install Istio with the default profile
istioctl install --set profile=default -y

# Enable Istio injection for the default namespace
kubectl label namespace default istio-injection=enabled
```
