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
2. Install a custom release with `--set replicaCount=3 --set service.type=NodePort`
3. Create a `custom-values.yaml` file with replicaCount, service type, and resource limits
4. Install another release using `-f custom-values.yaml`
5. Check overrides: `helm get values <release-name>`

**Verify:** Does the values file release have the correct replicas and service type?

---

### Task 5: Upgrade and Rollback
1. Upgrade: `helm upgrade my-nginx bitnami/nginx --set replicaCount=5`
2. Check history: `helm history my-nginx`
3. Rollback: `helm rollback my-nginx 1`
4. Check history again — rollback creates a new revision (3), not overwriting revision 2

Same concept as Deployment rollouts from Day 52, but at the full stack level.

**Verify:** How many revisions after the rollback?

---

### Task 6: Create Your Own Chart
1. Scaffold: `helm create my-app`
2. Explore the directory: `Chart.yaml`, `values.yaml`, `templates/deployment.yaml`
3. Look at the Go template syntax in templates: `{{ .Values.replicaCount }}`, `{{ .Chart.Name }}`
4. Edit `values.yaml` — set replicaCount to 3 and image to nginx:1.25
5. Validate: `helm lint my-app`
6. Preview: `helm template my-release ./my-app`
7. Install: `helm install my-release ./my-app`
8. Upgrade: `helm upgrade my-release ./my-app --set replicaCount=5`

**Verify:** After installing, 3 replicas? After upgrading, 5?

---

### Task 7: Clean Up
1. Uninstall all releases: `helm uninstall <name>` for each
2. Remove chart directory and values file
3. Use `--keep-history` if you want to retain release history for auditing

**Verify:** Does `helm list` show zero releases?

---

## Hints
- `helm show values <chart>` — see what you can customize
- `--set key=value` for single overrides, `-f values.yaml` for files
- Nested values use dots: `--set service.type=NodePort`
- `helm get values <release>` shows overrides, `--all` for everything
- `helm template` renders without installing — great for debugging
- `helm lint` validates chart structure before installing
- Templates: `{{ .Values.key }}`, `{{ .Chart.Name }}`, `{{ .Release.Name }}`

---

## Documentation
Create `day-59-helm.md` with:
- What Helm is and the three core concepts
- How to install, customize, upgrade, and rollback
- The structure of a Helm chart and how Go templating works
- Your `custom-values.yaml` with explanations

---

## Submission
1. Add `day-59-helm.md` and `custom-values.yaml` to `2026/day-59/`
2. Commit and push to your fork

---

## Learn in Public
Share on LinkedIn: "Learned Helm today — deployed charts, customized with values, performed rollbacks, and created my own chart from scratch. One command replaces dozens of YAML files."

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`

Happy Learning!
**TrainWithShubham**
