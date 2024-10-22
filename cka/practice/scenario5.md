
# Kubernetes Multi-Container Pod Setup

## Objective
The goal of this scenario is to practice setting up a Kubernetes Pod with multiple containers and configuring inter-container communication. Additionally, we will use ConfigMaps and Secrets to manage environment variables for the containers.

## Prerequisites
- Kubernetes cluster running (Minikube, MicroK8s, GKE, etc.)
- kubectl installed and configured
- Basic knowledge of Kubernetes Pods, ConfigMaps, and Secrets

## Step 1: Create a ConfigMap for Environment Variables

- `configmap.yaml`

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: my-config
  data:
    APP_ENV: "production"
    APP_VERSION: "1.0"
  ```

- Create the ConfigMap:
  ```bash
  kubectl apply -f configmap.yaml
  ```

## Step 2: Create a Secret for Sensitive Information

- `secret.yaml`

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: my-secret
  type: Opaque
  data:
    DB_USER: dXNlcm5hbWU=
    DB_PASS: cGFzc3dvcmQ=
  ````` 

Here, the values are base64 encoded (`username` and `password`).

- Create the Secret:
  ```bash
  kubectl apply -f secret.yaml
  ```

## Step 3: Create a Kubernetes Pod with Multiple Containers

- `pod-multi-container.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: container-one
      image: nginx
      ports:
        - containerPort: 80
      envFrom:
        - configMapRef:
            name: my-config
        - secretRef:
            name: my-secret
    - name: container-two
      image: busybox
      command: ['sh', '-c', 'while true; do echo "Hello from container two"; sleep 5; done']
      envFrom:
        - configMapRef:
            name: my-config
        - secretRef:
            name: my-secret
```

This YAML defines a Pod named `multi-container-pod` containing two containers:
1. **nginx**: Exposes port 80.
2. **busybox**: Continuously outputs a message every 5 seconds.

### Create the Pod
```bash
kubectl apply -f pod-multi-container.yaml
```

Verify the Pod:
```bash
kubectl get pods
```

## Step 4: Configure Inter-Container Communication

Since both containers are in the same Pod, they share the same network namespace. You can communicate between containers using `localhost`.

- To check communication:
  ```bash
  kubectl exec -it multi-container-pod -c container-two -- sh
  ```

- From within the busybox(container-two), try to access container-one:
  ```bash
  wget -qO- http://localhost:80
  ```

You should be able to download index.html of nginx container.

## Step 5: Access the Environment Variables in Containers

In the `pod-multi-container.yaml`, the environment variables from the ConfigMap and Secret are injected into the containers using `envFrom`.

To verify:
```bash
kubectl exec -it multi-container-pod -- env
```

You should see the environment variables from both the ConfigMap and Secret.

## Step 6: Clean Up Resources

To delete the Pod, ConfigMap, and Secret:
```bash
kubectl delete pod multi-container-pod
kubectl delete configmap my-config
kubectl delete secret my-secret
```

## Conclusion

You've now successfully set up a Kubernetes Pod with multiple containers, configured inter-container communication, and managed environment variables using ConfigMaps and Secrets.
