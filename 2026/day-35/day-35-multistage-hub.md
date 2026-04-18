### Task 1: The Problem with Large Images
1. Write a simple Go, Java, or Node.js app (even a "Hello World" is fine)
2. Create a Dockerfile that builds and runs it in a **single stage**
3. Build the image and check its **size**
```
$ vim Dockerfile

FROM node:18

WORKDIR /app

COPY app.js .

CMD ["node", "app.js"]
```
```
$ vim app.js

console.log("Hello, Docker!");
```
```
$ docker build -t heelo-docker .
```
Note down the size — you'll compare it later.
```
The image size 1.57gb
```
<img width="965" height="152" alt="image" src="https://github.com/user-attachments/assets/8ea78808-175b-4dd5-a76a-8c6b2e0249bf" />


---

### Task 2: Multi-Stage Build
1. Rewrite the Dockerfile using **multi-stage build**:
   - Stage 1: Build the app (install dependencies, compile)
   - Stage 2: Copy only the built artifact into a minimal base image (`alpine`, `distroless`, or `scratch`)
2. Build the image and check its size again
3. Compare the two sizes
```
$ vim Dockerfile

#--------------Stage1-------------------

FROM node:18 AS builder

WORKDIR /app

COPY app.js .

#--------------Stage2-------------------

FROM gcr.io/distroless/nodejs20-debian12

WORKDIR /app

COPY --from=builder /app/app.js .

CMD ["node", "app.js"]

```
```
$ docker build -t hello-docker-multi .

$ docker images
```

Write in your notes: Why is the multi-stage image so much smaller?
```
The image size 170mb

Only the final runtime files are included (not build tools)
Heavy layers (like compilers, npm cache, dev dependencies) are left behind in the builder stage
Uses a minimal base image (alpine) instead of a full OS
Reduces unused files → smaller attack surface + faster deployment

Multi-stage builds separate:

🏗️ Build environment (heavy, temporary)
⚡ Runtime environment (light, optimized)
```
<img width="968" height="140" alt="image" src="https://github.com/user-attachments/assets/7756e626-0faa-4d7a-bcd4-78f112930c04" />

---

### Task 3: Push to Docker Hub
1. Create a free account on [Docker Hub](https://hub.docker.com) (if you don't have one)
2. Log in from your terminal
3. Tag your image properly: `yourusername/image-name:tag`
4. Push it to Docker Hub
5. Pull it on a different machine (or after removing locally) to verify

---

### Task 4: Docker Hub Repository
1. Go to Docker Hub and check your pushed image
2. Add a **description** to the repository
3. Explore the **tags** tab — understand how versioning works
4. Pull a specific tag vs `latest` — what happens?

---

### Task 5: Image Best Practices
Apply these to one of your images and rebuild:
1. Use a **minimal base image** (alpine vs ubuntu — compare sizes)
2. **Don't run as root** — add a non-root USER in your Dockerfile
3. Combine `RUN` commands to **reduce layers**
4. Use **specific tags** for base images (not `latest`)

Check the size before and after.

---

## Hints
- Multi-stage: use `FROM ... AS builder` then `COPY --from=builder`
- Login: `docker login`
- Tag: `docker tag local-image:tag username/repo:tag`
- Push: `docker push username/repo:tag`
- Non-root user: `RUN adduser` + `USER`

---

## Submission
1. Add your Dockerfiles and `day-35-multistage-hub.md` to `2026/day-35/`
2. Include the link to your Docker Hub repo
3. Commit and push to your fork

---

## Learn in Public
Share your before/after image sizes on LinkedIn — the difference is always impressive.

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`

Happy Learning!
**TrainWithShubham**
