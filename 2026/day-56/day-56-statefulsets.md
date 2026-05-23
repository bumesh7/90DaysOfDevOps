### Task 1: Understand the Problem
1. Create a Deployment with 3 replicas using nginx

vim nginx-deployment.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        image: nginx
        ports:
        - containerPort: 80
```
```
kubectl apply -f nginx-deployment.yml

kubectl get pods
```
2. Check the pod names — they are random (`app-xyz-abc`)

<img width="804" height="115" alt="image" src="https://github.com/user-attachments/assets/4053caa3-5a45-42ff-b418-076c66164a36" />

3. Delete a pod and notice the replacement gets a different random name
```
kubectl delete pod <pod name>
```

<img width="1159" height="184" alt="image" src="https://github.com/user-attachments/assets/85f6ae85-7b76-4f24-8fd7-4ddf6c365a7d" />

This is fine for web servers but not for databases where you need stable identity.

| Feature | Deployment | StatefulSet |
|---|---|---|
| Pod names | Random | Stable, ordered (`app-0`, `app-1`) |
| Startup order | All at once | Ordered: pod-0, then pod-1, then pod-2 |
| Storage | Shared PVC | Each pod gets its own PVC |
| Network identity | No stable hostname | Stable DNS per pod |

Delete the Deployment before moving on.

```
kubectl delete deployment nginx-deployment
```

**Verify:** Why would random pod names be a problem for a database cluster?
```
Database clusters need:

stable identity
fixed hostnames
reliable communication between nodes

Each node stores:

replication information
cluster membership
synchronization data

If names suddenly change:

nodes cannot find each other
replication may fail
leader election may break
data consistency issues can happen
```
---

### Task 2: Create a Headless Service
1. Write a Service manifest with `clusterIP: None` — this is a Headless Service

vim headless-service.yml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```
```
kubectl apply -f headless-service.yml
```
2. Set the selector to match the labels you will use on your StatefulSet pods
3. Apply it and confirm CLUSTER-IP shows `None`

<img width="1159" height="184" alt="image" src="https://github.com/user-attachments/assets/794cbf20-324d-4064-8854-b22f81fd13b9" />

A Headless Service creates individual DNS entries for each pod instead of load-balancing to one IP. StatefulSets require this.

**Verify:** What does the CLUSTER-IP column show?
```
Cluster IP column shows None
```

---

### Task 3: Create a StatefulSet
1. Write a StatefulSet manifest with `serviceName` pointing to your Headless Service

vim statefulset.yml
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: nginx-headless
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
        image: nginx
        ports:
        - containerPort: 80

        volumeMounts:
        - name: web-data
          mountPath: /usr/share/nginx/html

  volumeClaimTemplates:
  - metadata:
      name: web-data
    spec:
      accessModes: [ "ReadWriteOnce" ]

      resources:
        requests:
          storage: 100Mi
```
```
kubectl apply -f statefulset.yml
```
2. Set replicas to 3, use the nginx image
<img width="1158" height="110" alt="image" src="https://github.com/user-attachments/assets/e9d91298-d0b4-46e1-8eb0-0d9bf7668b68" />

3. Add a `volumeClaimTemplates` section requesting 100Mi of ReadWriteOnce storage
4. Apply and watch: `kubectl get pods -l <your-label> -w`
```
kubectl get pods -l app=nginx -w
```
<img width="1164" height="193" alt="image" src="https://github.com/user-attachments/assets/3e71d62d-a37d-4a7a-a860-c988fb64fe0e" />

Observe ordered creation — `web-0` first, then `web-1` after `web-0` is Ready, then `web-2`.

Check the PVCs: `kubectl get pvc` — you should see `web-data-web-0`, `web-data-web-1`, `web-data-web-2` (names follow the pattern `<template-name>-<pod-name>`).

**Verify:** What are the exact pod names and PVC names?
```
kubectl get pods

kubectl get pvc
```
<img width="1430" height="247" alt="image" src="https://github.com/user-attachments/assets/191eb05b-f7cd-42ba-9fe6-01a8c0693d16" />

---

### Task 4: Stable Network Identity
Each StatefulSet pod gets a DNS name: `<pod-name>.<service-name>.<namespace>.svc.cluster.local`
```
syntax:

