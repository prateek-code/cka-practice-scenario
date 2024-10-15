## Scenario 3: Configuring Persistent Storage with PersistentVolume and PersistentVolumeClaim

### Objective
The goal of this scenario is to practice setting up persistent storage using a PersistentVolume (PV) and a PersistentVolumeClaim (PVC) in Kubernetes. You'll configure a pod to use this persistent storage and verify that data remains persistent across pod restarts.

### Prerequisites
- A Kubernetes cluster running version 1.27
- A namespace named `kubectl create namespace cka-practice`
- A context to the namespace `kubectl config set-context --current --namespace=cka-practice`
- Label one of the node which will be used to schedule the pod `kubectl label nodes <node-name> storage-node=true`

### Steps

1. **Create a PersistentVolume (PV)**
   - Define a PV named `cka-pv` with the following specifications:
     - Capacity: 1Gi
     - Access Modes: ReadWriteOnce
     - Storage Class: `manual`
     - pv_definition.yaml
        ```yaml
        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: cka-pv
          namespace: cka-practice
        spec:
          capacity:
            storage: 1Gi
          accessModes:
            - ReadWriteOnce
          storageClassName: manual
          hostPath:
            path: /mnt/test
        ```

   - Apply the PV configuration:
   ```bash
   kubectl apply -f pv_definition.yaml
   ```

2. **Create a PersistentVolumeClaim (PVC)**
   - Define a PVC named `cka-pvc` with the following specifications:
     - Request Storage: 500Mi
     - Access Modes: ReadWriteOnce
     - Storage Class: `manual`
     - pvc_definition.yaml

        ```yaml
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: cka-pvc
          namespace: cka-practice
        spec:
          accessModes:
            - ReadWriteOnce
          storageClassName: manual
          resources:
            requests:
              storage: 500Mi
        ```

   - Apply the PVC configuration:
   ```bash
   kubectl apply -f pvc_definition.yaml
   ```

3. **Verify the Binding of the PVC to the PV**
   - Check if the PVC is bound to the PV:
   ```bash
   kubectl get pvc
   kubectl get pv
   ```

4. **Deploy a Pod Using the PVC**
   - Create a pod named `busybox-pv` that uses the `busybox` image. The pod should mount the PVC to a path inside the container, such as `/mnt/test`.
   - pod_definition.yaml

      ```yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: busybox-pv
        namespace: cka-practice
      spec:
        containers:
        - name: busybox
          image: busybox
          command: ["/bin/sh"]
          args: ["-c", "while true; do echo $(date) >> /mnt/test/timestamp.log; sleep 5; done"]
          volumeMounts:
          - mountPath: /mnt/test
            name: cka-storage
        volumes:
        - name: cka-storage
          persistentVolumeClaim:
            claimName: cka-pvc
        nodeSelector:
          storage-node: "true"
      ```

   - Apply the pod configuration:
   ```bash
   kubectl apply -f pod_definition.yaml
   ```

5. **Verify Data Persistence**
   - Use `kubectl exec` to check the contents of the `timestamp.log` file to ensure that timestamps are being logged.

   ```bash
   kubectl exec busybox-pv -- cat /mnt/test/timestamp.log
   ```

6. **Test Data Persistence After Pod Deletion**
   - Delete the `busybox-pv` pod and then recreate it using the same PVC. Verify that the data in the `timestamp.log` file is still present.

   ```bash
   kubectl delete pod busybox-pv
   kubectl apply -f pod_definition.yaml
   kubectl exec busybox-pv -- cat /mnt/test/timestamp.log
   ```
7. **Cleanup**
   - Delete the pod `kubectl delete pod busybox-pv`
   - Delete the `kubectl delete pvc cka-pvc` and `kubectl delete pv cka-pv` PVs and PVCs.
   - Delete labelled nodes `kubectl label nodes <node-name> storage-node-`
### Deliverables
- Confirm that the persistent storage configuration works correctly and that data persists even after deleting and recreating the pod.

