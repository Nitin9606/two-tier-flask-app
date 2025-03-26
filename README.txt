# Flask App with MySQL using Docker

## 1. Clone the Repository
```sh
git clone <your-repo-url>
cd <your-repo-name>
```

## 2. Create a Docker Image for Flask App
```sh
docker build -t flaskapp:latest .
```

## 3. Tag and Push the Image to Docker Hub
```sh
docker tag flaskapp:latest nitinbijjwan/flaskapp:latest
docker push nitinbijjwan/flaskapp:latest
```

## 4. Create a Docker Network
```sh
docker network create mynetwork
```

## 5. Run MySQL and Flask Containers Manually
```sh
# Run MySQL Container
docker run -d --name mysql --network mynetwork -e MYSQL_DATABASE=myDB -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin -e MYSQL_ROOT_PASSWORD=admin -v mysql-data:/var/lib/mysql mysql:5.7

# Run Flask App Container
docker run -d --name flaskapp --network mynetwork -p 5000:5000 -e MYSQL_HOST=mysql -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin -e MYSQL_DB=myDB nitinbijlwan/flaskapp:latest
```

## 6. Configure MySQL
```sh
# Access MySQL container
docker exec -it mysql bash

# Log in to MySQL
mysql -u root -p

# Execute the following SQL commands inside MySQL shell
GRANT ALL PRIVILEGES ON myDB.* TO 'admin'@'%';
FLUSH PRIVILEGES;
CREATE DATABASE myDB;
SHOW DATABASES;
EXIT;
```

## 7. Restart Flask Container
```sh
docker restart flaskapp
```

## 8. Test the Application
### Access Flask App
Open a browser and visit:
```
http://localhost:5000
```

### Verify MySQL Data
```sh
# Access MySQL container
docker exec -it mysql bash

# Log in to MySQL
mysql -u root -p

# Check database and messages table
SHOW DATABASES;
USE myDB;
SHOW TABLES;
SELECT * FROM messages;
```

## 9. Use Docker Compose to Run Containers
Create a `docker-compose.yml` file:

```yaml


version: "3.8"

services:
  mysql:
    container_name: mysql
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: "admin"
      MYSQL_DATABASE: "myDB"
      MYSQL_USER: "admin"
      MYSQL_PASSWORD: "admin"
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./message.sql:/docker-entrypoint-initdb.d/message.sql
    networks:
      - my-network

  flask-app:
    container_name: flask-app
    image: nitinbijlwan/flaskapp:latest
    ports:
      - "5000:5000"
    environment:
      MYSQL_HOST: "mysql"
      MYSQL_USER: "admin"
      MYSQL_PASSWORD: "admin"
      MYSQL_DB: "myDB"
    depends_on:
      - mysql
    networks:
      - my-network
    restart: always

networks:
  my-network:

volumes:
  mysql-data:

~



## 10. Test the Application Again
### Access Flask App
Open a browser and visit:
```
http://localhost:5000
```

### Verify MySQL Data
```sh
# Access MySQL container
docker exec -it mysql bash

# Log in to MySQL
mysql -u root -p

# Check database and messages table
SHOW DATABASES;
USE myDB;
SHOW TABLES;
SELECT * FROM messages;
```

