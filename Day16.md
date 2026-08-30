# 🚀 CKA Preparation Series — Day 16: CoreDNS

Continuing my **CKA Preparation Series | Learning Kubernetes in Public**.

Today I focused on **CoreDNS** — an important component for service discovery and DNS communication inside a Kubernetes cluster.

---

## 🔹 What is CoreDNS?

**CoreDNS** is the DNS server used by Kubernetes to provide name resolution inside the cluster.

Instead of applications communicating directly using changing Pod IP addresses, they can communicate using Kubernetes DNS names.

For example:

```text
backend-service.default.svc.cluster.local
```

This makes service-to-service communication easier and more reliable.

---

## 🔹 How does CoreDNS work?

A typical request looks like:

```text
Pod
 ↓
DNS Query
 ↓
CoreDNS
 ↓
Service
 ↓
Backend Pods
```

CoreDNS watches Kubernetes resources such as Services and provides DNS records for them.

This allows applications to discover Services using stable DNS names.

---

## 🔹 Kubernetes Service DNS

A Service can be accessed using:

```text
<service-name>.<namespace>.svc.cluster.local
```

Example:

```text
backend-service.default.svc.cluster.local
```

If the application and Service are in the same namespace, the shorter name can usually be used:

```text
backend-service
```

---

## 🧪 Useful Commands

### Check CoreDNS Pods

```bash
kubectl get pods -n kube-system
```

### Check CoreDNS Service

```bash
kubectl get svc -n kube-system
```

### Check CoreDNS Configuration

```bash
kubectl -n kube-system get configmap coredns -o yaml
```

### Check CoreDNS Logs

```bash
kubectl logs -n kube-system <coredns-pod>
```

### Test DNS from a Pod

```bash
kubectl exec -it <pod-name> -- nslookup <service-name>
```

### Check Pod DNS Configuration

```bash
kubectl exec -it <pod-name> -- cat /etc/resolv.conf
```

---

## 🔍 Common DNS Troubleshooting

If one Pod cannot communicate with another using the Service name, I would check:

1. Is the Service running?

```bash
kubectl get svc
```

2. Are the Service selectors correct?

```bash
kubectl describe svc <service-name>
```

3. Are the backend Pods running and ready?

```bash
kubectl get pods
```

4. Does the Service have endpoints?

```bash
kubectl get endpoints <service-name>
```

5. Is CoreDNS running?

```bash
kubectl get pods -n kube-system
```

6. Is the Pod's DNS configuration correct?

```bash
kubectl exec -it <pod-name> -- cat /etc/resolv.conf
```

---

## ⚠️ Common Mistake

One mistake is assuming that if DNS resolution fails, CoreDNS must be the problem.

Instead, troubleshoot step by step:

```text
Application
     ↓
Service
     ↓
Endpoints
     ↓
DNS Resolution
     ↓
CoreDNS
```

A Service selector mismatch or missing endpoints can cause connectivity problems even when CoreDNS itself is working correctly.

---

## 🎯 CKA Exam Tip

Remember:

```text
Service Name
      ↓
DNS Resolution
      ↓
CoreDNS
      ↓
Service
      ↓
Backend Pods
```

Useful commands to remember:

```bash
kubectl get svc
kubectl get endpoints
kubectl get pods -n kube-system
kubectl get svc -n kube-system
kubectl exec -it <pod> -- nslookup <service>
```

When troubleshooting DNS, **don't immediately restart CoreDNS**. First identify which layer is actually causing the problem.

---

## 💡 What I Learned

The biggest takeaway for me today:

**Kubernetes DNS makes service-to-service communication practical in a dynamic environment.**

Instead of depending on changing Pod IP addresses, applications can communicate using stable Service names such as:

```text
backend-service
```

or:

```text
backend-service.default.svc.cluster.local
```

Understanding CoreDNS also makes it easier to troubleshoot Kubernetes **Services, Service Discovery, and Pod connectivity**.

---

## 📚 Next Step

Next, I'll move deeper into:

**Kubernetes Service Troubleshooting**

I'm continuing to learn Kubernetes by practicing each topic and documenting what I learn along the way.

📂 **GitHub — CKA Preparation Repository**

https://github.com/Mukund15kale/100DaysOfCKA

🔔 Follow **Mukund Kale** for the next post in the **CKA Preparation Series**.

---

## 🏷️ Tags

`#Kubernetes` `#CKA` `#CKAPreparation` `#DevOps` `#CloudComputing` `#KubernetesNetworking` `#CoreDNS` `#LearningInPublic` `#DevOpsLearning` `#100DaysOfCKA`
