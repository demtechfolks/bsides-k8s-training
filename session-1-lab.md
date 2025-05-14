# Session 1 Labs - Kubernetes Architecture Walkthrough

These labs guide students through foundational components of Kubernetes — the control plane and worker node architecture — with hands-on exploration and reflection.

---

## 🧠 Lab 1: Exploring the Kubernetes Control Plane

**Objective:** Understand the role and location of the API server, controller manager, scheduler, and etcd.

### Steps:

1. **Identify Control Plane Nodes**
   ```bash
   kubectl get nodes -o wide
   ```

2. **View System Pods in kube-system**
   ```bash
   kubectl get pods -n kube-system -o wide
   ```

3. **Inspect a Component (e.g., API Server)**
   ```bash
   kubectl describe pod kube-apiserver-<node-name> -n kube-system
   ```

4. **View Logs from a Control Plane Component**
   ```bash
   kubectl logs kube-scheduler-<node-name> -n kube-system
   ```

5. **Use kubectl to Talk to the API Server**
   ```bash
   kubectl get componentstatuses
   ```

### Reflection:
- What role does each control plane component play?
- How do they work together to manage the cluster?

---

## 🚀 Lab 2: Tracing Pod Scheduling and Worker Node Components

**Objective:** Follow a pod from definition to execution, observing the roles of the scheduler, kubelet, and kube-proxy.

### Steps:

1. **Deploy a Simple App**
   ```bash
   kubectl create deployment nginx --image=nginx
   ```

2. **Watch the Pod Scheduling Process**
   ```bash
   kubectl get pods -w
   ```

3. **Find the Assigned Node**
   ```bash
   kubectl get pod <nginx-pod-name> -o wide
   ```

4. **Describe the Pod**
   ```bash
   kubectl describe pod <nginx-pod-name>
   ```

5. **Observe kubelet's Role (via logs or systemd)**
   ```bash
   journalctl -u kubelet   # or examine container logs if running inside a container
   ```

6. **Explore kube-proxy Rules**
   ```bash
   kubectl get svc
   kubectl describe svc nginx
   ```

### Bonus:
- Run:
   ```bash
   kubectl explain pod
   ```
- Explore the Kubernetes API schema via:
   ```bash
   kubectl explain pod.spec.containers
   ```

### Reflection:
- What role did each component play in deploying your app?
- Why is each necessary?

---
