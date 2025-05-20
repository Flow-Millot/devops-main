# TP part 01

### 1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?

Security is the main reason because when we are defining environment variables like POSTGRES_PASSWORD inside the Dockerfile, they become part of the image permanently.

### 1-2 Why do we need a volume to be attached to our postgres container?

All database files (tables, indexes, ...) are stored outside the container in a persistent volume. Our data will survive from container restarts, crashes, rebuilds, and image updates. And we can also back up, inspect, or migrate the volume data independently.

### 1-3 Document your database container essentials: commands and Dockerfile.

Dockerfile: (sql files are stored in a folder called sql_files, and they are run with CreateSheme first and InsertData second)
```
FROM postgres:17.2-alpine

COPY sql_files /docker-entrypoint-initdb.d/
```

Build the Postgres image based on the Dockerfile:
```
docker build -t custom-postgres . 
```
Create a network:
```
docker network create app-network
```
Run the adminer:
```
docker run \                  
    -p "8090:8080" \
    --net=app-network \
    --name=adminer \
    -d \
    adminer
```
Then, create and run the container associated with the previous image, with a Volume:
```
docker run -d --name my-postgres \
  --network app-network \
  -e POSTGRES_DB=db \
  -e POSTGRES_USER=usr \
  -e POSTGRES_PASSWORD=pwd \
  -v "/Users/florentinmillot/Library/CloudStorage/OneDrive-FondationEPF/Cours/4A/DevOps/devops-main/tp01/data:/var/lib/postgresql/data" \
  custom-postgres
  ```

### 1-4 Why do we need a multistage build? And explain each step of this dockerfile.

The multistage build in Docker is used to separate the build environment (where we compile code, install tools) from the runtime environment (which is as lightweight as possible). We don’t ship Maven, source code, or temporary files in the final image but only the compiled .jar.

###### Step by step:

```
FROM eclipse-temurin:21-jdk-alpine AS myapp-build
```

Use a lightweight image with JDK 21 to compile our app. This stage is called `myapp-build`.



```
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME
```

Define an environment variable `MYAPP_HOME` to reuse the path easily and set it as the working directory.



```
RUN apk add --no-cache maven
```

Install Maven that is only needed in the build stage.
`--no-cache` to keep the code lighter



```
COPY pom.xml .
COPY src ./src
```

Copies the Maven build file `pom.xml` and the Java source code into the image



```
RUN mvn package -DskipTests
```

Runs Maven package to compile the project and build the .jar file. Skipping test




```
FROM eclipse-temurin:21-jre-alpine
```

Use a JRE only which is lighter than JDK and contains just enough to run the app



```
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
```

Copie the JAR file only from the previous stage into our container.



```
ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

Defines the default command to run the app when the container will start

### 1-5 Why do we need a reverse proxy?

A reverse proxy sits in front of our backend api and forwards incoming HTTP requests to the appropriate internal service

### 1-6 Why is docker-compose so important?

Docker Compose is essential because it simplifies managing multi-container applications by allowing you to define all services, networks, and configurations in a single file, faster. This centralized setup gives to us consistent and reproducible environments across development, testing, and production. It also manages service dependencies and startup order automatically, and making complex applications easier to run. Also, Docker Compose offers flexibility to scale services and integrates into development workflows. This gives accelerating deployment.

### 1-7 Document docker-compose most important commands.

| Command                     | Description                                                                 |
| --------------------------- | --------------------------------------------------------------------------- |
| `docker-compose up`         | Builds, (re)creates, starts, and attaches to containers for all services and gives logs|
| `docker-compose up -d`      | Starts containers in detached mode (in the background) without logs         |
| `docker-compose down -v`    | Stops and removes containers, networks, and default volumes                 |
| `docker-compose stop`       | Stops running containers but does not remove them                           | 
| `docker-compose restart`    | Restarts containers                                                         | 
| `docker-compose logs`       | Shows logs from containers                                                  | 
| `docker-compose ps`         | Lists containers with their status                                          | 
| `docker-compose exec`       | Runs a command inside a running container (like `docker exec`)              | 

### 1-8 Document your docker-compose file.

```
services:
  database:
    #Build a custom Postgres image from Dockerfile in ./sql_files
    build: ./sql_files
    image: custom-postgres  #Tag for the associated built image
    container_name: my-postgres-compose  #Container name
    env_file:
      - .env  #Load environment variables (DB_NAME, DB_USER, DB_PASSWORD, DB_HOST) from .env file
    volumes:
      #Persist database data on the host machine
      - /Users/florentinmillot/Library/CloudStorage/OneDrive-FondationEPF/Cours/4A/DevOps/devops-main/tp01/data-compose:/var/lib/postgresql/data
    networks:
      - internal  #Attach container to internal network
    restart: unless-stopped  #Restart container unless stopped by the user

  backend:
    #Build backend API service from ./simpleapi folder
    build: ./simpleapi
    image: simpleapi-java  #Image name for backend service
    container_name: simpleapi-compose
    depends_on:
      - database  #Start database first
    networks:
      - internal
      - public  #Connect to both internal and public networks
    env_file:
      - .env  #Environment variables for backend (POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD)
    restart: on-failure:3  #Restart up to 3 times on failure

  httpd:
    #Build HTTP server from ./http-server
    build: ./http-server
    image: http-server
    container_name: http-server-compose
    ports:
      - "80:80"  #Map host port 80 to container port 80 (only access)
    depends_on:
      - backend  #Start backend before httpd
    networks:
      - public  #Public network for external access

volumes:
  postgres-data:  #Named volume for persisting postgres data (not currently used)

networks:
  internal:
    name: internal-network  #Explicit network name for internal communication
    driver: bridge          #Default bridge network driver

  public:
    name: public-network
    driver: bridge
```

### 1-9 Document your publication commands and published images in dockerhub.

If you want to check my Docker repository:
```
https://hub.docker.com/repositories/flowmillot
```

Command used:
```
docker tag <local-image> <dockerhub-username>/<repository>:<tag>
```
```
docker push <dockerhub-username>/<repository>:<tag>
```
#### Summary

| Local Image       | Docker Hub Repository         | Tag | Purpose                      |
| ----------------- | ----------------------------- | --- | ---------------------------- |
| `custom-postgres` | `flowmillot/tp01-postgres`    | 1.0 | PostgreSQL database image    |
| `my-http-server`  | `flowmillot/tp01-http-server` | 1.0 | HTTP server image            |
| `simpleapi-java`  | `flowmillot/tp01-backend-api` | 1.0 | Backend API image            |

#### How to use it

To pull any image, use the command:

```
docker pull <repository>:<tag>
```
### 1-10 Why do we put our images into an online repo?

Storing Docker images in an online repository allows easy sharing and collaboration by providing a central place for my images. Also it helps for manage versions.