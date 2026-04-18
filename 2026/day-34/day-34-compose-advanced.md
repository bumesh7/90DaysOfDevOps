### Task 1: Build Your Own App Stack
Create a `docker-compose.yml` for a 3-service stack:
- A **web app** (use Python Flask, Node.js, or any language you know)
- A **database** (Postgres or MySQL)
- A **cache** (Redis)
```
app/
 ├── app.py
 ├── requirements.txt
 └── Dockerfile
docker-compose.yml
```
```
$ vim Dockerfile

FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["gunicorn", "-b", "0.0.0.0:5000", "app:app"]
```
```
$ vim docker-compose.yml

services:
  web:
    build: ./app
    container_name: flask_app
    ports:
      - "5000:5000"
    depends_on:
      - db
      - cache
    environment:
      - DB_HOST=db
      - DB_NAME=mydb
      - DB_USER=postgres
      - DB_PASSWORD=postgres
      - REDIS_HOST=cache

  db:
    image: postgres:15
    container_name: postgres_db
    restart: always
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

  cache:
    image: redis:7
    container_name: redis_cache
    ports:
      - "6379:6379"

volumes:
  db_data:
```
```
$ vim app/app.py

from flask import Flask
import psycopg2
import redis
import os

app = Flask(__name__)

def get_db_connection():
    conn = psycopg2.connect(
        host=os.getenv("DB_HOST"),
        database=os.getenv("DB_NAME"),
        user=os.getenv("DB_USER"),
        password=os.getenv("DB_PASSWORD")
    )
    return conn

def get_redis_connection():
    r = redis.Redis(host=os.getenv("REDIS_HOST"), port=6379)
    return r

@app.route("/")
def hello():
    try:
        # DB check
        conn = get_db_connection()
        cur = conn.cursor()
        cur.execute("SELECT 1;")
        cur.close()
        conn.close()
        db_status = "Connected ✅"
    except Exception as e:
        db_status = f"Failed ❌ ({e})"

    try:
        # Redis check
        r = get_redis_connection()
        r.set("test", "Hello Redis")
        redis_status = r.get("test").decode()
    except Exception as e:
        redis_status = f"Failed ❌ ({e})"

    return f"""
    <h1>Hello from Flask 🚀</h1>
    <p>Postgres: {db_status}</p>
    <p>Redis: {redis_status}</p>
    """

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
```
$ vim rewuirements.txt

flask
psycopg2-binary
redis
gunicorn
```
```
$ docker compose up --build

http://localhost:5000
```
Write a simple Dockerfile for the web app. The app doesn't need to be complex — even a "Hello World" that connects to the database is enough.
<img width="480" height="270" alt="image" src="https://github.com/user-attachments/assets/370a4db8-f437-42d7-b5b1-d870e2bef44c" />

```
$ docker compose down --rmi all --volumes --remove-orphans
```
---

### Task 2: depends_on & Healthchecks
1. Add `depends_on` to your compose file so the app starts **after** the database
2. Add a **healthcheck** on the database service
3. Use `depends_on` with `condition: service_healthy` so the app waits for the database to be truly ready, not just started
```
version: "3.9"

services:
  web:
    build: ./app
    container_name: flask_app
    ports:
      - "5000:5000"
    depends_on:
      db:
        condition: service_healthy   # 🔥 wait until DB is ready
      cache:
        condition: service_started
    environment:
      - DB_HOST=db
      - DB_NAME=mydb
      - DB_USER=postgres
      - DB_PASSWORD=postgres
      - REDIS_HOST=cache

  db:
    image: postgres:15
    container_name: postgres_db
    restart: always
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

    # ✅ Healthcheck for Postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d mydb"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 5s

  cache:
    image: redis:7
    container_name: redis_cache
    ports:
      - "6379:6379"

volumes:
  db_data:
