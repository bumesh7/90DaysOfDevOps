# Day 51 – Kubernetes Manifests and Your First Pods

## Task
Yesterday you set up a cluster. Today you actually deploy something. You will learn the structure of a Kubernetes manifest file and use it to create Pods — the smallest deployable unit in Kubernetes. By the end of today, you should be able to write a Pod definition from scratch without looking at docs.

---

## Expected Output
- At least 3 Pod manifests written by hand
- A markdown file: `day-51-pods.md`
- Screenshot of `kubectl get pods` showing your running pods

---

## The Anatomy of a Kubernetes Manifest

Every Kubernetes resource is defined using a YAML manifest with four required top-level fields:

```yaml
apiVersion: v1          # Which API version to use
kind: Pod               # What type of resource
metadata:               # Name, labels, namespace
  name: my-pod
  labels:
    app: my-app
spec:                   # The actual specification (what you want)
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

- `apiVersion` — tells Kubernetes which API group to use. For Pods, it is `v1`.
- `kind` — the resource type. Today it is `Pod`. Later you will use `Deployment`, `Service`, etc.
- `metadata` — the identity of your resource. `name` is required. `labels` are key-value pairs used for organization and selection.
- `spec` — the desired state. For a Pod, this means which containers to run, which images, which ports, etc.

---

## Challenge Tasks

### Task 1: Create Your First Pod (Nginx)
Create a file called `nginx-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Apply it:
```bash
kubectl apply -f nginx-pod.yaml
```


Verify:
```bash
kubectl get pods
kubectl get pods -o wide
```
<img width="1199" height="277" alt="image" src="https://github.com/user-attachments/assets/e6b14757-dd2f-4138-a621-6b4fae22c654" />


Wait until the STATUS shows `Running`. Then explore:
```bash
# Detailed info about the pod
kubectl describe pod nginx-pod

# Read the logs
kubectl logs nginx-pod

# Get a shell inside the container
kubectl exec -it nginx-pod -- /bin/bash

# Inside the container, run:
curl localhost:80
exit

```
<img width="1231" height="1000" alt="image" src="https://github.com/user-attachments/assets/93b1e336-29f6-43a1-8d3a-7cf59e430692" />
<img width="1862" height="848" alt="image" src="https://github.com/user-attachments/assets/6349e55d-34b2-40cd-8113-a6d1943daaa7" />
<img width="1004" height="670" alt="image" src="https://github.com/user-attachments/assets/69cccdf6-ae12-4c4c-b42d-2f58a36ad7ea" />

**Verify:** Can you see the Nginx welcome page when you curl from inside the pod?

<img width="1255" height="357" alt="image" src="https://github.com/user-attachments/assets/27a97817-de26-4376-847c-ef5e8b0323ed" />

---

### Task 2: Create a Custom Pod (BusyBox)
Write a new manifest `busybox-pod.yaml` from scratch (do not copy-paste the nginx one):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
```

Apply and verify:
```bash
kubectl apply -f busybox-pod.yaml
kubectl get pods
kubectl logs busybox-pod
```

Notice the `command` field — BusyBox does not run a long-lived server like Nginx. Without a command that keeps it running, the container would exit immediately and the pod would go into `CrashLoopBackOff`.

**Verify:** Can you see "Hello from BusyBox" in the logs?

<img width="1255" height="165" alt="image" src="https://github.com/user-attachments/assets/b3a8bee4-7608-4cff-80cc-c288c5eab394" />


---

### Task 3: Imperative vs Declarative
You have been using the declarative approach (writing YAML, then `kubectl apply`). Kubernetes also supports imperative commands:

```bash
# Create a pod without a YAML file
kubectl run redis-pod --image=redis:latest

# Check it
kubectl get pods
```
<img width="1255" height="272" alt="image" src="https://github.com/user-attachments/assets/42334ad7-a561-451f-a141-ff31daa4fc0d" />

Now extract the YAML that Kubernetes generated:
```bash
kubectl get pod redis-pod -o yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-03-18T15:44:06Z"
  generation: 1
  labels:
    run: redis-pod
  name: redis-pod
  namespace: default
  resourceVersion: "27504"
  uid: 78902e79-b8f7-4a0e-a251-bcde5ffd9e0c
