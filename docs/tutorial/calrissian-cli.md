## Calrissian Tutorial: Running a Water Bodies Detection Job on Kubernetes

This tutorial guides you through the steps to running Calrissian as a Cli in a kubernete pod.

This requires creating a pod with the `calrissian` executable installed and opening a shell in that pod.

This tutorial targets a minikube cluster.

### Configure minikube

Configure minikube to run calrissian:

```
curl -L https://calrissian-cwl.github.io/tutorial/assets/pvc.yaml | kubectl apply -n calrissian -f -
curl -L https://calrissian-cwl.github.io/tutorial/assets/service-account.yaml | kubectl apply -n calrissian -f -
curl -L https://calrissian-cwl.github.io/tutorial/assets/roles.yaml | kubectl apply -n calrissian -f -
curl -L https://calrissian-cwl.github.io/tutorial/assets/role-binding.yaml | kubectl apply -n calrissian -f -
```

### Step 1: Create a Container with calrissian

Below an example of a Dockerfile that suits the tutorial needs:

```
FROM docker.io/python:3.10-slim-bullseye

# Install necessary packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    curl \
    wget \
    vim \
    sudo && \
    rm -rf /var/lib/apt/lists/*

# Install kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl

# Create a user named 'calrissian' with sudo privileges
RUN useradd -m calrissian && \
    echo "calrissian:password" | chpasswd && \
    adduser calrissian sudo && \
    echo "calrissian ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Add requirements.txt and install Python dependencies
ADD requirements.txt /tmp/requirements.txt

RUN pip install --no-cache-dir calrissian

# Set the user to 'calrissian'
USER calrissian

# Add alias for ll="ls -l" to the bash profile of calrissian
RUN echo 'alias ll="ls -l"' >> /home/calrissian/.bashrc

WORKDIR /home/calrissian
```

Build the image with:

```
docker build -t calrissian-runtime -f Dockerfile .
```

Push the image to minikube:

```
minikube image load docker.io/library/calrissian-runtime
```

## Step 2: Create a Deployment with a pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: calrissian-session
  labels:
    app: calrissian
spec:
  replicas: 1
  selector:
    matchLabels:
      app: calrissian
  template:
    metadata:
      labels:
        app: calrissian
    spec:
      serviceAccountName: calrissian-sa
      containers:
        - name: calrissian-session
          image: docker.io/library/calrissian-runtime 
          command: ["sleep", "infinity"]
          volumeMounts:
            - mountPath: /calrissian
              name: calrissian-volume
          resources:
            limits:
              cpu: '2'
              memory: 1Gi
            requests:
              cpu: '1'
              memory: 512Mi
          env:
            - name: CALRISSIAN_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
      volumes:
        - name: calrissian-volume
          persistentVolumeClaim:
            claimName: calrissian-claim
```

Deploy it with:

```
curl -L https://calrissian-cwl.github.io/tutorial/assets/deployment.yaml | kubectl apply -n calrissian -f -
```