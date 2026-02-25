Task 1: Your First Dockerfile
Create a folder called my-first-image
$ mkdir my-first-image

Inside it, create a Dockerfile that:
$ vim Dockerfile

Uses ubuntu as the base image

FROM ubuntu:latest

# Avoid User input
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y curl

CMD ["echo", "Hello from my custom image!"]
<img width="722" height="223" alt="image" src="https://github.com/user-attachments/assets/37bc83fc-49ac-45de-96ae-8bf441bf3daf" />


Installs curl
Sets a default command to print "Hello from my custom image!"
Build the image and tag it my-ubuntu:v1  ($ docker build -t my-ubuntu:v1 .)
<img width="1271" height="603" alt="image" src="https://github.com/user-attachments/assets/96c16128-0aee-496b-be0a-6ee739313dbf" />

Run a container from your image
Verify: The message prints on docker run  ($ docker run --rm my-ubuntu:v1)
<img width="825" height="137" alt="image" src="https://github.com/user-attachments/assets/7755e420-6636-4f0a-a532-dd03c7954599" />


Task 2: Dockerfile Instructions
Create a new Dockerfile that uses all of these instructions:

FROM — base image
RUN — execute commands during build
COPY — copy files from host to image
WORKDIR — set working directory
EXPOSE — document the port
CMD — default command
Build and run it. Understand what each line does.

Task 3: CMD vs ENTRYPOINT
Create an image with CMD ["echo", "hello"] — run it, then run it with a custom command. What happens?
Create an image with ENTRYPOINT ["echo"] — run it, then run it with additional arguments. What happens?
Write in your notes: When would you use CMD vs ENTRYPOINT?
Task 4: Build a Simple Web App Image
Create a small static HTML file (index.html) with any content
Write a Dockerfile that:
Uses nginx:alpine as base
Copies your index.html to the Nginx web directory
Build and tag it my-website:v1
Run it with port mapping and access it in your browser
Task 5: .dockerignore
Create a .dockerignore file in one of your project folders
Add entries for: node_modules, .git, *.md, .env
Build the image — verify that ignored files are not included
Task 6: Build Optimization
Build an image, then change one line and rebuild — notice how Docker uses cache
Reorder your Dockerfile so that frequently changing lines come last
Write in your notes: Why does layer order matter for build speed?
