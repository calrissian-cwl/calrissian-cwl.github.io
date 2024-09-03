## Calrissian Tutorial: Running a Water Bodies Detection Job on Kubernetes

This tutorial guides you through the steps to set up and run a Calrissian job on Kubernetes for detecting water bodies using Sentinel-2 imagery. 

The workflow is packaged as a CWL (Common Workflow Language) file, and the necessary configurations are stored in ConfigMaps.

### Prerequisites

A running Kubernetes cluster with `kubectl` configured with:

* A Kubernetes namespace named `calrissian`
* A Kubernetes service account `calrissian-sa`
* Roles and role bindings mounted on the service account
* A ReadWriteMany claim and volume

### Step 1: Create the Parameters File ConfigMap

First, create a ConfigMap that contains the input parameters for the workflow, such as the STAC items (Sentinel-2 imagery) and the area of interest (AOI).

1. Create a `params.yaml` file with the following content:

```yaml
stac_items:
- "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2A_10TFK_20210708_0_L2A"
- "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2B_10TFK_20210713_0_L2A"
- "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2A_10TFK_20210718_0_L2A"
- "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2A_10TFK_20220524_0_L2A"
- "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2A_10TFK_20220514_0_L2A"
- "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2A_10TFK_20220504_0_L2A"
aoi: -121.399,39.834,-120.74,40.472
epsg: "EPSG:4326"
```

2. Create a ConfigMap named `params` in the calrissian namespace using the `params.yaml` file:

```bash
kubectl create -n calrissian configmap params --from-file=params.yaml
```

### Step 2: Get the Application Package

Next, download the CWL application package and create another ConfigMap to store it.

Download the CWL workflow file:

```bash
curl -L https://github.com/eoap/mastering-app-package/releases/download/1.0.0/app-water-bodies-cloud-native.1.0.0.cwl > app-package.cwl
```

Create a ConfigMap named app-package in the calrissian namespace using the app-package.cwl file:

```bash
kubectl create -n calrissian configmap app-package --from-file=app-package.cwl
```

### Step 3: Prepare the Kubernetes Job Manifest

The Kubernetes Job manifest defines how the workflow will be executed. 

The manifest includes configurations for the Calrissian container, resource limits, and the volumes needed to store input and output data.

Here is the job manifest:

```yaml
---
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
            - --stdout 
            - /calrissian/results.json
            - --stderr 
            - /calrissian/app.log
            - --max-ram 
            - 4G
            - --max-cores 
            - "8"
            - --tmp-outdir-prefix 
            - /calrissian/tmp/ 
            - --outdir
            - /calrissian/results
            - --usage-report 
            - /calrissian/usage.json
            - --tool-logs-basepath 
            - /calrissian/logs
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

### Step 4: Run the Kubernetes Job

Apply the job manifest to start the Calrissian job in Kubernetes:

```bash
kubectl apply -n calrissian -f k8s-job.yaml
```

This command submits the job to the Kubernetes cluster, where Calrissian will execute the CWL workflow defined in the app-package.cwl file using the parameters from params.yaml.

### Step 5: Monitor the Job

You can monitor the progress of the job by watching the Pods created by the job:

```bash
kubectl get pods -n calrissian -l job-name=water-bodies-detection -w
```

This command watches the Pods in real-time, displaying their status as they move through the lifecycle stages of Pending, Running, and Completed.

### Conclusion

By following these steps, you have successfully set up and run a Calrissian job on Kubernetes. The workflow processes Sentinel-2 imagery to detect water bodies within the specified area of interest. The outputs and logs are stored in the configured volumes, and the resource usage report provides detailed insights into the execution.