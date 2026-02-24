Task 1: Docker Images
Pull the nginx, ubuntu, and alpine images from Docker Hub
List all images on your machine — note the sizes
<img width="922" height="447" alt="image" src="https://github.com/user-attachments/assets/68a7a7c6-0459-4bc1-ab68-0ded1f89001b" />

nginx -> 161mb
ubuntu -> 78.1mb
alpine -> 8.44mb

Compare ubuntu vs alpine — why is one much smaller?

Ubuntu is larger beacause it consists of entire OS and package manager and libraries.
Alpine is smaller because it has minimal base system and few packages.

Inspect an image — what information can you see?
$ docker image inspect nginx
1) Image ID
2) Creation date
3) Architecture (amd64, arm64)
4) OS
5) Environment variables
6) Default command (CMD)
7) Entrypoint
8) Exposed ports
9) Layers
10) Working directory

Remove an image you no longer need

<img width="1022" height="335" alt="image" src="https://github.com/user-attachments/assets/95119de7-0d8f-4c64-b289-7152c0c2c75f" />

Task 2: Image Layers

Run docker image history nginx — what do you see?
<img width="1282" height="472" alt="image" src="https://github.com/user-attachments/assets/1e279c94-cecd-4a79-bd70-55fcc32e61d9" />

Each line is a layer. Note how some layers show sizes and some show 0B
-> Each row is one seperate layer. Some layers as 0mb and some has more mb because of the Meta Data and File system change.
-> Each layer represents a step in the image build process.

Write in your notes: What are layers and why does Docker use them?
-> Meta Data Layer like ENV, CMD, RUN, Expose doesn't require of adding files it just changes the configuration.
-> File system layer like COPY, ADD actually add data to the image.

=> A layer is a read-only filesystem change created by a single Dockerfile instruction.

1) Each instruction creates a new layer
2) Layers are stacked on top of each other
3) Final image = combination of all layers

Task 3: Container Lifecycle
Practice the full lifecycle on one container:

Create a container (without starting it)  $ docker create --name lifycycle nginx (Created but wont start container)
<img width="1158" height="403" alt="image" src="https://github.com/user-attachments/assets/c1b5655a-3bb4-4f5c-b4ac-7675e884c482" />

Start the container   $ docker start lifecycle  (start the created nginx container)
<img width="1250" height="185" alt="image" src="https://github.com/user-attachments/assets/1eecfed5-9163-4a3b-b3ba-48d12899ea4a" />

Pause it and check status  $ docker pause lifecycle  (Pause the container) Freezes container processes, memory is allocated and CPU reduces to 0.
<img width="1307" height="155" alt="image" src="https://github.com/user-attachments/assets/daa32698-bc0b-4d65-8480-07127e4df1b6" />

Unpause it  $ docker unpause lifecycle  (Resumes the container)
<img width="1165" height="150" alt="image" src="https://github.com/user-attachments/assets/552e136f-fbc5-4b30-8b40-3752c2e3190b" />

Stop it $ docker stop lifecycle  (Stop does Graceful Shutdown)
Graceful shutdown => safe termination process for OS/software and allowing active program to save data and finish pending task and close connection before shutdown.
<img width="1278" height="147" alt="image" src="https://github.com/user-attachments/assets/e3adae5f-d084-47ee-99a0-bec874ca1d02" />

Restart it  $ docker restart lifecycle (Restart = Stop + Start)
<img width="1207" height="147" alt="image" src="https://github.com/user-attachments/assets/e6a9c56b-cc77-4e7f-97b9-c5ce35e94d2f" />

Kill it  $ docker kill lifecycle  (Kills immediately with code 137 and no graceful shutdown)
<img width="1343" height="138" alt="image" src="https://github.com/user-attachments/assets/162a25c4-0a79-4984-961a-95f84b45bc99" />

Remove it  $ docker rm lifecycle  (Container is removed from the list)
<img width="827" height="125" alt="image" src="https://github.com/user-attachments/assets/ebdd4b3b-cf70-46d1-b759-83af60397899" />

create  →  Created
start   →  Running
pause   →  Paused
unpause →  Running
stop    →  Exited (0) Graceful shutdown
restart →   stop + Start => Running
kill    →  Exited (137) No Graceful Shutdown
rm      →  Removed

Check docker ps -a after each step — observe the state changes.

Task 4: Working with Running Containers
Run an Nginx container in detached mode  
$ docker run -d --name nginxdemo -p 8000:80 nginx

View its logs
$ docker logs nginxdemo

View real-time logs (follow mode)
$ docker logs -f nginxdemo

Exec into the container and look around the filesystem
$ docker exec -it nginxdemo bash
root# ls /
root# ls /etc/nginx
root# ls /usr/share/nginx/html
root# cat /usr/share/nginx/html/index.html

Run a single command inside the container without entering it
$ docker exec nginxdemo ls /etc/nginx 

Inspect the container — find its IP address, port mappings, and mounts
$ docker inspect nginxdemo

Task 5: Cleanup

Stop all running containers in one command
$ docker stop $(docker ps -q) 
<img width="675" height="115" alt="image" src="https://github.com/user-attachments/assets/e8999223-baac-43bc-aa46-5179c7e5baae" />

Remove all stopped containers in one command
$ docker container prune
<img width="898" height="206" alt="image" src="https://github.com/user-attachments/assets/b2f41407-2a5d-49c0-84db-404fde2f482b" />

Remove unused images

$ docker image prune -a -f => Remove all unused images (not used by any container) 
<img width="1056" height="432" alt="image" src="https://github.com/user-attachments/assets/ca2ccedf-90bf-45ea-87e8-9140e1210fda" />

Check how much disk space Docker is using
$ docker system df
<img width="702" height="172" alt="image" src="https://github.com/user-attachments/assets/38a71682-f6c3-41cb-a9aa-340eac994782" />

