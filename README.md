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









