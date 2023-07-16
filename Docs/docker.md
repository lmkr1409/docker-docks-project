# Docker command line Interface
```sh
docker --help
``` 
- Gives us the commands we can run using docker cli
- --help command will give help for every cli command. Below is the example command for running help on network command
```sh
docker network --help 
```

# Create docker container - Long way
-  Containers are created from container images. Container images are a compressed and pre-packaged file system that contain, no pun intended, your app along with its environment and configuration with an instruction on how to start your application. That instruction is called the entry point.
- First, we'll need to tell Docker to create a container from an image. If an image does not exist on your computer, Docker will try to retrieve it from a container image registry. By default, Docker always tries to pull from Docker Hub.
- Create image using docker container create command. First check help on container create
```sh
docker container create --help
```
will give 
```text
Usage: docker container create [OPTIONS] IMAGE [COMMAND] [ARGS...]
Create a new container
Options:
-- a bunch of options
```
- Create a hello-world image from docker-hub
```sh
docker container create hello-world:linux
```
-  It's important to note that the Docker container create command creates containers, but it does not actually start them.
- To check if the docker is running or not by listing the docker processes by using `ps` command
```sh
docker ps
```
> `docker ps` will show the containers that are actively running

- To see all the containers that are available 
```sh
docker ps --all
```
- To start the container
```sh
docker container start container_id
```
- If we check the status now using `docker ps --all` we will see `Exited(0)` status. Note: only status code 0 is successful, all other codes are failure state codes. 
- Check the logs while the container ran using first 3 characters
```sh
docker logs three_ch_of_container_id 
```
- If you want to list logs while running the container
```sh
docker container start --attach three_ch_of_container_id
```

# Create docker container - short way
- creating and starting container both using single command along with attaching the logs
```sh
docker run hello-world:linux
```
we can think `docker run` = `docker container create` + `docker container start` + `docker container attach`
- Note: `docker run` doesn't output the container id, to check we need to run the `docker ps` command explicitly 

# Create docker container - from Dockerfile
- To create container for our own application we can create using the Dockerfile. An example for creating image from Dockerfile
```docker
FROM ubuntu

LABEL maintainer="Manoj Kumar Reddy"

USER root

COPY ./test.py /

RUN apt -y update
RUN apt -y install python3
RUN chmod 755 /test.py

USER nobody

ENTRYPOINT [ "/test.py" ]
```
#### FROM

from tells Docker which existing Docker image to base your Docker image off of. This can be any existing image either local or from the internet. By default, Docker will try to get this image from Docker hub if it's not already on your machine.

#### LABEL
Some images will contain a label adding additional data like the maintainer of this image

#### USER 
user tells Docker which user to use for any Docker file commands underneath it. By default, Docker will use the "root user" to execute commands. Since most security teams do not like this, the user keyword is useful in changing a user that your app runs as to one that is less powerful, like "nobody" for example. 

#### COPY
Copy copies files from a directory provided to the Docker build command to the container image. The directory provided to Docker build is called the context. The context is usually your working directory, but it does not have to be.

#### RUN
Run statements are commands that customize our image. This is a great place to install additional software or configure files needed by your application.

#### ENTRYPOINT
The entry point tells Docker what command containers created from this image should run. You can also use the `CMD` command to do this, though there are differences.

## Building Images
- To create the image we can use the `docker build` command. But first check at `docker build --help` to check at the documentation first.
- We use `-t` or the `tag` to give the docker a tag, so we don't need to remember the docker id.
- `Docker context` is simply the folder containing files that Docker will include in your image.
-  Since the entry point script is in our working directory already, we can simply put a period here. If it were located in another folder, like say path/to/app, we would put that directory there instead of a period. 
```sh
docker build -t first-image . #If the context is present directory
docker build -t first-image path/to/app # Can we use folder in other directory other than the current context directory?
```
- By default `docker build` looks for a file named `Dockerfile` if we have a different name for the docker file then we need to provide the name of the docker file using `-f` or `--file` argument
```sh
docker build --file my_docker.Dockerfile --tag my-first-docker . 
```
- You might notice that it is printing what looks like image IDs after each command runs. Why is this? Well, Docker images are just layers of images compressed together. Because of this, Docker creates an image for every command in our Dockerfile. These are called intermediate images. When Docker finishes reading your Dockerfile, it squashes all of these images together into a single image. This is the final image that your container will be created from.

## Running containers
- You can run the docker using
```sh
docker run first-image
```
##### Note: 
If the entry point script is a never ending script like a kafka-consumer or a web-server which don't exit then above run command will hang in the terminal and we can't use `CRTL+C` to kill the process. 

