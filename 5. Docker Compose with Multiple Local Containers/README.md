## Web App

### App Overview
A web application which displays the number of visits on the site. <br/>
Web App 🠲 Node App 🠲 Redis <br/>
We will have separate docker containers for both the node application and the redis server. All the node containers will connect to the redis instance and store the current count variable.

### App Server Code
[index.js](https://github.com/cry0genic/Docker/blob/main/5.%20Docker%20Compose%20with%20Multiple%20Local%20Containers/visits/index.js) <br/>
[package.json](https://github.com/cry0genic/Docker/blob/main/5.%20Docker%20Compose%20with%20Multiple%20Local%20Containers/visits/package.json) <br/>

### Assembling a Dockerfile
[Dockerfile](https://github.com/cry0genic/Docker/blob/main/5.%20Docker%20Compose%20with%20Multiple%20Local%20Containers/visits/docker-compose.yml) <br/>
Inside the visits directory, run ```docker build .``` <br/>
Tag the image by ```docker build -t <yourDockerID>/<name of project> . ``` <br/>

### Introducing Docker Compose
Run the container by ```docker run <yourDockerID>/<name of project>``` <br/>

You will get an redis instance error. Run ```docker run redis``` and in another terminal once again execute the same command to run the container. Once again, we get an error. This is because, by default, each of the container having node and redis do not have any means of communication set by default. They are just two absolutely isolated processes.<br/>

Options for connecting these containers:
- use Docker CLI's network features (real pain in the neck xD)
- use Docker Compose
<br/>

We will use a separate CLI tool, called Docker-Compose!
- separate CLI that gets installed along with Docker
- used to start up multiple Docker containers at the same time
- Automates some of the long-winded arguments we were passing to 'docker run'
<br/>
Run  ```docker-compose```  in the terminal. It will list out all the commands that can be used with 'docker-compose'!
<br/>

### Docker Compose Files
We will write all the commands and configuration inside the docker-compose.yml file. <br/>
We define two services inside the docker-compose file and both these services take the form of these two different docker containers. <br/>
[docker-compose.yml](https://github.com/cry0genic/Docker/blob/main/5.%20Docker%20Compose%20with%20Multiple%20Local%20Containers/visits/docker-compose.yml)

### Networking with Docker Compose
By just defining these two services inside the file, docker-compose will automatically create both these containers on the same network and will have access to communicate with each other. <br/>

Inside [index.js](https://github.com/cry0genic/Docker/blob/main/5.%20Docker%20Compose%20with%20Multiple%20Local%20Containers/visits/index.js) , replace ```const client = redis.createClient()``` with ```const client = redis.createClient({host:'redis-server'})``` <br/>
Redis makes a connection request on ```host:redis-server``` . When the request goes out from the node application, docker is going to see that it's trying to access a host called 'redis-server' and connect it with the other container that has redis.

### Docker Compose Commands
```docker-compose up``` : docker run myimage <br/>
```docker-compose up --build``` : docker build + docker run myimage <br/>

Run ```docker-compose up``` and go to https://localhost:4001 and try to refresh the page. You will see the site visits change everytime you refresh.

### Stopping Docker Compose Containers
```docker-compose up -d``` launches containers in the background <br/>
```docker-compose down``` stops all these containers <br/>

### Container Maintenance with Compose
How to deal with containers that crash? <br/>
Status codes are quite important as this decides how docker will decide whether or not to restart the container.<br/>

###### Restart Policies
There are 4 different restart policies listed as:
- "no" : never attempt to restart this container if it ever stops or crashes (in yml file, no equals false)
- always : always attempt to restart this container
- on-failure : only restart if the container stops with an error code
- unless-stopped : always restart unless we forcibly stop it
<br/>
Add a restart policy by adding  ```restart: <policy>```  into the services. <br/>

### Container Status with Docker Compose
Print out the status of running containers by ```docker-compose ps```. This command will look for the 'docker-compose.yml' file in the directory from where the commands was ran, and then if it finds one, it will read that file and find all the running containers on the machine which belong to this docker-compose file.
