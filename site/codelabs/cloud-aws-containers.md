summary: Containerizing a Node.js Application and Running it in AWS
id: cloud-aws-containers
categories: Linux, Exoscale
tags: containers, introduction, exoscale, fhstp-nccs
status: Published
authors: Thomas Schuetz

# Containerizing a Node.js Application and Running it in AWS

<!-- ------------------------ -->

## What You’ll Learn

In this lab, you will:

- Make yourself familiar with the concepts of containerization
- Learn the basic commands in a Dockerfile
- Learn the basic commands to run a container
- How to write a Dockerfile for a Node.js application

## Set up an AWS Instance
Duration: 5

- Open the AWS Console
- Search for EC2
- Click on Launch a Virtual Machine
-   * Machine Name: my-docker-instance
* OS Image: Amazon Linux (2023)
* Instance Type: t2.micro
* Key Pair: Create a new key pair
* Tags:
  * Name: my-first-instance
* Launch instance

### Connect to the Instance
You can access the instance via SSH. Therefore, you need to download the key pair and use the following command to connect to the instance:

```bash
ssh -i my-first-instance.pem ec2-user@<public-ip>
```

Alternatively, you can use the AWS Console to connect to the instance.

## Installing Docker
Duration: 5

### What is Docker?
Docker is a platform for developing, shipping, and running applications in containers. Containers allow a developer to package up an application with all parts it needs, such as libraries and other dependencies, and ship it all out as one package.

### Installing Docker
To install Docker on your AWS instance, you can use the following commands:

```bash
sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker
sudo usermod -a -G docker ec2-user
```

The commands update the package list, install Docker, start the Docker service, and add the `ec2-user` to the `docker` group.

### Verify the Installation
You can verify the installation by running the following command:

```bash
docker --version
```

Furthermore, you can run your first container by running the following command:

```bash
docker run hello-world
```

This command will download the `hello-world` image from the Docker Hub and run it in a container. The output should look like this:

```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 ```

### What is Docker Hub?
Docker Hub is a cloud-based registry service that allows you to link to code repositories, build your images, test them, and store them. It provides a centralized resource for container image discovery, distribution, and change management, user and team collaboration, and workflow automation throughout the development pipeline.

## Writing your first Dockerfile
A dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using `docker build`, users can create an automated build that executes several command-line instructions in succession.

### Create a new directory
Create a new directory for your first dockerfile:

```bash
mkdir my-docker-app
cd my-docker-app
```

### Create a new file
Create a new file called `Dockerfile`:

```bash
touch Dockerfile
```

### Edit the Dockerfile
Edit the Dockerfile with the following content:

```Dockerfile
# Use the official "debian" image as the base image

FROM debian
```

This Dockerfile uses the official `debian` image as the base image. The base image is the image that your image is based on. It sets the foundation for your image and defines the operating system and other software that your image will contain.

### Build the Dockerfile
You can build the Dockerfile by running the following command:

```bash
docker build -t my-docker-app .
```

This command builds the Dockerfile and tags the image with the name `my-docker-app`. The `.` at the end of the command specifies the build context, which is the path to the directory containing the Dockerfile. In this case, it is the current directory. The `-t` flag is used to tag the image with a name, which makes it easier to reference the image later. The resulting image will be stored in your local Docker image registry and can be pushed to a remote registry like Docker Hub in the future.

### Docker Tags
Docker tags are used to identify different versions of an image. They are used to specify the version of the image and differentiate between different versions of the same image. The image we used for the base image is `debian`. As we used docker as the runtime, it assumes that we want to fetch the image from the Docker Hub (docker.io). You can also use custom tags to identify different versions of your image. For example, it would be possible to specify a specific version of the `debian` image by using the tag `buster` (e.g., `debian:buster`). As a result, if we would use the fully resolved name of the image, it would look like this: `docker.io/library/debian:buster`.

To ensure that you are always building the same image, you should use tags to identify different versions of your image. This way, you can always refer to a specific version of your image and ensure that you are using the correct version in your deployments.

