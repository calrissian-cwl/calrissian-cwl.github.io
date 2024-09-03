## How to set node selectors

The `--pod-nodeselectors` option allows you to specify a YAML file that defines node selectors for the Kubernetes Pods created by Calrissian during the execution of a workflow. 

Node selectors are key-value pairs that constrain the scheduling of Pods to specific nodes in the Kubernetes cluster that match the specified criteria.

The YAML file provided to the `--pod-nodeselectors` option should contain a list of key-value pairs that represent the node selector criteria.

When Calrissian creates Pods to run CWL tools, it applies these node selectors to ensure that the Pods are scheduled only on nodes that match the specified criteria.

This option is useful when you need to run certain workloads on specific types of nodes, such as nodes with specific hardware, geographic location, or other attributes.

### Usage Example:

```yaml
k8s.earth.com/pool-name: processing-node-pool-gpu
```

If this file is available in the `/calrissian` mount point, then in your Calrissian Job manifest do:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: water-bodies-detection
spec:
  ttlSecondsAfterFinished: 60
  template:
    spec:
      serviceAccountName: calrissian-sa
      securityContext:
        runAsUser: 1000
        runAsGroup: 1000
      containers:
        - name: calrissian
          image: ghcr.io/duke-gcb/calrissian/calrissian:0.16.0
          command: ["calrissian"]
          args:
            - --pod-nodeselectors
            - /calrissian/node-selector.yaml
            ...
            - /app-package/app-package.cwl
            - /inputs/params.yaml
          volumeMounts:
            - name: calrissian-volume
              mountPath: /calrissian
            - name: params-volume
              mountPath: /inputs
            - name: app-package-volume
              mountPath: /app-package
          env:
            - name: CALRISSIAN_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
      restartPolicy: Never
      volumes:
        - name: calrissian-volume
          persistentVolumeClaim:
            claimName: calrissian-claim 
        - name: params-volume
          configMap:
            name: params  
        - name: app-package-volume
          configMap:
            name: app-package
  backoffLimit: 3
```

If you want to mount it via a Config Map, create a ConfigMap named `node-selector` in the calrissian namespace using the `node-selector.yaml` file:

```bash
kubectl create -n calrissian configmap node-selector --from-file=node-selector.yaml
```

and then do:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: water-bodies-detection
spec:
  ttlSecondsAfterFinished: 60
  template:
    spec:
      serviceAccountName: calrissian-sa
      securityContext:
        runAsUser: 1000
        runAsGroup: 1000
      containers:
        - name: calrissian
          image: ghcr.io/duke-gcb/calrissian/calrissian:0.16.0
          command: ["calrissian"]
          args:
            - --pod-nodeselectors
            - /selector/node-selector.yaml
            ...
            - /app-package/app-package.cwl
            - /inputs/params.yaml
          volumeMounts:
            - name: calrissian-volume
              mountPath: /calrissian
            - name: params-volume
              mountPath: /inputs
            - name: app-package-volume
              mountPath: /app-package
            - name: node-selector-volume
              mountPath: /selector
          env:
            - name: CALRISSIAN_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
      restartPolicy: Never
      volumes:
        - name: calrissian-volume
          persistentVolumeClaim:
            claimName: calrissian-claim 
        - name: params-volume
          configMap:
            name: params  
        - name: app-package-volume
          configMap:
            name: app-package
        - name: node-selector-volume
          configMap:
            name: node-selector
  backoffLimit: 3
```