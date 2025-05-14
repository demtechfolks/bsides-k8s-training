# Kubernetes Hardening Student Labs

These labs align with your Kahoot question sets and reinforce hands-on security skills using CIS benchmarks, attack surface minimization, and real-world exploitation awareness.

---

## 🔒 Lab 1: Evaluate API Server Hardening

**Objective:** Verify `kube-apiserver` flags for CIS compliance and test for unauthenticated access.

### Steps:
1. SSH into your control plane node.
2. Inspect the manifest file:
   ```bash
   cat /etc/kubernetes/manifests/kube-apiserver.yaml
   ```
3. Confirm the following flags exist and are set:
   ```bash
   --anonymous-auth=false
   --authorization-mode=RBAC
   --client-ca-file=/etc/kubernetes/pki/ca.crt
   --audit-policy-file=/etc/kubernetes/audit-policy.yaml
   --audit-log-path=/var/log/kubernetes/audit.log
   ```
4. (Exploit) If `--anonymous-auth=true` is set:
   ```bash
   curl -k https://localhost:6443/api
   ```
   Note if the API responds without authentication.

---

## 🧹 Lab 2: Disable Unused Services on a Node

**Objective:** Minimize the OS attack surface.

### Steps:
1. List active services:
   ```bash
   systemctl list-units --type=service
   ```
2. Disable unneeded services:
   ```bash
   sudo systemctl disable --now avahi-daemon.service
   sudo systemctl disable --now bluetooth.service
   ```
3. Configure secure kernel parameters:
   ```bash
   echo "net.ipv4.conf.all.accept_redirects = 0" | sudo tee -a /etc/sysctl.d/99-custom.conf
   echo "net.ipv4.conf.all.rp_filter = 1" | sudo tee -a /etc/sysctl.d/99-custom.conf
   sudo sysctl --system
   ```
4. Verify changes:
   ```bash
   sysctl -a | grep 'redirects\|rp_filter'
   ```
5. (Exploit) Run an external `nmap` scan and document unexpected open ports.

---

## 🐳 Lab 3: Scan for Vulnerabilities Using Trivy

**Objective:** Identify vulnerabilities in container images.

### Steps:
1. Install Trivy:
   ```bash
   sudo apt install -y trivy
   ```
2. Scan an image:
   ```bash
   trivy image nginx:1.19
   ```
3. Document HIGH and CRITICAL findings.
4. (Exploit) If vulnerabilities allow shell access, exploit using known commands (e.g., outdated `wget`, `bash`).

---

## 📜 Lab 4: Deploy and Inspect Audit Logging

**Objective:** Enable and verify Kubernetes audit logs.

### Steps:
1. Create an audit policy YAML:
   ```bash
   sudo nano /etc/kubernetes/audit-policy.yaml
   ```
2. Add flags to the API server manifest (`kube-apiserver.yaml`).
3. Wait for the static pod to reload.
4. Trigger events:
   ```bash
   kubectl get pods
   ```
5. View audit logs:
   ```bash
   sudo tail -f /var/log/kubernetes/audit.log
   ```
6. Identify structure of one audit event.
7. (Exploit) Run:
   ```bash
   kubectl exec -it <pod> -- /bin/sh
   ```
   Then search logs to confirm evidence.

---

## 🛡 Lab 5: Harden the Kubelet

**Objective:** Restrict insecure Kubelet API exposure.

### Steps:
1. Inspect kubelet config file:
   ```bash
   cat /var/lib/kubelet/config.yaml
   ```
2. Set or confirm:
   ```yaml
   readOnlyPort: 0
   authentication:
     anonymous:
       enabled: false
   authorization:
     mode: Webhook
   ```
3. Restart the kubelet:
   ```bash
   sudo systemctl restart kubelet
   ```
4. Verify open ports:
   ```bash
   ss -tuln
   ```
5. (Exploit) If read-only port is open:
   ```bash
   curl http://localhost:10255/pods
   ```
   Document exposed info.

---

## 🌐 Lab 6: Apply NetworkPolicy for Isolation

**Objective:** Apply default deny and allow rules with NetworkPolicy.

### Steps:
1. Deploy two pods in the same namespace.
2. Apply a default deny policy:
   ```yaml
   kind: NetworkPolicy
   spec:
     podSelector: {}
     policyTypes:
       - Ingress
   ```
3. Confirm traffic is blocked.
4. Apply an allow rule:
   ```yaml
   kind: NetworkPolicy
   spec:
     podSelector:
       matchLabels:
         role: backend
     ingress:
       - from:
           - podSelector:
               matchLabels:
                 role: frontend
   ```
5. Confirm frontend can reach backend.
6. (Exploit) Attempt to curl from another pod and verify it is blocked.

---

These labs are compatible with any Kubernetes distribution (Minikube, kind, RKE2, etc.) and help develop both hardening and threat simulation skills.
