Task 1: Explore Default Namespaces

Kubernetes comes with built-in namespaces. List them:

kubectl get namespaces
You should see at least:

default — where your resources go if you do not specify a namespace
kube-system — Kubernetes internal components (API server, scheduler, etc.)
kube-public — publicly readable resources
kube-node-lease — node heartbeat tracking
Check what is running inside kube-system:

kubectl get pods -n kube-system
These are the control plane components keeping your cluster alive. Do not touch them.


Verify: How many pods are running in kube-system?

-> There are 14 running pods in the kube-system.

<img width="995" height="363" alt="image" src="https://github.com/user-attachments/assets/a2f4b021-7b25-45b2-b5bd-dec59ae8fc47" />


Task 2: Create and Use Custom Namespaces

Create two namespaces — one for a development environment and one for staging:

kubectl create namespace dev
kubectl create namespace staging

Verify they exist:

kubectl get namespaces
<img width="990" height="332" alt="image" src="https://github.com/user-attachments/assets/fde07e35-24bd-4ca1-80d0-b931697cdf68" />

You can also create a namespace from a manifest:

namespace.yaml

```
apiVersion: v1
kind: Namespace
metadata:
  name: production
```
kubectl apply -f namespace.yaml

Now run a pod in a specific namespace:

<img width="990" height="332" alt="image" src="https://github.com/user-attachments/assets/b3c569f4-0b7c-460d-86f6-5455b1118c50" />
```
kubectl run nginx-dev --image=nginx:latest -n dev
kubectl run nginx-staging --image=nginx:latest -n staging
```

List pods across all namespaces:

<img width="1155" height="100" alt="image" src="https://github.com/user-attachments/assets/87baa5d4-7410-4b62-b0a3-5566a664df31" />


kubectl get pods -A

<img width="1134" height="591" alt="image" src="https://github.com/user-attachments/assets/b96d2d57-c7fd-440a-94be-0adea0da3f85" />

Notice that kubectl get pods without -n only shows the default namespace. You must specify -n <namespace> or use -A to see everything.

Verify: Does kubectl get pods show these pods? What about kubectl get pods -A?

-> There were no pods in default namespaces and by using -A we can get entire pods running in all namespaces.

Task 3: Create Your First Deployment
A Deployment tells Kubernetes: "I want X replicas of this Pod running at all times." If a Pod crashes, the Deployment controller recreates it automatically.

