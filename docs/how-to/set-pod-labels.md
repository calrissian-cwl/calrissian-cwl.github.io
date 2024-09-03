## How to set pod labels

The `--pod-labels` option allows you to specify a YAML file that contains key-value pairs for labels to be applied to the Kubernetes Pods created by Calrissian during the execution of a workflow. 

These labels can be used for organizing, categorizing, and managing the Pods within the Kubernetes cluster.

The YAML file provided to the `--pod-labels` option should contain a list of key-value pairs that represent the labels you want to apply to the Pods.

When Calrissian creates Pods to run CWL tools, it automatically applies the specified labels to these Pods.

Labels can be used for various purposes, such as grouping related Pods, filtering Pods during monitoring, or applying policies within the cluster.

### Usage Example

Create a `pod-labels.yaml` file with the following content:

```yaml
app: water-bodies-detection
environment: production
workflow: sentinel-2-processing
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
            - --pod-labels
            - /calrissian/pod-labels.yaml
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

If you want to mount it via a Config Map, create a ConfigMap named `pod-labels` in the calrissian namespace using the `pod-labels.yaml` file:

```bash
kubectl create -n calrissian configmap pod-labels --from-file=pod-labels.yaml
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
            - --pod-labels
            - /labels/pod-labels.yaml
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
            - name: pod-labels-volume
              mountPath: /labels
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
        - name: pod-labels-volume
          configMap:
            name: pod-labels
  backoffLimit: 3
```