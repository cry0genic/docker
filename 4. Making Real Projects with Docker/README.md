## Real Projects with Docker

### Project Outline
Create NodeJS web app 🠲 Create a dockerfile 🠲 Build image from dockerfile 🠲 Run image as container 🠲 Connect to web app from browser<br/>
Disclaimer: We're going to do a few things slightly wrong! <br/>

### Node Server Setup
Download the [HTML file](source code), open it and follow the steps in it. <br/>

[index.js](https://github.com/cry0genic/Docker/blob/main/4.%20Making%20Real%20Projects%20with%20Docker/simpleweb/index.js) <br/>
[package.json](https://github.com/cry0genic/Docker/blob/main/4.%20Making%20Real%20Projects%20with%20Docker/simpleweb/package.json)<br/>


### A Few Planned Errors
Now that we have the source code, we will figure out how to wrap this up in a container.
<br/>
Specify a base image 🠲 Run some commands to install additional programs 🠲 Specify a command to run on cotainer startup
<br/>

```bash
# Specify a base image
FROM alpine

# Install some dependencies 
RUN npm install

#Default command
CMD ["npm", "start"]
```

<br/>

Run ```docker build . ```

<br/>
Error 


```bash
/bin/sh: npm: not found
```
<br/>

### Base Image Issues
The alpine image does not have a lot of programs included in it. It has only some default UNIX programs and some package managers.
<br/>
There are two solutions to this problem. Either find and use a different base image, or continue using the alpine image and  RUN some commands so as to install NodeJS and npm. <br/>

We will use an Image that already has Node preinstalled. Go to [Docker Hub](https://hub.docker.com) and find the [Node image](https://hub.docker.com/_/node) <br/>

In the Dockerfile, replace ```FROM alpine``` by ```FROM node:alpine``` <br/>
By using the alpine tag, we make sure that we do not get a bunch of additional preinstalled programs. <br/>
Alpine means as compact as possible. <br/>

Now run ```docker build .``` <br/>

### A Few Missing Files
When we ```RUN npm install``` , the image that comes ```FROM node:alpine``` does not have a package.json file. By default, none of the files in your project directory are available inside your container, unless you specifically allow it! <br/>

### Copying Build Files
The COPY instruction is used to move the files and folders from our local machine into the FS of the temporary container. <br/>
```COPY <path to folder on your machine relative to build context> <path to copy stuff inside the container>``` <br/>
Add ```COPY ./ ./``` above ```RUN npm install``` (because we want the files before npm is installed) <br/>

Now run ```docker build -t <yourDockerID>/<name of the project> . ```. This will now run succesfully and generate an Image. <br/>
If you get the error ```npm ERR! Tracker "idealTree" already exists ```, then specify a working directory(in the dockerfile) and copy all the required files there. This should probably solve the problem.<br/>
```bash
WORKDIR /usr/app
COPY ./ ./
```
<br/>
WORKDIR instruction enables us to run commands relative to a folder <br/>

<br/>
Now, run the container by  ```docker run <yourDockerID>/<name of the project>```   
<br/>
Now go to [https://localhost:808](https://localhost:808)

### Container Port Mapping
When we tried to visit https://localhost:8080, we got an error message. The browser is making a request to port 8080 on your current local machine. By default, no traffic coming into your localhost network is routed into the container.The container has its own isolated ports that cannot recieve traffic. To connect/bind both the ports, we set an explicit port mapping. <br/>

##### Docker Run with Port Mapping
```docker run -p 8080:8080 <image name>``` <br/>
The first 8080 is for route incoming requests to this port on localhost, and the second 8080 is the port inside the container.
<br/>
Now, run ```docker run -p 8080:8080 <yourDockerID>/<name of the project>``` <br/>
Go to https://localhost:8080 and now your web application will work just fine! 

### Unnecessary Rebuilds
Run the container by ```docker run -p 8080:8080 <yourDockerID>/<name of the project>``` <br/>
Go to [index.js](https://github.com/cry0genic/Docker/blob/main/4.%20Making%20Real%20Projects%20with%20Docker/simpleweb/index.js) and change "Hi there" with "Bye There" and refresh https://localhost:8080. Nothing changes. This is because anytime we are creating a container, we are taking a snapshot of the FS. Any changes made further will not be automatically detected inside the container. One solution to this is building the container again. All the instructions after COPY run and all the dependencies are again installed even though we only changed a single line of unrelated code. This is not ideal.

### Minimizing Cache Busting and Rebuilds
In the last part, we saw that we had to run every single step starting from COPY to RUN. This can be avoided. <br/>
```bash
COPY ./package.json ./
RUN npm install
COPY ./ ./
```

