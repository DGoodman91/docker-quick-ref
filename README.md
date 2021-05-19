# Docker Quick Reference

## Image Basics

Build an imagine called myimage using the Dockerfile in the current directory.

```
docker build -t myimage .
```


List all images.
```
docker image ls
```

[Scan image](https://docs.docker.com/engine/scan/) for vulnerabilities using Snyk. 
```
docker scan mysql:5.7
```

View the layers that make up an image with the *history* command.
```
docker image history --no-trunc mysql:5.7
```

### Layer caching

When a layer in our image changes, each downstream layer needs rebuilding. So with the following Dockerfile, if ANY file in our app changes, the yarn dependency installation layer will be rebuilt too, meaning downloading them all again

```docker
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
``` 

A big efficiency is to copy the package.json & yarn.lock files first (and hence in a separate layer), then install the deps, then copy the rest of the app files. This means the *yarn install* layer will **only** be rebuilt if there's a change to the two relevant files.

```docker
FROM node:12-alpine
WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn install --production
COPY . .
CMD ["node", "src/index.js"]
```

### Multi-stage builds

The following example of building a Java application image means that even though we need a JDK to compile the code, the JDK isn't included in the final image (since it's not needed).

```docker
FROM maven AS build
WORKDIR /app
COPY . .
RUN mvn package

FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/webapps 
```

---

## Container Basics

Run a container in detached mode and map port 3306 in the container to port 3307 on the Docker host. Use the *name* argument to name it, else it'll be given a random name by the docker service.
```
docker run --name my-db -d -p 3307:3306 mysql:5.7
```
Can use the *-e* argument to specify environment variables in the container. ***DON'T PASS SECRETS IN THIS WAY IN PRODUCTION!***
```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=myfirstdb mysql:5.7
```
Can combine the working directory argument *-w* with the positional *command* argument - supplied as the final arg to execute a command on container start.
```
docker run -dp 3000:3000 -w /app -v "$(pwd):/app" my-node-app sh -c "yarn install && yarn run dev"
```


List all containers, all states.
```
docker ps -a
```
Stop and remove a container by id using the force flag.
```
docker rm -f bb41eae63413
```

Tail the container logs by id.
```
docker logs -f bb41eae63413
```

Execute commands inside the container specified by the id. In this case we connect to a db container and run the mysql command line tool.
```
docker exec -it dfda05c2c20b mysql -p password
```

---

## Volumes

### Named volumes

Create a new named volume.
```
docker volume create my-named-volume
```
When running a container, mount the named volume *my-named-volume* into the container at */var/lib/mysql*.
```
docker run -dp 3306:3306 -v my-named-volume:/var/lib/mysql mysql:5.7
```
Get details of the volume including the created date and the location on the docker host's filesystem
```json
docker volume inspect my-named-volume

[
    {
        "CreatedAt": "2021-05-18T23:42:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "my-named-volume",
        "Options": {},
        "Scope": "local"
    }
]
```
Clean up volumes not being used by any containers.
```
docker volume prune
```

### Bind mounts

Can use bind mounts to mount a portion of the host filesystem into the container. Useful for development.
```
docker run -dit -v C:/Users/david/Documents/docker/folder:/directory/in/container my-app
```

---

## Networks

Create a new network.
```
docker network create my-network
```
Run a container and join it onto the network.
```
docker run -d --network my-network mysql:5.7
```
The the *network-alias* argument can be used to add a DNS record to the network to point to the container.
```
docker run -d --network my-network --network-alias my-networked-db mysql:5.7
```

---

## Useful Containers

### nicolaka/netshoot network tools

The [nicolaka/netshoot](https://github.com/nicolaka/netshoot) image is an alpine distribution with a bunch of networking tools. We can deploy a netshoot container and join it to a network to explore, troubleshoot and debug networking issues. Check the github link, but contains stuff like dig, netcat, nmap, tcpdump, ssl_client.

```
docker run -it --network todo-app nicolaka/netshoot
```

Note -it puts us straight into an interactive shell on the container.

### alpine/git

We can run an [alpine/git](https://hub.docker.com/r/alpine/git) container to check out a repository, then use the [*docker cp*](https://docs.docker.com/engine/reference/commandline/cp/) tool to copy the repo out of the container.

```
docker run --name repo alpine/git clone https://github.com/nicolaka/netshoot.git
docker cp repo:/git/netshoot/ .
```

### httpd

Can use the [official docker httpd](https://hub.docker.com/_/httpd) image to quickly spin up a website by bind-mounting the host's filesystem
```
docker run -d -p 8080:80 -v C:/Users/david/Documents/docker/simple-webapp:/usr/local/apache2/htdocs/ httpd:2.4
```

### mysql

Use the [official docker mysql](https://hub.docker.com/_/mysql) image to get a database up and running rapidly. The named volume will be auto-created if it doesn't exist.
```
docker run -d -p 3306:3306 -v my-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=dbname mysql:5.7
```

---

## Docker Compose

### Basic examples

The following is a simple compose example from Docker's Getting Started guide. It defines two images: a node app and a database - the latter using a named volume defined in the file. The node app connects to the database using the DNS name, based by default on the name of the service (in this case *mysql*). I've added in the netshoot container which is useful for hopping onto the network and exploring. Again, ***DON'T PASS SECRETS IN THIS WAY IN PRODUCTION!***
```yaml
version: "3.7"

services:
    app:
        image: "node:12-alpine"
        command: sh -c "yarn install && yarn run dev"
        ports:
            - 3000:3000
        working_dir: /app
        volumes:
            - ./:/app
        environment:
            MYSQL_HOST: mysql
            MYSQL_USER: root
            MYSQL_PASSWORD: secret
            MYSQL_DB: todos
    mysql:
        image: mysql:5.7
        volumes:
            - todo-mysql-data:/var/lib/mysql
        environment:
            MYSQL_ROOT_PASSWORD: secret
            MYSQL_DATABASE: todos
    netshoot:
        image: nicolaka/netshoot
        command: sh -c "tail -f /dev/null"

volumes:
    todo-mysql-data:

```

Bring up the services defined in the docker-compose.yml file in the current directory, using the *remove-orphaned* arg to clean up any old copies of the containers.
```
docker-compose up -d --remove-orphans
```
Tail the docker-compose logs.
```
docker-compose logs -f
```
Tear it down.
```
docker-compose down
```
Tear it down and remove volumes.
```
docker-compose down --volumes
```
