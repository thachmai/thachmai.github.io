---
layout: post
title: Running MongoDB as a Docker container
---

A main strength of [MongoDB](https://www.mongodb.org/) is arguably its ease-of-use. You can complete its installation and have your first database up-and-running in minutes. 

With [docker](https://www.docker.com/), this process can become even easier. In this post, we'll install and create a MongoDB database in a docker container in just a few simple commands. 

All you need to get started is an [installation](https://docs.docker.com/installation/#installation) of docker. The commands in this post assume that you're running Ubuntu.

# The official MongoDB container

MongoDB conveniently provides us with an [official container](https://registry.hub.docker.com/_/mongo/). To try it out:

```bash
mkdir ~/data
sudo docker run -d -p 27017:27017 -v ~/data:/data/db mongo
```

I'll explain the syntax of the `docker run` command in a bit. At this point, you should have a MongoDB instance listening on port 27017. Its data is stored in `~/data` directory of the docker host.

Now, let's populate the fresh database with some data.

# Connecting to your MongoDB container
```bash
# Install the MongoDB client
sudo apt-get install mongodb-clients

# Change mydb to the name of your DB
mongo localhost/mydb
```
After that, you'll get into a MongoDB prompt, like this:
![MongoDB client](/public/imgs/mongodb_client.png)

We want to store some cities in our database:

```
db.createCollection('cities')
db.cities.insert({ name: 'New York', country: 'USA' })
db.cities.insert({ name: 'Paris', country: 'France' })
db.cities.find()
```

You should see the entries that you created:
![MongoDB city query](/public/imgs/mongodb_cities.png)

So far, getting data **into** our MongoDB container has proven to be easy. How about taking data out of the container?

# Migrating data to a new installation
Let's start a new MongoDB container, this time running on port 37017 instead of the default 27017:

```bash
# Copy the data from the previous container
sudo cp -r ~/data ~/data_clone

# Start another MongoDB container
sudo docker run -d -p 37017:27017 -v ~/data_clone:/data/db mongo
```

We can query the data in the new container in the same way:
![MongoDB container on port 37017](/public/imgs/mongodb_verification.png)

# Stopping your MongoDB containers

To stop our runnings MongoDB queries, you run `sudo docker ps` to see the list of the running containers and `sudo docker stop HASH` to stop each container:
![docker ps and docker rm commands](/public/imgs/mongodb_docker_rm.png)

Alternatively, you can also execute `sudo docker stop $(sudo docker ps -q)` as a shortcut to stop *all* running containers.

# DOCKER RUN command syntax
The `docker run -d -p 27017:27107 -v ~/data:/data/db mongo` does 3 main things:

* `-d` tells docker to run the container as a daemon, which is the mode that'll you want to use for server containers.
* `-p 27017:27107` maps the port 27017 of the container to the port 27017 of the host. The syntax is `-p HOST_PORT:CONTAINER_PORT`. 
* `-v ~/data:/data/db` maps the /data/db directory of the container to the ~/data directory on the host. This is called a [data volume](https://docs.docker.com/userguide/dockervolumes/), the principal mechanism to import and export data with your docker container.

How do you know which port(s) the container expose and which directory the container uses to store data? What I did was simply reading the [Dockerfile](https://github.com/docker-library/mongo/blob/c9a1b066a0f35f679c2f8e1854a21e025867d938/3.0/Dockerfile). Look for the `VOLUME` command to know what directory the container uses for its data. The `EXPOSE` command tells you which network port(s) the container listens on.
