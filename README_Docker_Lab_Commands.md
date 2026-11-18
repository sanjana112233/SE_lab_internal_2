# Docker Lab Commands

Commands are in execution order. Run them in **Git Bash** unless mentioned otherwise.

## 1. Check Docker

```bash
docker --version
docker version
docker info
docker ps
```

## 2. Single Container – Redis

```bash
docker pull redis
docker run --name my-redis -d redis
docker ps
docker ps -a
docker exec -it my-redis redis-cli
```

Inside Redis CLI:

```text
SET name "Alice"
GET name
exit
```

Stop, start and remove:

```bash
docker stop my-redis
docker start my-redis
docker stop my-redis
docker rm my-redis
docker rmi redis
```

## 3. Create a Dockerfile and Build an Image

```bash
mkdir Redis
cd Redis
touch Dockerfile
code Dockerfile
```

Paste into `Dockerfile`:

```dockerfile
FROM redis:latest
CMD ["redis-server"]
```

Build and run:

```bash
docker build -t redisnew .
docker images
docker run --name myredisnew -d redisnew
docker ps
```

## 4. Push Image to Docker Hub

```bash
docker login -u sanjanaaavarma
docker ps -a
docker commit myredisnew sanjanaaavarma/redis1:latest
docker images
docker push sanjanaaavarma/redis1:latest
docker logout
```

Pull and run the pushed image:

```bash
docker pull sanjanaaavarma/redis1:latest
docker run --name redis-from-hub -d sanjanaaavarma/redis1:latest
docker exec -it redis-from-hub redis-cli
```

Inside Redis CLI:

```text
SET course "Docker"
GET course
exit
```

Cleanup:

```bash
docker stop redis-from-hub
docker rm redis-from-hub
docker stop myredisnew
docker rm myredisnew
docker rmi redisnew
```

## 5. Useful Single-Container Commands

```bash
docker ps
docker ps -a
docker images
docker logs CONTAINER_NAME
docker exec -it CONTAINER_NAME sh
docker stop CONTAINER_NAME
docker start CONTAINER_NAME
docker restart CONTAINER_NAME
docker rm CONTAINER_NAME
docker rmi IMAGE_NAME
docker stats
docker port CONTAINER_NAME
docker tag IMAGE_NAME USERNAME/IMAGE_NAME:latest
```

Run a container with a port:

```bash
docker run -d --name myapi -p 8080:5000 myapi
```

Save an image:

```bash
docker save -o myimage.tar IMAGE_NAME
```

## 6. Docker Compose – Nginx and PostgreSQL

```bash
mkdir compose-lab
cd compose-lab
touch docker-compose.yml
code docker-compose.yml
```

Paste into `docker-compose.yml`:

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8081:80"

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
```

Run and check:

```bash
docker compose up -d
docker compose ps
docker ps
```

Open: `http://localhost:8081`

## 7. Add Redis to Compose

Use this complete `docker-compose.yml`:

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8081:80"
    depends_on:
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db

  redis:
    image: redis:alpine
```

Commands:

```bash
docker compose up -d
docker compose ps
docker compose logs
```

## 8. Compose Network and Volume

Use this complete `docker-compose.yml`:

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8081:80"
    networks:
      - app-net
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-net

networks:
  app-net:

volumes:
  db-data:
```

Commands:

```bash
docker compose up -d
docker compose ps
docker network ls
docker volume ls
docker compose down
docker compose up -d
```

PostgreSQL terminal (optional):

```bash
docker compose exec db psql -U demo -d demo_db
```

Inside PostgreSQL:

```sql
CREATE TABLE test_data (id SERIAL PRIMARY KEY, item TEXT);
INSERT INTO test_data (item) VALUES ('reservation');
SELECT * FROM test_data;
\q
```

## 9. Flask with Docker Compose

Create `app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Flask + Docker!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

Create `Dockerfile`:

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY app.py /app/
RUN pip install flask
CMD ["python", "app.py"]
```

Use this `docker-compose.yml`:

```yaml
services:
  web:
    build: .
    ports:
      - "5000:5000"
```

Run/rebuild:

```bash
docker compose up --build
```

Open: `http://localhost:5000`

## 10. SET-7 – Metro Reservation + MySQL

Clone and build the project:

```bash
git clone https://github.com/deepthisagar7/metro-reservation.git
cd metro-reservation
mvn clean package
ls target
```

Pull images:

```bash
docker pull tomcat:9
docker pull mysql:8
```

Create `Dockerfile` in the project folder:

```dockerfile
FROM tomcat:9
RUN rm -rf /usr/local/tomcat/webapps/*
COPY target/metro-reservation.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
```

Build app image:

```bash
docker build -t metro-reservation:latest .
```

Create network and volume:

```bash
docker network create metro-net
docker volume create metro-mysql-data
```

Run MySQL:

```bash
docker run -d --name mysql --network metro-net -e MYSQL_ROOT_PASSWORD=RootPass123! -e MYSQL_DATABASE=metrodb -e MYSQL_USER=metro_user -e MYSQL_PASSWORD=MetroPass123! -v metro-mysql-data:/var/lib/mysql mysql:8
```

Run application:

```bash
docker run -d --name metro-app --network metro-net -p 8080:8080 metro-reservation:latest
```

Check:

```bash
docker ps
docker logs metro-app
docker logs mysql
docker network inspect metro-net
```

Open: `http://localhost:8080`

MySQL terminal:

```bash
docker exec -it mysql mysql -u metro_user -p metrodb
```

Cleanup:

```bash
docker stop metro-app mysql
docker rm metro-app mysql
docker network rm metro-net
```

Keep `metro-mysql-data` if database data is needed. Delete it only when you want to erase the data:

```bash
docker volume rm metro-mysql-data
```

**Important:** The Java project must also be configured to connect to `mysql:3306` with the database name and credentials above. The Docker commands alone do not configure the app's JDBC settings or database tables.

## 11. More Compose Commands

```bash
docker compose up -d
docker compose up --build
docker compose ps
docker compose logs
docker compose restart
docker compose down
docker compose build frontend
docker compose up -d --no-deps frontend
docker compose exec SERVICE_NAME sh
```

To copy the Compose folder to another Windows machine (PowerShell):

```powershell
Compress-Archive -Path ".\compose-lab\*" -DestinationPath ".\compose-lab.zip"
```

On the other machine, extract the ZIP, open a terminal in the folder and run:

```bash
docker compose up -d
docker compose ps
```

## 12. Git Commands to Push This README

Run these commands from the folder containing `README.md`:

```bash
git init
git add README.md
git commit -m "Add Docker lab commands"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
git push -u origin main
```

Replace the GitHub URL with your repository URL.

If the repository already has a remote:

```bash
git remote -v
git add README.md
git commit -m "Update Docker lab commands"
git push
```
