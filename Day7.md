# 🚀 CKA Preparation Series — Day 7: Annotations

Day 7 of my **CKA preparation journey**.

Today I learned about **Annotations** in Kubernetes.

At first, I thought annotations were basically the same as labels. But there is an important difference.

---

## 🔹 What are Annotations?

Annotations are **key-value metadata attached to Kubernetes resources**.

They are mainly used to store additional information that Kubernetes tools, controllers, or other applications can use.

For example:

```yaml
metadata:
  name: nginx
  annotations:
    description: "My nginx application"
    owner: "Mukund"
```

Unlike labels, annotations are **not normally used for selecting or grouping resources**.

---

## 🔹 Labels vs Annotations

This was the main thing I focused on today.

| Labels                         | Annotations                          |
| ------------------------------ | ------------------------------------ |
| Used to identify resources     | Used to store additional metadata    |
| Used for selection             | Not normally used for selection      |
| Can be queried using selectors | Generally not used with selectors    |
| Useful for grouping resources  | Useful for tool-specific information |

For example:

```yaml
labels:
  app: nginx
```

can be used by a Service to select Pods.

Whereas:

```yaml
annotations:
  description: "Production nginx application"
```

provides additional information about the resource.

### Simple way to remember:

```text
Labels       → Identification & Selection
Annotations  → Additional Metadata & Configuration
```

---

## 🧪 Adding an Annotation

I can add an annotation using:

```bash
kubectl annotate pod nginx description="My nginx pod"
```

To view it:

```bash
kubectl describe pod nginx
```

Or:

```bash
kubectl get pod nginx -o yaml
```

---

## 🔍 Viewing Annotations

I can inspect the complete resource metadata using:

```bash
kubectl get pod nginx -o yaml
```

Look for:

```yaml
metadata:
  annotations:
    description: My nginx pod
```

I can also use:

```bash
kubectl describe pod nginx
```

to quickly inspect the resource details.

---

## 💡 Real-World Use

Annotations become especially useful when working with Kubernetes controllers and external tools.

For example, **Ingress controllers** can use annotations to configure behavior such as:

* Rewrite rules
* SSL/TLS behavior
* Proxy settings
* Timeouts
* Authentication
* Other controller-specific settings

This makes annotations useful for providing **tool-specific configuration** without changing the core Kubernetes API structure.

---

## ⚠️ Common Mistake

One mistake I want to avoid is treating annotations like labels.

If I need to select Pods based on something like:

```text
app=nginx
```

I should use a **label**, not an annotation.

For example:

```yaml
metadata:
  labels:
    app: nginx
```

A Service can then use:

```yaml
selector:
  app: nginx
```

Annotations don't work this way.

---

## 🎯 CKA Tip

Remember this simple rule:

> **Labels help identify and select resources.**
> **Annotations provide extra information about resources.**

Also be comfortable inspecting metadata quickly:

```bash
kubectl get pod nginx -o yaml
```

and:

```bash
kubectl describe pod nginx
```

For the CKA, understanding **where metadata is stored and how to inspect it quickly** can save valuable time.

---

## 📚 What I Learned Today

The difference between labels and annotations looks small, but it becomes important when working with real Kubernetes configurations.

### Key Takeaway

```text
Labels
  ↓
Identification & Selection

Annotations
  ↓
Additional Metadata & Configuration
```

Today I learned:

* What annotations are
* How to add annotations
* How to inspect annotations
* The difference between labels and annotations
* How annotations can be used by controllers and Kubernetes tools
* Why annotations should not be used for resource selection

**Day 7 complete. ✅**

---

## 📂 Repository Structure

```text

```

![CKA Day 7](/images/day7.png)

---

## 🔗 Follow the Journey

I'm continuing to learn Kubernetes by practicing each topic and documenting what I learn along the way.

Follow **Mukund Kale** for the next post in the **CKA Preparation Series**.

📂 **GitHub — CKA Preparation Repository**

https://github.com/Mukund15k/100DaysOfCKA

---

### 🏷️ Tags

`#CKA` `#Kubernetes` `#DevOps` `#LearningInPublic` `#KubernetesAdministrator` `#CloudComputing` `#100DaysOfCKA`
