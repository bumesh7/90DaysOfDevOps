### Task 1: Create the Namespace (Day 52)
1. Create a `capstone` namespace
```
kubectl create namespace capstone
```
2. Set it as your default: `kubectl config set-context --current --namespace=capstone`
```
# Set capstone as the default namespace for current context
kubectl config set-context --current --namespace=capstone

# Verify current namespace
kubectl config view --minify | grep namespace
```
---

### Task 2: Deploy MySQL (Days 54-56)
1. Create a Secret with `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, and `MYSQL_PASSWORD` using `stringData`

vim mysql-secret.yml
```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: capstone
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: rootpass123
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wpuser
  MYSQL_PASSWORD: wppass123
```
```
kubectl apply -f mysql-secret.yml
```
2. Create a Headless Service (`clusterIP: None`) for MySQL on port 3306

vim mysql-headless-service.yml
```
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: capstone
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
```

3. Create a StatefulSet for MySQL with:
   - Image: `mysql:8.0`
   - `envFrom` referencing the Secret
   - Resource requests (cpu: 250m, memory: 512Mi) and limits (cpu: 500m, memory: 1Gi)
   - A `volumeClaimTemplates` section requesting 1Gi of storage, mounted at `/var/lib/mysql`

vim mysql-statefulset.yml
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: capstone
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0

          envFrom:
            - secretRef:
                name: mysql-secret

          ports:
            - containerPort: 3306

          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"
            limits:
              cpu: "500m"
              memory: "1Gi"

          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql

  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
```
```
kubectl apply -f mysql-statefulset.yml
```

4. Verify MySQL works: `kubectl exec -it mysql-0 -- mysql -u <user> -p<password> -e "SHOW DATABASES;"`
```
kubectl get pods
kubectl get pvc
kubectl get svc
```
**Verify:** Can you see the `wordpress` database?
```
kubectl exec -it mysql-0 -- mysql -u wpuser -pwppass123 -e "SHOW DATABASES;"
```

<img width="1433" height="438" alt="image" src="https://github.com/user-attachments/assets/927ccfc8-e9be-41c1-80ad-7a1dfba79e12" />

---

### Task 3: Deploy WordPress (Days 52, 54, 57)
1. Create a ConfigMap with `WORDPRESS_DB_HOST` set to `mysql-0.mysql.capstone.svc.cluster.local:3306` and `WORDPRESS_DB_NAME`

vim wordpress-configmap.yml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-config
  namespace: capstone
data:
  WORDPRESS_DB_HOST: mysql-0.mysql.capstone.svc.cluster.local:3306
  WORDPRESS_DB_NAME: wordpress
```
```
kubectl apply -f wordpress-configmap.yml
```
2. Create a Deployment with 2 replicas using `wordpress:latest` that:
   - Uses `envFrom` for the ConfigMap
   - Uses `secretKeyRef` for `WORDPRESS_DB_USER` and `WORDPRESS_DB_PASSWORD` from the MySQL Secret
   - Has resource requests and limits
   - Has a liveness probe and readiness probe on `/wp-login.php` port 80

vim wordpress-deployment.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: capstone

spec:
  replicas: 2

  selector:
    matchLabels:
      app: wordpress

  template:
    metadata:
      labels:
        app: wordpress

    spec:
      containers:
        - name: wordpress
          image: wordpress:latest

          ports:
            - containerPort: 80

          envFrom:
            - configMapRef:
                name: wordpress-config

          env:
            - name: WORDPRESS_DB_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_USER

            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_PASSWORD

          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"

          readinessProbe:
            httpGet:
              path: /wp-login.php
              port: 80
            initialDelaySeconds: 60
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 5

          livenessProbe:
            httpGet:
              path: /wp-login.php
              port: 80
            initialDelaySeconds: 120
            periodSeconds: 15
            timeoutSeconds: 5
            failureThreshold: 5
```
3. Wait until both pods show `1/1 Running`

