# DockerDoc
All the cmds for docker to run it on server. 


## Docker have three main things:
1. Image
2. Container (in-active)
3. Container (active)

Image is blueprint of a container same as what class is for an object. whenever we create a container we need to an image. So an image is required to create a contianer and a contianer is required to run an instannce of an image/container. 

We have `Jenkins` image in docker official docs. 

### Creation
For downloading an image: 
```
docker pull jenkins
```

to run the container:
```
docker run jenkins
```

***Note: We can run only second command for downloading and running a container at the same time***

to see all the running container:
```
docker ps
```


to see all the conatiners with status active or in-active:
```
docker ps -a
```


### Deletion

To delete a container, we need to stop any running(active) container then delete in-active container then delete the image. 

To stop the running conatiner:
```
docker stop *container-id*
```

To delete a container:
```
docker rm *container-id*
```

To delete image:
```
docker rmi *image-name*
```

To give arguments to a container, you can append at the end of the command:
For example: (to sleep a continer for 10 sec)
```
docker run ubuntu sleep 10
```

To give environment variables to a container use `-e` flag before name:
```
docker run -e NODE_ENV=development webapp
```

To give tag/name of a continer use `--tag` or `-t` for that:
```
docker run -t MyPersonalWebApp webapp
```
To inspect or check the details of an image or container:
```
docker inspect *container-name/image-name/id*
```

Docker use `Dockerfile` to create a custom image from local machine. 

let suppose we have Dockerfile:
```
FROM node:18
RUN mkdir -p /opt/app
WORKDIR /opt/app
COPY src/package.json src/package-lock.json .
RUN npm install
COPY src/ .
EXPOSE 3000
ENTRYPOINT [ "npm", "start"]
```

In the above code:
1. We are using node 18th version image to create custom image.
2. Copying all the code to the working directory of the container.
3. Runnning `npm install` to install `npm` packages. 
4. Using port `3000` to connect.
5. Running `npm start` command to start the application.


#### Difference between `ENTRYPOINT` and `CMD` in dockerfile:
ENTRYPOINT append arguemnts at the end of the command which we have written in the json array. 
for example:
Dockerfile
```
...
ENTRYPOINT[ "npm", "start"]
```
when we run this docker file using some arguments like:
```
docker run mywebapp sleep 45
```
system append `sleep 45` at the end of the command `npm start`
like this:`npm start sleep 45`

on the other hand in the case of CMD:
```
...
CMD[ "npm", "start"]
```

when we run above command it will override existing commad and run only arguments. 
~~npm start~~
```
sleep 45
```

##Docker Engine

Docker engine have 3 main components:

 1. Docker Deamon
    Manage docker Objects, Volumes, Images, Conatiner etc. 
 2. REST API Interface
    Communication between docker Deamon and docker cli
 3. Docker CLI
    a command line interface just to interact from third party. 
Note that Docker cli can be used from another system by spacitying command with the remote address of the docker Deamon.
like:
```
docker -H=10.123.2.1:2375 run nginx
```

By default docker do not specify any restriction on how much cpu or ram can a continer use. 
we can specify that by using following commands:

```
docker run --cpus=.5 ubuntu // use only 50% of CPU at max
docker run --memory=100m ubuntu // use only 100 mb of memory at max
```

##Docker storage
Docker have layered architecture, means it make layers when it is executing commands in docker file:

For example:
```
FROM UBUNTU
RUN apt-get update && apt-get install python
RUN pip install flask flask-mysql
COPY . /opt/source-code
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

when this docker file run with the command:
`docker build Dockerfile -t uzair/my-custom-app`

it makes layer something like this:
 - **Layer 1**: Base Ubuntu layer
 - **Layer 2**: Changes in api packages
 - **Layer 3**: Changes in pip packages
 - **Layer 4**: Source code
 - **Layer 5**: Update Entrypoint



All the above layers can also be called as image Lagers because it is making an image from a docker file. 

when an image is created. we need to run commad to run this image, to do that we need to create a container for that image. 

command: `docker run uzair/my-custom-app`

it makes another layer called Container layer. 
 - **Layer 6**: Container layer

**Note:** container layer is read/write but image layer is read only. 

when we wish to change code of our given files. like `app.py`. 
when we change to app.py file. change goes to the container and image files will remains the same becasue we can thing that there may be more than one conatiner running from this image. 
this is called **Copy-on-write**

when we stop and destroy the container, all of our changes will be removed permanently from the container and we cannot persist these change for our running code. 
to overcome above problem, docker introduce Volumes

We can create volume by running the following command:
`dcoker volume create my-volume`

there are two types of volumes:
 - Volume mounting
   using volumes that docker create for our containers to share space. 
 - Bind mounting
   using any folder or path from our system to use as a shared space.

to run docker command with mounting, docker introduce   ``--mount`` parameter, for example:
```
docker run \
--mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```


Who is responsile for the creating and managing these mouting and volumes. 
these are Storage drivers:
there are many storage drivers based on OS, docker use accordingly. 
 - AUFS
 - ZFS (Ubuntu OS by default)
 - BTRFS
 - Device Mapper
 - Overlay
 - Overlay2 (MacOS by default)



## Networks

There are three nextwork created by default when you install docker. 
 1. Bridge (default when to run a container)
    when we run a container it use bridge network by default means all the containers that is connected to bridge network can communicate to each other using local address. 
 2. None
    When using none it means our container is not connected to any network it is standalone container.
    ```
    docker run Ubuntu --network=none
    ```
 3. Host
    In case of Host, a container can be connected to use resporsed that our host/app use. means when we run this container it will connect to the host port and use that port to communicate out of the network.
    Only one container can be used in host network, as there is only port connected to host port. 
   ```
    docker run Ubuntu --network=host
    ```

###User defined Networks
Create network by using following command
```
docker network create\
  --driver bridge \
  --subnet 182.18.0.0.16 \
  custom-isolated-network
```

Inspect Network:
In a container settings JSON, there is a property called NetworkSettings, In this node, you can find all the details about that container network.
To view the Network setting, run the following command:
```
docker inspect container-name
```

To view all the available networks in a system:
```
docker network ls
```

To inspect a network: like Gateway, subnet etc
```
docker network inspect network-name
```

Docker has build in DNS server. so Continaers can reach other by using name.
Docker use network Embeded DNS to get the ip address of the required contianer. (this is the prevention of getting other network container within a host)

## Docker Regtistry
Central repository for docker images.
Storage and distribution system for the named Docker images
just like npm, Crate, etc
when we write `image: nginx/nginx` it search on the docker hub for that image and download that image from docker hub. (docker.io)

###Own Private Registry
to create private registry:
```
docker run -d -p 5000:5000 --restart=always --name my-registry registry:2
```
Now, If we want to push some images to our registry server. i.e. nginx:latest
```
docker pull nginx:latest
docker image tag nginx:latest localhost:5000/nginx:latest
docker push localhost:5000/nginx:latest
```

You can pull that image from this private registry using this command:
```
docker pull localhost:5000/nginx
```



