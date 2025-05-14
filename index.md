# Kubernetes Security Labs

These labs reinforce concepts around Kubernetes **threat detection**, **incident response**, and **audit logging** using tools like **Falco**, **Trivy**, and native Kubernetes features.

---

## 🔍 Lab 1: Enable and Inspect Kubernetes Audit Logs

**Objective:** Configure audit logging on a kubeadm cluster and inspect security-relevant events.

### Steps:
1. SSH into your control plane node.
2. Create a policy file:  
   `/etc/kubernetes/audit-policy.yaml`
3. Edit the API server manifest:  
   `/etc/kubernetes/manifests/kube-apiserver.yaml`  
   Add:
   ```bash
   --audit-policy-file=/etc/kubernetes/audit-policy.yaml
   --audit-log-path=/var/log/kubernetes/audit.log
   ```
4. Wait for kubelet to reload the API server.
5. Trigger events:
   ```bash
   kubectl get pods
   kubectl delete pod <pod-name>
   ```
6. View logs:
   ```bash
   jq . < /var/log/kubernetes/audit.log | less
   ```
7. Search for `verb`, `user.username`, `objectRef`.

---

## 🛡 Lab 2: Install and Trigger Falco Alerts

**Objective:** Deploy Falco and simulate suspicious behavior.

### Steps:
1. Install Falco via Helm:
   ```bash
   helm repo add falcosecurity https://falcosecurity.github.io/charts
   helm install falco falcosecurity/falco
   ```
2. View logs:
   ```bash
   kubectl logs -n falco -l app=falco
   ```
3. Simulate an alert:
   ```bash
   kubectl exec -it <pod-name> -- touch /etc/passwd
   ```
4. Check Falco logs for alerts.
5. Optional: Review `/etc/falco/falco_rules.yaml`.

---

## 🔁 Lab 3: Roll Back a Deployment After a Simulated Breach

**Objective:** Simulate a risky image deployment and recover using rollback.

### Steps:
1. Create a deployment:
   ```bash
   kubectl create deployment web --image=nginx:1.19
   ```
2. Simulate an update:
   ```bash
   kubectl set image deployment web nginx=nginx:1.12
   ```
3. View history:
   ```bash
   kubectl rollout history deployment web
   ```
4. Roll back:
   ```bash
   kubectl rollout undo deployment web
   ```
5. Confirm:
   ```bash
   kubectl get pods -o wide
   ```

---

## 🔒 Lab 4: Apply NetworkPolicy for Pod Isolation

**Objective:** Use NetworkPolicy to isolate a compromised pod.

### Steps:
1. Deploy two pods labeled `frontend` and `backend`.
2. Confirm connectivity:
   ```bash
   kubectl exec -it frontend -- curl backend:80
   ```
3. Apply deny-all NetworkPolicy to `backend`.
4. Confirm that traffic is blocked.
5. Modify the policy to allow `frontend` only.

---

## 🧪 Lab 5: Scan Kubernetes with Trivy Operator

**Objective:** Scan workloads for known vulnerabilities.

### Steps:
1. Install Trivy Operator:
   ```bash
   helm install trivy-operator aquasecurity/trivy-operator -n trivy-system --create-namespace
   ```
2. Deploy a sample app:
   ```bash
   kubectl create deployment web --image=nginx
   ```
3. Wait for reports:
   ```bash
   kubectl get vulnerabilityreports -A
   ```
4. Inspect a report:
   ```bash
   kubectl describe vulnerabilityreport <name>
   ```

---

## ✅ Recommendations

- Run these labs in a local or test cluster (kubeadm or Minikube).
- Use `kubectl explain` to explore object schemas.
- Combine with a SIEM or Slack integration to forward alerts from Falco or Trivy.
