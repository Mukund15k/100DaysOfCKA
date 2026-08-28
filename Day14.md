Below is a detailed `README.md` you can place inside your `Day-14-Ingress` folder. I’ve kept the structure consistent with your previous CKA days and added hands-on commands, troubleshooting, YAML, and exam-focused notes.

# 🚀 CKA Preparation Series — Day 14: Kubernetes Ingress

Continuing my **CKA Preparation Series | Learning Kubernetes in Public**.

Today I focused on **Kubernetes Ingress** — an important networking concept for routing external HTTP/HTTPS traffic to applications running inside a Kubernetes cluster.

In the previous days, I learned about:

* Pod networking
* Services
* Service Discovery
* Kubernetes DNS

Today, I connected these concepts together with **Ingress**.

The basic traffic flow is:

```text
Client
   ↓
Ingress
   ↓
Service
   ↓
Pod
```

---

# 📚 Table of Contents

* [What is Ingress?](#-what-is-ingress)
* [Why Do We Need Ingress?](#-why-do-we-need-ingress)
* [Ingress vs Service](#-ingress-vs-service)
* [Ingress Architecture](#-ingress-architecture)
* [Ingress Controller](#-ingress-controller)
* [Ingress Routing](#-ingress-routing)
* [Path-Based Routing](#-path-based-routing)
* [Host-Based Routing](#-host-based-routing)
* [Simple Ingress Example](#-simple-ingress-example)
* [Hands-On Lab](#-hands-on-lab)
* [TLS and HTTPS](#-tls-and-https)
* [Useful kubectl Commands](#-useful-kubectl-commands)
* [Troubleshooting](#-ingress-troubleshooting)
* [Common Mistakes](#-common-mistakes)
* [CKA Exam Tips](#-cka-exam-tips)
* [Key Takeaways](#-key-takeaways)

---

# 🔹 What is Ingress?

A Kubernetes **Ingress** is an API resource used to define rules for routing external **HTTP and HTTPS traffic** to Services inside a Kubernetes cluster.

Instead of exposing every application separately, we can use an Ingress to define routing rules.

For example:

```text
                     Internet
                        |
                        v
                    Ingress
                   /   |   \
                  /    |    \
                 v     v     v
             Frontend Backend Admin
              Service  Service Service
                 |       |       |
                Pods    Pods     Pods
```

Ingress can route traffic based on:

* Hostnames
* URL paths
* HTTP/HTTPS
* TLS configuration

---

# 🔹 Why Do We Need Ingress?

Suppose we have three applications:

```text
Frontend
Backend API
Admin Application
```

One approach would be to expose each application separately.

For example:

```text
frontend.example.com
api.example.com
admin.example.com
```

Or expose different external ports.

This can become difficult to manage as the number of applications increases.

Ingress provides a centralized HTTP/HTTPS routing layer:

```text
                       Internet
                          |
                          v
                       Ingress
                     /    |    \
                    /     |     \
                   v      v      v
              Frontend  Backend  Admin
               Service  Service  Service
```

This allows multiple applications to share a common entry point.

---

# 🔹 Main Capabilities of Ingress

Ingress can provide:

### ✅ Host-Based Routing

Route requests based on the hostname.

```text
app.example.com
        ↓
Frontend Service
```

```text
api.example.com
        ↓
Backend Service
```

---

### ✅ Path-Based Routing

Route requests based on the URL path.

```text
example.com/
        ↓
frontend-service
```

```text
example.com/api
        ↓
backend-service
```

```text
example.com/admin
        ↓
admin-service
```

---

### ✅ TLS Termination

Ingress can be configured to handle HTTPS/TLS and forward traffic to Services.

```text
Client
  |
HTTPS
  |
  v
Ingress
  |
HTTP
  |
  v
Service
  |
  v
Pod
```

The exact TLS behavior depends on the Ingress Controller configuration.

---

# 🔹 Ingress vs Service

This distinction is important for the CKA.

## Service

A Service provides a stable network endpoint for Pods.

```text
Service
   ↓
Pods
```

It can provide:

* Stable IP
* Stable DNS name
* Load balancing across selected Pods
* Internal or external exposure depending on Service type

---

## Ingress

Ingress defines HTTP/HTTPS routing rules.

```text
Ingress
   ↓
Service
   ↓
Pods
```

So the simplified relationship is:

```text
Internet
   ↓
Ingress
   ↓
Service
   ↓
Pods
```

### Remember:

**Service = stable access to Pods**

**Ingress = HTTP/HTTPS routing to Services**

---

# 🔹 Ingress Architecture

A simplified Kubernetes architecture looks like:

```text
                     External Client
                           |
                           v
                    Load Balancer /
                    NodePort / IP
                           |
                           v
                  Ingress Controller
                           |
                           v
                      Ingress Rules
                           |
             +-------------+-------------+
             |                           |
             v                           v
      frontend-service            backend-service
             |                           |
             v                           v
        Frontend Pods               Backend Pods
```

The exact infrastructure depends on the Kubernetes environment and the Ingress Controller being used.

---

# 🔹 Ingress Controller

One of the most important concepts I learned today:

> **Creating an Ingress resource does not automatically route traffic.**

An **Ingress Controller** must be running to watch Ingress resources and implement their routing rules.

The relationship is:

```text
Ingress Resource
       ↓
Routing Rules
       ↓
Ingress Controller
       ↓
Actual Traffic Routing
```

Examples of Ingress Controller implementations include:

* NGINX Ingress Controller
* Traefik
* HAProxy
* Kong
* Cloud-provider-specific controllers

The exact controller available depends on the Kubernetes environment.

---

# 🔹 Ingress Resource vs Ingress Controller

This is an important distinction.

### Ingress

The Ingress is a Kubernetes API object containing routing configuration.

For example:

```yaml
kind: Ingress
```

It defines:

```text
Host
Path
Backend Service
Port
TLS
```

---

### Ingress Controller

The Controller is the component that actually processes those rules and configures the underlying proxy/load-balancing behavior.

Think of it this way:

```text
Ingress
= Configuration

Ingress Controller
= Implementation
```

---

# 🔹 Path-Based Routing

Path-based routing allows different URL paths to be routed to different Services.

For example:

```text
example.com/
     ↓
frontend-service
```

```text
example.com/api
     ↓
backend-service
```

```text
example.com/admin
     ↓
admin-service
```

The traffic flow becomes:

```text
                         example.com
                              |
                              v
                           Ingress
                       /      |      \
                      /       |       \
                     v        v        v
                    /        /api    /admin
                    |         |        |
                    v         v        v
               Frontend    Backend    Admin
                Service    Service   Service
```

---

# 🔹 Host-Based Routing

Host-based routing routes traffic based on the requested hostname.

For example:

```text
app.example.com
        ↓
frontend-service
```

```text
api.example.com
        ↓
backend-service
```

```text
admin.example.com
        ↓
admin-service
```

This is useful when multiple applications use different domains or subdomains.

---

# 🔹 Simple Ingress Example

Suppose we already have:

```text
backend-service
```

running on port `80`.

We can create an Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80
```

Apply it:

```bash
kubectl apply -f ingress.yaml
```

Check it:

```bash
kubectl get ingress
```

---

# 🔹 Understanding the Ingress YAML

Let's break it down.

### API Version

```yaml
apiVersion: networking.k8s.io/v1
```

This is the API version used for the current Ingress API.

---

### Kind

```yaml
kind: Ingress
```

This tells Kubernetes that we are creating an Ingress resource.

---

### Name

```yaml
metadata:
  name: app-ingress
```

The name of the Ingress resource.

---

### Host

```yaml
host: example.com
```

This specifies the hostname for the routing rule.

---

### Path

```yaml
path: /api
```

Requests matching `/api` will be routed according to this rule.

---

### Path Type

```yaml
pathType: Prefix
```

`Prefix` means the path is matched based on the URL path prefix.

For example:

```text
/api
/api/users
/api/orders
```

can match the `/api` prefix rule.

---

### Backend Service

```yaml
backend:
  service:
    name: backend-service
    port:
      number: 80
```

The request is forwarded to:

```text
backend-service:80
```

The Service then routes traffic to its selected Pods.

---

# 🔹 Complete Traffic Flow

Putting everything together:

```text
Browser
   |
   | HTTP/HTTPS
   v
Ingress Controller
   |
   | Reads Ingress rules
   v
Ingress
   |
   | /api
   v
backend-service
   |
   | Service selector
   v
Backend Pods
```

This is the mental model I want to remember.

---

# 🧪 Hands-On Lab

Let's create a simple application with two Services:

```text
Frontend
Backend
```

Then use an Ingress to route traffic.

---

## Step 1 — Create Frontend Deployment

```bash
kubectl create deployment frontend \
  --image=nginx \
  --replicas=2
```

Check the Pods:

```bash
kubectl get pods
```

---

## Step 2 — Create Frontend Service

```bash
kubectl expose deployment frontend \
  --name=frontend-service \
  --port=80 \
  --target-port=80
```

Check:

```bash
kubectl get svc
```

---

## Step 3 — Create Backend Deployment

```bash
kubectl create deployment backend \
  --image=nginx \
  --replicas=2
```

---

## Step 4 — Create Backend Service

```bash
kubectl expose deployment backend \
  --name=backend-service \
  --port=80 \
  --target-port=80
```

Check:

```bash
kubectl get svc
```

---

# 🔹 Step 5 — Create Ingress

Create:

```text
ingress.yaml
```

with:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80

          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80
```

Apply:

```bash
kubectl apply -f ingress.yaml
```

---

# 🔍 Step 6 — Check the Ingress

```bash
kubectl get ingress
```

For more details:

```bash
kubectl describe ingress app-ingress
```

Check the backend Services:

```bash
kubectl get svc
```

Check Pods:

```bash
kubectl get pods -o wide
```

---

# 🔹 Understanding the Routing

With the above configuration:

```text
http://example.com/
        ↓
frontend-service
        ↓
Frontend Pods
```

And:

```text
http://example.com/api
        ↓
backend-service
        ↓
Backend Pods
```

The Ingress Controller evaluates the request and routes it according to the configured rules.

---

# 🔹 PathType

The `networking.k8s.io/v1` Ingress API requires a `pathType`.

Common values include:

### Prefix

```yaml
pathType: Prefix
```

Matches URL paths based on a prefix.

Example:

```text
/api
/api/users
/api/orders
```

---

### Exact

```yaml
pathType: Exact
```

Matches the exact path.

For example:

```text
/api
```

would match exactly `/api`, subject to the Kubernetes path matching rules.

---

# 🔹 TLS and HTTPS

Ingress can also define TLS configuration.

A simplified example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
spec:
  tls:
    - hosts:
        - example.com
      secretName: example-tls

  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

The TLS Secret might contain the certificate and private key.

Example:

```text
TLS Secret
    ↓
Ingress Controller
    ↓
HTTPS
    ↓
Service
    ↓
Pods
```

The exact TLS setup depends on the Ingress Controller and certificate-management approach being used.

---

# 🔹 Creating a TLS Secret

For a lab environment, a TLS Secret can be created from certificate files:

```bash
kubectl create secret tls example-tls \
  --cert=tls.crt \
  --key=tls.key
```

Check it:

```bash
kubectl get secret example-tls
```

---

# 🔍 Useful kubectl Commands

### List Ingresses

```bash
kubectl get ingress
```

or:

```bash
kubectl get ing
```

---

### Describe Ingress

```bash
kubectl describe ingress <ingress-name>
```

---

### Get Ingress YAML

```bash
kubectl get ingress <ingress-name> -o yaml
```

---

### List Services

```bash
kubectl get svc
```

---

### Check Endpoints

```bash
kubectl get endpoints
```

---

### Check EndpointSlices

```bash
kubectl get endpointslices
```

---

### Check Pods

```bash
kubectl get pods -o wide
```

---

### Check Ingress Controller Pods

Depending on the controller and installation:

```bash
kubectl get pods -A
```

Then identify the namespace and Pods belonging to the Ingress Controller.

---

# 🔍 Ingress Troubleshooting

When an Ingress isn't working, I want to troubleshoot from the outside in.

---

## 1. Check the Ingress

```bash
kubectl get ingress
```

Then:

```bash
kubectl describe ingress <ingress-name>
```

Check:

* Host
* Path
* Backend Service
* Backend port
* Events

---

## 2. Check the Ingress Controller

Remember:

> An Ingress resource alone doesn't implement traffic routing.

Check whether an appropriate Ingress Controller is running:

```bash
kubectl get pods -A
```

Look for the controller used by your cluster.

---

## 3. Check the Service

```bash
kubectl get svc
```

Then:

```bash
kubectl describe svc <service-name>
```

Make sure:

```text
Service Port
Target Port
Selector
```

are correct.

---

## 4. Check Endpoints

```bash
kubectl get endpoints <service-name>
```

If there are no endpoints, check the Service selector and Pod labels.

For example:

```text
Service selector:
app=backend
```

Pod:

```text
app=backend
```

These need to match appropriately.

---

## 5. Check Pod Status

```bash
kubectl get pods
```

Then:

```bash
kubectl describe pod <pod-name>
```

Check:

* Pod status
* Readiness
* Container status
* Events

---

## 6. Test the Service Directly

Before blaming Ingress, verify that the Service itself works.

For example, from a test Pod:

```bash
kubectl run curl-test \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Then test:

```bash
curl http://frontend-service
```

and:

```bash
curl http://backend-service
```

If the Service doesn't work directly, troubleshoot the Service and Pods before troubleshooting Ingress.

---

# 🔥 Recommended Troubleshooting Flow

A useful CKA troubleshooting sequence:

```text
Client
  ↓
Ingress Controller
  ↓
Ingress Rules
  ↓
Service
  ↓
Endpoints
  ↓
Pods
```

Check each layer:

```text
1. Is the Ingress Controller running?
             ↓
2. Is the Ingress configured correctly?
             ↓
3. Does the hostname/path match?
             ↓
4. Does the backend Service exist?
             ↓
5. Does the Service have endpoints?
             ↓
6. Do the Pods have the correct labels?
             ↓
7. Are the Pods Ready?
```

This approach prevents jumping directly into random configuration changes.

---

# ⚠️ Common Mistakes

## ❌ Mistake 1 — No Ingress Controller

Creating:

```yaml
kind: Ingress
```

doesn't automatically create the component that implements the routing.

Remember:

```text
Ingress Resource
      ↓
Ingress Controller
      ↓
Traffic Routing
```

---

## ❌ Mistake 2 — Wrong Service Name

Ingress:

```yaml
service:
  name: backend-service
```

But the actual Service is:

```text
backend
```

The routing won't work as expected.

---

## ❌ Mistake 3 — Wrong Service Port

Ingress:

```yaml
port:
  number: 80
```

Make sure the referenced Service exposes the expected port.

---

## ❌ Mistake 4 — Service Has No Endpoints

Check:

```bash
kubectl get endpoints <service-name>
```

If no endpoints exist, inspect:

```bash
kubectl get pods --show-labels
```

and compare with the Service selector.

---

## ❌ Mistake 5 — Incorrect Host

Ingress:

```yaml
host: example.com
```

But the request is sent to:

```text
api.example.com
```

The request may not match the intended rule.

---

## ❌ Mistake 6 — Incorrect Path

For example:

```yaml
path: /api
```

but the expected application behavior assumes a different path.

Always verify the path and `pathType`.

---

# 🎯 CKA Exam Tips

For the CKA, I want to be comfortable with:

### Creating an Ingress

Know the basic structure:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80
```

---

### Inspecting an Ingress

```bash
kubectl get ingress
```

```bash
kubectl describe ingress <name>
```

```bash
kubectl get ingress <name> -o yaml
```

---

### Troubleshooting

Remember:

```text
Ingress
   ↓
Ingress Controller
   ↓
Service
   ↓
Endpoints
   ↓
Pods
```

---

# 🧠 Important CKA Concepts

### Ingress

Defines HTTP/HTTPS routing rules.

```text
Ingress = Routing Configuration
```

### Ingress Controller

Implements the routing rules.

```text
Controller = Routing Implementation
```

### Service

Provides stable access to Pods.

```text
Service = Stable Network Endpoint
```

### Pod

Runs the application workload.

```text
Pod = Application Workload
```

Together:

```text
Client
  ↓
Ingress
  ↓
Ingress Controller
  ↓
Service
  ↓
Pod
```

---

# 🔄 Real-World Example

Consider an e-commerce application:

```text
                    Internet
                       |
                       v
                 Ingress Controller
                       |
                       v
                     Ingress
                  /     |      \
                 /      |       \
                v       v        v
            Frontend  Backend   Admin
             Service  Service  Service
                |        |        |
                v        v        v
              Pods      Pods      Pods
```

The routing rules could look like:

```text
shop.example.com
        ↓
frontend-service
```

```text
shop.example.com/api
        ↓
backend-service
```

```text
admin.example.com
        ↓
admin-service
```

This provides a centralized HTTP/HTTPS entry point for multiple applications.

---

# 💡 What I Learned Today

The biggest takeaway from today is that **Ingress is primarily about HTTP/HTTPS routing**.

Instead of exposing every application individually, I can define routing rules that direct traffic to different Services.

The mental model I want to remember is:

```text
External Client
      ↓
    Ingress
      ↓
   Service
      ↓
     Pod
```

And the most important distinction:

```text
Ingress
   ↓
Routing Rules

Ingress Controller
   ↓
Implements the Rules
```

Ingress became much easier to understand after connecting it with the concepts I learned in previous days:

```text
Pods
  ↓
Services
  ↓
Service Discovery
  ↓
Ingress
```

---

# 📌 Key Takeaways

Today I learned:

* What Kubernetes Ingress is
* Why Ingress is useful
* Ingress vs Service
* Ingress vs Ingress Controller
* Host-based routing
* Path-based routing
* `pathType`
* TLS configuration
* Ingress-to-Service communication
* How to inspect Ingress resources
* How to troubleshoot Ingress
* How Services and Endpoints fit into the Ingress traffic flow

### Quick Revision

```text
                    Internet
                       |
                       v
                     Ingress
                       |
                       v
              Ingress Controller
                       |
                       v
                    Service
                       |
                       v
                 Service Selector
                       |
                       v
                      Pods
```

**Day 14 complete. ✅**

Next: **Day 15 — Kubernetes NetworkPolicies**

---

# 📂 Repository Structure

```text
100DaysOfCKA/
├── Day-01-Kubernetes-Architecture/
├── Day-02-Kubernetes-Objects/
├── Day-03-Pods/
├── Day-04-YAML-kubectl/
├── Day-05-Namespaces/
├── Day-06-Labels-Selectors/
├── Day-07-Annotations/
├── Day-08-Deployments/
├── Day-09-ReplicaSets/
├── Day-10-Jobs-CronJobs/
├── Day-11-Kubernetes-Networking/
├── Day-12-Kubernetes-Services/
├── Day-13-Service-Discovery/
├── Day-14-Ingress/
│   ├── README.md
│   ├── ingress.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backend-deployment.yaml
│   └── backend-service.yaml
├── Day-15-NetworkPolicies/
└── images/
    └── day14.png
```

---

# 🔗 Follow the Journey

I'm continuing to learn Kubernetes by practicing each topic and documenting what I learn along the way.

📂 **GitHub — CKA Preparation Repository**

[https://github.com/Mukund15kale/100DaysOfCKA](https://github.com/Mukund15kale/100DaysOfCKA)

Follow **Mukund Kale** for the next post in the **CKA Preparation Series**.

---

## 🏷️ Tags

`#CKA` `#Kubernetes` `#CKAPreparation` `#DevOps` `#KubernetesNetworking` `#Ingress` `#CloudNative` `#CloudComputing` `#LearningInPublic` `#DevOpsEngineer` `#100DaysOfCKA`

This is ready to save as `Day-14-Ingress/README.md`.