<pod-name>.<service-name>.<namespace>.svc.cluster.local

example:

web-0.nginx-headless.default.svc.cluster.local
```
1. Run a temporary busybox pod and use `nslookup` to resolve `web-0.<your-headless-service>.default.svc.cluster.local`
```
kubectl run busybox --image=busybox:1.28 --rm -it --restart=Never -- sh

nslookup web-0.nginx-headless.default.svc.cluster.local
nslookup web-1.nginx-headless.default.svc.cluster.local
nslookup web-2.nginx-headless.default.svc.cluster.local
```
<img width="1707" height="490" alt="image" src="https://github.com/user-attachments/assets/941aa140-8d19-4bee-8e9a-acd4be7b6b5c" />

2. Do the same for `web-1` and `web-2`
3. Confirm the IPs match `kubectl get pods -o wide`
<img width="1269" height="138" alt="image" src="https://github.com/user-attachments/assets/67f98955-9cbb-4e61-b210-a7658371b0ef" />

**Verify:** Does the nslookup IP match the pod IP?
```
yes it matches
```
---

### Task 5: Stable Storage — Data Survives Pod Deletion
1. Write unique data to each pod: `kubectl exec web-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"`
```
kubectl exec web-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"

kubectl exec web-0 -- cat /usr/share/nginx/html/index.html
```
2. Delete `web-0`: `kubectl delete pod web-0`
```
kubectl delete pod web-0
```
3. Wait for it to come back, then check the data — it should still be "Data from web-0"
<img width="1571" height="292" alt="image" src="https://github.com/user-attachments/assets/f3debe28-f579-495b-9733-cc5ec292c712" />

The new pod reconnected to the same PVC.

**Verify:** Is the data identical after pod recreation?
```
Data is restored

Even though the pod was deleted:

the PVC was NOT deleted
StatefulSet attached the same storage back to web-0

Databases must not lose data when pods restart.

StatefulSets guarantee:

stable storage
stable identity
pod-to-volume mapping

That is why StatefulSets are used for databases.
```
---

### Task 6: Ordered Scaling
1. Scale up to 5: `kubectl scale statefulset web --replicas=5` — pods create in order (web-3, then web-4)
```
kubectl scale statefulset web --replicas=5
```
<img width="1553" height="379" alt="image" src="https://github.com/user-attachments/assets/61d354c9-534f-4be4-ab51-3fd374f0fa23" />

2. Scale down to 3 — pods terminate in reverse order (web-4, then web-3)
```
kubectl scale statefulset web --replicas=3
```
<img width="1548" height="332" alt="image" src="https://github.com/user-attachments/assets/57584201-6a1f-4581-bd6c-fe8dd93d920a" />

3. Check `kubectl get pvc` — all five PVCs still exist. Kubernetes keeps them on scale-down so data is preserved if you scale back up.
```
If you scale back to 5 later:

web-3 gets its old storage
web-4 gets its old storage

This prevents data loss.
```
**Verify:** After scaling down, how many PVCs exist?
```
Yes 6 PVC exists.
```
---

### Task 7: Clean Up
1. Delete the StatefulSet and the Headless Service
```
kubectl delete statefulset web

kubectl delete service nginx-headless

kubectl get pods
```
<img width="1124" height="278" alt="image" src="https://github.com/user-attachments/assets/67d6db8c-4fe3-4038-bc6b-8e7c96a568fc" />

2. Check `kubectl get pvc` — PVCs are still there (safety feature)
```
kubectl get pvc
```
```
Kubernetes treats persistent storage carefully.

Even if pods or StatefulSets are deleted:

storage remains
data remains safe

This prevents accidental database data loss.
```
3. Delete PVCs manually
```
kubectl delete pvc --all
```
<img width="1429" height="387" alt="image" src="https://github.com/user-attachments/assets/3b1bcc8b-16bf-4f17-8818-319173bc78e4" />

**Verify:** Were PVCs auto-deleted with the StatefulSet?
```
PVCs were NOT auto-deleted.

They remained after deleting the StatefulSet and had to be removed manually.
```
---

