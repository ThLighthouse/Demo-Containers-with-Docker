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
