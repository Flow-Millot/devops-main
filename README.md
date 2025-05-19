# devops-main

## TP part 01

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


