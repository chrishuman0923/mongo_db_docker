# MongoDB via Docker

## Table of Contents

- [Overview & Purpose](#overview)
- [Pre-Requisites](#pre-reqs)
- [Installation](#install)
- [Start the Container](#start)
- [Login to the Container](#login-container)
- [Login to MySQL Server](#login-server)
- [Connect MongoDB Compass](#compass)
- [Persistent Data](#persistent)
- [To Do's](#todos)

<a name="overview" />

## Overview & Purpose

This repo provides a step-by-step guide for getting MongoDB up and running on your development machine using Docker and Docker Compose.

You may be asking yourself:

> "Why would I want MongoDB running in a container rather than directly on my machine?"

1. There is less clustter on your device.
1. You can spin up multiple MongoDB instances using slightly different verisons to cater to different projects or to experiment before making the leap to a new version.
1. You can reliably create identical / consistent MongoDB instances independent of your device's OS...as long as the OS supports Docker 🙂
1. Your development environment is "documented", automated, and reproducible.

<a name="pre-reqs"></a>

## Pre-Requisites

1. Have Docker and Docker Compose installed on your device.
1. Have access to the `docker` and `docker-compose` CLI commands.
   - You can test this by running `docker -v` & `docker-compose -v`
     - If either of these commands do no return a version or build number, do **NOT** continue until you have them working.
1. Run `docker ps` to confirm Docker engine is currently running.

<a name="install"></a>

## Installation

1. Clone this repo

   ```
   git clone https://github.com/chrishuman0923/mongodb_docker.git
   ```

1. Create an .env file at the root of this directory for sensitive values using the following template:

   ```
   # The username for a new user
   DB_USER=

   # The password for the new user defined above.
   DB_PASS=
   ```

1. Confirm that the MongoDB version you want to use is defined in docker-compose.yml
   - If you are unsure which version of MongoDB to use, consult the MongoDB Docker page [here](https://hub.docker.com/_/mongo) or just use `mongo:latest` to get the latest version available.

<a name="start"></a>

## Start the Container

1. Open a terminal inside this directory and run the following command.

   ```
   docker-compose up -d
   ```

   - This creates and runs the container as defined by the docker-compose.yml file

   - The `-d` flag simply means to run the container in the background (aka "detached mode").

   - "Detached mode" allows you to regain use of your terminal after the container is started.

If you want to see the logs produced by the running container while in detached mode, you need to login to the running container with the terminal command `docker -it {CONTAINER_NAME} sh` and review the log files produced.

However, two easier ways to review the logs of a container are to print the live logs to the terminal with `docker logs {CONTAINER_NAME} --follow` or view them in the Docker Dashboard GUI.

<a name="login-container"></a>

## Login to the Container

Once the container is up and running, you should be able to view it using `docker ps` and login to it using `docker -it {CONTAINER_NAME} sh`.

- In this case, the container name is defined in the docker-compose.yml on line #6.

<a name="login-server"></a>

## Login to MongoDB

1. Once logged into the container, you will need to then login to MongoDB using `mongo -u {username} -p`.

1. You will be prompted to enter the password for the user.

<a name="compass"></a>

## Connect MongoDB Compass

Now, you may be thinking:

> That is all well and good that I can access my database via the terminal but I am a developer that still likes a GUI. How do I connect my other tools to the database now that it is inside a container? Am I out of luck?

**Of course not!!**

If you look back at our docker-compose.yml you will notice the `ports` definition on line #17. This is our ticket to connecting outside services to our database.

What line #17 and #18 are doing is mapping a port on our host machine (the device running docker) to a port within our container.

`27017:27017` is read as:

> Map all traffic from `localhost:27017` to `container:27017`.

This means that we can set tools like MongoDB Compass to connect to our database on `localhost:27017` and it will be re-routed into our container and reach our database.

Now, `27017` is the default listening port for MongoDB and I made the ports match for simplicity.

However, lets say that `27017` on my machine was already occupied by something. I could have changed the host port to anything and still connect to my database.

E.G. If I had set the port definition in docker-compose.yml to `8080:27017` then I could still connect to my database by accessing `localhost:8080` and it would still be mapped to `container:27017`.

<a name="persistent"></a>

## Persistent Data

Now, a typical problem with containerizing applications is the non-persistent storage. This means that when the contain stops, the data is gone. Now, for a database, that is un-acceptable! We need our data to be retained regardless how many times we stop and start our container. How do we fix that?

The answer is a persistent volume.

In essence, we need to map the directory(ies) in our container that hold our data to a location on our host machine.

In our docker-compose.yml (line #24), you can see how we mapped our persisent volume. This reads the same as our port mapping.

> Map all the information to the `./db/data` directory on our host machine from the `/data/db` directory in the container.

In MongoDB, the `/data/db` directory holds all of the data from the server.

This means that as long as we retain the `./db/data` directory, we can stop, start, and delete our container without fear of losing any information.

<a name='todos'></a>

## To Do's

- [ ] Research proper methods for authentication and authorization of users within container
