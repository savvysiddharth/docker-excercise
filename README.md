## Deploying Your First Docker Container

* to search among all pre-existing images in docker registery
```
docker search <image-name>
```

* to see all images on host
```
docker images
```

* running docker image in a container without exposing to the host in background (-d) (tag is optional)
```
docker run -d <image-name:tag>
```

* to see list of running container
```
docker ps
```

* running docker image with ports predefined (host_port:container_port)
```
docker run -d -p 80:80 <image-name:tag>
```

* running docker image with name specified and also port 
```
docker run -d --name redisHostPort -p 6379:6379 redis:latest
```

* running docker with randomly available port exposed on host
```
docker run -d --name redisDynamic -p 6379 redis:latest
```

* to check port of any running container
```
docker port redisDynamic 6379
```

* running docker image with persistant directory defined (host-dir:container-dir)
```
docker run -d --name redisMapped -v /opt/docker/data/redis:/data redis
```

* running a command from docker image instace and getting output on host
```
docker run ubuntu ps
```

* running a docker image on foreground (eg: accessing ubuntu bash)
docker run -it ubuntu bash

* to get more details about running container
```
docker inspect <friendly-name|container-id>
```

* to see logs that a particular container is logging out on std error/std output
```
docker logs <friendly-name|container-id>
```

## Deploy Static HTML Website as Container

* Dockerfile:
```
FROM nginx:alpine
COPY . /usr/shar/nginx/html
```

* building image from a dockerfile with image-name:tag
```
docker build -t webserver-image:v1 .
```

* list all images on host
```
docker images
```

* running that image we created on port 80 and exposing to port 80
```
docker run -d -p 80:80 webserver-image:v1
```

## Building Container Images

* To define a base image we use the instruction FROM <image-name>:<tag>

* RUN <command> allows you to execute any command as you would at a command prompt

* COPY <src> <dest> allows you to copy files from the directory containing the Dockerfile to the container's image.

* Using the EXPOSE <port> command you tell Docker which ports should be open and can be bound to. You can define multiple ports on the single command, for example, EXPOSE 80 433 or EXPOSE 7000-8000

* The CMD line in a Dockerfile defines the default command to run when a container is launched. If the command requires arguments then it's recommended to use an array, for example ["cmd", "-a", "arga value", "-b", "argb-value"], which will be combined together and the command cmd -a "arga value" -b argb-value would be run.