Create a file nginx-deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
```
Key differences from a standalone Pod:

kind: Deployment instead of kind: Pod
apiVersion: apps/v1 instead of v1
replicas: 3 tells Kubernetes to maintain 3 identical pods
selector.matchLabels connects the Deployment to its Pods
template is the Pod template — the Deployment creates Pods using this blueprint
Apply it:

kubectl apply -f nginx-deployment.yaml
Check the result:READY → Pods that are healthy right now
UP-TO-DATE → Pods using latest version
AVAILABLE → Pods ready to handle users

kubectl get deployments -n dev
kubectl get pods -n dev
You should see 3 pods with names like nginx-deployment-xxxxx-yyyyy.

<img width="868" height="204" alt="image" src="https://github.com/user-attachments/assets/212d60d5-9bf3-4615-86a4-3a64e76967d7" />


Verify: What do the READY, UP-TO-DATE, and AVAILABLE columns mean in the deployment output?

READY → Pods that are healthy right now
UP-TO-DATE → Pods using latest version
AVAILABLE → Pods ready to handle users

Task 4: Self-Healing — Delete a Pod and Watch It Come Back
This is the key difference between a Deployment and a standalone Pod.

# List pods
kubectl get pods -n dev

<img width="868" height="204" alt="image" src="https://github.com/user-attachments/assets/d7e95b10-253e-47ee-9651-ad120842a82b" />

# Delete one of the deployment's pods (use an actual pod name from your output)
kubectl delete pod <pod-name> -n dev

<img width="1195" height="411" alt="image" src="https://github.com/user-attachments/assets/ac658581-2c09-4aec-bfc7-4cf45f4ce31e" />


# Immediately check again
kubectl get pods -n dev
The Deployment controller detects that only 2 of 3 desired replicas exist and immediately creates a new one. The deleted pod is replaced within seconds.

<img width="1195" height="411" alt="image" src="https://github.com/user-attachments/assets/066b9850-de02-49c2-8d2f-519fcf5ddf39" />

Verify: Is the replacement pod's name the same as the one you deleted, or different?
-> No the old pod got deleted and new pod got deleted
old pod = nginx-deployment-5d9c84579f-csv4d
new pod = nginx-deployment-5d9c84579f-pxgg8

Task 5: Scale the Deployment

Change the number of replicas:

# Scale up to 5
kubectl scale deployment nginx-deployment --replicas=5 -n dev
kubectl get pods -n dev

# Scale down to 2
kubectl scale deployment nginx-deployment --replicas=2 -n dev
kubectl get pods -n dev
Watch how Kubernetes creates or terminates pods to match the desired count.

<img width="1195" height="411" alt="image" src="https://github.com/user-attachments/assets/9eae1998-d088-404f-80c7-e9f9ce3a3a58" />


You can also scale by editing the manifest — change replicas: 4 in your YAML file and run kubectl apply -f nginx-deployment.yaml again.

Verify: When you scaled down from 5 to 2, what happened to the extra pods?
-> The extra pods got deleted.

Task 6: Rolling Update
Update the Nginx image version to trigger a rolling update:

kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev

Watch the rollout in real time:

kubectl rollout status deployment/nginx-deployment -n dev

<img width="1309" height="366" alt="image" src="https://github.com/user-attachments/assets/705461f0-8a3f-4a60-b790-5564cb08b177" />


Kubernetes replaces pods one by one — old pods are terminated only after new ones are healthy. This means zero downtime.

Check the rollout history:

kubectl rollout history deployment/nginx-deployment -n dev

<img width="1300" height="169" alt="image" src="https://github.com/user-attachments/assets/8349a59e-c20a-4932-b2d6-e4b6321f680b" />

Now roll back to the previous version:

kubectl rollout undo deployment/nginx-deployment -n dev
kubectl rollout status deployment/nginx-deployment -n dev

<img width="1307" height="126" alt="image" src="https://github.com/user-attachments/assets/3a609041-152c-40aa-89fa-425193ed934a" />

Verify the image is back to the previous version:

kubectl describe deployment nginx-deployment -n dev | grep Image

<img width="1311" height="297" alt="image" src="https://github.com/user-attachments/assets/b2d03c58-6362-4717-b0f3-2fbeef36d8e8" />

Verify: What image version is running after the rollback?
-> Image: 1.24

Task 7: Clean Up
kubectl delete deployment nginx-deployment -n dev
kubectl delete pod nginx-dev -n dev
kubectl delete pod nginx-staging -n staging
kubectl delete namespace dev staging production

<img width="1294" height="255" alt="image" src="https://github.com/user-attachments/assets/cf77a2bf-b54c-472e-bc50-e396e731f521" />

Deleting a namespace removes everything inside it. Be very careful with this in production.

kubectl get namespaces
kubectl get pods -A
Verify: Are all your resources gone?

-> Yes all the pods are deleted from all the namespaces.


kubectl get <resource> -n <namespace> — target a specific namespace
kubectl get <resource> -A — list resources across all namespaces
selector.matchLabels in a Deployment must match template.metadata.labels — if they do not match, the Deployment will not manage the Pods
kubectl scale deployment <name> --replicas=N — quick way to scale
kubectl set image updates a container image without editing the YAML
kubectl rollout undo rolls back to the previous revision
kubectl rollout history shows past revisions of a Deployment
Deployments create ReplicaSets behind the scenes — you can see them with kubectl get replicasets -n <namespace>


What namespaces are and why you would use them

-> Namespaces are like folders inside Kubernetes
-> They help separate resources

Your Deployment manifest and an explanation of each section

What happens when you delete a Pod managed by a Deployment vs a standalone Pod

-> New pod gets created when the old pod is deleted the pod created using deployment
-> Once standalone pod is deleted its gone for ever.

How scaling works (both imperative and declarative)

imperative = kubectl scale deployment nginx-deployment --replicas=5  => using command
declarative = manually update the replicas in the deployment.yml

How rolling updates and rollbacks work

Rolling => Create new pods and delete the old pods
Rollback => Move back to the previous version