spec:
  containers:
  - image: redis:latest
    imagePullPolicy: Always
    name: redis-pod
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-b88lk
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: umesh-cluster-worker2
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-b88lk
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2026-03-18T15:44:25Z"
    observedGeneration: 1
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2026-03-18T15:44:07Z"
    observedGeneration: 1
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-03-18T15:44:25Z"
    observedGeneration: 1
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-03-18T15:44:25Z"
    observedGeneration: 1
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-03-18T15:44:07Z"
    observedGeneration: 1
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://5c268942fd302b79a5c2bdc065bab30fb76368d14ecad8756e3f256b2b90af50
    image: docker.io/library/redis:latest
    imageID: docker.io/library/redis@sha256:315270d166080f537bbdf1b489b603aaaa213cb55a544acfa51feb7481abb1c0
    lastState: {}
    name: redis-pod
    ready: true
    resources: {}
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2026-03-18T15:44:24Z"
    user:
      linux:
        gid: 0
        supplementalGroups:
        - 0
        uid: 0
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-b88lk
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 172.18.0.5
  hostIPs:
  - ip: 172.18.0.5
  observedGeneration: 1
  phase: Running
  podIP: 10.244.3.2
  podIPs:
  - ip: 10.244.3.2
  qosClass: BestEffort
  startTime: "2026-03-18T15:44:07Z"

```

Compare this output with your hand-written manifests. Notice how much extra metadata Kubernetes adds automatically (status, timestamps, uid, resource version).

You can also use dry-run to generate YAML without creating anything:
```bash
kubectl run test-pod --image=nginx --dry-run=client -o yaml
```

This is a powerful trick — use it to quickly scaffold a manifest, then customize it.

**Verify:** Save the dry-run output to a file and compare its structure with your nginx-pod.yaml. What fields are the same? What is different?

---

```
$ kubectl run test-pod --image=nginx --dry-run=client -o yaml > auto-generated-pod-script.yaml

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test-pod
  name: test-pod
spec:
  containers:
  - image: nginx
    name: test-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

### Task 4: Validate Before Applying
Before applying a manifest, you can validate it:

```bash
# Check if the YAML is valid without actually creating the resource
kubectl apply -f nginx-pod.yaml --dry-run=client

YAML syntax is correct
Required fields exist
Basic structure is valid

# Validate against the cluster's API (server-side validation)
kubectl apply -f nginx-pod.yaml --dry-run=server

Everything client does
API version validity
Admission controllers / policies
Cluster-specific rules
Whether the resource would actually be accepted

Use client = while writing YAML quickly

Use server = before deploying to production
```

Now intentionally break your YAML (remove the `image` field or add an invalid field) and run dry-run again. See what error you get.

<img width="1255" height="236" alt="image" src="https://github.com/user-attachments/assets/3d2a5543-c158-47ec-8624-5a71ff70b9e0" />


**Verify:** What error does Kubernetes give when the image field is missing?

-> The Pod "nginx-pod" is invalid: spec.containers[0].image: Required value

---

### Task 5: Pod Labels and Filtering
Labels are how Kubernetes organizes and selects resources. You added labels in your manifests — now use them:

```bash
# List all pods with their labels
kubectl get pods --show-labels

# Filter pods by label
kubectl get pods -l app=nginx
kubectl get pods -l environment=dev

# Add a label to an existing pod
kubectl label pod nginx-pod environment=production

# Verify
kubectl get pods --show-labels

# Remove a label
kubectl label pod nginx-pod environment-
```

Write a manifest for a third pod with at least 3 labels (app, environment, team). Apply it and practice filtering.

---

### Task 6: Clean Up
Delete all the pods you created:

```bash
# Delete by name
kubectl delete pod nginx-pod
kubectl delete pod busybox-pod
kubectl delete pod redis-pod

# Or delete using the manifest file
kubectl delete -f nginx-pod.yaml

# Verify everything is gone
kubectl get pods
```

Notice that when you delete a standalone Pod, it is gone forever. There is no controller to recreate it. This is why in production you use Deployments (coming on Day 52) instead of bare Pods.

---

## Hints
- `kubectl apply -f` creates or updates a resource from a file
- `kubectl get pods -o wide` shows the node and IP address
- `kubectl describe pod <name>` shows events — very useful for debugging
- `kubectl logs <name>` shows container stdout/stderr
- `kubectl exec -it <name> -- /bin/sh` gives you a shell (use `/bin/sh` if `/bin/bash` is not available)
- Labels are just key-value pairs — they have no meaning to Kubernetes itself, only to selectors
- `--dry-run=client -o yaml` is your best friend for generating manifest templates

---

## Documentation
Create `day-51-pods.md` with:
- The four required fields of a Kubernetes manifest and what each does
- Your nginx, busybox, and third pod manifests
- Difference between imperative (`kubectl run`) and declarative (`kubectl apply -f`)
- Screenshot of your pods running
- What happens when you delete a standalone Pod?

---

## Submission
1. Add `day-51-pods.md` and your YAML files to `2026/day-51/`
2. Commit and push to your fork

---

## Learn in Public
Share on LinkedIn: "Wrote my first Kubernetes Pod manifests from scratch today. Created pods, got a shell inside them, and learned the difference between imperative and declarative approaches."

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`

Happy Learning!
**TrainWithShubham**
