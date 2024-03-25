# Docker Essential Training

## What is Docker?

### What Docker does?

Docker carves up a running Linux system into small containers each of which is it's own sealed little world, with it's own everything. All isolated from anything else. Docker does the work of getting to and from your Computers/System. Docker is a social platform for you to find and share containers, which are different from virtual machines. 

### what is a Container?

the Container is self-contained sealed unit of software. It contains everything required to run the code such as Operating System, Libraries, dependencies, code, configurations, process and networking. 


## Format the Output

Below is a shell script that make the output of some the docker commands set nicely on the screen - will change output from vertically to horizontally:

```bash 
export FORMAT="\nID\t{{.ID}}\nIMAGE\t{{.Image}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.RunningFor}}\nSTATUS\t{{.Status}}\nPORTS\t{{.Ports}}\nNAMES\t{{.Names}}\n"

# this shell will only work on bash
```


## Docker Installation 

To install Docker you need to check which OS you are running. In my case I'm running Fedora so to Identify which Fedora Linux Distribution version and Kernel version run below command:

```bash 
cat /etc/os-release | grep -iE "(^NAME|^VERSION=)"  
NAME="Fedora Linux"  
VERSION="39 (Workstation Edition)"

uname -r       
6.7.4-200.fc39.x86_64
```

Then you can visit this [link](https://docs.docker.com/engine/install/) to check the installation instructions to follow based on operating system version you have.

![](./attachments/Pasted%20image%2020240212141504.png)

This is the link to install Docker on fedora [Installation Steps](https://docs.docker.com/engine/install/fedora/) 


## Docker Flow 

From Fixed image, we can run many containers, each container start as same as image, but once we start manipulate containers the change is not reflected to the image if you stop the container the change you made will be lost.  

![](./attachments/Pasted%20image%2020240212235810.png)

to save the changes made by us in the running containers we can use `docker commit container_Name/Container_ID`

![](./attachments/Pasted%20image%2020240213000653.png)

## Docker Networking 

Connect to anther docker through the docker host machine network:
> can be done by expose the ports to host OS

![](./attachments/Pasted%20image%2020240217160747.png)

Connect Directly between containers :
> - It can be done by link the client to the server
> - It can be done through create private network and link the client to server [Docker Link](###docker-link) [Docker Network](#docker-network)

![](attachments/Pasted%20image%2020240217160922.png)

## Docker Command

> 1- Don't let your containers fetch dependencies when they start.
> 2- Don't leave important things in unnamed stopped container.

### Test Docker working

```bash 
sudo docker run hello-world
```

### Docker Info

This command to display the docker information such as version, number of CPU cores, 

```bash 
sudo docker info 
```

### Docker Run


```bash
## to exit the Docker interactive terminal you can run exit or CTRL+D
sudo Docker run -it --rm ubuntu bash 

# --rm to delete the container once the command finished
# --it means interactive terminal 

sudo docker run -d --it ubuntu bash

# -d deattached the container and keep it running  

```


### Docker Attach

This command help us to attach to running docker container

```bash 
sudo docker attach e697bead00ce
# to exist and detach again use below short cut
# CTRL+P & CTRL+Q # without shift
```

### Docker PS

```
sudo docker ps -a 
sudo docker ps -l

export FORMAT="\nID\t{{.ID}}\nIMAGE\t{{.Image}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.RunningFor}}\nSTATUS\t{{.Status}}\nPORTS\t{{.Ports}}\nNAMES\t{{.Names}}\n"
sudo docker ps -l --format=$FORMAT
```

### Docker Commit

Docker commit uses to write the changes that has been done to the container to a new docker image. 

```bash 
# get the list of container 
sudo docker container ls -a 
# get the container ID that we need to save it as docker Image 
sudo docker commit 95c8e1aa6930
# Give the new Image a tag or Name 
sudo docker tag 36d9b50c90313afbf267ecba170088288e6111dae30f5389143eaa  
672a74232d my-new-image

# you can give the iamge a tag during the docker commit
sudo docker commit container_name new_image_name
```

### Docker EXEC

create a new process to the running container, this will help in debugging and access container for docking more stuff.

```bash
# access to container as an interactive shell 
sudo docker exec -it container_name bash 
```

### Docker LOGS

this command helps us in troubleshooting the containers and getting more clear idea whats going on inside the containers.

```bash
sudo docker logs container_name
```

### DOCKER KILL

```bash 
sudo docker kill container_name

```

### DOCKER RM

```bash 
# you should stop or kill the container first to be able to remove the container 
sudo docker container rm container_name
```

### Docker Resource Constraints

```bash 
# memory limits 
docker run --memory maximum-allowed-memory image-name command 

# CPU Limits 
sudo docker run --cpu-shares relative to other containers 
sudo docker run --cpu-quota to limit it in general 

```

### Docker EXPOSE Port

```bash
# below command will expose the two ports to the host from docker 
docker run --rm --it -p 45678:45678 -p 45679:45679 ubuntu bash 

# to check opened ports for specific container
docker port container_name 

# to specify the protocol of the port 
docker run -p host_port:cont_port/tcp -p host_port:cont_port/udp container_name 


```

### Docker LINK

to connect containers between each other we use link option 

```bash
# server container 
docker run -it --rm --name server ubuntu bash
nc -lp 1234

# client container 
docker run -it --rm --link server --name client ubuntu bash
nc -nv server 1234 


```

### Docker NETWORK

```bash
# create network 
docker network create example 

# use server network 
docker run --rm -it --net=example --name server ubuntu bash 

# use client network 
docker run -it --rm --link server --net=example --name client ubuntu bash
```

### Docker Volume

Docker has different type of volumes see this [article](https://semaphoreci.com/blog/docker-volumes)

Two main varieties : 

* Persistent Volume : data resides and stay on the host 
* Ephemeral Volume : data will be removed once no containers use this volume 

```bash 
# sharing data with the host 
docker run -it -v Host_Folder_absolute_Path:Docker_Folder_Path ubuntu bash 

# sharing single file into a container using volumes-from
docker run -it --name ubuntu-sharedfile -v /shared-data ubuntu bash
docker run -it -volumes-from ubuntu-sharedfile ubuntu bash 

```

### Docker Registry 

you can search the registry, 

```bash
# you can search the registry,
docker search ubuntu

# access Docker Registry 
docker login

# tag the container that we need to push 
docker tag debian:sid myimages/test-image:v11.0
dokcer push 

```
### Dockerfile

```bash
# build image with tag name-of-result. and you should have Dockerfile in the path 
docker build -t name-of-result .

```
