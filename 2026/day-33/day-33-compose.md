Task 1: Install & Verify

Check if Docker Compose is available on your machine
$ sudo apt install docker-compose

Verify the version
$ docker-compose --version

Task 2: Your First Compose File

Create a folder compose-basics
$ mkdir compose-file
$ cd compose-fil

Write a docker-compose.yml that runs a single Nginx container with port mapping
$ vim dokcer-compose.yml

services:
  nginx:
    image: nginx:latest
    container_name: nginx-demo
    ports:
      - "5555:80"

Start it with docker compose up
$ docker-compose up

Access it in your browser
<img width="967" height="390" alt="image" src="https://github.com/user-attachments/assets/ab76c5d3-7055-4239-972b-b52ed3b80c87" />


Stop it with docker compose down
$ docker-compose down

Task 3: Two-Container Setup
Write a docker-compose.yml that runs:

A WordPress container
A MySQL container
They should:

Be on the same network (Compose does this automatically)
MySQL should have a named volume for data persistence
WordPress should connect to MySQL using the service name
Start it, access WordPress in your browser, and set it up.

services:
  mysql:
    image: mysql:8
    container_name: wordpress-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword 
      MYSQL_USER: myuser
      MYSQL_PASSWORD: mypassword
      MYSQL_DATABASE: mydb
    volumes:
      - wb-data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    container_name: wordpress-app
    depends_on:
      - mysql
    restart: always
    ports:
      - "8888:80"
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: myuser
      WORDPRESS_DB_PASSWORD: mypassword
      WORDPRESS_DB_NAME: mydb

volumes:
  wb-data:

$ docker-compose up -d
$ docker-compose dowm -v

<img width="1313" height="947" alt="image" src="https://github.com/user-attachments/assets/b0986ea9-aa31-4ee3-be2d-b3f5dc4059e8" />


Verify: Stop and restart with docker compose down and docker compose up — is your WordPress data still there?
shows blog page not setup page.
<img width="1517" height="562" alt="image" src="https://github.com/user-attachments/assets/d7c293b5-32f4-435c-b504-ca058eb08df9" />





Task 4: Compose Commands
Practice and document these:

Start services in detached mode
View running services
View logs of all services
View logs of a specific service
Stop services without removing
Remove everything (containers, networks)
Rebuild images if you make a change

Task 5: Environment Variables
Add environment variables directly in your docker-compose.yml
Create a .env file and reference variables from it in your compose file
Verify the variables are being picked up
