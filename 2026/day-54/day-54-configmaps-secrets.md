### Task 1: Create a ConfigMap from Literals
1. Use `kubectl create configmap` with `--from-literal` to create a ConfigMap called `app-config` with keys `APP_ENV=production`, `APP_DEBUG=false`, and `APP_PORT=8080`

```
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=APP_PORT=8080
```
   
2. Inspect it with `kubectl describe configmap app-config` and `kubectl get configmap app-config -o yaml`

```
$ kubectl describe configmap app-config

Name:         app-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
APP_DEBUG:
----
false

APP_ENV:
----
production

APP_PORT:
----
8080


BinaryData
====

$ kubectl get configmap app-config -o yaml

apiVersion: v1
data:
  APP_DEBUG: "false"
  APP_ENV: production
  APP_PORT: "8080"
kind: ConfigMap
metadata:
  creationTimestamp: "2026-03-24T13:40:44Z"
  name: app-config
  namespace: default
  resourceVersion: "371740"
  uid: f74bed5a-e31d-47ed-8f56-b948d1d88c99

```
   
3. Notice the data is stored as plain text — no encoding, no encryption

-> The data is covered in the plain text

**Verify:** Can you see all three key-value pairs?

<img width="995" height="837" alt="image" src="https://github.com/user-attachments/assets/6e8fb8de-913b-432a-8762-fde153234695" />


---

### Task 2: Create a ConfigMap from a File
1. Write a custom Nginx config file that adds a `/health` endpoint returning "healthy"
   ```
   $ vi default.conf

    server {
        listen 80;

        location / {
            return 200 'Welcome to Nginx!';
        }

        location /health {
            return 200 'healthy';
            add_header Content-Type text/plain;
        }
    }
   ```
   
2. Create a ConfigMap from this file using `kubectl create configmap nginx-config --from-file=default.conf=<your-file>`
   ```
   kubectl create configmap nginx-config --from-file=default.conf=default.conf

   default.conf=default.conf

   Left side → key name in ConfigMap
   Right side → your local file
   ```

4. The key name (`default.conf`) becomes the filename when mounted into a Pod

**Verify:** Does `kubectl get configmap nginx-config -o yaml` show the file contents?
```
apiVersion: v1
data:
  default.conf: |
    server {
        listen 80;

        location / {
            return 200 'Welcome to Nginx!';
        }

        location /health {
            return 200 'healthy';
            add_header Content-Type text/plain;
        }
    }
kind: ConfigMap
metadata:
  creationTimestamp: "2026-03-25T16:39:02Z"
  name: nginx-config
  namespace: default
  resourceVersion: "377767"
  uid: e8152189-6eb1-4232-9d1c-f4d9e7175d16
```

<img width="978" height="519" alt="image" src="https://github.com/user-attachments/assets/31b37701-d970-4418-8ef5-f0bf7d455096" />


---

### Task 3: Use ConfigMaps in a Pod
1. Write a Pod manifest that uses `envFrom` with `configMapRef` to inject all keys from `app-config` as environment variables. Use a busybox container that prints the values.

```
$ vi env-pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config

$ kubectl apply -f env-pod.yml 
```

```
$ kubectl logs env-pod

KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
APP_DEBUG=false
HOSTNAME=env-pod
SHLVL=1
HOME=/root
APP_PORT=8080
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
APP_ENV=production
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```

2. Write a second Pod manifest that mounts `nginx-config` as a volume at `/etc/nginx/conf.d`. Use the nginx image.

```
$ vim nginx-pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-config-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config

$ kubectl apply -f nginx-pod.yml
```

3. Test that the mounted config works: `kubectl exec <pod> -- curl -s http://localhost/health`

```
$ kubectl apply -f nginx-pod.yml

$ kubectl exec <pod> -- curl -s http://localhost/health

```

Use environment variables for simple key-value settings. Use volume mounts for full config files.

**Verify:** Does the `/health` endpoint respond?
<img width="1270" height="142" alt="image" src="https://github.com/user-attachments/assets/17a46633-4dea-4a83-a297-843d5bf46108" />

---

### Task 4: Create a Secret
1. Use `kubectl create secret generic db-credentials` with `--from-literal` to store `DB_USER=admin` and `DB_PASSWORD=s3cureP@ssw0rd`

