## Multiple Container Deployment to AWS

### Multi-Container Definition Files
This time, we don't have a single Dockerfile. We have many folders which have their own Dockerfiles. If we just push to code to AWS BS, it won't be able to figure out what to do. Therefore, we will now add configuration to tell EB how to run our project.

Inside of our project directory, we will create a new file called [Dockerrun.aws.json](source code). It will tell EB where to pull our images from, which resources to allocate to each one, how to set up port mappings and some associated information. 

This 'Dockerrun.aws.json' file can be thought of as a production version  of 'docker-compose.yml' file. The docker-compose file has instructions on how to build an image whereas the dockerrun file is going to pull the already built image and configure stuff.

### Finding Docs on Container Definitions
Elastic BS doesn't know how to run containers, hence it passes the task to Amazon Elastic Container Service(ECS). We work with Amazon ECS by creating 'Task Definitions' which are essentially instructions on how to run a single container.

The docs on Amazon ECS Task Definitions can be found [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)<br/>
The docs on Task Definition-Container Definition parameters can be found [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definitions)

### Adding Container Definitions to DockerRun
The configuration for Clinet, Server and worker are:

```bash
{
    "AWSEBDockerrunVersion": 2,
    "containerDefinitions": [
        {
            "name":"client",
            "image":"cry0genic/multi-client",
            "hostname":"client",
            "essential":false,
            "memory": 128
        },
        {
            "name":"server",
            "image":"cry0genic/multi-server",
            "hostname":"api",
            "essential":false,
            "memory":128
        },
        {
            "name":"worker",
            "image":"cry0genic/multi-worker",
            "hostname":"worker",
            "essential":false,
            "memory":128
        }
        
    ]
}
```

### Forming Container Links
We will now add the nginx to containerDefinitions

```bash
{
            "name":"nginx",
            "image":"cry0genic/multi-nginx",
            "hostname":"nginx",
            "essential":true,
            "portMappings":[
                {
                    "hostPort":80,
                    "containerPort":80
                }
            ],
            "links":["client","server"],
            "memory":128
        }
```

[Dockerrun.aws.json](source code)

### Creating the EB Environment
Create a new application on AWS EB. Create an environment that maps up to the application that you just created. Choose 'Multi-Container Docker' in the preconfigured platform. 

### Managed Data Service Providers
In our Dockerrun file, we have no references to containers that are running Postgres or Redis.Why?

In development, we were running the Postgres & Redis services inside the container. In production, we are going to take an alternate approach. 

Nginx, Express Server, Nginx w/ Prod files and the Worker will all be running in the EB Instance. We wired up all the instructions for this in the [Dockerrun.aws.json](source code) file. The instance for Redis & Postgres which will be serving data for our application however will not be inside of EB instance. Instead we are going to rely upon two external services to fulfill all our dataneeds for our application. 

The two services that we are going to use are AWS Relational Database Service(RDS) and AWS Elastic Cache. These are general data-services that can be used with just about any application. 

##### Why use these services?
AWS Elastic Cache:
- automatically creates and maintains Redis instances for you
- super easy to scale
- built in logging + maintenance
- probably better security than what we can do
- easier to migrate off to EB with
<br/>

AWS Relational Database Service:
- automatically creates and maintains Redis instances for you
- super easy to scale
- built in logging + maintenance
- probably better security than what we can do
- easier to migrate off to EB with
- automated backups and rollbacks

### Overview of AWS VPC's and Security Groups
We will connect the EB instance to RDS instance and the EC instance.

When we created our EB instance, it was created in a very specific region of the world. In each of these regions, you get created a Virtual Private Cloud(VPC). A VPC is it's own private little network so that any instance/service that you create is isolated to just your account. It is also used in security and also to connect to different instances on AWS. When you create an application, you automatically get one default VPC per region.

To connect different instances with each other, we need to create a 'Security Group'. It is basically firewall rules. When we created the EB instance, a security group was automatically created that allows any incoming traffic on Port 80 from any IP. 

We are going to create a new security group and this security group is going to say as a rule let any traffic access this instance if it belongs to this security group. So, after creating a security group, we are going to attach it to all three of these different services. 

Follow the tutorial for most of the next sections as they are quite crucial.

### RDS Database Creation
Follow tutorial for detailed steps

### ElastiCache Redis Creation
Follow tutorial for detailed steps

### Creating a Custom Security Group
Follow tutorial for detailed steps

### Applying Security Group to Resources
Follow tutorial for detailed steps

### Setting Environment Variables
Follow tutorial for detailed steps

### IAM Keys for Deployment
Follow tutorial for detailed steps

### Travis Deploy Script
```bash
deploy:
  provider: elasticbeanstalk  
  region: us-east-2
  app: multi-docker
  env: MultiDocker-env
  bucket_name: elasticbeanstalk-us-east-2-761185896481
  bucket_path: docker-multi
  on:
    branch: master

  access_key_id: $"AWS_ACCESS_KEY"
  secret_access_key: 
    secure: $"AWS_SECRET_KEY"  
```

[.travis.yml](source code)

## ggwp


