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













