1.	Run a Postgres or MySQL container
$ docker run --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=secret \
  -d mysql:8
 
2.	Create some data inside it (a table, a few rows — anything)
$ docker exec -it mysql-test mysql -uroot -psecret
CREATE DATABASE demo;
USE demo;
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);
INSERT INTO users (name) VALUES ('Alice'), ('Bob');
SELECT * FROM users;
 
3.	Stop and remove the container
$ docker stop mysql-test
$ docker rm mysql-test
4.	Run a new one — is your data still there?
$ docker run --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=secret \
  -d mysql:8

$ docker exec -t mysql-test mysql -uroot -psecret

 
Write what happened and why.
The previously created data is lost because volume is not attached to it.

\

________________________________________
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
 
5.	Is the data still there?
Yes data is present because the same volume is attached this time and data is stored in volume.
Verify: docker volume ls, docker volume inspect
________________________________________
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
 
4.	Edit the index.html on your host — refresh the browser
$ vim index.html
<h1>Hi</h1>
 

Write in your notes: What is the difference between a named volume and a bind mount?
As the volume is bind to PWD the content updated will be automatically reflected in browser.


________________________________________

Task 4: Docker Networking Basics
1.	List all Docker networks on your machine
$ docker network ls
 
2.	Inspect the default bridge network
$ docker network inspect bridge
3.	Run two containers on the default bridge — can they ping each other by name?
$ docker run -dit -–name c1 alpine sh
$ docker run -dit -–name c2 alpine sh
$ docker exec -it c1 sh
# apk add iputils
# ping c2
	We cant ping each other container via name 
 
4.	Run two containers on the default bridge — can they ping each other by IP?
$ docker inspect c2 | grep IPAddress
$ docker ecex -it c1 sh
# ping 172.17.0.5
 
________________________________________
Task 5: Custom Networks
1.	Create a custom bridge network called my-app-net
$ docker network create my-app-nw
$ docker network ls
 
2.	Run two containers on my-app-net
$ docker run -dit --name app1 --network my-app-net alpine sh 
$ docker exec -it app1 sh
# apk add iputils
$ docker run -dit --name app2 --network my-app-net alpine sh
$ docker exec -it app2 sh
# apk add iputils
# ping app1
 
Can they ping each other by name now?
 
3.	Write in your notes: Why does custom networking allow name-based communication but the default bridge doesn't?
Default bridge don’t have embedded DNS, container can communicate only via IP.
User-define or custome bridge has inbuilt DNS so docker register container names and hence container can communicate via names. 
________________________________________
Task 6: Put It Together
1.	Create a custom network
$ docker network create custom-network
$ docker volume create custom-volume
 
2.	Run a database container (MySQL/Postgres) on that network with a volume for data
$ docker run -d \
--name my-postgres \
--network custom-network \
-e POSTGRES_USER=myusername \
-e POSTGRES_PASSWORD=mypassword \
-e POSTGRES_DB=mydb \
-v custom-volume:/var/lib/postgresql/data \
postgres:15

 
3.	Run an app container (use any image) on the same network
$ docker run -it --name my-app \
--network custom-network \
alpine:latest sh


  
4.	Verify the app container can reach the database by container name

$ apk add --no-cache postgresql-client

$ psql -h my-postgres -U myuser -d mydb

# Enter Password = mypassword

#mydb

