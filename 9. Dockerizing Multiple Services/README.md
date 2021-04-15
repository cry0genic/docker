## Dockerizing Multiple Services

### Dockerizing an App
Copy over package.json 🠲 Run npm install 🠲 Copy over everthing else 🠲 Set a dockerfile which in turn sets up volumes for each of the project

### Dockerizing a React App
[Dockerfile.dev](https://github.com/cry0genic/Docker/blob/main/9.%20Dockerizing%20Multiple%20Services/complex/client/Dockerfile.dev) <br/>
In the client directory, run ```docker build -f Dockerfile.dev .``` <br/>
Grab the ID of the image, and then run ```docker run <imageID>``` <br/>

### Dockerizing Generic Node Apps
Server [Dockerfile.dev](https://github.com/cry0genic/Docker/blob/main/9.%20Dockerizing%20Multiple%20Services/complex/server/Dockerfile.dev) <br/>
Worker [Dockerfile.dev](https://github.com/cry0genic/Docker/blob/main/9.%20Dockerizing%20Multiple%20Services/complex/worker/Dockerfile.dev) <br/>

### Adding Postgres as a Service
We will now make a docker-compose file so as to make the process the process of starting each images as containers with proper arguments in an easy manner.<br/>

We will first add the express server, the postgres service and the redis service.<br/>

```bash
version: '3'
services: 
    postgres:
        image: 'postgres:latest'
```

<br/>

### Docker-compose Config

Redis service setup <br/>

```bash
redis:
        image: 'redis:latest'
```

<br/>
Server service setup <br/>

```bash
server:
        build: 
            dockerfile: Dockerfile.dev
            context: ./server
        volumes: 
            - /app/node_modules
            - ./server:/app
```

### Environment Variables with Docker Compose
Inside the server service, we will configure it in such a way to access environment variables. <br/>

Environment Variables: variableName = value <br/>

```bash
environment: 
            - REDIS_HOST=redis
            - REDIS_PORT=6379
            - PGUSER=postgres
            - PGHOST=postgres
            - PGDATABASE=postgres
            - PGPASSWORD=postgres_password
            - PGPORT=5432
```

[docker-compose.yml](https://github.com/cry0genic/Docker/blob/main/9.%20Dockerizing%20Multiple%20Services/complex/docker-compose.yml)

### The Worker & Client Services
We need to add services for both the Worker and Client projects.<br/>

Client Service <br/>

```bash
client:
        build: 
            dockerfile: Dockerfile.dev
            context: ./client
        volumes: 
            - /app/node_modules
            - ./client:/app
```

<br/>
Worker Service<br/>

```bash
worker:
        build: 
            dockerfile: Dockerfile.dev
            context: ./worker
        volumes: 
            - /app/node_modules
            - ./worker:/app
```

<br/>
But we haven't set up the port mapping yet

### Nginx Path Routing
The browser is going to make static file requests to React server as well as API requests to Express server.Hence, we need a routing that sends static requests to the React server and API requests to the Express server. That is the purpose of the Nginx server that we are going to add in. <br/>

We will add another service which will essentially create a container with an instance of Nginx server running in it. The Nginx server is going to look at the request path and redirect them to correct servers. Also, if a route is '/api/values/all' nginx is going to chop off api and send '/values/all' to the server(not automatic, we configure it).<br/>

### Routing with Nginx
We will make a [default.conf](https://github.com/cry0genic/Docker/blob/main/9.%20Dockerizing%20Multiple%20Services/complex/nginx/default.conf) file which adds configuration rules to Nginx. <br/>
Make a new folder 'nginx' inside the root directory and make a file inside is as [default.conf](https://github.com/cry0genic/Docker/blob/main/9.%20Dockerizing%20Multiple%20Services/complex/nginx/default.conf) <br/>

The break keyword ensures that no extra rewrites happen<br/>

### Building a Custom Nginx Image
Now we are going to put together a [Dockerfile.dev](https://github.com/cry0genic/Docker/blob/main/9.%20Dockerizing%20Multiple%20Services/complex/nginx/Dockerfile.dev) that'll create a new custom Nginx image and and apply the [default.conf](https://github.com/cry0genic/Docker/blob/main/9.%20Dockerizing%20Multiple%20Services/complex/nginx/default.conf) file to it.<br/>

Now, we just need to add nginx as a service in the [docker-compose.yml](https://github.com/cry0genic/Docker/blob/main/9.%20Dockerizing%20Multiple%20Services/complex/docker-compose.yml) file.<br/>

```bash
nginx:
        restart: always
        build: 
            dockerfile: Dockerfile.dev
            context: ./nginx
        ports: 
            - '6969:80'
```

### Troubleshooting Startup Bugs
Run ```docker-compose up --build``` <br/>
Exit the container by Ctrl+C <br/>
Run the container again by ```docker-compose up``` <br/>

Look out for typos

### Opening Websocket Connections
We got our application working, but in the chrome console, we got an error message about Websocket Connection. The browser and the react app in it wants to have an active connection to the development react server.<br/>

Nginx Conf File Changes:

```bash
location /sockjs-node {
        proxy_pass http://client;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
```

<br/>

[default.conf](https://github.com/cry0genic/Docker/blob/main/9.%20Dockerizing%20Multiple%20Services/complex/nginx/default.conf)