* Dockerfile
```
FROM nginx:1.11-alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

* building image from a dockerfile with friendly image-name:tag
```
docker build -t my-nginx-image:latest .
```

* see all images on host
```
docker images
```

* launching the image
```
docker run -d -p 80:80 my-nginx-image:latest
```

* see list of running containers
```
docker ps
```

## Dockerizing Node.js applications

* We can define a working directory using WORKDIR <directory> to ensure that all future commands are executed from the directory relative to our application.

* Dockerfile
```
FROM node:10-alpine
RUN mkdir -p /src/app
WORKDIR /src/app
COPY package.json /src/app/package.json
RUN npm install
COPY . /src/app
EXPOSE 3000
CMD ["npm", "start"]
```

* building image from dockerfile
```
docker build -t my-nodejs-app .
```

* running the built image
```
docker run -d --name my-running-app -p 3000:3000 my-nodejs-app
```

* testing container
```
curl http://docker:3000
```

* With Docker, environment variables can be defined when you launch the container. For example with Node.js applications, you should define an environment variable for NODE_ENV when running in production.

* Using -e option, you can set the name and value as -e NODE_ENV=production

```
docker run -d --name my-production-running-app -e NODE_ENV=production -p 3000:3000 my-nodejs-app
```

## Creating data containers

* Data Containers are containers whose sole responsibility is to be a place to store/manage data.

* To create a Data Container we first create a container with a well-known name for future reference. We use busybox as the base as it's small and lightweight in case we want to explore and move the container to another host.

* When creating the container, we also provide a -v option to define where other containers will be reading/saving data.

```
docker create -v /config --name dataContainer busybox
```

* To copy files into a container you use the command docker cp. The following command will copy the config.conf file into our dataContainer and the directory config.

```
docker cp config.conf dataContainer:/config/
```

* Using the --volumes-from <container> option we can use the mount volumes from other containers inside the container being launched.

* we'll launch an Ubuntu container which has reference to our Data Container. When we list the config directory, it will show the files from the attached container.

```
docker run --volumes-from dataContainer ubuntu ls /config
```

* If we wanted to move the Data Container to another machine then we can export it to a .tar file.

```
docker export dataContainer > dataContainer.tar
```

* import data container back to docker 

```
docker import dataContainer.tar
```

## Load Balancing Container

* 3 key properties to configure

* The first is binding the container to port 80 on the host using -p 80:80. This ensures all HTTP requests are handled by the proxy.

* The second is to mount the docker.sock file.

* Finally, we can set an optional  -e DEFAULTHOST=<domain>

* launching nginx proxy
```
docker run -d -p 80:80 -e DEFAULT_HOST=proxy.example -v /var/run/docker.sock:/tmp/docker.sock:ro --name nginx jwilder/nginx-prox
```

* For Nginx-proxy to start sending requests to a container you 
need to specify the VIRTUAL_HOST environment variable. This variable defines the domain where requests will come from and should be handled by the container.

* In this scenario we'll set our HOST to match our DEFAULT_HOST so it will accept all requests.

```
docker run -d -p 80 -e VIRTUAL_HOST=proxy.example katacoda/docker-http-server
```

* We now have successfully created a container to handle our HTTP requests. If we launch a second container with the same VIRTUAL_HOST then nginx-proxy will configure the system in a round-robin load balanced scenario. This means that the first request will go to one container, the second request to a second container and then repeat in a circle. There is no limit to the number of nodes you can have running.

* Launching a second container using the same command as we did before.

```
docker run -d -p 80 -e VIRTUAL_HOST=proxy.example katacoda/docker-http-server
```

## Orchestration using Docker Compose

* Docker Compose is based on a docker-compose.yml file. This file defines all of the containers and settings you need to launch your set of clusters. The properties map onto how you use the docker run commands, however, are now stored in source control and shared along with your code.

* the file needs to name the container 'web' and set the build property to the current directory. We'll cover the other properties in future steps.

```
web:
  build: .
```
* This will define a container called web, which is based on the build of the current directory.

* To link two containers together to specify a links property and list required connections. For example, the following would link to the redis source container defined in the same file and assign the same name to the alias.

```
links:
    - redis
```

* The same format is used for other properties such as ports

```
ports:
    - "3000"
    - "8000"
```

* Define the second container with the name redis which uses the image redis

```
redis:
  image: redis:alpine
  volumes:
    - /var/redis/data:/data
```

* docker-compose.yml
```
web:
  build: .

  links:
    - redis

  ports:
    - "3001"
    - "8000"

redis:
  image: redis:alpine
  volumes:
    - /var/redis/data:/data
```

Launch your application using 
```
docker-compose up -d
```

* to see the details of the launched containers you can use

```
docker-compose ps
```

* To access all the logs via a single stream you use 
```
docker-compose logs
```

* Scale the number of web containers you're running using the command

```
docker-compose scale web=3
```

* As when we launched the application, to stop a set of containers you can use the command 

```
docker-compose stop
```

* To remove all the containers use the command 

```
docker-compose rm
```

## Docker Stats

* The environment has a container running with the name nginx. You can find the stats for the container by using:

```
docker stats nginx
```

* This launches a terminal window which refreshes itself with live data from the container.

* By combining the two we can take the list of all our running containers provided by docker ps and use them as the argument for docker stats. This gives us an overview of the entire machine's containers.

```
docker ps -q | xargs docker stats
```
