This is my personal code for the course Docker and Kubernetes: The Complete Guide by Stephen Grider in Udemy.


- [Introduction](#introduction) 
- [Installing](#installing) 
- [Manipulating Containers with Docker Client](#manipulating-containers-with-docker-client) 
- [Building Custom Images Thourgh Docker Server](#building-custom-images-through-docker-server) 
- [Port Mapping](#port-mapping)
- [Copying files to the container](#copying-files-to-the-container)
- [Building React-App and serving with Nginx](#building-react-app-and-serving-with-nginx)
- [Docker Componse and Multiple containers](#docker-componse-and-multiple-containers)
- [Docker Volume and hot-reloading](#docker-volume-and-hot-reloading)
- [Tests](#tests)
- [Building React-App and serving with Nginx](#building-react-app-and-serving-with-nginx)

https://www.udemy.com/docker-and-kubernetes-the-complete-guide

# Introduction

## What is Docker, Why and When is it helpfull?
Docker is a tool to help us run Containers, by container we might think of a virtualized environment with the purpouse of isolating dependencies and minimizing error prone doing the setup of the environment.

By the words of Stephen, it would be an easy way to install things without taking care of it's dependencies in your OS.

I would improve this quote by saying that it delivers functionality. If you need a specific tool, with Docker you won't need to do the setup. You will only instantiate an "isolated virtualized environment" using Docker. 
Many tools requires a lot of setups like redis, Mysql, Postgres, MongoDB,Node.js, and so many tools used in Software Development that requires dependencies or setups to be used.

By isolated virtual enviroment it is a minimum linux image that will only install what you ask for it.


## Installing


For linux users try looking here:

https://docs.docker.com/install/linux/docker-ce/ubuntu/#set-up-the-repository

And later install docker-compose:

https://docs.docker.com/compose/install/#install-compose


# Docker simplified workflow

Even though Docker runs in a virtualized environment it still needs resources from our machine like CPU, RAM, Hard-Drive memory and so on. So how does this communication happens. Well resource is allocated for the docker image and it "runs like a service". 

So when you run for example the basic:
```
docker run hello-world
```

You will be running a docker image called "hello-world" which is just an example for how it works. 

Now the workflow:

1. Docker searches for an image locally
2. If it finds he will run it
3. If not he will search for in the Docker hub (internet-docker-hub)
4. Which will search for such image and attempt to run it.

So here we find the idea of image and docker-server.

Docker image is what I would call the container with a specific service. In the hello-world example is just a prompt of a message. It could be a container with node-js image, or MongoDb and all other setups we would find usefull for our projects. Again, all with the idea of centralizing the responsibility of setting up those tools into Docker.


# Manipulating Containers with Docker Client

List of useful docker commands that helps us:

- docker ps: List us all the docker images are currently running in our docker-server, which in local context is our machine.
- docker ps --all: List all the status of all images that had been instanciated and their status.
- docker run "image-name": Attempt to run an image container
- docker start/stop/restart -a "image-id": Attempt to start/stop/restart an image already in our docker-server.
- docker system prune: Ask to delete current docker images cached in our machine locally, which in case we want to use them again would lead to another download of the image. If accepted will return also the ammount of disk space reclaimed.
- docker logs "container id": will helps us checking out what is going on in the console of the container image we are running.
- docker stop: when you have a running container and you need to stop it you can run docker stop "image-hash" and you will stop the container. If docker stop doesn't stop the container, docker server will run "docker kill" to stop the container.
- docker kill "container id": It immediately stops the container.
- docker exec -it "container id: helps us to run commands into a running container.
- docker exec -it "container id" sh: helps us to run commands into a running container allowing us to have access to a console inside the container.

Example using docker exec

Lets use our setup of redis to help us understand how useful docker exec -it can help us.
We can install redis with docker by running:
```
docker run redis
```
With this we well start a container with an instance of redis-server inside, but how we would be able to start a redis-cli on it? If we run the normal redis-cli on our terminal we will not be able to connect with the redis-server that is running in our docker container. 
Well there is where another tag of the docker-cli helps us: "exec -it"
This command helps us running other commands inside an existing(and running) docker container. 
If you run:
```
docker exec -it 68ddca21392c redis-cli
```

You console will show a successful connection with our redis-server inside the docker container.

### Building Custom Images Thourgh Docker Server

Here in this section Stephen helps us understanding how to create a docker image. To create we follow some steps:

1. Create a folder with the name you desire you want to the image
2. Create a file with the name "Dockerfile"
3. Fill the Dockerfile with your custom setup
4. Run "docker build ." command to build an image
5. Run the container id after the build to check it out your image in action.

Steps 1 and 2 are very straightforward so I won't cover. Now, step 3 can be explained by three other steps:

1. Using a base docker image
2. Install dependencies
3. Send commands to our container to do something

This can be described in our redis-image example:

```yml
# Use an existing docker image as a base
FROM alpine
# Download and install a dependency
RUN apk add --update redis
## tell the image what to do when it starts as a container
CMD ["redis-server"]
```
- FROM says will receive what image we will use. 
- RUN install a dependency. apk is a package manager for the alpine image.
- CMD runs commands into the docker container

### Tagging a docker image
To build our image and tag it you can create by using the command:
```bash
 docker build -t vinipachecov/redis:latest .
```
This command will create a new image to be used and tagged as "vinipachecov/redis"


### Port Mapping
Lets take as an example the super simple express app below:

```js
const express = require('express');

const app = express();

app.get('/', (req, res) => {
  res.send('Hi there');
})

app.listen(8080, () => {
  console.log('Listening on port 8080');
})
```

If we create a file index.js with this code and create a Dockerfile to run it on a container, we will need an appropriate docker image to run node.js, as alpine itself doesn't have node.js. 
The node:alpine image is the image suggested by the instructor to run node.js. Alpine is a keyword for a very small image in the docker world.

To map a port we use -p tag in the command line: 

```bash
docker run -p 3000:3000 "container-id/image"
```

### Copying files to the container

In the simple-web folder we have a Dockerfile (that started) like this:
```yml
FROM node:alpine
# set a workspace directory
WORKDIR /usr/app
COPY ./ ./
RUN npm install
CMD ["npm", "start"]
```

The problem is that whenever you run a build or have any file changes you will have to go through all the build process again and again: installing npm install and copying data and so on.
Now let's look to the final version introduced by the instructor:

```yml
FROM node:alpine
# set a workspace directory
WORKDIR /usr/app
COPY ./package.json ./
RUN npm install
COPY ./ ./
CMD ["npm", "start"]
```
Here we can see we have another COPY command. The idea here is help caching your commands for faster builds by separating the operations to start the container. In this version of the file we will have faster builds as the copy commands were splitted into checking changes into package.json and checking changes into files AND THEN checking out to install stuff.
This way we are just copying package.json with our list of dependencies and only after installing them (node_modules is heavy!) we copy our source folder and run our app! This way the heavy lifting is done inside the container instead of doing it in our build process.

### Docker Componse and Multiple containers

Currently we were using things by using the docker-cli, but when we have to deal with networking multiple containers, it is usually more interesting to use docker-compose. However, we need to write docker-compose.yml files which have a different syntax than Dockerfile.

```yml
version: '3'
services: 
  redis-server:
    image: 'redis'
  node-app:
    build: .
    ports:
      - "8081:8081"
```

- Version: the version of docker-compose to be used by this file.
- services: services to be instanciated by this file
- image: tells docker-compose to use a specific image from the dockerhub
- "build: .": tells  the docker-compose to find a Dockerfile and build it.
   
Some docker-compose commands:

- docker-compose build
- docker-compose up
- docker-compose down
- docker compose up --build

Why? Well because now we are dealing with multiple containers and not just one. docker-compose looks for the .yml file and decompose that file into multiple docker-cli commands to make the use of docker easier.

### Docker Volume and volume mapping
Well copying seems to be working for our purposes so far. Still we have to rebuilt everytime we have changes in our files, which is not really an optimized workflow.

To handle this on the fly changes, we have to do something similar that we were doing before on ports, but now on folders and files. Here we can run the following command with -v tags to map the folders we want to check changings.

```bash
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app  acd94ec021e8
```

Well that is a long line of code! It does map our folder and files changes but it is absolutely difficult to remember everything. Well, that reminds us that docker-compose comes with the idea of simplifying docker-cli commands.
Let's check how docker-compose can helps us in a .yml file:

```yml
version: '3'
services: 
  react:    
    build: 
      context: .
      dockerfile: Dockerfile.dev
    ports: 
      - "3000:3000"
    volumes: 
      - /app/node_modules
      - .:/app
  tests:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm","run","test"]
```
Look to the "volumes" inside the react tag. Here we set the volumes /app/node_modules to be mapping and in the second part of the volumes settings we map "." into "/app" which is mapping every file of our local machine into the container /app folder. With this we get back the hot-reloading that react provides us when running the development server. 

PS: All of this means volume mapping or folder mapping into our container, but this does not trigger hot-reload, it only propagate forward the file changes into the mapped folder inside the docker container. This means that if you use Node.js you will still require nodemon or any other lib to watch changes into source code and restart your service inside the container.

### Tests
Now, how about the the service "tests"? Well if we also want to have hot-reloading test runs, we will need to map folders aswell. So you might guess that we will need to run it and map it also. 

* PROS and CONS
- With docker-compose we don't have access to console input, so no interaction with jest.
- With docker full command we have interaction with jest but with the cost of the long command lines.


### Building React-App and serving with Nginx
Well, so far we were doing development and testing, but how about production? Well to serve efficiently static files generated by our React build process we need a web server that serves it. In this course the instructor uses Nginx as a web-server.
The build process follows this pattern:

1. Setup react-app image
2. Setup nginx in the app image after react-build

```yml
# setting a tag helps us knowing 
# that everything below is related to it
FROM node:alpine as builder
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# Remember that our build will be at /app/build inside our container!
# Now to the Nginx setup

FROM nginx
# I want to copy something from the builder phase
COPY --from=builder /app/build /usr/share/nginx/html
# As nginx container image already deals wit hthe startup command
# we don't need call a RUN command.
```

Here we can see we are using an alias to the node:alpine image, so docker can understand that all commands after it are related only to it and that maybe we can call it later using its name.
