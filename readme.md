# 3 Tier Application

- [3 Tier Application](#3-tier-application)
  - [Introduction](#introduction)
  - [Prepare the dockerfile's](#prepare-the-dockerfiles)
    - [Database](#database)
    - [Service](#service)
    - [Frontend](#frontend)
  - [Create the `docker-compose.yaml` and connect the tiers](#create-the-docker-composeyaml-and-connect-the-tiers)
    - [DB](#db)
    - [Service](#service-1)
    - [App](#app)
  - [Running the three-tier application and conclusion](#running-the-three-tier-application-and-conclusion)
  - [Networking and conclusion](#networking-and-conclusion)

## Introduction
The goal of this is to create a simple three tier architecture which contains three separate docker containers which are combined by using the docker-compose.yml file.
It will consist of the following parts:

**Database tier**: A simple postgres db with one table 
**Service tier**: A  node app that has a get endpoint to get and send data from the DB.  
**Frontend tier**: Basic node web app which uses an express app to make a request to the service tier and send a html table with the data as response to the browser

## Prepare the dockerfile's
First we need to create the docker files for each of the tiers.

### Database
 The only thing we do is creating an `init.sql` file which contains some sql code to insert sample data to our DB.

### Service
Navigate into the service folder and create new project with `npm init`. Now create a `server.js` file and copy the sample code in. Now run `npm install express pg` to install the dependencies. Also create a `.dockerignore` file and add the `node_modules` folder and the `npm-debug.log` file to it.

### Frontend
For the frontend we create a new node project and create an `index.js` file and paste the example code in it. Run `npm install request express` to get the dependencies. Next we createa docker file which is the exact same as for the backend, except we expose port 3001 instead of 3000. Again create a .dockerignore file with the same content (node_modules and npm-debug.log )

## Create the `docker-compose.yaml` and connect the tiers
Now we can create the `docker-compose.yaml` file in the root directory. In there we need to create a service for each tier. 

### DB
For the database we  use a public image for postgres. And we also set environment variables which the image requires to setup the DB credentials and db name. An important step is to mount our folder db to the init volume of the DB container. Because every .sql file which is in that init directory is executed when the db starts up. Like that we are sure that we have the demo data available.

### Service
For the service we specify the path were our dockerfile is for the build property, like that docker-compose knows which image it needs to build/use. We also need to specify an environment variable for the connection string to our database, because we use that in the code to connect to the DB. Finally we also need to specify that the service depends on the db, to make sure the db is always started up first, else we could not connect to it.

### App
For the app we  specify the path to the dockerfile. Then we set it dependent on the service to make sure the API is available to our frontend app before we start it. 
On the app for the frontend we also want to create a port mapping for port 3000, so we can access it from our host machine.

## Running the three-tier application and conclusion
We have no successfully configured a three tier application with docker. To run it we use `docker-compose up --build` ,the `--build` makes sure that we build the image, if it is not yet available.
If everything is correct you should be able to go to `http://localhost:3000/`

## Networking and conclusion
 Our compose basically has two networks, `network-service` and `network-app`. The `network-service` allows the db and service tiers to communicate to each other. While the `network-app` connects the app and the service tier together. That means that the app can access the API inside the service layer but cannot access the db layer.
