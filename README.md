This is my personal code for the course Docker and Kubernetes: The Complete Guide by Stephen Grider in Udemy.


- [Introduction](#introduction) 
- [Installing](#installing) 
- [Manipulating Containers with Docker Client](#manipulating-containers-with-docker-client) 

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
- docker exec -it: helps us to run commands into a running container.

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