```
$ kubectl create secret generic db-credentials \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=s3cureP@ssw0rd

kubectl create secret: The base command to create a Secret resource in your cluster. 

generic: Specifies the type of secret to create, which in this case is Opaque. This is the default type for general-purpose sensitive data. 

db-credentials: The user-defined name of the Secret object. 
```
<img width="1035" height="129" alt="image" src="https://github.com/user-attachments/assets/7e5a30ce-ea57-4bec-908d-6a8f43a79244" />

   
2. Inspect with `kubectl get secret db-credentials -o yaml` — the values are base64-encoded

```
$ kubectl get secret db-credentials -o yaml

apiVersion: v1
data:
  DB_PASSWORD: czNjdXJlUEBzc3cwcmQ=
  DB_USER: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2026-03-25T17:52:31Z"
  name: db-credentials
  namespace: default
  resourceVersion: "384848"
  uid: 3494ea82-ee9a-4141-9c75-a701fa982b4d
type: Opaque

```
<img width="990" height="326" alt="image" src="https://github.com/user-attachments/assets/0a82475a-dc80-4ccf-a533-d1f2203e023e" />


3. Decode a value: `echo '<base64-value>' | base64 --decode`

```
$ echo "czNjdXJlUEBzc3cwcmQ=" | base64 --decode
s3cureP@ssw0rd

$ echo "YWRtaW4=" | base64 --decode
adminumesh
```

**base64 is encoding, not encryption.** Anyone with cluster access can decode Secrets. The real advantages are RBAC separation, tmpfs storage on nodes, and optional encryption at rest.

**Verify:** Can you decode the password back to plaintext?

```
-> Yes the password can be decoded to plain text again

RBAC = Only authorized users can access Secrets
Separation of Concerns = Keep sensitive data out of Pod YAML
Safer Storage
-> Stored in tmpfs (memory) on nodes
-> Not written to disk like normal files
Optional Encryption at Rest = Can be enabled in cluster
```
### Task 5: Use Secrets in a Pod
1. Write a Pod manifest that injects `DB_USER` as an environment variable using `secretKeyRef`
2. In the same Pod, mount the entire `db-credentials` Secret as a volume at `/etc/db-credentials` with `readOnly: true`
3. Verify: each Secret key becomes a file, and the content is the decoded plaintext value

**Verify:** Are the mounted file values plaintext or base64?

---

### Task 6: Update a ConfigMap and Observe Propagation
1. Create a ConfigMap `live-config` with a key `message=hello`
2. Write a Pod that mounts this ConfigMap as a volume and reads the file in a loop every 5 seconds
3. Update the ConfigMap: `kubectl patch configmap live-config --type merge -p '{"data":{"message":"world"}}'`
4. Wait 30-60 seconds — the volume-mounted value updates automatically
5. Environment variables from earlier tasks do NOT update — they are set at pod startup only

**Verify:** Did the volume-mounted value change without a pod restart?

---

### Task 7: Clean Up
Delete all pods, ConfigMaps, and Secrets you created.

---

## Hints
- `--from-literal=KEY=VALUE` for command-line values, `--from-file=key=filename` for file contents
- `envFrom` injects all keys; `env` with `valueFrom` injects individual keys
- `echo -n 'value' | base64` — always use `-n` to avoid encoding a trailing newline
- Volume-mounted ConfigMaps/Secrets auto-update; environment variables do not
- `kubectl get secret <name> -o jsonpath='{.data.KEY}' | base64 --decode` extracts and decodes a value

---

## Documentation
Create `day-54-configmaps-secrets.md` with:
- What ConfigMaps and Secrets are and when to use each
- The difference between environment variables and volume mounts
- Why base64 is encoding, not encryption
- How ConfigMap updates propagate to volumes but not env vars

---

## Submission
1. Add `day-54-configmaps-secrets.md` to `2026/day-54/`
2. Commit and push to your fork

---

## Learn in Public
Share on LinkedIn: "Learned Kubernetes ConfigMaps and Secrets today. Injected config as environment variables and volume mounts, and discovered that base64 encoding is not encryption."

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`

Happy Learning!
**TrainWithShubham**
