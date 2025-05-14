# Kubernetes Networking, NetworkPolicy, and DNS Labs

## Lab 1: Understanding Pod-to-Pod Communication

Objective: Confirm that pods can communicate across nodes by default.

Steps:
1. Create two pods in the same namespace:
   kubectl run backend --image=nginx --port=80
   kubectl run frontend --image=busybox -it --restart=Never -- sh

2. From the 'frontend' pod, run:
   wget --spider backend

3. Exit the shell.

Expected Result: The frontend pod should reach the backend pod successfully.

## Lab 2: Create and Test a ClusterIP Service

Objective: Understand how internal services use DNS and IP for routing.

Steps:
1. Deploy a basic app:
   kubectl create deployment myapp --image=httpd

2. Expose it:
   kubectl expose deployment myapp --port=80 --target-port=80 --name=myapp-service

3. Verify access from a temporary pod:
   kubectl run tester --image=busybox -it --restart=Never -- sh
   wget --spider myapp-service

Expected Result: DNS resolves the service name and connects to the app.

## Lab 3: Deploy and Test a NetworkPolicy (Ingress Restriction)

Objective: Restrict which pods can access the backend app.

Steps:
1. Label the backend pod:
   kubectl label pod backend role=backend

2. Apply this NetworkPolicy (save as deny-all.yaml):
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: deny-all
   spec:
     podSelector:
       matchLabels:
         role: backend
     policyTypes:
     - Ingress
     ingress: []

3. Apply the policy:
   kubectl apply -f deny-all.yaml

4. Try wget backend from the frontend pod.

Expected Result: The request is blocked.

## Lab 4: Allow Ingress from a Specific Pod

Objective: Allow frontend to access backend through NetworkPolicy.

Steps:
1. Label the frontend pod:
   kubectl label pod frontend role=frontend

2. Create and apply this policy:
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: allow-frontend
   spec:
     podSelector:
       matchLabels:
         role: backend
     ingress:
     - from:
       - podSelector:
           matchLabels:
             role: frontend

3. Retry wget backend from frontend.

Expected Result: Request should succeed now.

## Lab 5: DNS Resolution and Debugging

Objective: Understand how CoreDNS resolves services.

Steps:
1. Create a pod with a DNS tool:
   kubectl run dnsutils --image=busybox:1.28 -it --restart=Never -- sh

2. Run:
   nslookup myapp-service

3. Test what happens with a nonexistent service:
   nslookup invalid-service

Expected Result: DNS should resolve valid services and fail gracefully on invalid ones.

## Lab 6: Egress Control via NetworkPolicy

Objective: Restrict external internet access from a pod.

Steps:
1. Create a pod:
   kubectl run test-egress --image=busybox -it --restart=Never -- sh

2. Verify internet access:
   wget google.com

3. Apply this egress deny-all policy:
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: deny-egress
   spec:
     podSelector:
       matchLabels: {}
     policyTypes:
     - Egress
     egress: []

4. Try wget google.com again.

Expected Result: Internet access is now blocked.

