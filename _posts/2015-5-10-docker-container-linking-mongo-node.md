---
layout: post
title: Docker Container Linking - MongoDB and NodeJS
---

In this post, I'll use [Docker](https://www.docker.com/) to perform the following tasks:

* Setting up a MongoDB installation using the [official docker image](https://registry.hub.docker.com/_/mongo/).
* Setting up a NodeJS installation using the [official docker image](https://registry.hub.docker.com/_/node/).
* Connecting the MongoDB container to the NodeJS container using [docker container linking](https://docs.docker.com/userguide/dockerlinks/).

The whole process should take only a few minutes if you have a fast internet connection. I hope that this example will demonstrate that docker container linking is quite easy and practical, an impression you might not share if you wade through the official Docker documentation.

# Retrieving the MongoDB and NodeJS images

For simplicity, we'll use the official images of MongoDB and NodeJS in this example. First off, we need to retrieve the images from Docker hub:

```bash
docker pull mongo
docker pull node
```

These commands will download a few hundred MBs worth of data, so it might take a little while.

# Starting a MongoDB instance

For a more complete example of running MongoDB in a container, take a look at my [previous post](/2015/04/30/running-mongodb-container/). For this exercise, we'll simply run MongoDB as container named `myMongoDB`:

```bash
docker run -d --name myMongoDB mongo
```

![Running a named MongoDB container](/public/imgs/containerLinking_mongod.png)

Running that command should simply return the hash of the running container as illustrated. We didn't specify the port mapping for our MongoDB container, container linking will take care of that as you'll see shortly.

Next, let's start up our NodeJS container.

# Starting a NodeJS container
```bash
docker run --link=myMongoDB:mongodb -it node /bin/bash
```

That should bring up a bash shell inside your newly created NodeJS container. We specified `--link=myMongoDB:mongodb` to instruct Docker to use the container named `myMongoDB` as a linked container, and name it `mongodb` inside our NodeJS container.

Linking a container simply extracts the runtime information of your MongoDB container (its IP address, the exposed ports) and exposes that information into your new container. To see the effects of linking, run these commands in your NodeJS bash shell:

```bash
cat /etc/hosts
env | grep MONGODB
```

![Effects of linking MongoDB to your NodeJS container](/public/imgs/containerLinking_node.png)

As you can see, linking the container created an entry in your container's hosts file and a bunch of environment variables starting with `MONGODB`.

For our purposes, we're mostly interested in the following 2 entries:

```
MONGODB_PORT_27017_TCP_ADDR=172.17.0.1
MONGODB_PORT_27017_TCP_PORT=27017
```

Using these 2 variables should be enough for us to connect to MongoDB. Let's do that next.

# Connecting to the linked MongoDB with mongoskin

I'm using [Mongoskin](https://github.com/kissjs/node-mongoskin) to connect to MongoDB within Node. The concepts are the same for any other client.

First off, we need to install Mongoskin:

```bash
mkdir /data
cd /data
npm install mongoskin
```

Then, we use Mongoskin to connect to our MongoDB database:

```bash
node
```

```js
var mongo = require('mongoskin');
var db = mongo.db('mongodb://' + process.env.MONGODB_PORT_27017_TCP_ADDR + ':' + process.env.MONGODB_PORT_27017_TCP_PORT + '/mydb');
```

Your `db` variable should now point to a functional MongoDB installation.



In this example, we're running both MongoDB and NodeJS on the same host using docker container linking. Running your containers on different hosts require more work, with a [lot](https://github.com/docker/swarm) [of](http://kubernetes.io/) [competing](https://puppetlabs.com/blog/docker-and-puppet-for-application-management) [solutions](http://www.ansible.com/docker). Personally, I'm learning towards [Ansible](http://www.ansible.com) for the combination of ease-of-use and flexibility, a topic I hope to address in a future post.
