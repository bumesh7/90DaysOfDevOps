### Task 1: Resource Requests and Limits
1. Write a Pod manifest with `resources.requests` (cpu: 100m, memory: 128Mi) and `resources.limits` (cpu: 250m, memory: 256Mi)

vim resource-equests.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
    - name: nginx-container
      image: nginx
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "250m"
          memory: "256Mi"
```
2. Apply and inspect with `kubectl describe pod` — look for the Requests, Limits, and QoS Class sections
```
kubectl apply -f resource-request.yml

kubectl describe pod resource-demo
```
<img width="460" height="142" alt="image" src="https://github.com/user-attachments/assets/db674990-085f-4c93-9319-b464354dc008" />

3. Since requests and limits differ, the QoS class is `Burstable`. If equal, it would be `Guaranteed`. If missing, `BestEffort`.

CPU is in millicores: `100m` = 0.1 CPU. Memory is in mebibytes: `128Mi`.

<img width="940" height="138" alt="image" src="https://github.com/user-attachments/assets/ee5570f4-8b8f-4d5b-a49e-d409071c0851" />

**Requests** = guaranteed minimum (scheduler uses this for placement). **Limits** = maximum allowed (kubelet enforces at runtime).

**Verify:** What QoS class does your Pod have?
```
Burstable

Guaranteed → requests = limits for CPU and memory
Burstable → requests/limits exist but differ
BestEffort → no requests or limits defined
```
---

### Task 2: OOMKilled — Exceeding Memory Limits
1. Write a Pod manifest using the `polinux/stress` image with a memory limit of `100Mi`
vim memory-stress.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: memory-stress
spec:
  containers:
    - name: stress-container
      image: polinux/stress
      resources:
        limits:
          memory: "100Mi"
      command: ["stress"]
      args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]
```
2. Set the stress command to allocate 200M of memory: `command: ["stress"] args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]`
3. Apply and watch — the container gets killed immediately
```
kubectl get pods -w
kubectl describe pod memory-stress
```
<img width="939" height="166" alt="image" src="https://github.com/user-attachments/assets/31cf7b28-a06f-47d1-8ab6-3bf4cd2d5756" />

CPU is throttled when over limit. Memory is killed — no mercy.

Check `kubectl describe pod` for `Reason: OOMKilled` and `Exit Code: 137` (128 + SIGKILL).

**Verify:** What exit code does an OOMKilled container have?
```
Exit Code: 137
```
<img width="720" height="388" alt="image" src="https://github.com/user-attachments/assets/ae6e6128-ae68-4e88-a532-bd62030c32b0" />

---

### Task 3: Pending Pod — Requesting Too Much
1. Write a Pod manifest requesting `cpu: 100` and `memory: 128Gi`

vim huge-request.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: huge-request-pod
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
        requests:
          cpu: "100"
          memory: "128Gi"
```

2. Apply and check — STATUS stays `Pending` forever
```
kubectl apply -f huge-request.yml
```
3. Run `kubectl describe pod` and read the Events — the scheduler says exactly why: insufficient resources
<img width="911" height="117" alt="image" src="https://github.com/user-attachments/assets/3edc6bec-1785-4401-a161-c7c4532100f9" />

<img width="1853" height="114" alt="image" src="https://github.com/user-attachments/assets/b6c09883-af7f-4219-8d51-bbae04ce5012" />

**Verify:** What event message does the scheduler produce?
```
FailedScheduling: Insufficient cpu, Insufficient memory

This happens because:

Scheduler checks Pod requests
No node has enough free CPU or RAM
So the Pod remains in Pending state forever until resources become available
```
---

### Task 4: Liveness Probe
A liveness probe detects stuck containers. If it fails, Kubernetes restarts the container.

1. Write a Pod manifest with a busybox container that creates `/tmp/healthy` on startup, then deletes it after 30 seconds
2. Add a liveness probe using `exec` that runs `cat /tmp/healthy`, with `periodSeconds: 5` and `failureThreshold: 3`
3. After the file is deleted, 3 consecutive failures trigger a restart. Watch with `kubectl get pod -w`

**Verify:** How many times has the container restarted?

---

### Task 5: Readiness Probe
A readiness probe controls traffic. Failure removes the Pod from Service endpoints but does NOT restart it.

1. Write a Pod manifest with nginx and a `readinessProbe` using `httpGet` on path `/` port `80`
2. Expose it as a Service: `kubectl expose pod <name> --port=80 --name=readiness-svc`
3. Check `kubectl get endpoints readiness-svc` — the Pod IP is listed
4. Break the probe: `kubectl exec <pod> -- rm /usr/share/nginx/html/index.html`
5. Wait 15 seconds — Pod shows `0/1` READY, endpoints are empty, but the container is NOT restarted

**Verify:** When readiness failed, was the container restarted?

---

### Task 6: Startup Probe
A startup probe gives slow-starting containers extra time. While it runs, liveness and readiness probes are disabled.

1. Write a Pod manifest where the container takes 20 seconds to start (e.g., `sleep 20 && touch /tmp/started`)
2. Add a `startupProbe` checking for `/tmp/started` with `periodSeconds: 5` and `failureThreshold: 12` (60 second budget)
3. Add a `livenessProbe` that checks the same file — it only kicks in after startup succeeds

**Verify:** What would happen if `failureThreshold` were 2 instead of 12?

---

### Task 7: Clean Up
Delete all pods and services you created.

---

## Hints
- CPU is compressible (throttled); memory is incompressible (OOMKilled)
- CPU: `1` = 1 core = `1000m`. Memory: `Mi` (mebibytes), `Gi` (gibibytes)
- QoS: Guaranteed (requests == limits), Burstable (requests < limits), BestEffort (none set)
- Probe types: `httpGet`, `exec`, `tcpSocket`
- Liveness failure = restart. Readiness failure = remove from endpoints. Startup failure = kill.
- `initialDelaySeconds`, `periodSeconds`, `failureThreshold` control probe timing
- Exit code 137 = OOMKilled (128 + SIGKILL)

---

## Documentation
Create `day-57-resources-probes.md` with:
- Requests vs limits (scheduling vs enforcement)
- What happens when CPU or memory limits are exceeded
- Liveness vs readiness vs startup probes
- Screenshots of OOMKilled, Pending, and probe events

---

## Submission
1. Add `day-57-resources-probes.md` to `2026/day-57/`
2. Commit and push to your fork

---

## Learn in Public
Share on LinkedIn: "Set resource requests and limits in Kubernetes today, watched a pod get OOMKilled, and added liveness, readiness, and startup probes for self-healing."

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`

Happy Learning!
**TrainWithShubham**
