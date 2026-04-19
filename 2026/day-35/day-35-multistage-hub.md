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

CMD ["app.js"]

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
```
$ docker tag hello-docker-multi:latest umesh4999/hello-docker:latest
```
4. Push it to Docker Hub
```
$ docker push umesh4999/hello-docker:latest
```
5. Pull it on a different machine (or after removing locally) to verify
```
$ docker rmi umesh4999/hello-docker:latest

$ docker pull umesh4999/hello-docker:latest

$ docker run yourusername/hello-docker:latest
```
```
ex:

docker build -t hello-docker-multi .

docker tag hello-docker-multi umesh4999/hello-docker:latest

docker push umesh4999/hello-docker:latest

docker rmi umesh4999/hello-docker:latest

docker pull umesh4999/hello-docker:latest

docker run umesh4999/hello-docker:latest
```
<img width="1455" height="79" alt="image" src="https://github.com/user-attachments/assets/b78430e3-bf15-4765-8fb4-1f250c03607f" />

---

### Task 4: Docker Hub Repository
1. Go to Docker Hub and check your pushed image
2. Add a **description** to the repository
3. Explore the **tags** tab — understand how versioning works
4. Pull a specific tag vs `latest` — what happens?
```
docker build -t hello-docker-multi .

docker tag hello-docker-multi umesh4999/hello-docker:v1

docker push umesh4999/hello-docker:v1

docker rmi umesh4999/hello-docker:v1

docker pull umesh4999/hello-docker:v1

docker run umesh4999/hello-docker:v1
```
---

### Task 5: Image Best Practices
Apply these to one of your images and rebuild:
1. Use a **minimal base image** (alpine vs ubuntu — compare sizes)
2. **Don't run as root** — add a non-root USER in your Dockerfile
3. Combine `RUN` commands to **reduce layers**
4. Use **specific tags** for base images (not `latest`)
```
# Stage 1
FROM node:18-alpine3.19 AS builder
WORKDIR /app
COPY app.js .

# Stage 2
FROM node:18-alpine3.19

# create non-root user
RUN adduser -D appuser

WORKDIR /app
COPY --from=builder /app/app.js .

USER appuser

CMD ["app.js"]
```
```
$ docker tag build -t hello-docker-final:latest .

$ docker tag hello-docker-final:latest umesh4999/hello-docker-final:v1

$ docker push umesh4999/hello-docker-final:v1
```

Check the size before and after.
```
Previously it was 200MB and now it is 181MB
```
<img width="1022" height="161" alt="image" src="https://github.com/user-attachments/assets/5161d718-5edc-4183-aa97-818a10accd1e" />

---
