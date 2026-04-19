## Self-Assessment Checklist
Mark yourself honestly — **can do**, **shaky**, or **haven't done**:

- [ ] Run a container from Docker Hub (interactive + detached)
```
docker run -it nginx /bin/bash
docker run -d nginx
```
- [ ] List, stop, remove containers and images
```
docker ps
docker ps -a
docker stop <container_id>
docker rm <container_id>
docker rmi <image_id>
```
- [ ] Explain image layers and how caching works
```
Each Dockerfile step creates a layer. If nothing changes, Docker reuses cached layers to speed up builds.
```
- [ ] Write a Dockerfile from scratch with FROM, RUN, COPY, WORKDIR, CMD
```
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```
- [ ] Explain CMD vs ENTRYPOINT
```
CMD = default command (can be overridden)
ENTRYPOINT = fixed command (always runs)
```
- [ ] Build and tag a custom image
```
docker build -t myapp:latest .
```
- [ ] Create and use named volumes
```
docker volume create myvolume
docker run -v myvolume:/data nginx
```
- [ ] Use bind mounts
```
docker run -v $(pwd):/app nginx
```
- [ ] Create custom networks and connect containers
```
docker network create mynet
docker run -d --network=mynet --name app1 nginx
docker run -d --network=mynet --name app2 nginx
```
- [ ] Write a docker-compose.yml for a multi-container app
```
version: "3"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
  db:
    image: mongo
    volumes:
      - data:/data/db

volumes:
  data:
```
- [ ] Use environment variables and .env files in Compose
```
.env file:
PORT=3000

docker-compose.yml:
services:
  app:
    build: .
    environment:
      - PORT=${PORT}
```
- [ ] Write a multi-stage Dockerfile
```
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm install

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app .
CMD ["npm", "start"]
```
- [ ] Push an image to Docker Hub
```
docker login
docker tag myapp username/myapp
docker push username/myapp
```
- [ ] Use healthchecks and depends_on
```
services:
  app:
    build: .
    depends_on:
      - db
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000"]
      interval: 30s
      retries: 3

  db:
    image: mongo
```
---

## Quick-Fire Questions
Answer from memory, then verify:
1. What is the difference between an image and a container?
```
Image = blueprint (read-only)  
Container = running instance of that image 
```
2. What happens to data inside a container when you remove it?
```
Data is Deleted unless stored in volumes or bind mounts 
```
3. How do two containers on the same custom network communicate?
```
They use container names as hostnames
```
4. What does `docker compose down -v` do differently from `docker compose down`?
```
`down` → removes containers + network  
`down -v` → also removes volumes (data loss) 
```
5. Why are multi-stage builds useful?
```
Smaller images, cleaner, no unnecessary build tools  
```
6. What is the difference between `COPY` and `ADD`?
```
COPY = simple file copy  
ADD = copy + extra features (auto extract, URLs)
```
7. What does `-p 8080:80` mean?
```
mapping Host port 8080 → Container port 80  
```
8. How do you check how much disk space Docker is using?
```
docker system df
```
---

## Build Your Docker Cheat Sheet
Create `docker-cheatsheet.md` organized by category:
- **Container commands** — run, ps, stop, rm, exec, logs
- **Image commands** — build, pull, push, tag, ls, rm
- **Volume commands** — create, ls, inspect, rm
- **Network commands** — create, ls, inspect, connect
- **Compose commands** — up, down, ps, logs, build
- **Cleanup commands** — prune, system df
- **Dockerfile instructions** — FROM, RUN, COPY, WORKDIR, EXPOSE, CMD, ENTRYPOINT

Keep it short — one line per command, something you'd actually reference on the job.

---
