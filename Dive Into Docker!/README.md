## Why use Docker?
Docker makes it really easy and straightforward to install and run software on any machine(personal machines, web servers, cloud based computing platform, etc.) without worrying about setup or dependencies. 

## What is Docker?
Docker is a platform/ecosystem around creating and running containers.

### Image
Single file with all the deps and config required to run a program

- FS Snapshot - copy paste of very specific directory or files
- Startup Command - starts up the container with a specific set of instructions

### Container
- Instance of an image
- Runs a program

#### What really is a container?
- Namespacing - isolating resources per process (Processes, Hard Drive, Users, etc)
- Control Groups - Limits amount of resources used per process (Memory, CPU Usage, Network Bandwith)

Container is a set of processes that have a grouping of resources specifically assigned to it. 



## Docker Package
- Docker Client(Docker CLI) - issue commands to
- Docker Server(Docker Daemon) - responsible for creating images, running containers, etc

Client and daemon can run on same or different hosts

### Installation on Linux
[Follow this tutorial](https://github.com/cry0genic/Docker/blob/main/Dive%20Into%20Docker!/Installing%20Docker%20on%20Linux.html) You might have to download the file and open it.

### Docker Version:
Lists out the CLI and Server Version
```docker version```

### Hello-World
- hello-world: 
```docker run hello-world```

- Docker server checks for a local copy of the image 'hello-world' in the image cache. If the image does not exist, it reaches out to Docker Hub(free public repository for images) and saves the image to your PC. 
Now, the image is used to generate a container and run a program whose sole purpose is to print what you see on your screen. 


