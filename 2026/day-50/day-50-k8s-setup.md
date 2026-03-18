Task 1: Recall the Kubernetes Story

Before touching a terminal, write down from memory:

Do not look anything up yet. Write what you remember from the session, then verify against the official docs.
```
Why was Kubernetes created? What problem does it solve that Docker alone cannot?

-> Kubernetes was created to auto scale, auto heal and to manage multiple containers in a cluster. Solving problems like deployment, scaling and networking.

Who created Kubernetes and what was it inspired by?

-> Google developed the Kubernetes, it was inspired by borg internal cluster managment system. It was previously used to manage Google search, youtube and gmail.

What does the name "Kubernetes" mean?

-> K8S means "pilot" the person who steers a ship.
-> Just like a helmsman/pilot controls a ship, Kubernetes controls containers in a cluster.
```

Task 2: Draw the Kubernetes Architecture

From memory, draw or describe the Kubernetes architecture. Your diagram should include:

Control Plane (Master Node):

API Server — the front door to the cluster, every command goes through it
etcd — the database that stores all cluster state
Scheduler — decides which node a new pod should run on
Controller Manager — watches the cluster and makes sure the desired state matches reality
Worker Node:

kubelet — the agent on each node that talks to the API server and manages pods
kube-proxy — handles networking rules so pods can communicate
Container Runtime — the engine that actually runs containers (containerd, CRI-O)
After drawing, verify your understanding:

<img width="817" height="646" alt="image" src="https://github.com/user-attachments/assets/95b2edea-49b2-4684-8d5c-203ab905c1f2" />

```
What happens when you run kubectl apply -f pod.yaml? Trace the request through each component.

Send → kubectl sends YAML to API server
Save → API server stores it in etcd
Decide → Scheduler picks a node
Run → Kubelet starts the container

APPLY FLOW:
kubectl → API → etcd → scheduler → kubelet → container

What happens if the API server goes down?

kubectl commands (no entry point)
Creating/updating/deleting resources cant happen
Controllers can’t find state
Scheduler can’t assign new Pods

API SERVER DOWN:
No control, but running pods stay alive

What happens if a worker node goes down?

Node becomes Not Ready
Pods on it are lost

NODE DOWN:
Pods die → recreated (if managed)
```

Task 3: Install kubectl
kubectl is the CLI tool you will use to talk to your Kubernetes cluster.

Install it:

# macOS
brew install kubectl

# Linux (amd64)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Windows (with chocolatey)
choco install kubernetes-cli
Verify:

```
kubectl version --client

Client Version: v1.34.5
Kustomize Version: v5.7.1
Server Version: v1.35.0
```

Task 4: Set Up Your Local Cluster
Choose one of the following. Both give you a fully functional Kubernetes cluster on your machine.

Option A: kind (Kubernetes in Docker)

# Install kind
# macOS
brew install kind

# Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create a cluster
kind create cluster --name devops-cluster

# Verify
kubectl cluster-info
kubectl get nodes

<img width="1047" height="289" alt="image" src="https://github.com/user-attachments/assets/c82b1882-ea37-4572-8742-a1a283e8c935" />


Option B: minikube

# Install minikube
# macOS
brew install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start a cluster
minikube start

# Verify
kubectl cluster-info
kubectl get nodes

Write down: Which one did you choose and why?

```
KIND

Runs Kubernetes inside Docker containers
Each node = a Docker container
Very lightweight and fast

MINIKUBE

running a mini computer inside your laptop and its heavier.

```

Task 5: Explore Your Cluster
Now that your cluster is running, explore it:

# See cluster info
kubectl cluster-info

# List all nodes
kubectl get nodes

# Get detailed info about your node
kubectl describe node <node-name>

# List all namespaces
kubectl get namespaces

# See ALL pods running in the cluster (across all namespaces)
kubectl get pods -A
Look at the pods running in the kube-system namespace:

kubectl get pods -n kube-system
You should see pods like etcd, kube-apiserver, kube-scheduler, kube-controller-manager, coredns, and kube-proxy. These are the architecture components you drew in Task 2 — running as pods inside the cluster.

Verify: Can you match each running pod in kube-system to a component in your architecture diagram?

Task 6: Practice Cluster Lifecycle
Build muscle memory with cluster operations:

# Delete your cluster
kind delete cluster --name devops-cluster


# (or: minikube delete)

# Recreate it
kind create cluster --name devops-cluster
# (or: minikube start)

<img width="1199" height="448" alt="image" src="https://github.com/user-attachments/assets/2174ccf8-8be5-4472-866c-5ab3b28794c3" />

# Verify it is back
kubectl get nodes
Try these useful commands:

# Check which cluster kubectl is connected to
kubectl config current-context

# List all available contexts (clusters)
kubectl config get-contexts

<img width="1199" height="123" alt="image" src="https://github.com/user-attachments/assets/cd31605e-5c79-433e-85ae-a846a6fd1293" />


# See the full kubeconfig
kubectl config view
```
umesh@umesh-Aspire-A515-57G:~$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:39959
  name: kind-umesh-cluster
contexts:
- context:
    cluster: kind-umesh-cluster
    user: kind-umesh-cluster
  name: kind-umesh-cluster
current-context: kind-umesh-cluster
kind: Config
users:
- name: kind-umesh-cluster
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```

Write down: What is a kubeconfig? Where is it stored on your machine?
```
A kubeconfig is a configuration file used by Kubernetes to connect to and interact with a cluster.

It contains:

Cluster details (API server endpoint)
User credentials (authentication info)
Contexts (which cluster + user + namespace to use)

path: ~/.kube/config
```
