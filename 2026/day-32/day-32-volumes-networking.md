1.	Run a Postgres or MySQL container
$ docker run --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=secret \
  -d mysql:8
  	<img width="940" height="404" alt="image" src="https://github.com/user-attachments/assets/54e9b310-4e84-4fbe-b209-34ade2d481b2" />

 
3.	Create some data inside it (a table, a few rows — anything)
$ docker exec -it mysql-test mysql -uroot -psecret
CREATE DATABASE demo;
USE demo;
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);
INSERT INTO users (name) VALUES ('Alice'), ('Bob');
SELECT * FROM users;
 <img width="940" height="605" alt="image" src="https://github.com/user-attachments/assets/7743bbd8-df79-4465-a770-4d70d3872a5d" />

4.	Stop and remove the container
$ docker stop mysql-test
$ docker rm mysql-test

6.	Run a new one — is your data still there?
$ docker run --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=secret \
  -d mysql:8

$ docker exec -t mysql-test mysql -uroot -psecret

<img width="940" height="622" alt="image" src="https://github.com/user-attachments/assets/082784d2-f23b-42ca-8afc-49eec34b1ed4" />

 
Write what happened and why.
The previously created data is lost because volume is not attached to it.


Task 2: Named Volumes
1.	Create a named volume
$ docker volume create mysqldata
$ docker volume ls
2.	Run the same database container, but this time attach the volume to it
$ docker run –-name mysql-test \
-e MYSQL_ROOT_PASSWORD=secret \
-v mysqldata:/var/lib/mysql \
-d mysql:8
  
3.	Add some data, stop and remove the container
$ docker exec -it mysql-test mysql -uroot -psecret
CREATE DATABASE demo;
USE demo;
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);
INSERT INTO users (name) VALUES ('Alice'), ('Bob');
SELECT * FROM users;
4.	Run a brand new container with the same volume
$ docker run --name mysql-testing \
 -e MYSQL_ROOT_PASSWORD=secret \ 
 -v mysqldata:/var/lib/mysql \
 -d mysql:8
 <img width="940" height="559" alt="image" src="https://github.com/user-attachments/assets/e287496a-ce72-4e8f-b94b-ca8e2fb5020c" />

5.	Is the data still there?
Yes data is present because the same volume is attached this time and data is stored in volume.
Verify: docker volume ls, docker volume inspect

Task 3: Bind Mounts
1.	Create a folder on your host machine with an index.html file
$ mkdir nginx-bind
$ cd nginx-bind
2.	Run an Nginx container and bind mount your folder to the Nginx web directory
$ vim index.html
<h1>Hi</h1>
$ docker run -d \
 --name nginxbind \
--p 8999:80 \
 -v $(pwd):/usr/share/nginx/html/ \
 nginx

3.	Access the page in your browser
   <img width="940" height="155" alt="image" src="https://github.com/user-attachments/assets/5b3d1e8a-89b5-415d-af97-c7c82eefd146" />
 
5.	Edit the index.html on your host — refresh the browser
$ vim index.html
<h1>Hi</h1>
 <img width="940" height="180" alt="image" src="https://github.com/user-attachments/assets/1922e963-f83d-414b-857a-7efdb45fe162" />
 
Write in your notes: What is the difference between a named volume and a bind mount?
As the volume is bind to PWD the content updated will be automatically reflected in browser.

Task 4: Docker Networking Basics
1.	List all Docker networks on your machine
$ docker network ls
<img width="940" height="167" alt="image" src="https://github.com/user-attachments/assets/5666f631-2ee5-4ee8-aa56-d974965c555e" />
 
3.	Inspect the default bridge network
$ docker network inspect bridge
4.	Run two containers on the default bridge — can they ping each other by name?
$ docker run -dit -–name c1 alpine sh
$ docker run -dit -–name c2 alpine sh
$ docker exec -it c1 sh
apk add iputils
ping c2

<img width="940" height="442" alt="image" src="https://github.com/user-attachments/assets/16ec517a-095a-4e27-8021-d3ef246cfd7e" />

->We cant ping each other container via name 

6.	Run two containers on the default bridge — can they ping each other by IP?
$ docker inspect c2 | grep IPAddress
$ docker ecex -it c1 sh
ping 172.17.0.5
<img width="940" height="387" alt="image" src="https://github.com/user-attachments/assets/5fd23aea-a1d4-43b2-8c2d-5a2a2c84665a" />

Task 5: Custom Networks

1.	Create a custom bridge network called my-app-net
$ docker network create my-app-nw
$ docker network ls
<img width="940" height="202" alt="image" src="https://github.com/user-attachments/assets/3dca99e2-0d32-4998-8ee6-a1547ed9b9f2" />

3.	Run two containers on my-app-net
$ docker run -dit --name app1 --network my-app-net alpine sh 
$ docker exec -it app1 sh
apk add iputils
$ docker run -dit --name app2 --network my-app-net alpine sh
$ docker exec -it app2 sh
apk add iputils
ping app1
 <img width="940" height="327" alt="image" src="https://github.com/user-attachments/assets/fc015a62-24b1-4e69-b69f-ccf8baa834be" />

Can they ping each other by name now?
<img width="940" height="470" alt="image" src="https://github.com/user-attachments/assets/0a52f0c2-3849-4df0-bc64-cbecc13abeb4" />

3.	Write in your notes: Why does custom networking allow name-based communication but the default bridge doesn't?
Default bridge don’t have embedded DNS, container can communicate only via IP.
User-define or custome bridge has inbuilt DNS so docker register container names and hence container can communicate via names. 

Task 6: Put It Together
1.	Create a custom network
$ docker network create custom-network
$ docker volume create custom-volume

<img width="940" height="113" alt="image" src="https://github.com/user-attachments/assets/c56dc1e5-b324-48f8-a929-97b3e6f7c766" />

2.	Run a database container (MySQL/Postgres) on that network with a volume for data
$ docker run -d \
--name my-postgres \
--network custom-network \
-e POSTGRES_USER=myusername \
-e POSTGRES_PASSWORD=mypassword \
-e POSTGRES_DB=mydb \
-v custom-volume:/var/lib/postgresql/data \
postgres:15
<img width="800" height="284" alt="image" src="https://github.com/user-attachments/assets/e156cfb4-73c6-4692-8cfd-0939b70a2af5" />
 
3.	Run an app container (use any image) on the same network
$ docker run -it --name my-app \
--network custom-network \
alpine:latest sh
<img width="940" height="413" alt="image" src="https://github.com/user-attachments/assets/715e7aa7-d439-464a-b4bc-f0d3002c6fce" />
  
6.	Verify the app container can reach the database by container name

$ apk add --no-cache postgresql-client

$ psql -h my-postgres -U myuser -d mydb

Enter Password = mypassword

#mydb
<img width="1518" height="667" alt="image" src="https://github.com/user-attachments/assets/564b0262-4a44-4a4f-b602-6ccc57427737" />

