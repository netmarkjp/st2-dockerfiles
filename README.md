# StackStorm Docker Containers
[![Go to Docker Hub](https://img.shields.io/badge/Docker%20Hub-%E2%86%92-blue.svg)](https://hub.docker.com/r/stackstorm/)
> *Warning:*  Docker is not officially supported by StackStorm.<br>
> Repository provides basic skeleton, best effort helping you to [Build & Use](#build-and-deploy-stackstorm-components-to-docker-hub) own StackStorm images.<br>


This repository contains Dockerfiles for StackStorm. For a specific Dockerfile you can browse directly to a container sub directory.

## About StackStorm

[StackStorm](https://stackstorm.com/) is a powerful automation tool that wires together all of your apps, services and workflows. It’s extendable, flexible, and built with love for DevOps and ChatOps. It consists of bunch of components which bring Event-driven automation to you!

Here you can find Dockerfiles for *st2api, st2actionrunner, st2notifier etc* and all the components which StackStorm consists of.

## stackstorm container

Stackstorm container ([Dockerfile](Stackstorm/Dockerfile))  **is the base** for all of the **st2 components**. The full StackStorm stack is available inside the container. Notably that component containers such as *st2api, st2actionrunner etc* are basically the same container with the *stackstorm* container. The only hidden difference is the *ENV* settings. This approach minimizes the download snapshot when you want to run many stackstorm containers on the same node.

## Configuration

StackStorm components use Mongo database and RabbitMQ message queue service. When bringing up containers each component must know where db and queue are. All components except st2api also should know where st2api is.
There many ways to set this configuration, namely:

 - Using environment variables.
 - Using docker links.
 - Passing `/etc/st2/st2.conf` as volume.
 
These ways are sufficient for any use case, starting from a small docker compose development environment finishing with big discovery managed installations. Let's cover these.

### Using environment variables

St2 components the following environment variables:

 - **AMQP_URL** - url of rabbitmq, **default** is: `amqp://guest:guest@rabbitmq:5672/`.
 - **DB_HOST** - mongo database hostname or ip address, **default** is: `mongo`.
 - **DB_PORT** - mongo database listen port, **default** is: `27017`.
 - **API_URL** - stackstorm api endpoint. 

If you need any custom configuration, simple pass these environment variables to st2 docker containers and you are ready to go.


### Build your own StackStorm Docker images
Here is an example to build all StackStorm components and deploy them to Docker Hub.
It shows current automated CI & Deployment logic.

##### 1. Build `st2` and StackStorm components

Build all st2 components via docker-compose:

`st2` is base image, which will be used as parent for StackStorm components.

```sh
ST2_VERSION=2.0.1 ST2_REVISION=3 ST2_REPO=staging-stable docker-compose build
```

where `2.0.1` is version and `3` is revision number you can obtain from the [`PackageCloud repo`](https://packagecloud.io/StackStorm/staging-stable). 

##### 2. Usage
Start all st2 components via docker-compose:
```
ST2_VERSION=2.0.1 docker-compose up -d
```

Optionally run several `st2actionrunner` services:
```sh
ST2_VERSION=2.0.1 docker-compose scale actionrunner=4
```

You can use StackStorm now: 
```
# show st2 version
docker-compose run --rm client st2 --version

# list packs
docker-compose run --rm client st2 action list

# install github pack
docker-compose run --rm client st2 run packs.install packs=github
```

##### 4. Deploy to Docker Hub (optional)
```
docker login -e ${DOCKER_EMAIL} -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}

ST2_VERSION=2.0.1 
docker push stackstorm/st2actionrunner:${ST2_VERSION:?}
docker push stackstorm/st2api:${ST2_VERSION:?}
docker push stackstorm/st2auth:${ST2_VERSION:?}
docker push stackstorm/st2notifier:${ST2_VERSION:?}
docker push stackstorm/st2resultstracker:${ST2_VERSION:?}
docker push stackstorm/st2rulesengine:${ST2_VERSION:?}
docker push stackstorm/st2sensorcontainer:${ST2_VERSION:?}
docker push stackstorm/st2garbagecollector:${ST2_VERSION:?}
```
