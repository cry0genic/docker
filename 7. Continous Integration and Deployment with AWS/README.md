## Integration and Deployment with AWS

### Travis CI FLow
We will use Travis CI to test our code, and if the test is successful, automatically deploy it to AWS

### Travis YML File Configuration
We will tell Travis to test our file using '.travis.yml' file. This file will have a series of directions which will tell Travis CI exactly what we want it to do. <br/>

Tell Travis we need a copy of docker running 🠲 Build our image using Dockerfile.dev 🠲 Tell Travis how to run the test suite 🠲 Tell Travis how to deploy our code to AWS <br/>

[.travis.yml](https://github.com/cry0genic/Docker/blob/main/7.%20Continous%20Integration%20and%20Deployment%20with%20AWS/production/frontend/.travis.yml) <br/>

When we run ```docker run <imageTag> npm run test``` the terminal just hangs and waits for input. To avoid this, we will use ```docker run <imageTag> npm run test -- --coverage``` <br/>

### AWS Elastic Beanstalk
Go to AWS Console, and then search Elastic Beanstalk. Create a new application.<br/>

A 'Load Balancer' facililates request and response between the website and the AWS environment. It forwards any requests to a VM running docker. Inside the VM, we have our docker container running with our app inside it.<br/>

If the traffic reaches a specific threshold, Elastic Beanstalk is automatically going to add VM and then route the request to the one with the least traffic.

### Travis Config for Deployment
We will now configure Travis CI to automatically deploy our application to AWS once it passes the test.<br/>

```bash
sudo: required

services:
  - docker

before_install:
  - docker build -t cry0genic/docker-react -f Dockerfile.dev .

script:
  - docker run cry0genic/docker-react npm run test -- --coverage
```

<br/>
This was the inital '.travis.yml' file.<br/>

### Exposing Ports Through The Dockerfile
After the deployment is successful, if we head over to the URL given by AWS, the page won't load. This is because we haven't done any port mapping. Add ```EXPOSE 80``` after ```FROM nginx``` in the [Dockerfile](https://github.com/cry0genic/Docker/blob/main/7.%20Continous%20Integration%20and%20Deployment%20with%20AWS/production/frontend/Dockerfile).

