## Kubernetes cluster configuration

## Service account

It's a best practice to create a dedicated Service Account with a manifest like:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: calrissian-sa
```

## Roles and Role Bindings

`calrissian` executes CWL workflows by running steps as Pods in a kubernetes cluster. 

Create the Roles to manage pods and pod logs:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-manager-role
rules:
- apiGroups: [""] 
  resources: ["pods"]
  verbs: ["create","patch","delete","list","watch","get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: log-reader-role
rules:
- apiGroups: [""] 
  resources: ["pods/log"]
  verbs: ["get","list"]
```

Bind these roles to the `calrissian-sa` Service Account with the manifest: 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-manager-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-manager-role
subjects:
- kind: ServiceAccount
  name: calrissian-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: log-reader-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: log-reader-role
subjects:
- kind: ServiceAccount
  name: calrissian-sa
```

## Persistent Volume Claim

Calrissian needs a ReadWriteMany volume than can be defined with:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: calrissian-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10G
  storageClassName: default
```

Note: Update the `storageClassName` according to the Kubernetes cluster.