Additionally though, Docker containers are not interactive by default. This means that containers will not accept keystrokes from our terminal even if we are attached to them, and in this case, the application tells us the magic keystroke we need to press to end the program.

Workaround to kill the process is open new terminal and run `docker ps` to identify the image and kill it using `docker kill container_id`.

- To start the container and not attach the container to the terminal we can use `-d` to the run command
```sh
docker run -d first-image
```
- We can name the container using `--name` tag
```sh
docker run -d --name first-container first-image 
```
- We can run `docker exec`  t run additional commands from this container. This can be helpful while troubleshooting problems or testing images created by your application's Docker file.
```sh
docker exec container_id date
```
- To run shell scripts on the docker container we can use 
```sh
docker exec --interactive --tty container_id bash
```
- `--tty` is used so that our terminal can properly interact with the container's terminal. `CRTL+d` to exit the interactive mode.

## Stopping and removing containers
- To stop the containers from running we can simply use the `docker stop` command
```sh
docker stop container_id
```
- This will take sometime depends on what docker container running. This is because the docker will terminate the containers gracefully. To stop the containers immediately we can use `-t` argument to `docker stop` command. Note that this may cause data loss.
```sh
docker stop -t 0 container_id
``` 
- Stopped containers will be there in the server, they are just stopped and not removed. To remove the containers from we can use `docker rm` command.
```sh
docker rm container_id
```
- `docker rm` does not stop the container which is running. To stop and remove the container we can use the `-f` tag.
```sh
docker rm -f container_id
```
- If we want to remove the entire list of containers we can use the below command
```sh
docker ps -aq | xargs docker rm
```
- `docker ps -a` gives all the container details but to fetch only id's we can use `docker ps -aq`. `|`(Pipe) will fetch the result from left side of command and feed it to the right side of the command. `xargs` will feed the response to the command used in right side as arguments. So `docker ps -aq | xargs` result will fed to `docker rm`.

- To remove the images we can use
```sh
docker rmi image_id
docker rmi first-image
```
- Any image that is running in a container can not be removed directly. We can use `-f` tag to remove the image from running.
```sh
docker rmi -f first-image
```

## Binding Ports to Container
- Docker provides the ability to access network ports within the container, with something called port binding. This feature allows Docker to take a port on your machine, and map it to a port within the container.
- If we are running a webserver on a port `localhost:8000`, let's say 8000 then we started the application in the `ENTRYPOINT` script and we started the server and docker is running fine. Now, if we try to access the application in our system it won't allow us to access the server because we haven't mapped the port. To map the port we can use the following command
```sh
docker run -d --name first-container -p 8002:8000 first-image
```
##### Note:
- Mounting port is from host-system to docker is the format above
- 8002 - port is from host-system and 8000 is from 8000.
- Means we are mapping to 8002 port in our system from 8000 port from docker container.
- Now we can access the application using `localhost:8002` 

## Save data from containers
> outside:inside
- Containers are disposable means when we delete containers everything will be deleted we saved in container
- Create a sample file in the container using below command
```sh
docker run --rm --entrypoint sh ubuntu -c "echo 'Hello there.' > /tmp/file && cat /tmp/file"
```
- Above script will create a container from ubuntu, create a file, add some text, display the text and removes the container.
- If we tries to access the file from our local system it will throw error says file is not detected.
- To get the data from the docker we will use `VOLUME` bind using `--volume` or `-v` tag

```sh
docker run --rm --entrypoint sh -v /tmp/container:/tmp ubuntu -c "echo 'Hello there.' > /tmp/file && cat /tmp/file"
```

- If we run the above command then there won't be any change in the output but a file would be created in our system. To check we can use `cat /tmp/container/file` will result "Hello there."

##### Note: 
- If we try to map a file on our computer that does not exist, it will be mapped as a directory within the container.
###### Example: 
Create a file in our system
```sh
touch /tmp/change_this_file
docker run --rm --entrypoint sh -v /tmp/change_this_file:/tmp ubuntu -c "echo 'Hello there.' > /tmp/file && cat /tmp/file"
```
docker will look for the file 'change_this_file' and since it exists it adds the contents to the file.

Now, if we try 
```sh
touch /tmp/change_this_file
docker run --rm --entrypoint sh -v /tmp/this_file_not_exist:/tmp ubuntu -c "echo 'Hello there.' > /tmp/file && cat /tmp/file"
```
docker says `/tmp/file` is a directory as docker is looking at `/tmp/this_file_not_exist` as a directory and it create the directory.
- If we use `cat /tmp/this_file_not_exist` then our system say this is directory since it is created by docker as directory.
