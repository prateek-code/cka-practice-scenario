Scenario: Configuring a Kubernetes Ingress
Objective: Configure an Ingress resource to route traffic to two different services based on the URL path. 

# Kubernetes Ingress with Custom HTML for Service1 and Service2

## Prerequisites

- Kubernetes cluster
- `kubectl` configured to interact with your cluster
- NGINX Ingress Controller installed

## Steps

1. **Install NGINX Ingress Controller:**

    Using Helm:
    ```sh
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm install ingress-nginx ingress-nginx/ingress-nginx
    ```

    Using Kubernetes manifests:
    ```sh
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
    ```

2. **Create ConfigMap for Service1:**

    ```sh
    cat <<EOF | kubectl apply -f -
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: service1-html
    data:
      index.html: |
        <html>
        <head><title>Service 1</title></head>
        <body><h1>Welcome to Service 1</h1></body>
        </html>
    EOF
    ```

3. **Create ConfigMap for Service2:**

    ```sh
    cat <<EOF | kubectl apply -f -
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: service2-html
    data:
      index.html: |
        <html>
        <head><title>Service 2</title></head>
        <body><h1>Welcome to Service 2</h1></body>
        </html>
    EOF
    ```

4. **Create and apply the deployment for Service1:**

    ```sh
    cat <<EOF | kubectl apply -f -
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: service1-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: service1
      template:
        metadata:
          labels:
            app: service1
        spec:
          containers:
          - name: service1-container
            image: nginx:1.21
            volumeMounts:
            - name: custom-html
              mountPath: /usr/share/nginx/html
      volumes:
      - name: custom-html
        configMap:
          name: service1-html
    EOF
    ```

5. **Create and apply the deployment for Service2:**

    ```sh
    cat <<EOF | kubectl apply -f -
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: service2-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: service2
      template:
        metadata:
          labels:
            app: service2
        spec:
          containers:
          - name: service2-container
            image: nginx:1.21
            volumeMounts:
            - name: custom-html
              mountPath: /usr/share/nginx/html
      volumes:
      - name: custom-html
        configMap:
          name: service2-html
    EOF
    ```

6. **Create and apply the services for Service1 and Service2:**

    ```sh
    cat <<EOF | kubectl apply -f -
    apiVersion: v1
    kind: Service
    metadata:
      name: service1
    spec:
      selector:
        app: service1
      ports:
      - protocol: TCP
        port: 80
        targetPort: 80
    EOF

    cat <<EOF | kubectl apply -f -
    apiVersion: v1
    kind: Service
    metadata:
      name: service2
    spec:
      selector:
        app: service2
      ports:
      - protocol: TCP
        port: 80
        targetPort: 80
    EOF
    ```

7. **Create and apply the Ingress resource (Routing Rules):**

    ```sh
    cat <<EOF | kubectl apply -f -
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: example-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - http:
          paths:
          - path: /service1
            pathType: Prefix
            backend:
              service:
                name: service1
                port:
                  number: 80
          - path: /service2
            pathType: Prefix
            backend:
              service:
                name: service2
                port:
                  number: 80
    EOF
    ```

8. **Verify the Ingress resource:**

    ```sh
    kubectl get ingress
    ```

9. **Test the Ingress routing:**

    - Forward the port to access the Ingress controller locally:

    ```sh
    kubectl port-forward svc/ingress-nginx-controller 8080:80 -n ingress-nginx
    ```

    - Use `curl` to test the routing from the local Linux system:

    ```sh
    curl http://localhost:8080/service1
    curl http://localhost:8080/service2
    ```

    - You should see the respective custom HTML content for Service 1 and Service 2 in the output of the `curl` commands.