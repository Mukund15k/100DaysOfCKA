# 🚀 CKA Preparation Series — Day 15: Kubernetes NetworkPolicy

Continuing my **CKA Preparation Series | Learning Kubernetes in Public**.

Today I focused on **Kubernetes NetworkPolicy** — an important topic in Kubernetes networking and security.

![Kubernetes NetworkPolicy](images/day15-networkpolicy.png)

Until now, I was learning how Pods communicate with each other and how Services and Ingress route traffic.

But there is another important question:

> **Who should be allowed to communicate with whom?**

That's where **NetworkPolicy** comes in.

---

## 📚 Table of Contents

- [What is NetworkPolicy?](#-what-is-networkpolicy)
- [Why NetworkPolicy?](#-why-networkpolicy)
- [How NetworkPolicy Works](#-how-networkpolicy-works)
- [Ingress and Egress](#-ingress-and-egress)
- [Pod Selectors](#-pod-selectors)
- [Simple NetworkPolicy Example](#-simple-networkpolicy-example)
- [Understanding the YAML](#-understanding-the-yaml)
- [Real-World Example](#-real-world-example)
- [Default Deny Policies](#-default-deny-policies)
- [Allowing Specific Traffic](#-allowing-specific-traffic)
- [Hands-On Lab](#-hands-on-lab)
- [Useful kubectl Commands](#-useful-kubectl-commands)
- [Troubleshooting](#-troubleshooting)
- [Common Mistakes](#-common-mistakes)
- [CKA Exam Strategy](#-cka-exam-strategy)
- [Key Takeaways](#-key-takeaways)

---

# 🔹 What is NetworkPolicy?

A **NetworkPolicy** is a Kubernetes API object used to control network traffic **to and from Pods**.

It can define which traffic is allowed based on things such as:

- Pod selectors
- Namespace selectors
- IP blocks
- Ports
- Protocols
- Traffic direction

A simple example:

```text
Frontend
   |
   v
Backend
   |
   v
Database
```

We may want:

```text
Frontend  ───────► Backend       ✅ Allowed
Backend   ───────► Database      ✅ Allowed
Frontend  ───────► Database      ❌ Not Allowed
Other Pod ───────► Database      ❌ Not Allowed
```

NetworkPolicy allows us to express these communication requirements.

---

# 🔹 Why NetworkPolicy?

Without network restrictions, workloads may be able to communicate more broadly than necessary, depending on the cluster networking configuration.

For a microservices application, this can create unnecessary network exposure.

For example:

```text
             Kubernetes Cluster

Frontend ──────► Backend ──────► Database
   │                │
   │                │
   └───────────────► Database
```

We may not want the Frontend to communicate directly with the Database.

Instead:

```text
Frontend
   |
   v
Backend
   |
   v
Database
```

This follows the security principle:

> **Allow only the traffic that is actually required.**

This is commonly associated with **least-privilege network access**.

---

# 🔹 How NetworkPolicy Works

A NetworkPolicy uses a `podSelector` to identify the Pods to which the policy applies.

Example:

```yaml
spec:
  podSelector:
    matchLabels:
      app: backend
```

This means the policy applies to Pods with:

```text
app=backend
```

The policy can then define:

- `ingress` → incoming traffic
- `egress` → outgoing traffic

The basic model is:

```text
                 NetworkPolicy
                       |
              +--------+--------+
              |                 |
           Ingress            Egress
          Incoming           Outgoing
           Traffic             Traffic
```

---

# 🔹 Ingress and Egress

Understanding the direction of traffic is extremely important for the CKA.

## Ingress

**Ingress = incoming traffic to the selected Pod**

For example:

```text
Frontend
   |
   | Incoming request
   v
Backend Pod
```

A NetworkPolicy controlling traffic **to the Backend Pod** is controlling ingress to that Pod.

---

## Egress

**Egress = outgoing traffic from the selected Pod**

For example:

```text
Backend Pod
   |
   | Outgoing request
   v
Database
```

A NetworkPolicy controlling traffic **from the Backend Pod** is controlling egress from that Pod.

---

## Quick Memory Trick

```text
Ingress = In
Egress  = Exit
```

---

# 🔹 Pod Selectors

Pod labels are extremely important when working with NetworkPolicies.

Suppose the Pods have:

```yaml
labels:
  app: frontend
```

and:

```yaml
labels:
  app: backend
```

A NetworkPolicy can select the backend:

```yaml
podSelector:
  matchLabels:
    app: backend
```

Then it can allow traffic from frontend Pods:

```yaml
from:
  - podSelector:
      matchLabels:
        app: frontend
```

The relationship becomes:

```text
Frontend Pods
app=frontend
      |
      | Allowed
      v
Backend Pods
app=backend
```

---

# 🔹 Simple NetworkPolicy Example

Here is a basic policy that allows traffic to backend Pods only from frontend Pods:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
```

The policy selects:

```text
app=backend
```

Pods.

It then allows ingress traffic from:

```text
app=frontend
```

Pods.

Conceptually:

```text
Frontend
app=frontend
     |
     | Allowed
     v
Backend
app=backend
```

---

# 🔹 Understanding the YAML

Let's break the manifest down.

## API Version

```yaml
apiVersion: networking.k8s.io/v1
```

This specifies the Kubernetes NetworkPolicy API.

---

## Kind

```yaml
kind: NetworkPolicy
```

This tells Kubernetes that we are creating a NetworkPolicy.

---

## Metadata

```yaml
metadata:
  name: backend-policy
```

This gives the policy a name.

---

## Pod Selector

```yaml
podSelector:
  matchLabels:
    app: backend
```

This identifies the Pods that the policy applies to.

In this case:

```text
app=backend
```

---

## Policy Types

```yaml
policyTypes:
  - Ingress
```

This tells Kubernetes that the policy governs ingress traffic for the selected Pods.

For both directions:

```yaml
policyTypes:
  - Ingress
  - Egress
```

---

## Ingress Rule

```yaml
ingress:
  - from:
      - podSelector:
          matchLabels:
            app: frontend
```

This allows traffic from Pods with:

```text
app=frontend
```

to the selected backend Pods.

---

# 🔹 Real-World Example

Consider a typical microservices application:

```text
                  Frontend
                     |
                     v
                  Backend
                 /       \
                v         v
             Redis     Database
```

We could define communication requirements such as:

```text
Frontend → Backend       ✅
Backend  → Redis         ✅
Backend  → Database      ✅

Frontend → Database      ❌
Frontend → Redis         ❌
Other Pods → Database    ❌
```

NetworkPolicies can be used to enforce these boundaries.

The goal is not to block networking randomly.

The goal is to define:

> **Which workloads actually need to communicate?**

---

# 🔐 Default Deny Policies

A common security pattern is to start with a **default-deny** policy and then explicitly allow required traffic.

For example, deny all ingress traffic to selected Pods:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

The empty selector:

```yaml
podSelector: {}
```

selects all Pods in the policy's namespace.

This creates an important security model:

```text
Default
   ↓
Deny
   ↓
Explicitly allow required traffic
```

A similar policy can be created for egress:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}
  policyTypes:
    - Egress
```

Be careful with default-deny egress policies because they can block required DNS or other outbound communication unless those flows are explicitly allowed.

---

# 🔹 Allowing Specific Traffic

NetworkPolicies can also restrict traffic by port.

For example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

This policy allows:

```text
Frontend
   |
   | TCP :8080
   v
Backend
```

Other traffic is not allowed by this policy.

---

# 🔹 Namespace Selectors

NetworkPolicy can also use a `namespaceSelector`.

For example:

```yaml
ingress:
  - from:
      - namespaceSelector:
          matchLabels:
            environment: production
```

This can allow traffic from Pods located in namespaces matching:

```text
environment=production
```

A combination of selectors can be used when the requirement calls for it.

For example:

```yaml
from:
  - namespaceSelector:
      matchLabels:
        environment: production
    podSelector:
      matchLabels:
        app: frontend
```

The exact selector combination matters, so always read the YAML carefully during the CKA.

---

# 🧪 Hands-On Lab

Let's create a small environment to practice NetworkPolicies.

## Step 1 — Create a Namespace

```bash
kubectl create namespace network-policy-demo
```

Switch to it:

```bash
kubectl config set-context --current --namespace=network-policy-demo
```

---

## Step 2 — Create Frontend Pod

```bash
kubectl run frontend \
  --image=nginx \
  --labels="app=frontend"
```

---

## Step 3 — Create Backend Pod

```bash
kubectl run backend \
  --image=nginx \
  --labels="app=backend"
```

Check the Pods:

```bash
kubectl get pods --show-labels
```

Expected labels:

```text
frontend   app=frontend
backend    app=backend
```

---

# 🔹 Step 4 — Test Connectivity Before Policy

Before applying a policy, test communication between Pods.

First identify the backend IP:

```bash
kubectl get pods -o wide
```

Then test from the frontend Pod if the image contains the required networking utility.

For example:

```bash
kubectl exec -it frontend -- sh
```

Inside the container:

```bash
curl http://<backend-pod-ip>
```

The exact result depends on the container image and networking environment.

---

# 🔹 Step 5 — Create NetworkPolicy

Create:

```text
backend-policy.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
```

Apply it:

```bash
kubectl apply -f backend-policy.yaml
```

---

# 🔍 Step 6 — Verify the Policy

```bash
kubectl get networkpolicy
```

or:

```bash
kubectl get netpol
```

Describe it:

```bash
kubectl describe networkpolicy backend-policy
```

---

# 🔹 Step 7 — Check Pod Labels

Always verify labels:

```bash
kubectl get pods --show-labels
```

You should be able to clearly identify:

```text
frontend → app=frontend
backend  → app=backend
```

---

# 🔹 Step 8 — Test Again

Test connectivity from an allowed source:

```text
frontend → backend
```

Then test from another Pod that does not match:

```text
other-pod → backend
```

The expected behavior depends on the other policies present in the namespace and cluster.

This is important:

> **NetworkPolicy behavior is the result of all applicable policies, not just one policy viewed in isolation.**

---

# 🔹 Useful kubectl Commands

### List NetworkPolicies

```bash
kubectl get networkpolicy
```

Short form:

```bash
kubectl get netpol
```

---

### Describe a NetworkPolicy

```bash
kubectl describe networkpolicy <policy-name>
```

---

### Get NetworkPolicy YAML

```bash
kubectl get networkpolicy <policy-name> -o yaml
```

---

### List Pods with Labels

```bash
kubectl get pods --show-labels
```

---

### Get Pods with IP Addresses

```bash
kubectl get pods -o wide
```

---

### Execute a Shell in a Pod

```bash
kubectl exec -it <pod-name> -- sh
```

---

### Test Connectivity

Depending on the image:

```bash
curl http://<destination>
```

or:

```bash
wget -qO- http://<destination>
```

or:

```bash
nc -zv <destination> <port>
```

Not every container image includes these tools.

---

# 🔍 Troubleshooting NetworkPolicy

When a NetworkPolicy doesn't behave as expected, I want to troubleshoot systematically.

## 1. Check the Policy

```bash
kubectl get networkpolicy
```

Then:

```bash
kubectl describe networkpolicy <name>
```

Check:

- `podSelector`
- `policyTypes`
- `ingress`
- `egress`
- Allowed sources/destinations
- Ports

---

## 2. Check Pod Labels

```bash
kubectl get pods --show-labels
```

Compare the policy selector with the actual labels.

For example:

Policy:

```yaml
matchLabels:
  app: backend
```

Pod:

```text
app=backend
```

These match.

But:

```text
app=web
```

would not match.

---

## 3. Check the Namespace

NetworkPolicies are **namespaced resources**.

Make sure the policy is being created in the correct namespace:

```bash
kubectl get networkpolicy -A
```

Check the current namespace:

```bash
kubectl config view --minify --output 'jsonpath={..namespace}'
```

---

## 4. Check Policy Direction

Ask:

```text
Is the problem incoming traffic?
        ↓
Ingress

Is the problem outgoing traffic?
        ↓
Egress
```

A policy controlling ingress does not automatically define egress behavior.

---

## 5. Check Ports

If the application listens on:

```text
TCP 8080
```

but the policy allows:

```text
TCP 80
```

the expected traffic will not be allowed by that rule.

---

## 6. Check the Network Plugin

NetworkPolicy enforcement depends on the cluster's networking implementation.

The NetworkPolicy object itself does not magically enforce packet filtering.

The cluster's CNI/networking implementation must support and enforce NetworkPolicy.

---

# ⚠️ Common Mistakes

## ❌ Mistake 1 — Thinking NetworkPolicy is a Cluster-Wide Firewall

A NetworkPolicy is not automatically a firewall for the entire cluster.

It applies to Pods selected by the policy and within its namespace scope.

---

## ❌ Mistake 2 — Forgetting Labels

NetworkPolicy relies heavily on selectors.

If:

```yaml
matchLabels:
  app: backend
```

but the Pod has:

```text
app=web
```

the policy won't select that Pod.

---

## ❌ Mistake 3 — Confusing Ingress and Egress

Remember:

```text
Ingress = traffic coming IN
Egress  = traffic going OUT
```

---

## ❌ Mistake 4 — Assuming One Policy Replaces Another

NetworkPolicies are generally **additive**.

If multiple policies select the same Pod, their allowed traffic is combined according to the NetworkPolicy model.

Don't assume that adding a second policy automatically removes traffic allowed by another applicable policy.

---

## ❌ Mistake 5 — Forgetting DNS with Egress Restrictions

If you apply a restrictive egress policy, applications may lose DNS resolution unless DNS traffic is explicitly permitted.

For example:

```text
Application
    |
    | DNS query
    v
CoreDNS
```

If DNS egress is blocked, service-name resolution can fail.

---

## ❌ Mistake 6 — Testing Only the Policy YAML

A valid YAML file doesn't guarantee the desired traffic behavior.

Always verify:

```text
Policy
  ↓
Selected Pods
  ↓
Source/Destination
  ↓
Ports
  ↓
CNI enforcement
  ↓
Actual connectivity test
```

---

# 🎯 CKA Exam Strategy

When solving a NetworkPolicy question, **don't start writing YAML immediately**.

First understand the traffic requirement.

## Step 1 — Identify the Target Pods

Ask:

> Which Pods should this policy apply to?

Find their labels:

```bash
kubectl get pods --show-labels
```

---

## Step 2 — Identify the Source

Ask:

> Who should be allowed to communicate?

For example:

```text
frontend
```

---

## Step 3 — Identify the Direction

Ask:

```text
Incoming?
→ Ingress

Outgoing?
→ Egress
```

---

## Step 4 — Identify the Port

Ask:

> Which port should be allowed?

For example:

```text
TCP 8080
```

---

## Step 5 — Build the Policy

Only after understanding the requirement, create the YAML.

---

## Step 6 — Verify

```bash
kubectl get networkpolicy
kubectl describe networkpolicy <name>
kubectl get pods --show-labels
```

Then test connectivity.

---

# 🧠 CKA Quick Revision

### NetworkPolicy

Controls network traffic to and from selected Pods.

### podSelector

Identifies which Pods the policy applies to.

### Ingress

Controls incoming traffic.

### Egress

Controls outgoing traffic.

### policyTypes

Specifies whether the policy governs:

```text
Ingress
Egress
```

or both.

### from

Defines allowed ingress sources.

### to

Defines allowed egress destinations.

### ports

Defines allowed ports/protocols.

---

# 🔐 Security Mental Model

A useful way to think about NetworkPolicy is:

```text
                    NetworkPolicy
                          |
          +---------------+---------------+
          |                               |
       Ingress                           Egress
       Incoming                          Outgoing
          |                               |
          v                               v
   Who can reach me?              Where can I connect?
```

The security objective is:

```text
Default / broad connectivity
          ↓
Identify required communication
          ↓
Restrict unnecessary traffic
          ↓
Allow only required flows
```

---

# 📊 Example Communication Matrix

For a three-tier application:

| Source | Destination | Port | Allowed |
|---|---|---:|---|
| Frontend | Backend | 8080 | ✅ |
| Backend | Database | 5432 | ✅ |
| Backend | Redis | 6379 | ✅ |
| Frontend | Database | 5432 | ❌ |
| Frontend | Redis | 6379 | ❌ |
| Other Pods | Database | 5432 | ❌ |

This type of communication matrix is useful before writing the NetworkPolicy.

---

# 💡 What I Learned

The biggest takeaway for me today:

> **Kubernetes networking isn't only about connectivity — it's also about controlling connectivity.**

NetworkPolicy provides an important layer for implementing **least-privilege network communication** between workloads.

The mental model I want to remember is:

```text
Pod
 ↓
Labels
 ↓
NetworkPolicy
 ↓
Ingress / Egress Rules
 ↓
Allowed Communication
```

And when troubleshooting:

```text
Policy
  ↓
Pod Selector
  ↓
Labels
  ↓
Source / Destination
  ↓
Port
  ↓
CNI Enforcement
  ↓
Connectivity Test
```

---

# 📌 Key Takeaways

Today I learned:

- What Kubernetes NetworkPolicy is
- Why NetworkPolicy is important
- How `podSelector` works
- Difference between Ingress and Egress
- How labels affect NetworkPolicy
- How to allow traffic from specific Pods
- How to restrict traffic by port
- How namespace selectors can be used
- Default-deny concepts
- Why DNS matters with egress restrictions
- How multiple policies interact
- How to troubleshoot NetworkPolicy
- How to approach NetworkPolicy questions in the CKA

### One-line revision:

> **NetworkPolicy defines who can communicate with selected Pods and in which direction.**

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
├── Day-15-NetworkPolicy/
│   ├── README.md
│   ├── backend-policy.yaml
│   ├── default-deny-ingress.yaml
│   ├── default-deny-egress.yaml
│   └── images/
│       └── day15-networkpolicy.png
└── ...
```

---

# 🔗 Follow the Journey

I'm continuing to learn Kubernetes by practicing each topic and documenting what I learn along the way.

📂 **GitHub — CKA Preparation Repository**

https://github.com/Mukund15kale/100DaysOfCKA

Follow **Mukund Kale** for the next post in the **CKA Preparation Series**.

---

## 🏷️ Tags

`#CKA` `#Kubernetes` `#CKAPreparation` `#DevOps` `#NetworkPolicy` `#KubernetesNetworking` `#CloudComputing` `#CyberSecurity` `#LearningInPublic` `#DevOpsEngineer` `#100DaysOfCKA`
