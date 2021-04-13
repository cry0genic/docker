## Docker Images

### Creating Docker Images
Dockerfile : config to define how our container behaves <br/>
Docker Server : It takes the dockerfile, looks at all the lines of configuration and then builds a usable image.

Specify a base image 🠲 Run some commands to install additional programs 🠲 Specify a command to run on container startup

### Building a Dockerfile
```docker build .``` <br/>
[Source Code]( https://github.com/cry0genic/Docker/blob/main/3.%20Building%20Custom%20Images%20Through%20Docker%20Server/redis-image/Dockerfile) <br/> 

### Dockerfile Teardown
FROM, RUN, CMD - Instruction telling Docker Server what to do... <br/>
alpine, apk add --update redis, ["redis-server"] - Argument to the instruction(customizes how the instruction is executed) <br/>

### What's a base Image
Gives us an inital set of programs which can be further used to customise our image. <br/>
FROM alpine (we use alpine as the base image because it comes with a preinstalled set of programs) <br/>

### The Build Process in Detail
For every line of configuration we put into the Dockerfile, we get a step. <br/>
```bash
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM alpine
 ---> 49f356fa4513
Step 2/3 : RUN apk add --update redis
 ---> Running in 33e75564b8a3
fetch https://dl-cdn.alpinelinux.org/alpine/v3.13/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.13/community/x86_64/APKINDEX.tar.gz
(1/1) Installing redis (6.0.11-r0)
Executing redis-6.0.11-r0.pre-install
Executing redis-6.0.11-r0.post-install
Executing busybox-1.32.1-r5.trigger
OK: 8 MiB in 15 packages
Removing intermediate container 33e75564b8a3
 ---> 35ed10e3a35a
Step 3/3 : CMD ["redis-server"]
 ---> Running in 54e30de3e509
Removing intermediate container 54e30de3e509
 ---> 7df48aa3910a
Successfully built 7df48aa3910a
``` 
<br/>
- FROM alpine : downloads the Alpine Image from Docker Hub <br/>
- RUN apk add --update redis : Looked at the image from the previous step, and created a new container out of it (Running in 33e75564b8a3 in Step 2/3). After creating the temporary container, the command (RUN apk add --update redis) was executed as the primary running process. It downloaded and installed redis along with few other dependencies. We then stop the container and take a FS snapshot of that container and we save it as a temporary image(35ed10e3a35a). <br/>
- CMD ["redis-server"] : Take the image from the previous step (35ed10e3a35a) and create a container with primary default command 'redis-server' (note that it sets this as the default command, and does not execute it). We again stop the container and take a FS snapshot of the container and its primary command and save it as a image (7df48aa3910a). This is the final image with redis and primary startup command. 
<br/>

### Rebuilds with Cache
``` bash
---> Using cache
---> 35ed10e3a35a
```
<br/>
Order of operations are important. If we exchange the positions of gcc & redis, the docker will no longer use the cached version. It will build the image once again (from the changed line).

### Tagging an Image
Tagging an image:
```docker build -t <yourDockerId>/<project name>:latest .``` <br/>
Start a container using this image by ```docker run <yourDockerId>/<project name>``` <br/>

### Manual Image Generation with Docker Commit
```bash
$ docker run -it alpine sh
$ apk add --update redis
``` 
<br/>
In another terminal <br/>

```bash
$ docker ps
$ docker commit -c 'CMD ["redis-server"]' <container id>
$ docker run <container id>
```

<br/>
This is the manual process of basically doing what the Dockerfile did.