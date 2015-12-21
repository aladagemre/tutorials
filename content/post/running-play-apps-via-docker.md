+++
tags = ["Scala", "Docker", "Play"]
Description = ""
date = "2015-10-03T16:40:59+02:00"
menu = "main"
title = "Running Play Apps via Docker"

+++


In this tutorial, I will guide you running your Play framework application (named MyProject in our tutorial) via Docker. I assume you have created a Play project with a build.sbt (or equivalent) file and make sure it's running on port 9000 with sbt run command.

Our demo project pulls tweets from Twitter of a specified person so it requires environment variables *CONSUMER_KEY, CONSUMER_SECRET, ACCESS_TOKEN, ACCESS_TOKEN_SECRET* to be set before running.

## Docker Installation

Make sure you have installed the latest version of Docker Toolbox. In order to use docker command, Docker needs some environment variables to be set. To do this, Docker introduces Docker QuickStart Terminal.

Run Docker Quickstart Terminal from Launchpad. A new terminal tab will open either on Terminal or iTerm app (depending on which one you chose at the installation). Check if any docker containers exist:

    $ docker ps -a

Now lets check the DOCKER_HOST (we will use this IP instead of localhost when reaching the app).

    $ echo $DOCKER_HOST

You will see something like tcp://192.168.99.100:2376. This means we will use IP address 192.168.99.100 for reaching our apps.

## Building Docker Image with SBT

Now that we have Docker installed and running, we can build a Docker image for our Play App. We can use SBT Native Packager plugin's docker feature to build a Docker image for the Play app (named MyProject). Before using the plugin, we have to add some config parameters to build.sbt of the Play project:

    dockerExposedPorts := Seq(9000, 9443)
    dockerExposedVolumes := Seq("/opt/docker/logs")

This will add instructions to the Dockerfile to expose 9000 and 9443 ports and let us read logs outside the container. Then build the image:

    $ cd MyProject
    $ sbt docker:publishLocal

Now check docker images:

    $ docker images

you will see a table listing all docker images available.


| REPOSITORY | TAG          | IMAGE ID     | CREATED       | VIRTUAL SIZE |
|------------|--------------|--------------|---------------|--------------|
| myproject  | 1.0-SNAPSHOT | cd25bf99efa9 | 2 minutes ago | 925.8 MB     |
| java       | latest       | 7547e52aac4b | 3 weeks ago   | 817.5 MB     |


## Creating the Container

We now see the image named myproject created for our app. Let's create a container with this image with environment variables and ports added.

    $ docker run --name MyProject -p 9000:9000 -e CONSUMER_KEY=abcd -e CONSUMER_SECRET=abcd -e ACCESS_TOKEN=abcd -e ACCESS_TOKEN_SECRET=abcd myproject:1.0-SNAPSHOT

This command means:

* use myproject image with 1.0-SNAPSHOT tag.
* expose port 9000 (right one) of inner Play app to 9000 (left one) of the outer system.
* name the container as myproject.
* give environment variables CONSUMER_KEY, CONSUMER_SECRET... with -e parameter for each.

Console will show logs for the app and navigate to [http://192.168.99.100:9000](http://192.168.99.100:9000) (the DOCKER_HOST) to reach the app. You can stop the container with Ctrl/CMD+C.

## Restarting the Container

    $ docker ps -a

you will see the container has exited.


| CONTAINER ID | IMAGE                  | COMMAND         | CREATED        | STATUS                      | PORTS | NAMES          |
|:-------------|:-----------------------|:----------------|:---------------|:----------------------------|:------|:---------------|
| c9b768dac51e | myproject:1.0-SNAPSHOT | "bin/myproject" | 10 minutes ago | Exited (130) 30 seconds ago |       | MyProject      |


Now restart the container (initials of the id is sufficient):

    $ docker start c9b
    $ docker ps -a

And we see the running containers:


| CONTAINER ID | IMAGE                  | COMMAND         | CREATED        | STATUS                      | PORTS                                 | NAMES      |
|--------------|------------------------|-----------------|----------------|-----------------------------|---------------------------------------|------------|
| c9b768dac51e | myproject:1.0-SNAPSHOT | "bin/myproject" | 11 minutes ago | Up 13 seconds               | 0.0.0.0:9000->9000/tcp, 9443/tcp      |  MyProject  |


## Stopping the Container

When you restart the container, it will run in the deamon mode. you can stop it via:

    $ docker stop c9b

We're done!
