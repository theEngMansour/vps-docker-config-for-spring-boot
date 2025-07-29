# Docker and Spring Boot Setup Guide
```bash
  by: @theengmansour
```
## 1. Create a Docker Network

Create a new Docker network to allow your containers to communicate:

```bash
docker network create {network-name}
```

Example:

```bash
docker network create test-net
```

---

## 2. Run MySQL Container

Run a MySQL container attached to the created network:

```bash
docker run --name mysql --network test-net -e MYSQL_ROOT_PASSWORD=12345678 -p 3306:3306 -d mysql:{tag}
```

- Replace `{tag}` with the desired MySQL version tag.  
- Example:

```bash
docker run --name mysql --network test-net -e MYSQL_ROOT_PASSWORD=12345678 -p 3306:3306 -d mysql:8.0.43-oraclelinux9
```

---

## 3. Create a New Database in MySQL

Access the MySQL container and create a new database:

```bash
docker exec -it mysql mysql -u root -p
```

Enter the password (`12345678` in this example), then run:

```sql
CREATE DATABASE {database_name};
SHOW DATABASES;
EXIT;
```

---

## 4. Connect to MySQL Using phpMyAdmin

You can also use phpMyAdmin to manage your MySQL container easily.

Run the MySQL container (if not already running):

```bash
docker run --name mysql --network test-net -e MYSQL_ROOT_PASSWORD=12345678 -p 3306:3306 -d mysql:{tag}
```

Run phpMyAdmin container connected to the same network:

```bash
docker run --name phpmyadmin --network test-net -d -e PMA_HOST=mysql -p 8080:80 phpmyadmin
```

- Open your browser at `http://localhost:8080` to access phpMyAdmin.
- Use `root` and your password (`12345678`) to log in.

---

## 5. Build Spring Boot Application Package

In your `pom.xml` add:

```xml
<packaging>jar</packaging>
```

---

## 6. Create Dockerfile for Spring Boot Application

Create a `Dockerfile` with the following content:

```dockerfile
FROM openjdk:24
LABEL authors="theengmansour"
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "-Dspring.profiles.active=cloud", "app.jar"]
```

---

## 7. Configure Spring Boot `application-cloud.yaml`

Example configuration:

```yaml
server:
  port: 2000

spring:
  application:
    name: connections
  datasource:
    url: jdbc:mysql://host.docker.internal:3306/patients_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
    username: root
    password: 12345678

  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.MySQL8Dialect
```

---

## 8. Build Docker Image for Spring Boot

Run this command in the folder containing your `Dockerfile`:

```bash
docker build -t connection:v1.6 .
```

---

## 9. Run Spring Boot Container

Run the Spring Boot container attached to your Docker network:

```bash
docker run --name spring-app --network test-net -p 8080:2000 connection:v1.6
```

---