### Running the Docker Container
You can run the Docker container by running the following command:

```bash
docker run my-docker-app
```

This command runs the Docker container with the image `my-docker-app`. The container will start and run the commands specified in the Dockerfile. In this case, it will use the `debian` image as the base image.

You will see the output of the container in your terminal. If everything worked correctly, you should see the output of the `debian` image which should be quite empty in this case.

### Adding a Web Server to the Dockerfile
Now that you have successfully built and run a Docker container, you can add a web server to the Dockerfile. In our case, this will be a simple nginx web server.

Edit the Dockerfile with the following content:

```Dockerfile
# Use the official "debian" image as the base image
FROM debian

# Install nginx
RUN apt-get update && apt-get install -y nginx
```

This Dockerfile installs the `nginx` web server on top of the `debian` image. The `RUN` command is used to execute commands in the Dockerfile. In this case, it updates the package list and installs the `nginx` package.

After editing the Dockerfile, you can build and run the Docker container as before:

```bash
docker build -t my-docker-app .
```

In this case, you will see that the `nginx` web server is installed in the container, but it will not start the nginx service. To start the nginx service, you can add the following command to the Dockerfile:

```Dockerfile
# Use the official "debian" image as the base image
FROM debian

# Install nginx
RUN apt-get update && apt-get install -y nginx

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

The `CMD` command is used to specify the command that should be run when the container starts. In this case, it starts the `nginx` web server with the command `nginx -g 'daemon off;'`.

After editing the Dockerfile, you can build and run the Docker container as before:

```bash
docker build -t my-docker-app .
docker run -d -p 80:80 my-docker-app
```

This command runs the Docker container in detached mode (`-d`) and maps port 80 of the host machine to port 80 of the container (`-p 80:80`). This allows you to access the nginx web server running in the container from your host machine.

You can show the content of the nginx web server by opening a web browser and navigating to `http://127.0.0.1`.

### Summary
Until here, you learned how to create a simple container with a web server running in it. You used the `debian` image as the base image and installed the `nginx` web server on top of it. You also learned how to start the nginx service in the container and access the web server from your host machine.

## Writing a Dockerfile for a Node.js Application
You can also write a Dockerfile for a Node.js application. In this case, you will use the official `node` image as the base image and install the necessary dependencies to run a Node.js application.

### Create a new directory
Create a new directory for your Node.js application:

```bash
mkdir -p my-node-app/app
cd my-node-app/app
```

### Create a simple Node.js application
Create a new file called `app.js` and open it in your favorite text editor:

```bash
nano app.js
```

Add the following content to the `app.js` file:

```javascript
const http = require('http');
const os = require('os');

const hostname = os.hostname();
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end(`Hello World from ${hostname}\n`);
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

And let's create a `package.json` file with the following content:

```json
{
  "name": "my-node-app",
  "version": "1.0.0",
  "description": "A simple Node.js application",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "author": "Your Name",
  "license": "MIT"
}
```

This Node.js application creates a simple HTTP server that responds with a `Hello World` message and the hostname of the server.

As you might have noticed, we didn't install any node components on our machine. This is because we will use a Docker container to run our Node.js application.

### Create a new Dockerfile
In this example, we will use the official `node` image as the base image. Create a new file called `Dockerfile`:

```bash
cd ..
touch Dockerfile
```

Edit the Dockerfile with the following content:

```Dockerfile
# Use the official "node" image as the base image
FROM node

# Create a directory for the application
WORKDIR /usr/src/app

# Copy the application files to the container
COPY app/package*.json .
COPY app/app.js .

# Install the application dependencies
RUN npm install

# Expose the port the application runs on
EXPOSE 3000

