## Multiple Services Integration

### Production Multi-Container Deployments
Push code to GitHub 🠲 Travis automatically pulls repo 🠲 Travis builds a test image, tests code 🠲 Travis builds prod images 🠲 Travis pushes built prod images to Docker Hub 🠲 Travis pushes project to AWS EB 🠲 EB pull images from Docker Hub, deploys

### Production Dockerfiles
Worker [Dockerfile](source code)<br/>
Server [Dockerfile](source code)<br/>
Nginx [Dockerfile](source code)<br/>

### Multiple Nginx Instances
There will be multiple nginx instances this time. Routing, nginx with production react files etc.

### Alterning Nginx's Listen Port
Nginx [default.cong](source code)<br/>
Client [Dockerfile](source code)<br/>

### Travis Configuration Setup
Specify docker as a dependency 🠲 Build test version of React project 🠲 Run tests 🠲 Build production version of all projects 🠲 Push all to Docker Hub 🠲 Tell EB to update

As a before_install step, we will be building the test version of client project and run some tests inside of it. For this, we will be using the development Dockerfile.dev .

```bash

sudo: required
servives:
    - docker

before_install:
    - docler build -t cry0genic/react-test -f ./client/Dockerfile.dev ./client

script:
    - docker run cry0genic/react-test npm test -- --coverage

after_success:
    - docker run -t cry0genic/multi-client ./client 
    - docker run -t cry0genic/multi-nginx ./nginx
    - docker run -t cry0genic/multi-server ./server
    - docker run -t cry0genic/multi-worker ./worker
```

### Pushing Images to Docker Hub
Add DOCKER_ID and DOCKER_PASSWORD in the environment variable provided by Travis CI. 

``` - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_ID" --password-stdin ``` 

The echo part is going to retrieve your docker password from the enviroment variable and essentially emit that over STDIN as input to the next command.

[.travis.yml](Source Code)

Now push all the changes to GitHub and let Travis do it's thing.

