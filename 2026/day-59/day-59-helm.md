### Task 1: Install Helm
1. Install Helm (brew, curl script, or chocolatey depending on your OS)
```
sudo snap install helm --classic
```
2. Verify with `helm version` and `helm env`

Three core concepts:
- **Chart** — a package of Kubernetes manifest templates
- **Release** — a specific installation of a chart in your cluster
- **Repository** — a collection of charts (like a package repo)

**Verify:** What version of Helm is installed?
```
version.BuildInfo{Version:"v4.2.0", GitCommit:"06468084e85c244c712834933d25ea232a4c2093", GitTreeState:"clean", GoVersion:"go1.26.3", KubeClientVersion:"v1.36"}
```
---

### Task 2: Add a Repository and Search
1. Add the Bitnami repository: `helm repo add bitnami https://charts.bitnami.com/bitnami`
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
2. Update: `helm repo update`
```
helm repo update
```
3. Search: `helm search repo nginx` and `helm search repo bitnami`
```
helm search repo nginx

helm search repo bitnami
```
**Verify:** How many charts does Bitnami have?
```
144 charts

with header = 145
helm search repo bitnami | wc -l

without header = 144
helm search repo bitnami -o yaml | grep -c "name:"
```
---

### Task 3: Install a Chart
1. Deploy nginx: `helm install my-nginx bitnami/nginx`
```
helm install my-nginx bitnami/nginx
```
<img width="1845" height="975" alt="image" src="https://github.com/user-attachments/assets/15a4af62-bdc1-4b8e-9df4-903a1affa820" />

2. Check what was created: `kubectl get all`
```
kubectl get all
```
<img width="984" height="297" alt="image" src="https://github.com/user-attachments/assets/7c3d81df-b3ca-4643-8532-813d58504dfa" />

3. Inspect the release: `helm list`, `helm status my-nginx`, `helm get manifest my-nginx`
```
helm list

helm status my-nginx

helm get manifest my-nginx
```
<img width="1335" height="129" alt="image" src="https://github.com/user-attachments/assets/34ff2d24-3796-4165-8419-f6aba20fbc65" />

<img width="1151" height="936" alt="image" src="https://github.com/user-attachments/assets/66285da0-f5e4-4215-a533-3bedb4c34a03" />

One command replaced writing a Deployment, Service, and ConfigMap by hand.

**Verify:** How many Pods are running? What Service type was created?
```
kubectl get pods

kubectl get svc
```
<img width="984" height="173" alt="image" src="https://github.com/user-attachments/assets/be29af89-9feb-4dd8-a627-4369b81e92b6" />

---

### Task 4: Customize with Values
1. View defaults: `helm show values bitnami/nginx`
```
helm show values bitnami/nginx
```
2. Install a custom release with `--set replicaCount=3 --set service.type=NodePort`
```
helm install nginx-custom bitnami/nginx \
  --set replicaCount=3 \
  --set service.type=NodePort
```
<img width="1847" height="957" alt="image" src="https://github.com/user-attachments/assets/d5df9cc1-04db-462a-9c30-d63523656396" />

3. Create a `custom-values.yaml` file with replicaCount, service type, and resource limits

vim custom-values.yml
```
replicaCount: 2

service:
  type: NodePort

resources:
  limits:
    cpu: "200m"
    memory: "256Mi"
  requests:
    cpu: "100m"
    memory: "128Mi"
```
4. Install another release using `-f custom-values.yaml`
```
helm install nginx-values bitnami/nginx -f custom-values.yml
```
5. Check overrides: `helm get values <release-name>`
```
helm get values nginx-values

kubectl get pods

kubectl get svc

kubectl get deploy
```
**Verify:** Does the values file release have the correct replicas and service type?
```
Yes, the values file release has the correct replica count and service type.

Example:

Replicas: 2
Service type: NodePort
```
<img width="1028" height="729" alt="image" src="https://github.com/user-attachments/assets/893d62b6-ba2d-4e4d-bd81-8d734f4d0cac" />

---

### Task 5: Upgrade and Rollback
1. Upgrade: `helm upgrade my-nginx bitnami/nginx --set replicaCount=5`
```
helm upgrade my-nginx bitnami/nginx --set replicaCount=5
```
2. Check history: `helm history my-nginx`
```
kubectl get deploy

helm history my-nginx
```
<img width="1171" height="235" alt="image" src="https://github.com/user-attachments/assets/960afa1e-b764-42a2-8b1b-58d0c8abe99c" />

2. Rollback: `helm rollback my-nginx 1`
```
helm rollback my-nginx 1
```
<img width="1153" height="408" alt="image" src="https://github.com/user-attachments/assets/41b3559c-be81-4d25-82bf-8c1cdefa53b7" />

3. Check history again — rollback creates a new revision (3), not overwriting revision 2

Same concept as Deployment rollouts from Day 52, but at the full stack level.

**Verify:** How many revisions after the rollback?
```
Total 3 revisions
```
<img width="1151" height="146" alt="image" src="https://github.com/user-attachments/assets/80314d2f-4e87-4f63-9838-34cc8f6276f0" />

---

### Task 6: Create Your Own Chart
1. Scaffold: `helm create my-app`
```
helm create my-app

cd my-app

ls -l
```
2. Explore the directory: `Chart.yaml`, `values.yaml`, `templates/deployment.yaml`
3. Look at the Go template syntax in templates: `{{ .Values.replicaCount }}`, `{{ .Chart.Name }}`
4. Edit `values.yaml` — set replicaCount to 3 and image to nginx:1.25

<img width="1445" height="317" alt="image" src="https://github.com/user-attachments/assets/f20091ec-f74f-45d4-84a9-f9e68f4a741d" />

5. Validate: `helm lint my-app`
6. Preview: `helm template my-release ./my-app`
7. Install: `helm install my-release ./my-app`
<img width="985" height="171" alt="image" src="https://github.com/user-attachments/assets/80d46d91-5499-4369-8466-c48a43ff7df0" />

8. Upgrade: `helm upgrade my-release ./my-app --set replicaCount=5`
<img width="985" height="171" alt="image" src="https://github.com/user-attachments/assets/51056521-7c94-4c88-88b0-91c9fe1dbda4" />

**Verify:** After installing, 3 replicas? After upgrading, 5?
```
After install → 3 replicas
After upgrade → 5 replicas
```
<img width="1713" height="607" alt="image" src="https://github.com/user-attachments/assets/1907f0b1-451d-4e7e-8642-58ac03c3b113" />

---

### Task 7: Clean Up
1. Uninstall all releases: `helm uninstall <name>` for each
```
helm list

helm uninstall my-nginx
helm uninstall nginx-custom
helm uninstall nginx-values
helm uninstall my-release
```
2. Remove chart directory and values file
3. Use `--keep-history` if you want to retain release history for auditing

**Verify:** Does `helm list` show zero releases?
```
helm list

kubectl get all
```

---

