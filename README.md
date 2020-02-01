# Docker-Spark-Tutorial
This repo is intended to be a tutorial walkthrough in how to set up and use a Spark cluster running inside Docker containers. 

I assume some familiarity with Docker and its basic commands such as build and run. Everything else will be explained in this file.

We will build up the complexity to eventually a full architecture of a Spark cluster running inside of Docker containers in a 
sequential fashion so as to hopefully build the understanding. The table of contents below shows 
the different architectures we will work through.

## TABLE OF CONTENTS
* [Create Docker images](#create-docker-images)
    * [Building images from scratch](#building-images-from-scratch)
    * [Pulling images from repository](#pulling-images-from-repository)
* [Docker networking](#docker-networking) 
  * [Containers on single machine](#containers-on-single-machine)
  * [Containers on different machine](#containers-on-different-machine)
  
  
## Create Docker images

#### Building images from scratch
If you wish to build the Docker images from scratch clone this repo.

Next, run the following commands
```
docker build -f Dockerfile_base -t {YOUR_TAG} .
docker build -f Dockerfile_master -t {YOUR_TAG} .
docker build -f Dockerfile_worker -t {YOUR_TAG} .
```
and then you may wish to push these images to your repository on Docker or somewhere else 
using the 'Docker push' command.

#### Pulling images from repository
Alternatively, you can pull the pre-built images from my repository. Simply run
the following commands
```
docker pull sdesilva26/spark_master:latest
docker pull sdesilva26/spark_worker:latest
```
*NOTE: You do not need to pull sdesilva26/spark_base:0.0.1 because, as the name suggests,
this image is used as a base image for the other two images and so is not needed on your machine.*

## Docker networking

Now let's see how to get the containers communicating with each other. First
we shall show how to do when the containers are running on the same machine and
then on different machines.

### Containers on a single machine

Most of the following is taken from Docker's website in their section 
about [networking with standalone containers](#https://docs.docker.com/network/network-tutorial-standalone/).

Docker running on your machine under the hood starts a Docker daemon which interacts
with tthe linux OS to execute your commands.(*NOTE: if you're running Docker on a 
non-linux host, docker actually runs a VM with a linux OS and then runs the Docker
daemon on top of the VM's OS.*)

By default, when you ask the Docker daemon to run a container, it adds it to the
default bridge network called 'bridge'. You can also define a bridge network yourself.

We will do both. First let's use the default bridge network. Let's also run a container
with the image for the Spark master and also one for the Spark worker. Our architecture therefore
looks like the figure below. 

![Alt text](/images/docker_single_machine.png "docker_on_single_machine")

Start the two containers up using
```
docker run -dit --name spark-master sdesilva26/spark_master:0.0.1 bash
docker run -dit --name spark-worker sdesilva26/spark_worker:0.0.1 bash
```
The d flag means to run the container dettached, so in the background, the i flag
indicates it should be interactive so that we can type into it, and the t flag specifies 
the container should be started with a TTY so you can see the input and output.

Check that the containers have started using 
```
docker container ls
```
and you should get something similar to

![Alt text](/images/screenshot1.png "screenshot1")

You can see which containers are connected to a network by using
```
docker network inspect bridge
```
and you should see something similar to the following

![Alt text](/images/screenshot2.png "screenshot2")

Attach to the spark-master container using
```
docker attach spark-master
```
You should the prompt to change now to a # which indicates that you are
the root user inside of the container.

Get the network interfaces as they look from within the container using
```
ip addr show
```
The interface with the title starting with 'eth0"' should display that
this interface has the same IP address as what was shown for the spark-master
container in the network inspect command from above.

Check internet connection from within container using
```
ping -c 2 google.com
```
Now ping spark-worker container using its IP address from the network
```
ping -c 2 172.17.0.3
```
If you know ping by container name it will familiarity
```
ping -c 2 spark-worker
```
Now dettach from spark-master whilst leaving it running by holding down
ctl and hitting p and then q.
### Containers on different machines