# Run the application
CMD ["npm", "start"]
```

This Dockerfile uses the official `node` image as the base image. It creates a directory for the application, copies the application files to the container, installs the application dependencies, exposes the port the application runs on, and runs the application.

### Build the Dockerfile
You can build the Dockerfile by running the following command:

```bash
docker build -t my-node-app .
```

This command builds the Dockerfile and tags the image with the name `my-node-app`.

### Running the Docker Container
You can run the Docker container by running the following command:

```bash
docker run -d -p 3000:3000 my-node-app
```

This command runs the Docker container in detached mode (`-d`) and maps port 3000 of the host machine to port 3000 of the container (`-p 3000:3000`). This allows you to access the Node.js application running in the container from your host machine.

You can show the content of the Node.js application by running the following command:

```bash
curl http://localhost:3000
```

Congratulations! You have successfully containerized a Node.js application and run it in a Docker container.

## (Optional) Enhancing the Node.js Application with Docker Compose
Docker Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services. Then, with a single command, you create and start all the services from your configuration.

In our case, we could enhance our Node.js application by adding a database service to it. For example, we could use simple additional service that will return a JSON object with the current time.

Therefore, create a new file called `time-service.js`:

```bash
mkdir time-service
cd time-service
touch time-service.js
```

Edit the `time-service.js` file with the following content:

```javascript
const http = require('http');
const os = require('os');

const hostname = os.hostname();

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'application/json');
  res.end(JSON.stringify({ time: new Date() }));
});

server.listen(3001, hostname, () => {
  console.log(`Time service running at http://${hostname}:3001/`);
});
```

This service will return a JSON object with the current time.

### Create a new Dockerfile for the Time Service
Create a new Dockerfile for the Time Service:

```bash
cd ..
touch Dockerfile.time-service
```

Edit the Dockerfile with the following content:

```Dockerfile
# Use the official "node" image as the base image
FROM node

# Create a directory for the application
WORKDIR /usr/src/app

# Copy the application files to the container
COPY time-service.js .

# Expose the port the application runs on
EXPOSE 3001

# Run the application
CMD ["node", "time-service.js"]
```

This Dockerfile uses the official `node` image as the base image. It creates a directory for the application, copies the application files to the container, exposes the port the application runs on, and runs the application.

Furthermore, let's change the `app.js` file to call the time service:

```javascript
const http = require('http');
const os = require('os');

const hostname = os.hostname();
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.write(`Hello World from ${hostname}\n`);

  const timeServiceReq = http.request('http://time-service:3001', (timeServiceRes) => {
    let data = '';

    timeServiceRes.on('data', (chunk) => {
      data += chunk;
    });

    timeServiceRes.on('end', () => {
      const time = JSON.parse(data).time;
      res.end(`The current time is: ${time}`);
    });
  });

  timeServiceReq.on('error', (e) => {
    console.error(`Problem with request: ${e.message}`);
  });

  timeServiceReq.end();
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

This change will call the time service and display the current time in the response.

### Install Docker Compose
To install Docker Compose, you can run the following commands:

```bash
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

docker-compose version
```

### Create a Docker Compose File
To start both services together, you can use Docker Compose. Create a new file called `docker-compose.yml`:

```bash
touch docker-compose.yml
```

Edit the `docker-compose.yml` file with the following content:

```yaml
version: '3'

services:
  my-node-app:
    build: 
      context: 
      dockerfile: Dockerfile
    ports:
      - "3000:3000"

  time-service:
    build:
      context: .
      dockerfile: Dockerfile.time-service
    ports:
      - "3001:3001"
```

This Docker Compose file defines two services: `my-node-app` and `time-service`. The `my-node-app` service uses the `Dockerfile` in the current directory to build the image, and the `time-service` service uses the `Dockerfile.time-service` in the current directory to build the image. Both services expose their respective ports to the host machine.

### Running the Docker Compose File
You can run the Docker Compose file by running the following command:

```bash
docker-compose up
```

This command will build the images for both services and start the containers. You can access the Node.js application at `http://localhost:3000` and the Time Service at `http://localhost:3001`.

### Testing the Node.js Application
You can test the Node.js application by running the following command:

```bash
curl http://localhost:3000
```

This command will return the response from the Node.js application, which should include the current time from the Time Service.

Congratulations! You have successfully enhanced your Node.js application with a Time Service using Docker Compose.



