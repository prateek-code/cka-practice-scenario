# CKA Practice Scenario 1: Multi-Container Pod with Shared Volume

**Objective:** Create a multi-container pod where one container writes logs to a shared volume, and the other container reads those logs.

## Steps:

1. **Create a Namespace:**
   - Create a namespace named `cka-practice`.
  
      ```bash
      kubectl create namespace cka-practice
      kubectl config set-context --current --namespace=cka-practice
      ```
2. **Define the Multi-Container Pod:**
   - Use a YAML file to define a pod with two containers:
     - **Container 1:** `busybox-container` that writes the current date and time to a log file every 5 seconds.
     - **Container 2:** `nginx-container` that serves the log file through HTTP.
     - Use an `emptyDir` volume to share the log file between the two containers.
     - multi_container_pod.yaml:
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: multi-container-pod
        spec:
          containers:
          - name: nginx-container
            image: nginx:1.27
            ports:
            - containerPort: 80
            volumeMounts:
            - mountPath: /var/log/pod-shared
              name: shared-logs
          - name: busybox-container
            image: busybox
            command: ["/bin/sh"]
            args: ["-c", "while true; do echo $(date) >> /var/log/pod-shared/log.txt; sleep 5; done"]
            volumeMounts:
            - mountPath: /var/log/pod-shared
              name: shared-logs
          volumes:
          - name: shared-logs
            emptyDir: {}
        ```   

4. **Apply the Pod Configuration:**
   - Deploy the pod.
      ```bash
      kubectl apply -f multi_container_pod.yaml
      ```

5. **Verify Log Writing:**
   - Use `kubectl exec` to check the contents of the log file from the `nginx-container`.
      ```bash
      kubectl exec -n cka-practice nginx-container -- cat /var/log/pod-shared/log.txt
      ```
6. **Cleanup:**
   - Remove the pod `kubectl delete pod multi-container-pod`.
   - Remove files `rm multi_container_pod.yaml`
## Expected Result:
You should see log entries showing the current date and time, confirming that the `busybox-container` is writing to the log file and you should be able to read it from nginx-container.
