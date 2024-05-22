# DockerDoc
All the commands for running Docker on a server.

## Docker has three main components:
1. Image
2. Container (inactive)
3. Container (active)

An image is a blueprint of a container, similar to how a class is for an object. Whenever we create a container, we need an image. Therefore, an image is required to create a container, and a container is required to run an instance of an image/container.

We have the `Jenkins` image in Docker's official docs.

### Creation
To download an image:
```bash
docker pull jenkins
```

To run the container:
```bash
docker run jenkins
```

***Note: You can run only the second command to download and run a container at the same time.***

To see all the running containers:
```bash
docker ps
```

To see all the containers with status active or inactive:
```bash
docker ps -a
```

### Deletion
To delete a container, you need to stop any running (active) container, then delete the inactive container, and finally delete the image.

To stop a running container:
```bash
docker stop <container-id>
```

To delete a container:
```bash
docker rm <container-id>
```

To delete an image:
```bash
docker rmi <image-name>
```

To give arguments to a container, you can append them at the end of the command:
For example, to sleep a container for 10 seconds:
```bash
docker run ubuntu sleep 10
```

To provide environment variables to a container, use the `-e` flag before the name:
```bash
docker run -e NODE_ENV=development webapp
```

To tag/name a container, use `--tag` or `-t`:
```bash
docker run -t MyPersonalWebApp webapp
```

To inspect or check the details of an image or container:
```bash
docker inspect <container-name/image-name/id>
```

Docker uses a `Dockerfile` to create a custom image from a local machine.

Let's suppose we have a Dockerfile:
```Dockerfile
FROM node:18
RUN mkdir -p /opt/app
WORKDIR /opt/app
COPY src/package.json src/package-lock.json .
RUN npm install
COPY src/ .
EXPOSE 3000
ENTRYPOINT ["npm", "start"]
```

In the above code:
1. We are using the Node.js 18 image to create a custom image.
2. Copying all the code to the working directory of the container.
3. Running `npm install` to install `npm` packages.
4. Using port `3000` to connect.
5. Running the `npm start` command to start the application.

#### Difference between `ENTRYPOINT` and `CMD` in a Dockerfile:
`ENTRYPOINT` appends arguments at the end of the command specified in the JSON array. 
For example:
Dockerfile
```Dockerfile
...
ENTRYPOINT ["npm", "start"]
```
When we run this Dockerfile with some arguments like:
```bash
docker run mywebapp sleep 45
```
The system appends `sleep 45` at the end of the command `npm start`:
```bash
npm start sleep 45
```

On the other hand, in the case of `CMD`:
```Dockerfile
...
CMD ["npm", "start"]
```
When we run the above command, it will override the existing command and run only the arguments:
~~`npm start`~~
```bash
sleep 45
```

## Docker Engine
Docker Engine has 3 main components:

1. Docker Daemon:
   Manages Docker objects, volumes, images, containers, etc.
2. REST API Interface:
   Facilitates communication between the Docker Daemon and Docker CLI.
3. Docker CLI:
   A command-line interface to interact from third parties.

Note that Docker CLI can be used from another system by specifying the command with the remote address of the Docker Daemon, like:
```bash
docker -H=10.123.2.1:2375 run nginx
```

By default, Docker does not specify any restriction on how much CPU or RAM a container can use. We can specify that using the following commands:
```bash
docker run --cpus=.5 ubuntu  # use only 50% of CPU at max
docker run --memory=100m ubuntu  # use only 100 MB of memory at max
```

## Docker Storage
Docker has a layered architecture, meaning it creates layers when executing commands in a Dockerfile:

For example:
```Dockerfile
FROM ubuntu
RUN apt-get update && apt-get install python
RUN pip install flask flask-mysql
COPY . /opt/source-code
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

When this Dockerfile runs with the command:
```bash
docker build -t uzair/my-custom-app .
```

It creates layers like this:
- **Layer 1**: Base Ubuntu layer
- **Layer 2**: Changes in apt packages
- **Layer 3**: Changes in pip packages
- **Layer 4**: Source code
- **Layer 5**: Update ENTRYPOINT

All the above layers can also be called image layers because it creates an image from a Dockerfile.

When an image is created, we need to run a command to run this image. To do that, we need to create a container for that image:
```bash
docker run uzair/my-custom-app
```

It creates another layer called the container layer:
- **Layer 6**: Container layer

**Note:** The container layer is read/write, but the image layer is read-only.

When we wish to change the code of our given files, like `app.py`, the changes go to the container, and image files remain the same because there may be more than one container running from this image. This is called **Copy-on-write**.

When we stop and destroy the container, all of our changes will be permanently removed from the container, and we cannot persist these changes for our running code. To overcome this problem, Docker introduces volumes.

We can create a volume by running the following command:
```bash
docker volume create my-volume
```

There are two types of volumes:
- Volume mounting:
  Using volumes that Docker creates for our containers to share space.
- Bind mounting:
  Using any folder or path from our system as shared space.

To run a Docker command with mounting, Docker introduces the `--mount` parameter, for example:
```bash
docker run \
--mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

Who is responsible for creating and managing these mountings and volumes? Storage drivers handle this. There are many storage drivers based on the OS; Docker uses them accordingly:
- AUFS
- ZFS (Ubuntu OS by default)
- BTRFS
- Device Mapper
- Overlay
- Overlay2 (MacOS by default)

## Networks
There are three networks created by default when you install Docker:
1. Bridge (default network when running a container):
   When we run a container, it uses the bridge network by default. This means all the containers connected to the bridge network can communicate with each other using local addresses.
2. None:
   When using none, it means our container is not connected to any network; it is a standalone container.
   ```bash
   docker run ubuntu --network=none
   ```
3. Host:
   In the case of Host, a container can be connected to use resources that our host/app uses. This means when we run this container, it will connect to the host port and use that port to communicate outside the network.
   Only one container can use the host network, as there is only one port connected to the host port.
   ```bash
   docker run ubuntu --network=host
   ```

### User-defined Networks
Create a network using the following command:
```bash
docker network create \
  --driver bridge \
  --subnet 182.18.0.0/16 \
  custom-isolated-network
```

Inspect Network:
In a container settings JSON, there is a property called NetworkSettings. In this node, you can find all the details about that container's network. To view the network settings, run the following command:
```bash
docker inspect <container-name>
```

To view all the available networks in a system:
```bash
docker network ls
```

To inspect a network (e.g., gateway, subnet, etc.):
```bash
docker network inspect <network-name>
```

Docker has a built-in DNS server, so containers can reach each other by using names. Docker uses network-embedded DNS to get the IP address of the required container. (This prevents other network containers within a host from being accessed.)

## Docker Registry
A central repository for Docker images.
Storage and distribution system for named Docker images, similar to npm, Crate, etc.
When we write `image: nginx/nginx`, it searches on Docker Hub for that image and downloads it from Docker Hub (docker.io).

### Own Private Registry
To create a private registry:
```bash
docker run -d -p 5000:5000 --restart=always --name my-registry registry:2
```
Now, if we want to push some images to our registry server (e.g., nginx:latest):
```bash
docker pull nginx:latest
docker image tag nginx:latest localhost:5000/nginx:latest
docker push localhost:5000/nginx:latest
```

You can pull that image from this private registry using this command:
```bash
docker pull localhost:5000/nginx
```