**Verify:** Are both WordPress pods running and ready?
<img width="914" height="133" alt="image" src="https://github.com/user-attachments/assets/93417b25-340c-4d15-9790-d38186da51af" />

---

### Task 4: Expose WordPress (Day 53)
1. Create a NodePort Service on port 30080 targeting the WordPress pods

vim wordpress-service.yml
```
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: capstone

spec:
  type: NodePort

  selector:
    app: wordpress

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```
```
kubectl apply -f wordpress-service.yml

kubectl get svc -n capstone
```

2. Access WordPress in your browser:
   - Minikube: `minikube service wordpress -n capstone`
   - Kind: `kubectl port-forward svc/wordpress 8080:80 -n capstone`

```
kubectl port-forward svc/wordpress 8080:80 -n capstone
```
<img width="902" height="903" alt="image" src="https://github.com/user-attachments/assets/2100e521-c415-456d-914e-1dacf306edfb" />

3. Complete the setup wizard and create a blog post

**Verify:** Can you see the WordPress setup page?
<img width="1677" height="899" alt="image" src="https://github.com/user-attachments/assets/f9753c40-aba9-4a78-8f04-8d0b2c232989" />

---

### Task 5: Test Self-Healing and Persistence
1. Delete a WordPress pod — watch the Deployment recreate it within seconds. Refresh the site.
```
kubectl get pods -n capstone

kubectl delete pod <wordpress-pod-name> -n capstone
```
2. Delete the MySQL pod: `kubectl delete pod mysql-0 -n capstone` — watch the StatefulSet recreate it
```
kubectl delete pod <mysql>
```
3. After MySQL recovers, refresh WordPress — your blog post should still be there

**Verify:** After deleting both pods, is your blog post still there?
```
Yes data is restored.
```
---

### Task 6: Set Up HPA (Day 58)
1. Write an HPA manifest targeting the WordPress Deployment with CPU at 50%, min 2, max 10 replicas

vim wordpresss-hpa.yml
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: wordpress-hpa
  namespace: capstone

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: wordpress

  minReplicas: 2
  maxReplicas: 10

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```
```
kubectl apply -f wordpress-hpa.yml
```

2. Apply and check: `kubectl get hpa -n capstone`
```
kubectl get hpa -n capstone
```
3. Run `kubectl get all -n capstone` for the complete picture
```
kubectl get all -n capstone
```
**Verify:** Does the HPA show correct min/max and target?
```
yes 2%/50%
```
<img width="1262" height="494" alt="image" src="https://github.com/user-attachments/assets/0b4d7fdf-54e0-441e-ab2f-580e3a44fe99" />

---

### Task 7: (Bonus) Compare with Helm (Day 59)
1. Install WordPress using `helm install wp-helm bitnami/wordpress` in a separate namespace
```
kubectl create namespace helm-demo
```
2. Compare: how many resources did each approach create? Which gives more control?
```
helm repo add bitnami https://charts.bitnami.com/bitnami

helm repo update
```
```
helm install wp-helm bitnami/wordpress -n helm-demo

kubectl get all -n helm-demo
```
3. Clean up the Helm deployment
```
helm uninstall wp-helm -n helm-demo

kubectl delete namespace helm-demo

kubectl get all -n helm-demo
```
---

### Task 8: Clean Up and Reflect
1. Take a final look: `kubectl get all -n capstone`
2. Count the concepts you used: Namespace, Secret, ConfigMap, PVC, StatefulSet, Headless Service, Deployment, NodePort Service, Resource Limits, Probes, HPA, Helm — twelve concepts in one deployment
3. Delete the namespace: `kubectl delete namespace capstone`
4. Reset default: `kubectl config set-context --current --namespace=default`

**Verify:** Did deleting the namespace remove everything?
```
Yes deleteing namespace everything is deleted.
```
```
kubectl delete namespace capstone && kubectl config set-context --current --namespace=default
```
---

