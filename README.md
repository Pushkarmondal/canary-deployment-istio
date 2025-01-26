# canary-deployment-istio
Here’s the **README.md** file in Markdown format for your GitHub repository:
https://www.aviator.co/blog/implementing-canary-releases-in-kubernetes-with-istio/#

```markdown
# Canary Releases with Istio on Kubernetes

This project demonstrates how to implement **canary releases** in Kubernetes using **Istio**. Canary releases allow you to roll out new versions of your application to a small subset of users, monitor their performance, and gradually shift traffic to the new version if everything looks good.

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

---

### 2. Deploy the Application
Deploy the **stable version** (`v1`) of your application:
```yaml
# deployment-v1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: docker.io/khabdrick/quotes:v1
        ports:
        - containerPort: 8000
```

Apply the deployment:
```bash
kubectl apply -f deployment-v1.yaml
```

---

### 3. Create a Kubernetes Service
Create a service to expose your application:
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  ports:
  - port: 8000
    name: http
  selector:
    app: myapp
```

Apply the service:
```bash
kubectl apply -f service.yaml
```

---

### 4. Deploy the Canary Version
Deploy the **canary version** (`v2`) of your application:
```yaml
# deployment-v2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      version: v2
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
      - name: myapp
        image: docker.io/khabdrick/quotes:v2
        ports:
        - containerPort: 8000
```

Apply the deployment:
```bash
kubectl apply -f deployment-v2.yaml
```

---

### 5. Configure Istio Routing
Use Istio's **Gateway**, **VirtualService**, and **DestinationRule** to control traffic routing.

#### Gateway
```yaml
# istio.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: myapp-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 8080
      name: http
      protocol: HTTP
    hosts:
    - "*"
```

#### VirtualService
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp-vs
spec:
  hosts:
  - "*"
  gateways:
  - myapp-gateway
  http:
  - route:
    - destination:
        host: myapp
        subset: v1
        port:
          number: 8000
      weight: 90
    - destination:
        host: myapp
        subset: v2
        port:
          number: 8000
      weight: 10
```

#### DestinationRule
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: myapp-destination
spec:
  host: myapp
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

Apply the configurations:
```bash
kubectl apply -f istio.yaml
```

---

## Traffic Shifting with Istio
- Initially, **90% of traffic** goes to `v1` and **10% to `v2`**.
- Gradually increase the traffic to `v2` by updating the `weight` in the `VirtualService`.
- Example: Shift 50% traffic to `v2`:
  ```yaml
  - route:
    - destination:
        host: myapp
        subset: v1
        port:
          number: 8000
      weight: 50
    - destination:
        host: myapp
        subset: v2
        port:
          number: 8000
      weight: 50
  ```

---

## Monitoring and Rollback
- Use **Prometheus** and **Grafana** to monitor metrics like error rates and latency.
- If the canary version (`v2`) performs poorly, roll back by shifting all traffic to `v1`:
  ```yaml
  - route:
    - destination:
        host: myapp
        subset: v1
        port:
          number: 8000
      weight: 100
  ```

---

## Conclusion
This project demonstrates how to implement **canary releases** in Kubernetes using Istio. By following these steps, you can safely roll out new versions of your application, monitor their performance, and ensure a smooth transition with minimal risk.

For more details, refer to the original blog post: [Implementing Canary Releases in Kubernetes with Istio](https://www.aviator.co/blog/implementing-canary-releases-in-kubernetes-with-istio/).

---

Let me know if you need further assistance! 🚀
```

You can copy and paste this into a `README.md` file in your GitHub repository. Let me know if you need any adjustments! 😊