```
**Test:** Bring everything down and up — does the app wait for the DB?
```
$ docker compose up --build -d
$ docker compsoe logs -f
$ docker compose down --rmi all --volumes --remove-orphans
```
---

### Task 3: Restart Policies
1. Add `restart: always` to your database service
```
db:
  image: postgres:15
  container_name: postgres_db
  restart: always   # 👈 add this
  environment:
    POSTGRES_DB: mydb
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
```
2. Manually kill the database container — does it come back?
```
$ docker ps
$ docker exec -it postgres_db kill 1
$ docker ps
```
<img width="1850" height="458" alt="image" src="https://github.com/user-attachments/assets/0a6a28b8-71b3-49db-a16a-962d7c62d0ec" />

3. Try `restart: on-failure` — how is it different?
```
Policy	Behavior
always	Restarts no matter what (crash, stop, reboot)
on-failure	Restarts only if app crashes (non-zero exit)
```
```
👉 on-failure

Restarts only on internal crashes
Ignores manual stop/kill
Based on exit code

👉 always

Restarts on crash + daemon restart
May not restart if manually stopped via Compose
```
4. Write in your notes: When would you use each restart policy?
```
🔹 restart: always

Use when:

Critical services (DB, backend APIs)
You want maximum uptime
Even if container is manually stopped or Docker restarts

👉 Example:

Postgres
Redis
Backend services
🔹 restart: on-failure

Use when:

You only want restart on crashes
You don’t want restart after manual stop
Useful for batch jobs / workers

👉 Example:

ETL jobs
Background workers
One-time scripts
```
---

### Task 4: Custom Dockerfiles in Compose
1. Instead of using a pre-built image for your app, use `build:` in your compose file to build from a Dockerfile
2. Make a code change in your app
```
update you app.py code
```
3. Rebuild and restart with one command

<img width="1850" height="458" alt="image" src="https://github.com/user-attachments/assets/a5dbad13-cd76-49dc-b2fd-2c6ef3e3ff73" />

---

### Task 5: Named Networks & Volumes
1. Define **explicit networks** in your compose file instead of relying on the default
2. Define **named volumes** for database data
3. Add **labels** to your services for better organization
```
services:
  web:
    build: ./app
    container_name: flask_app
    ports:
      - "5000:5000"
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    environment:
      - DB_HOST=db
      - DB_NAME=mydb
      - DB_USER=postgres
      - DB_PASSWORD=postgres
      - REDIS_HOST=cache
    networks:
      - frontend
      - backend
    labels:
      - "app=flask"
      - "tier=frontend"
      - "env=dev"

  db:
    image: postgres:15
    container_name: postgres_db
    restart: always
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend
    labels:
      - "app=postgres"
      - "tier=database"
      - "env=dev"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d mydb"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 5s

  cache:
    image: redis:7
    container_name: redis_cache
    networks:
      - backend
    labels:
      - "app=redis"
      - "tier=cache"
      - "env=dev"

# ✅ Named Volume
volumes:
  db_data:
    name: postgres_data_volume

# ✅ Explicit Networks
networks:
  frontend:
    name: app_frontend_network
  backend:
    name: app_backend_network
```
```
$ docker network ls
$ docker volume ls
$ docker network inspect app_backend_network
```
<img width="1145" height="490" alt="image" src="https://github.com/user-attachments/assets/a40defe6-a845-4168-ab6e-104951678983" />
<img width="1119" height="390" alt="image" src="https://github.com/user-attachments/assets/69c4916d-acf9-4da7-a32c-93aa1b576d4f" />
<img width="1093" height="188" alt="image" src="https://github.com/user-attachments/assets/35f542e9-1922-43e7-96c9-b46152e64275" />

---

### Task 6: Scaling (Bonus)
1. Try scaling your web app to 3 replicas using `docker compose up --scale`
```
$ docker compose up -d --scale web=3
```
<img width="1855" height="158" alt="image" src="https://github.com/user-attachments/assets/b88c0f2b-33ad-4075-a365-758cfa92d0fa" />
<img width="1855" height="208" alt="image" src="https://github.com/user-attachments/assets/b7a2aca6-3abc-415c-ba7e-8260e0d22954" />

2. What happens? What breaks?
```
What breaks when scaling?

Custom container names
Prevent multiple instances
Must be removed for scaling

Port mapping

Multiple containers cannot bind same host port
```
3. Write in your notes: Why doesn't simple scaling work with port mapping?
```
Because:

Multiple containers cannot bind to the same host port
Multiple containers cannot have custome container names
```
---
