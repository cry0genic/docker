## Docker CLI

### Docker Run
Creating and running a container from an Image:
```docker run <image name>```

### Overriding Default Commands
Override the default startup command
<br/>
Execution : ```docker run <image name> command```
Example : ```docker run busybox echo hi there```
Example : ```docker run busybox ls``` (lists out everything in the container)
<br/>
Echo and ls exist inside the busybox FS Image, hence are executable.

### Listing Running Containers
List all the running containers on your machine:
```docker ps```
- Container ID : id of the container
- Image : image used for the container
- Command : command issued
- Created : created x seconds ago
- Status : up for y seconds
- Ports : ports opened for outside access
- Names : randomly generated name

#### Listing all Containers that have ever been created
Lists out all the containers that have ever been created:
```docker ps --all```


### Container Lifecycle
Creating a container and actually starting are two separate processes. <br/>
```docker run``` = ```docker create``` + ```docker start``` <br/>

- Create a Container
    ```docker create <image name>``` <br/>
    Prints out the id of the container created as the output
- Start a Container
    ```docker start -a <container id>```<br/>
    -a command makes Docker watch for output from the container and prints it to your terminal
    
### Restarting Stopped Containers  
List out all the containers using ```docker ps -all``` and then to restart a stopped container, grab it's id and then run ```docker start -a <container id>``` <br/>
The default command cannot be replaced. The container will start with the default command which was given at the time of creating the container. <br/>

### Removing Stopped Containers
To delete all stopped containers:
```docker system prune```

### Retrieving Log Outputs
Get logs from a container:
```docker logs <container id>```

### Stopping Containers
- Stop a running container:
    ```docker stop <container id>``` <br/>
    "SIGTERM" message is sent(short for terminate signal) which enables to shut down the container giving it some time to clean up(10s is given, else the container is killed).
- Kill a running container:
    ```docker kill <container id>```<br/>
    "SIGKILL" message is sent(short for kill signal) which instantaneously shuts down the container.

### Executing Commands in Running Containers
Execute an additional command in a container:
```docker exec -it <container id> <command>``` <br/>
##### -it flag 
allows us to provide input to the container <br/>

Every process that we create, has 3 channels attached to it. STDIN, STDOUT, STDERR
```-it``` =  ```-i``` + ```-t``` 
"-i" : attach terminal to STDIN <br>
"-t" : makes sure whatever the I/O is, if formatted in a nicely, shortely manner xD!

### Getting a Command Prompt in a Container
```docker exec -it <container id> sh``` <br/>
Ctrl+D to exit the container's shell (or just type ```exit```)

##### What is sh?
Sh is the name of a program that is being executed inside of the container. It is a command processor/shell. It allows us to type commands and have them being executed inside the container. 
    
### Starting with a Shell
```docker run -it busybox sh``` <br/>
Start up a new container out of the busybox image, run the sh program(which is a shell), and attach to STDIN of that program.

### Container Isolation
Between two containers, they do not automatically share their FileSystem.   
