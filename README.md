# Developing with Docker

A hands-on demo project completed during the Docker module of the [Techworld with Nana DevOps Bootcamp]
(https://techworld-with-nana.teachable.com/courses/).

The project demonstrates how a Node.js application can be connected to MongoDB, packaged into a Docker image, stored in a private Nexus registry, and deployed as part of a multi-container environment using Docker Compose.

> The original application and starter Docker configuration were provided by Nana Janashia for the DevOps Bootcamp.
> My work in this repository documents the hands-on implementation, configuration, deployment, troubleshooting, and lessons learned while following the course.

##  Application Overview

The application is a simple user profile page where profile information can be saved to and retrieved from MongoDB.

The stack consists of:

- a frontend written with HTML, CSS, and JavaScript;
- a Node.js backend using Express;
- MongoDB for storing profile data;
- Mongo Express for managing MongoDB through a web interface;
- a Dockerfile for building the application image;
- Docker Compose for managing the containers;
- a Docker Volume for persistent MongoDB data;
- Nexus Repository as a private Docker registry.

## Project Implementation Stages

This demo project was developed step by step.

### 1. Runing the Application Locally

I cloned the original demo project and tested Node.js application locally;

```
cd app
npm install
node server.js
```

The application became available at:

`http://localhost:3000`

At this stage, the Node.js application was running directly on the host machine.

### 2. Running MongoDB Containers manually

I pulled and started MongoDB and Mongo Express containers using Docker commands.

First, I created a dedicated Docker network:

`docker network create mongo-network`

Then I started MongoDB:

```
docker run -d \ 
-p 27017:27017 \ 
-e MONGO_INITDB_ROOT_USERNAME=admin \ 
-e MONGO_INITDB_ROOT_PASSWORD=password \ 
--net mongo-network \ 
--name mongodb \ 
mongo
```
After that, I started Mongo Express:

```

docker run -d \
-p 8081:8081 \
-e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \ 
-e ME_CONFIG_MONGODB_ADMINPASSWORD=password \ 
-e ME_CONFIG_BASICAUTH_USERNAME=user \ 
-e ME_CONFIG_BASICAUTH_PASSWORD=pass \ 
-e ME_CONFIG_MONGODB_SERVER=mongodb \ 
-e ME_CONFIG_MONGODB_URL=mongodb://mongodb:27017 \ 
--net mongo-network \ 
--name mongo-express \ 
mongo-express
```
Mongo Express became available at:

`http://localhost:8081`

Inside Mongo Express, I created:

- database: `user-account`
- collection: `users`

The Node.js application was still running locally, while MongoDB and Mongo Express were running in containers.

### 3.Connecting Containers through a Docker Network

MongoDB and Mongo Express were connected to the same custom Docker network.

Inside this network, Mongo Express could reach MongoDB using the container name:
This demonstrated that containers on the same Docker network communicate using container or service names instead of host IP addresses.

### 4.Managing Services with Docker Compose

After running the containers manually, I used Docker Compose to describe and manage MongoDB and Mongo Express in one configuration file.

Start the services:

`docker compose -f docker-compose.yaml up .`

Check their status:

`docker compose ps`

View their logs:

`docker compose logs -f`

Stop the services:

`docker compose down`

At this stage, Docker Compose started MongoDB and Mongo Express, while the Node.js application could still be started locally from the `app` directory.

## Building the Application Image

I used the Dockerfile to package the Node.js application and its dependencies into a Docker image:

`docker build -t my-app:1.0 .`

I then checked that image was created:

`docker image my-app`

The resulting image contains:

- the Node.js runtime;
- the application source code;
- npm dependencies;
- the command required to start the application.

When the application runs inside Docker Compose, it must connect to MongoDB using the Compose service name:

`mongodb://admin:password@mongodb`

It must not use `localhost`, because `localhost` inside the application container refers to the application container itself.

## Private Nexus Docker Registry

For the Private Nexus Registry I created EC2 Instance in AWS and configured a Docker-hosted repository in Nexus and used it as a private Docker image registry.

The Nexus services used:

- port `8081` for the Nexus UI;
- port `8083` fo the Docker-hosted repository.

### Authenticate with the Registry

```bash
`docker login <nexus-host>:8083`
```

This command authenticates the local Docker client with the private Docker registry.
Also configured a file /etc/docker/daemon.json to connect to the repository via http

`
{
	"insecure-registries" : ["<repostiry-host>:8083"]
}
`



### Part of Nana's README.md file

## demo app - developing with Docker

This demo app shows a simple user profile app set up using 
- index.html with pure js and css styles
- nodejs backend with express module
- mongodb for data storage

All components are docker-based

### With Docker

#### To start the application

Step 1: Create docker network

    docker network create mongo-network 

Step 2: start mongodb 

    docker run -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --name mongodb --net mongo-network mongo    

Step 3: start mongo-express
    
    docker run -d -p 8081:8081 -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin -e ME_CONFIG_MONGODB_ADMINPASSWORD=password --net mongo-network --name mongo-express -e ME_CONFIG_MONGODB_SERVER=mongodb -e ME_CONFIG_MONGODB_URL=mongodb://mongodb:27017 mongo-express   

_NOTE: creating docker-network in optional. You can start both containers in a default network. In this case, just emit `--net` flag in `docker run` command_

Step 4: open mongo-express from browser

    http://localhost:8081

Step 5: create `user-account` _db_ and `users` _collection_ in mongo-express

Step 6: Start your nodejs application locally - go to `app` directory of project 

    cd app
    npm install 
    node server.js
    
Step 7: Access you nodejs application UI from browser

    http://localhost:3000

### With Docker Compose

#### To start the application

Step 1: start mongodb and mongo-express

    docker-compose -f docker-compose.yaml up
    
_You can access the mongo-express under localhost:8080 from your browser_
    
Step 2: in mongo-express UI - create a new database "user-account"

Step 3: in mongo-express UI - create a new collection "users" in the database "user-account"       
    
Step 4: start node server 

    cd app
    npm install
    node server.js
    
Step 5: access the nodejs application from browser 

    http://localhost:3000

#### To build a docker image from the application

    docker build -t my-app:1.0 .       
    
The dot "." at the end of the command denotes location of the Dockerfile.

## Amazon ECR — Private Docker Registry

As part of the AWS Services module of the TechWorld with Nana DevOps Bootcamp, I extended this Docker project by using **Amazon Elastic Container Registry (ECR)** to store and manage Docker images in AWS.

The goal was to create a private ECR repository, build Docker images locally, and push two different versions of the application to the registry.

### 1. Creating a Private ECR Repository

I created a private repository through the AWS Management Console with the following configuration:

- **Repository name:** `devops-bootcamp/js-app`
- **AWS Region:** `eu-central-1` (Frankfurt)
- **Image tag mutability:** Mutable
- **Encryption:** AES-256

### 2. Authenticating Docker with Amazon ECR

First, I installed AWS CLI v2 on my local Linux machine and authenticated using an existing IAM user through a named AWS CLI profile.

```bash
aws login --profile devops-bootcamp --region eu-central-1
```

I verified the AWS identity:

```bash
aws sts get-caller-identity --profile devops-bootcamp
```

Then I authenticated Docker with the private ECR registry:

```bash
aws ecr get-login-password \
  --region eu-central-1 \
  --profile devops-bootcamp \
| docker login \
  --username AWS \
  --password-stdin \
  <account-id>.dkr.ecr.eu-central-1.amazonaws.com
```

This uses a temporary authentication token obtained through AWS CLI.

### 3. Building and Pushing the First Docker Image

I reused the existing Node.js application and Dockerfile from the Docker module.

First, I built the image locally:

```bash
docker build -t js-app:ecr-demo .
```

Then I tagged the image with the ECR repository address:

```bash
docker tag js-app:ecr-demo \
  <account-id>.dkr.ecr.eu-central-1.amazonaws.com/devops-bootcamp/js-app:ecr-demo
```

Finally, I pushed the image to Amazon ECR:

```bash
docker push \
  <account-id>.dkr.ecr.eu-central-1.amazonaws.com/devops-bootcamp/js-app:ecr-demo
```

The image was successfully uploaded and became available in the private ECR repository.

### 4. Building and Pushing a Second Version

To practice managing different image versions, I made a small change to the application.

In `app/index.html`, I updated the page heading to:

`User Profile - ECR Demo v2`

I rebuilt the application with a new image tag:

```bash
docker build -t js-app:ecr-demo-v2 .
```

Then I tagged and pushed the updated image to the same ECR repository:

```bash
docker tag js-app:ecr-demo-v2 \
  <account-id>.dkr.ecr.eu-central-1.amazonaws.com/devops-bootcamp/js-app:ecr-demo-v2

docker push \
  <account-id>.dkr.ecr.eu-central-1.amazonaws.com/devops-bootcamp/js-app:ecr-demo-v2
```

### 5. Final Result

Both Docker images were successfully pushed to Amazon ECR:

- `ecr-demo` — initial application version.
- `ecr-demo-v2` — updated application version.

I verified both image tags in the AWS ECR Console.

### What I Learned

- Creating and configuring a private Amazon ECR repository.
- Authenticating AWS CLI using an IAM user and named profile.
- Authenticating Docker with ECR using temporary credentials.
- Building and tagging Docker images for a private registry.
- Pushing different versions of an application to the same repository.
- Understanding the difference between `docker build`, `docker tag`, and `docker push`.

This demo helped me understand how Docker images can be stored and versioned in AWS, building on my previous experience with Docker Hub and Nexus Repository.

#### Final results

![Docker images stored in Amazon ECR](screenshots/ecr-images.png)