### Task 1: See the Problem — Data Lost on Pod Deletion
1. Write a Pod manifest that uses an `emptyDir` volume and writes a timestamped message to `/data/message.txt`

```
$ vim empty_dir.yml

kind: Pod
apiVersion: v1
metadata:
 name: empty-dir

spec:
 containers:
 - name: busybox
   image: busybox
   command: ["/bin/sh", "-c"]
   args:
    - while true; do date >> /data/message.txt; sleep 5; done
   volumeMounts:
   - name: data-volume
     mountPath: /data
 volumes:
 - name: data-volume
   emptyDir: {}

$ kubectl apply -f empty_dir.yml

$ kubectl get pods

$ kubectl exec -it <pod-name> -- cat /data/message.txt
```

2. Apply it, verify the data exists with `kubectl exec`
   
<img width="1078" height="340" alt="image" src="https://github.com/user-attachments/assets/966cdaf8-a73d-413b-8506-40fd3e2fb012" />

3. Delete the Pod, recreate it, check the file again — the old message is gone

```
$ kubectl delete pod <pod-name>

$ kubectl apply -f empty_dir.yml

$ kubectl exec -it <pod-name> -- cat /data/message.txt
```

**Verify:** Is the timestamp the same or different after recreation?
```
The time stamp is different as the old pod was deleted, the data also lost and once new pod is created new data is showing.
```
<img width="1077" height="317" alt="image" src="https://github.com/user-attachments/assets/1ed25b06-d0e9-4370-b4a3-cf83c5cc2268" />

---

### Task 2: Create a PersistentVolume (Static Provisioning)
1. Write a PV manifest with `capacity: 1Gi`, `accessModes: ReadWriteOnce`, `persistentVolumeReclaimPolicy: Retain`, and `hostPath` pointing to `/tmp/k8s-pv-data`

```
$ vim pv.yml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/k8s-pv-data

$ kubectl apply -f pv.yml
```

2. Apply it and check `kubectl get pv` — status should be `Available`

```
$ kubectl get pv
```

Access modes to know:
- `ReadWriteOnce (RWO)` — read-write by a single node
- `ReadOnlyMany (ROX)` — read-only by many nodes
- `ReadWriteMany (RWX)` — read-write by many nodes

`hostPath` is fine for learning, not for production.

**Verify:** What is the STATUS of the PV?

```
The status of pv is available
```
<img width="1248" height="173" alt="image" src="https://github.com/user-attachments/assets/26e01873-3c2d-4eed-9189-7c8be9c888d5" />

---

### Task 3: Create a PersistentVolumeClaim
1. Write a PVC manifest requesting `500Mi` of storage with `ReadWriteOnce` access

```
$ vim pvc.yml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ""
  resources:
    requests:
      storage: 500Mi

$ kubectl apply -f pvc.yml
```

2. Apply it and check both `kubectl get pvc` and `kubectl get pv`

```
$ kubectl get pv

$ kubectl get pvc
```

3. Both should show `Bound` — Kubernetes matched them by capacity and access mode

<img width="1317" height="166" alt="image" src="https://github.com/user-attachments/assets/641a11f9-a1b9-42db-8a66-c3e68c95246d" />


**Verify:** What does the VOLUME column in `kubectl get pvc` show?

```
It shows the name of the Persistent volume that claimed the Persistent Volume Claim.
```
---

### Task 4: Use the PVC in a Pod — Data That Survives
1. Write a Pod manifest that mounts the PVC at `/data` using `persistentVolumeClaim.claimName`
```
$ vim pvc-pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo
spec:
  nodeName: umesh-cluster-worker  

  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c"]
    args:
      - while true; do date >> /data/message.txt; sleep 5; done
    volumeMounts:
    - name: data-volume
      mountPath: /data

  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: my-pvc
$ kubectl apply -f pvc-pod.yml

$ kubectl get pods

$ kubectl exec -it pvc-demo -- cat /data/message.txt
```

2. Write data to `/data/message.txt`, then delete and recreate the Pod

```
$ kubectl exec -it pvc-demo -- cat /data/message.txt

$ kubectl get pods

$ kubect delete pod <podname>
```
<img width="1301" height="121" alt="image" src="https://github.com/user-attachments/assets/92622dae-2c10-4cd0-8f59-602fcbe753be" />

3. Check the file — it should contain data from both Pods

<img width="1330" height="364" alt="image" src="https://github.com/user-attachments/assets/2b37b8ba-ad96-47b1-8b46-944536e551bb" />

**Verify:** Does the file contain data from both the first and second Pod?

```
Yes the data exists in the second pod, idf the pod is created in another node the data is mismatch.
So we need to create the new pod in the same working node.
```
---

### Task 5: StorageClasses and Dynamic Provisioning
1. Run `kubectl get storageclass` and `kubectl describe storageclass`

<img width="1835" height="359" alt="image" src="https://github.com/user-attachments/assets/ed2986d0-a7da-4e97-aa80-91bbabe69f0f" />

2. Note the provisioner, reclaim policy, and volume binding mode

```
rovisioner → who creates dynamic volumes automatically  (rancher.io/local-path)
ReclaimPolicy → When PVC is deleted → PV also deleted  (Delete)
VolumeBindingMode → Volume created only when Pod is scheduled      (WaitForFirstConsumer)
                    Helps avoid wrong node issues

You create PVC
Kubernetes uses StorageClass
It automatically creates a PV (volume)
```

3. With dynamic provisioning, developers only create PVCs — the StorageClass handles PV creation automatically

```
Yes once PVC is created the PV is created automatically.
```

**Verify:** What is the default StorageClass in your cluster?
```
standard → name of the StorageClass
(default) → Kubernetes will use this automatically if you don’t specify anything in your PVC
```

---

### Task 6: Dynamic Provisioning
1. Write a PVC manifest that includes `storageClassName: standard` (or your cluster's default)

```
$ vim dynamic-pvc.yml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 500Mi

$ kubectl apply -f dynamic-pvc.yml
```

2. Apply it — a PV should appear automatically in `kubectl get pv`

```
$ kubect get pvc

$ kubectl get pv

PVC is in pending state once the PVC is used by the pod then PV will be created.
```

<img width="1334" height="180" alt="image" src="https://github.com/user-attachments/assets/7b5d2631-38df-4d78-8055-4b38ba9cfabb" />

3. Use this PVC in a Pod, write data, verify it works

```
$ vim dynamic-pod.yml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 500Mi

$ kubectl apply -f dynamic-pod.yml

$ kubectl get pvc

$ kubectl get pv

```
<img width="1716" height="193" alt="image" src="https://github.com/user-attachments/assets/8117de5c-68c1-471d-ad6b-0f74033667ea" />

**Verify:** How many PVs exist now? Which was manual, which was dynamic?

```
2 pv exixts

my-pv was created manually
pvc-*********  was created by dymamic pvc
```
---

### Task 7: Clean Up
1. Delete all pods first
2. Delete PVCs — check `kubectl get pv` to see what happened
3. The dynamic PV is gone (Delete reclaim policy). The manual PV shows `Released` (Retain policy).
4. Delete the remaining PV manually

```
$ kubectl delete get pods

$ kubectl delete pod --all

$ kubect delete pvc --all

$ kubectl get pv

$ kubectl delete pv <pv-name>
```

<img width="1716" height="532" alt="image" src="https://github.com/user-attachments/assets/8d61b811-dda1-4f0b-a0b6-a8afd1ed8b2f" />

**Verify:** Which PV was auto-deleted and which was retained? Why?

```
The PV which was created using dynamic PVC was deleted automatic.
Manually created PV was retained 
```




