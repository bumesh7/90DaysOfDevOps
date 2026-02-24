Task 1: What is Docker?
Research and write short notes on:

What is a container and why do we need them?
-> A Docker container is a lightweight, standalone, executable package that includes everything needed to run an application.
-> It contains code, dependencies, Runtime(Java, Python etc), Libraries.

Containers vs Virtual Machines — what's the real difference?
-> Container 
1) Uses Host OS
2) Light Weight
3) Less Secure as it uses shared OS kernel
4) Fast
   
-> Virtual Machine
1) Independent Operating System
2) Heavy
3) High Security
4) Slow
 
What is the Docker architecture? (daemon, client, images, containers, registry)
Docker = It is a container platform that allows you to build container images, run containers, share images.

Docker Architecture
1) Docker CLI => It is used to interact and run commands and send requests to docker daemon. ex docker run, docker build
2) Docker Daemon => It listens API request from CLI, Daemon can build images, run containers, manage volumes, manage networks.
3) Docker Images => It is a blueprint used to create the containers, it is read only templates.
4) Docker Containers => Running instance of images.
5) Docker Registry => Location where all the images are stored.

Image  = Class
Container = Object

Client sends request to daemon
Daemon checks local images
If not found then pulls image from Docker Hub
Creates container from image
Starts container
App runs inside isolated process


Draw or describe the Docker architecture in your own words.

Docker CLI -> Docker Daemon -> Docker Image -> Docker Container 
                                  |-> Docker Registry

Task 2: Install Docker

Install Docker on your machine (or use a cloud instance)
Verify the installation
<img width="802" height="310" alt="image" src="https://github.com/user-attachments/assets/9240b568-b98e-415c-be16-92d7bd280a34" />

Run the hello-world container
Read the output carefully — it explains what just happened
<img width="1875" height="743" alt="image" src="https://github.com/user-attachments/assets/433a3481-c52c-4a15-ad10-cc2302931b3f" />

Client sends request to daemon
Daemon checks local images
If not found then pulls image from Docker Hub
Creates container from image
Starts container


Task 3: Run Real Containers

Run an Nginx container and access it in your browser
<img width="767" height="43" alt="image" src="https://github.com/user-attachments/assets/0f99f148-3617-489b-9547-6e5ac2fe8ab9" />

Run an Ubuntu container in interactive mode — explore it like a mini Linux machine


List all running containers
List all containers (including stopped ones)
Stop and remove a container
<img width="1502" height="487" alt="image" src="https://github.com/user-attachments/assets/48e70c11-8111-49c8-aacd-bbca0debe4a5" />
<img width="1071" height="797" alt="image" src="https://github.com/user-attachments/assets/e522cc83-5e32-40d7-a352-4e4a87505ecb" />


Task 4: Explore

Run a container in detached mode — what's different?  -> It will execute in the background and the details will not be shown.
Give a container a custom name
<img width="1360" height="625" alt="image" src="https://github.com/user-attachments/assets/c9bfb94e-7c37-4c48-ba36-1bfb30f02e99" />

Map a port from the container to your host
<img width="736" height="52" alt="image" src="https://github.com/user-attachments/assets/e36ec53b-fe53-4a05-a9c1-7bc1b805a0f3" />

Check logs of a running container
<img width="1918" height="602" alt="image" src="https://github.com/user-attachments/assets/c25b1325-bead-4b31-80b8-d8a991c3adf9" />


Run a command inside a running container

<img width="1606" height="153" alt="image" src="https://github.com/user-attachments/assets/bda42da6-16fc-45a2-b42e-bf95470f1776" />

