
# Scenario: Configure Basic Authentication for NGINX Using .htpasswd

## Objective
Set up basic authentication on an NGINX server using `.htpasswd` and test it using a test-curl pod to confirm access is restricted.

## Steps

### 1. Create a Namespace for the Scenario
```bash
kubectl create namespace nginx-auth-practice
```

### 2. Set Up the `.htpasswd` File
- Install the `htpasswd` utility: `sudo apt-get install apache2-utils`
- Use the following command to create a `.htpasswd` file with a username and password (e.g., user: `admin`, password: `password123`).
  ```bash
  htpasswd -c -B -b .htpasswd admin password123
  ```
- Create a Kubernetes secret from the `.htpasswd` file:
  ```bash
  kubectl create secret generic basic-auth-secret --from-file=.htpasswd -n nginx-auth-practice
  ```

### 3. Create the NGINX ConfigMap
- This ConfigMap enables the basic auth in NGINX by editing `nginx.conf`:
  ```bash
  kubectl create configmap nginx-auth-config --from-literal=nginx.conf="
  server {
      listen 80;
      server_name localhost;
      location / {
          auth_basic 'Restricted';
          auth_basic_user_file /etc/nginx/.htpasswd;
          root /usr/share/nginx/html;
          index index.html;
      }
  }" -n nginx-auth-practice --dry-run=client -o yaml | kubectl apply -f -
  ```

### 4. Deploy the NGINX Server with Basic Authentication
- Create a YAML file named `nginx-auth.yaml` with the following content:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-auth
    namespace: nginx-auth-practice
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: nginx-auth
    template:
      metadata:
        labels:
          app: nginx-auth
      spec:
        containers:
        - name: nginx
          image: nginx
          volumeMounts:
          - name: auth-volume
            mountPath: "/etc/nginx/conf.d"
          - name: auth-secret
            mountPath: "/etc/nginx/.htpasswd"
            subPath: ".htpasswd"
          ports:
          - containerPort: 80
        volumes:
        - name: auth-volume
          configMap:
            name: nginx-auth-config
        - name: auth-secret
          secret:
            secretName: basic-auth-secret
  ```

### 5. Expose the NGINX Deployment
- Create a service to expose the NGINX server:
  ```bash
  kubectl expose deployment nginx-auth --port=80 --target-port=80 -n nginx-auth-practice
  ```

### 6. Deploy the test-curl Pod for Testing
- Run a test-curl pod to test access to NGINX:
  ```bash
  kubectl run test-curl --rm -i -t --image=alpine -n nginx-auth-practice -- /bin/sh
  ```
- Add curl to the test-curl pod `apk add curl`
- Inside the test-curl pod, use curl to test authentication:
  ```bash
  curl -u admin:password123 http://nginx-auth.nginx-auth-practice.svc.cluster.local
  ```

### 7. Cleanup
- To remove all resources created in this scenario:
  ```bash
  kubectl delete namespace nginx-auth-practice
  ```

## Explanation
- `.htpasswd`: This file contains the encrypted username and password for basic authentication.
- **Nginx ConfigMap**: Configures the NGINX server to use basic authentication, only allowing access with valid credentials.
- **test-curl Testing**: A lightweight container to test the access restrictions using HTTP requests.

By following these steps, you can verify that only users with the correct credentials can access the NGINX server.
