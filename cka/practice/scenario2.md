# CKA Practice Scenario 2: Implementing a Kubernetes Network Policy

**Objective:** Create a network policy to restrict traffic between pods in a namespace.

## Steps:

1. **Create a New Namespace:**
   - Create a namespace named `network-policy-demo`.

   ```bash
   kubectl create namespace network-policy-demo
   kubectl config set-context --current --namespace=network-policy-demo
   ```
2. **Deploy Two Pods in the Namespace:**
     - **Pod 1:** `frontend` (using `nginx:1.27`)
     - **Pod 2:** `backend` (using `nginx:1.27`)
     - **Service 1:** `backend`
     - pod_definition.yaml
      ```yaml
      ---
      apiVersion: v1
      kind: Pod
      metadata:
      name: frontend
      labels:
         app: frontend
      spec:
      containers:
      - name: nginx-container
         image: nginx:1.27
         ports:
         - containerPort: 80
      ---
      apiVersion: v1
      kind: Pod
      metadata:
      name: backend
      labels:
         app: backend
      spec:
      containers:
      - name: nginx-container
         image: nginx:1.27
         ports:
         - containerPort: 80
      ---
      apiVersion: v1
      kind: Service
      metadata:
      name: backend
      namespace: network-policy-demo
      spec:
      selector:
         app: backend
      ports:
         - protocol: TCP
            port: 80
            targetPort: 80
      ```

   - Deploy two pods and a backend service: 
   ```bash 
   kubectl apply -f pod_definition.yaml 
   ```

3. **Verify Pod Connectivity Before Applying the Network Policy:**
   - Test connectivity between the two pods.
   ```bash
   kubectl exec  frontend -- curl -s backend
   ```
4. **Create a Network Policy to Restrict Traffic:**
   - Define a network policy named `deny-all` that denies all incoming traffic to the `backend` pod by default and allows traffic only from pods labeled `access: allowed`.

   - deny_policy.yaml
      ```yaml
      apiVersion: networking.k8s.io/v1
      kind: NetworkPolicy
      metadata:
      name: deny-all
      namespace: network-policy-demo
      spec:
      podSelector:
         matchLabels:
            app: backend
      ingress:
      - from:
         - podSelector:
            matchLabels:
                  access: allowed
      policyTypes:
      - Ingress
      ```

5. **Apply the Network Policy:**
   ```bash
   kubectl apply -f deny_policy.yaml
   ```

6. **Verify Pod Connectivity again:**
   - Test connectivity between the two pods.
   ```bash
   kubectl exec  frontend -- curl -s backend
   ```

6. **Label the Pods Accordingly:**
   - Label the `backend` pod:
   ```bash
   kubectl label pod backend  app=backend
   ```
   - Label the `frontend` pod:
   ```bash
   kubectl label pod frontend  access=allowed
   ```

7. **Verify the Network Policy Enforcement:**
   - Test connectivity from the `frontend` pod to the `backend` pod.
   ```bash
   kubectl exec  frontend -- curl -s backend
   ```

## Expected Result:
After applying the network policy, the `frontend` pod should not be able to access the `backend` pod, however once labeled as access=allowed it should be able to access while other pods without the correct labels should be denied access.
