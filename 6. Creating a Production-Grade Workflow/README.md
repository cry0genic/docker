## Production-Grade Workflow

### Development Workflow
Development 🠲 Testing 🠲 Deployment (in a cycle)

### Flow Specifics
Github Repository : Serve as the central point of cordination for all the code. Two branches(feature, master). Pull and push into the feature branch. Create a pull request, and then merge the feature branch with the master branch. This will automatically send the changes to Travis CI. It will run some tests and if the test is successful it will automatically take our codebase and push it over to AWS hosting.

### Docker's Purpose
Docker is not a requirement. Docker is a just tool that'll help us execute some of task a lot easier.

### Project Generation
Use a project generator for the code. <br/>
Create a react app using ```create-react-app <appname>``` <br/>

Set up the react app. <br/>
We will be creating two dockerfiles, one for development and another for production!

### Creating the Dev Dockerfile
We will create the development dockerfile. <br/>
In the main directory, create a file 'Dockerfile.dev'. <br/>
[Dockerfile.dev](https://github.com/cry0genic/Docker/blob/main/6.%20Creating%20a%20Production-Grade%20Workflow/prod/frontend/Dockerfile.dev) <br/>
In order to make sure that we build our project using the Dockerfile.dev file, we tweak the docker build command.<br/>
Run ```docker build -f Dockerfile.dev . ``` <br/>
-f flag is used to specify the file.

### Starting the Container
Start the container by ```docker run -p 3000:3000 <imageID>```. <br/>
Go to https://localhost:3000.<br/>
Replace the default code ```Edit <code>src/App.js</code> and save to reload.``` in the [App.js](https://github.com/cry0genic/Docker/blob/main/6.%20Creating%20a%20Production-Grade%20Workflow/prod/frontend/src/App.js) with ```hi there``` and then go back and try to refresh. Nothing changes, as expected(since the snapshot of FS does not change). We can rebuild the container or think of a clever solution. 

### Docker Volumes
We copy the /src and /public folders into the /app directory. Hence anytime we make any changes, it is not detected by default. Hence we abandon the straight COPY approach. We will use a Docker Volume. Using it, we set up a placeholder of some sort inside the docker container and hence we will no longer copy the entire /src or /public directory. Instead, we will put in a reference. The reference is essentially going to point back to local machine and give us access to these files and folders inside the local machine(can be thought of as a mapping from a folder inside the container to outside the container).<br/>
We didn't use docker volumes before, because it's annoying. See the run command yourself: <br/>
```bash
$ docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <imageID>
```

### Shorthand with Docker Compose
[docker-compose.yml](https://github.com/cry0genic/Docker/blob/main/6.%20Creating%20a%20Production-Grade%20Workflow/prod/frontend/docker-compose.yml) <br/>
Setting this will help us reduce the code a lot. <br/>
Run ```docker-compose up``` <br/>
You will get an error saying that the Dockerfile does not exist

### Overriding Dockerfile Selection
We add a context to build to override the dockerfile selection and then we can specify the dockerfile. <br/>
```bash
build:
            context: .
            dockerfile: Dockerfile.dev
```            
<br/>
If node_modules is denying permission, run  ```chmod -R a+rw .```  from the same dir where the docker-compose.yml file is present. Took me half a day to solve this error :(

### Do We Need Copy?
No, since we already added a volume in the docker-compose file, COPY instruction isn't compulsory. It's better to leave the COPY instruction since the volumes are being handled by docker-compose, which you might not use in future. 

### Executing Tests
Build the Dockerfile.dev by ```docker build -f Dockerfile.dev .``` <br/>
Run the container by ```docker run -it <container id> npm run test``` <br/>

### Live Updating Tests
Make some modifications in the [App.test.js](https://github.com/cry0genic/Docker/blob/main/6.%20Creating%20a%20Production-Grade%20Workflow/prod/frontend/src/App.test.js) file. We see that the test suite does not detect the changes. This is because we made a container specifically for running this test. When we created this container, we took a snapshot of all the files and folders and put that inside the temporary test container. This container does not has all the volume stuff set up. <br/>

We can create another service and set up a volume to counter this. But we will take a different approach. Instead of creating another service, we will attach to the existing container disk. <br/>
Run ```docker-compose up``` and open up a second terminal.<br/>
Get the id of running container by ```docker ps``` <br/>
Execute a new command inside the container by ```docker exec -it <container id> npm run test```

### Docker Compose for Running Tests
Another solution is, creating a second service whose sole purpose is to run tests.<br/>
```bash
tests:
        build:
            context: .
            dockerfile: Dockerfile.dev
        volumes: 
            - .:/usr/app
            - /usr/app/node_modules
        command: ["npm","run","test"]
``` 
<br/>

We override the ```npm run start``` command by using ```npm run test``` <br/>
Now everytime we run ```docker-compose up``` we will start up one container which will be responsible for hosting our development server and a second container which will be solely responsible for running our tests. The downside to this appraoch is that you cannot enter any standard commands in the docker-compose default terminal interactive test suit menu<br/>
Everytime you add on a service, remember to use ```docker-compose up --build```


### Shortcomings on Testing
By default, the STDIN of the Test Container isn't receiving any inputs from our terminal. We will now try docker attach command, so as to forward input from our terminal to STDIN of the Test Container. <br/>
Run ```docker-compose up``` and then open a second terminal windows. <br/>
Get the ID of the running container by ```docker ps``` (grab the one with npm run start) <br/>
Run ```docker attach <containerID> ``` <br/>
This does not work.Any text we enter does not work. Unfortunately, we cannot manipulate the test suite while using docker-compose. We will now explore why. <br/>
Open a new terminal, grab the ID of the container and run ```docker exec -it <containerID> sh``` <br/>
Run ```ps``` which which show all the processes running inside the container. <br/>
When we run ```npm run test``` we are actually not directly running it, instead we are running process npm which further looks for more arguments and uses those arguments to decide what to do. Npm is eventually going to start up a second process that is actually running our test. This is the process that is actually executing the test suite. When we run ```docker attach``` we always attach to the STDIN of the primary process of the container. The primary npm process isn't responsible for taking in inputs for the test suite, it's the secondary process started by npm which is. 

### Need for Nginx
Nginx is an extremely popular web server. It takes incoming traffic and responds to it with static files. 

### Multi-Step Docker Builds
We are now going to start working on a second Dockerfile which is going to create an Image which will be specifically used in production.<br/>
Use node:alpine 🠲 Copy the package.json file 🠲 Install dependencies 🠲 Run 'npm run build' 🠲 Start nginx <br/>
Dependencies are only required to build the application, after which they are no longer required. Only the 'build' directory is required for production. Every other file and folder isn't required in the production environment. It would be nice if we could avoid carrying 150+ MBs worth of dependencies. Another issue is nginx server setup. <br/>
We will build a dockerfile which has multi-step build process. This is because the best solution is to have two base images, one 'node:alpine' and another 'nginx'.Inside this Dockerfile we will have two different blocks of configurations(build phase and run phase).<br/>

### Implementing Multi-Step Builds
We now create the [Dockerfile](https://github.com/cry0genic/Docker/blob/main/6.%20Creating%20a%20Production-Grade%20Workflow/prod/frontend/Dockerfile).<br/>
Now that we are onto production, we don't much care about changing our source code and don't have to make use of volumes.

### Running Nginx
Build the image by running ```docker build .``` <br/>
Copy the ID of the image built and run ```docker run -p 8080:80 <imageID>``` <br/>
Go to https://localhost:8080 and there you can see your React application. <br/>






