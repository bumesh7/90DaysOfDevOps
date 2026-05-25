### Task 1: Install the Metrics Server

1. Check if it is already running: `kubectl get pods -n kube-system | grep metrics-server`
```
kubectl get pods -n kube-system | grep metrics-server
```
2. If not, install it:
   - Minikube: `minikube addons enable metrics-server`
   - Kind/kubeadm: apply the official manifest from the metrics-server GitHub releases
```
minikube: minikube addons enable metrics-server

kind: kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
3. On local clusters, you may need the `--kubelet-insecure-tls` flag (never in production)
```
kubectl edit deployment metrics-server -n kube-system
```
```
- --kubelet-insecure-tls

example:

args:
  - --cert-dir=/tmp
  - --secure-port=10250
  - --kubelet-insecure-tls
```

4. Wait 60 seconds, then verify: `kubectl top nodes` and `kubectl top pods -A`
```
kubectl get pods -n kube-system -w
```
**Verify:** What is the current CPU and memory usage of your node?
```
kubectl top pods -A
```
<img width="1028" height="405" alt="image" src="https://github.com/user-attachments/assets/f5d5bbf3-682f-44b1-95b3-4eabf1f6d63f" />

---

### Task 2: Explore kubectl top
1. Run `kubectl top nodes`, `kubectl top pods -A`, `kubectl top pods -A --sort-by=cpu`

<img width="1048" height="946" alt="image" src="https://github.com/user-attachments/assets/8b5e7f5b-e10e-4c6f-9a03-fa3ace54f667" />

2. `kubectl top` shows real-time usage, not requests or limits — these are different things
3. Data comes from the Metrics Server, which polls kubelets every 15 seconds

**Verify:** Which pod is using the most CPU right now?
```
NAMESPACE            NAME                                                  CPU(cores)   MEMORY(bytes)   
kube-system          coredns-7d764666f9-pr7vg                              4m           37Mi 
```
---

### Task 3: Create a Deployment with CPU Requests
1. Write a Deployment manifest using the `registry.k8s.io/hpa-example` image (a CPU-intensive PHP-Apache server)

vim php-apache.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 200m
```
```
kubectl apply -f php-apache.yaml

kubectl get pods

kubectl expose deployment php-apache --port=80
```
<img width="1098" height="85" alt="image" src="https://github.com/user-attachments/assets/2db8c2ec-2077-43ec-9c60-bee7818fa94c" />

2. Set `resources.requests.cpu: 200m` — HPA needs this to calculate utilization percentages
3. Expose it as a Service: `kubectl expose deployment php-apache --port=80`

Without CPU requests, HPA cannot work — this is the most common HPA setup mistake.

**Verify:** What is the current CPU usage of the Pod?
```
Current cpu usage is 1m
```
<img width="1102" height="150" alt="image" src="https://github.com/user-attachments/assets/743db678-5f08-4cbf-b2cb-78044b90ec59" />

---

### Task 4: Create an HPA (Imperative)
1. Run: `kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`
```
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```
2. Check: `kubectl get hpa` and `kubectl describe hpa php-apache`
3. TARGETS may show `<unknown>` initially — wait 30 seconds for metrics to arrive

This scales up when average CPU exceeds 50% of requests, and down when it drops below.

**Verify:** What does the TARGETS column show?
```
Meaning:

Current average CPU usage = 2%
Target CPU usage = 50%

current CPU utilization / target CPU utilization
```
<img width="1486" height="133" alt="image" src="https://github.com/user-attachments/assets/cf9c6f21-7b29-4c4b-ae31-b0d2d89e87d0" />

---

### Task 5: Generate Load and Watch Autoscaling
1. Start a load generator: `kubectl run load-generator --image=busybox:1.36 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"`
```
kubectl run load-generator \
  --image=registry.k8s.io/hpa-example \
  --restart=Never \
  -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"
```
<img width="942" height="405" alt="image" src="https://github.com/user-attachments/assets/31341c3c-4716-4c70-8ebc-86ea9c49475c" />

2. Watch HPA: `kubectl get hpa php-apache --watch`
3. Over 1-3 minutes, CPU climbs above 50%, replicas increase, CPU stabilizes
4. Stop the load: `kubectl delete pod load-generator`
5. Scale-down is slow (5-minute stabilization window) — you do not need to wait

**Verify:** How many replicas did HPA scale to under load?
```
Max of 3 replicas.
```
---

### Task 6: Create an HPA from YAML (Declarative)
1. Delete the imperative HPA: `kubectl delete hpa php-apache`
```
kubectl delete hpa php-apache
```
2. Write an HPA manifest using `autoscaling/v2` API with CPU target at 50% utilization

vim pho-apache-hpa.yml
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache

  minReplicas: 1
  maxReplicas: 10

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15

    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
```
```
kubectl apply -f php-apache-hpa.yml
```
3. Add a `behavior` section to control scale-up speed (no stabilization) and scale-down speed (300 second window)
4. Apply and verify with `kubectl describe hpa`

`autoscaling/v2` supports multiple metrics and fine-grained scaling behavior that the imperative command cannot configure.

**Verify:** What does the `behavior` section control?
```
The behavior section controls how aggressively HPA scales up and down.

stabilizationWindowSeconds: 0
scale up immediately
no waiting period

stabilizationWindowSeconds: 300
wait 5 minutes before reducing replicas
prevents rapid up/down scaling (“flapping”)
remove at most 50% of replicas per minute
```
<img width="954" height="507" alt="image" src="https://github.com/user-attachments/assets/ec63d16a-fad2-4698-8c8a-602da3d94f1f" />

---

### Task 7: Clean Up
Delete the HPA, Service, Deployment, and load-generator pod. Leave the Metrics Server installed.

```
kubectl delete hpa php-apache
kubectl delete svc php-apache
kubectl delete deployment php-apache
kubectl delete pod load-generator

kubectl get all
```
<img width="986" height="282" alt="image" src="https://github.com/user-attachments/assets/5d29ec17-06b0-4f47-844a-a0bc88b7461f" />


---
