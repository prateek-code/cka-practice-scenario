
## Scenario 4: Configuring Kubernetes Resource Quotas and Limits

### Objective
In this scenario, you will learn how to manage Kubernetes resources by setting up resource quotas and limits within a namespace. The goal is to practice controlling resource consumption to ensure efficient and fair usage of cluster resources.

### Prerequisites
- A Kubernetes cluster running version 1.27
- A namespace named `kubectl create namespace cka-practice`
- A context to the namespace `kubectl config set-context --current --namespace=cka-practice`

### Steps

1. **Create a Resource Quota**
   - Define a resource quota named `cpu-memory-quota` with the following specifications:
     - Limits the total CPU to `2` cores and memory to `4Gi`.
     - quota_definition.yaml
        ```yaml
        apiVersion: v1
        kind: ResourceQuota
        metadata:
          name: cpu-memory-quota
          namespace: cka-practice
        spec:
          hard:
            requests.cpu: "2"
            requests.memory: 4Gi
            limits.cpu: "2"
            limits.memory: 4Gi
        ```

   - Apply the resource quota configuration:
      ```bash
      kubectl apply -f quota_definition.yaml
      ```

2. **Create a LimitRange for Pod Limits**
   - Define a LimitRange named `container-limits` that sets default CPU and memory limits and requests for containers in the namespace.
   - limitrange_definition.yaml
      ```yaml
      apiVersion: v1
      kind: LimitRange
      metadata:
        name: container-limits
        namespace: cka-practice
      spec:
        limits:
        - default:
            cpu: "500m"
            memory: "512Mi"
          defaultRequest:
            cpu: "200m"
            memory: "256Mi"
          type: Container
      ```

   - Apply the LimitRange configuration:
      ```bash
      kubectl apply -f limitrange_definition.yaml
      ```

3. **Deploy a Pod to Verify Quotas and Limits**
   - Create a pod named `busybox-resource` with the `busybox` image. The pod should run a simple command that sleeps for a long time.
   - pod_definition.yaml
      ```yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: busybox-resource
        namespace: cka-practice
      spec:
        containers:
        - name: busybox
          image: busybox
          command: ["sleep"]
          args: ["3600"]
      ```

   - Apply the pod configuration:
      ```bash
      kubectl apply -f pod_definition.yaml
      ```

4. **Check the Resource Usage**
   - Verify that the pod was deployed successfully and the requested resources were within the specified quotas.
      ```bash
      kubectl describe pod busybox-resource
      ```

5. **Test Exceeding the Quota**
   - Try deploying a pod that exceeds the resource quota or limits, and observe how Kubernetes prevents it.
   - test_pod_definition.yaml
      ```yaml
      apiVersion: v1
      kind: Pod
      metadata: 
        name: test-pod
        namespace: cka-practice
      spec:
        containers:
        - name: test-container
          image: busybox
          command: ["sleep"]
          args: ["3600"]
          resources:
            requests:
              cpu: "3"
              memory: "3Gi"
            limits:
              cpu: "4"
              memory: "4Gi"
      ```

   - Apply the pod configuration:
      ```bash
      kubectl apply -f test_pod_definition.yaml
      ```
6. **Cleanup**
   - Delete the pod `kubectl delete pod busybox-resource`
   - Delete the `kubectl delete quota cpu-memory-quota` resource quota and `kubectl delete limitrange container-limits` limit range.
   - Delete yaml files `rm pod_definition.yaml quota_definition.yaml limitrange_definition.yaml test_pod_definition.yaml`

### Deliverables
- Confirm that resource quotas and limits are applied correctly and that resource usage is enforced as expected.
