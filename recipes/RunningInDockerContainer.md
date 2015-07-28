#How to run ArangoDB in a Docker container

##Problem

How do you make ArangoDB run in a Docker container?

##Solution

Running ArangoDB is as simple as...

    docker run -p 8529:8529 arangodb/arangodb

I've created an automated build repository on docker, so that you can easily start a docker container with the latest stable release. Thanks to frodenas, hipertracker, joaodubas, webwurst who also created dockerfiles.

Please note that this recipe is a general overview. There is also a recipe explaining how to install an application consisting of a node.js application and an ArangoDB database server.

You can get more information about [Docker and how to use it](https://github.com/arangodb/arangodb-docker) in our Docker repository.

###Usage

First of all you need to start an ArangoDB instance.

In order to start an ArangoDB instance run

    unix> docker run --name arangodb-instance -d arangodb/arangodb

By default ArangoDB listen on port 8529 for request and the image includes `EXPOST 8529`. If you link an application container, it is automatically available in the linked container. See the following examples.

###Using the instance

In order to use the running instance from an application, link the container

    unix> docker run --name my-app --link arangodb-instance

###Running the image

In order to start an ArangoDB instance run

    unix> docker run -p 8529:8529 -d arangodb/arangodb

ArangoDB listen on port 8529 for request and the image includes `EXPOST 8529`. The `-p 8529:8529` exposes this port on the host.

In order to get a list of supported options, run

    unix> docker run -e help=1 arangodb/arangodb

###Persistent Data

ArangoDB use the volume `/data` as database directory to store the collection data and the volume `/apps` as apps directory to store any extensions. These directory are marked as docker volumes.

See `docker run -e help=1 arangodb` for all volumes.

You can map the container's volumes to a directory on the host, so that the data is kept between runs of the container. This path `/tmp/arangodb` is in general not the correct place to store you persistent files - it is just an example!

    unix> mkdir /tmp/arangodb
    unix> docker run -p 8529:8529 -d \
              -v /tmp/arangodb:/data \
              arangodb/arangodb

This will use the `/tmp/arangodb` directory of the host as database directory for ArangoDB inside the container.

Alternatively you can create a container holding the data.

    unix> docker run -d --name arangodb-persist -v /data ubuntu:14.04 true

And use this data container in your ArangoDB container.

    unix> docker run --volumes-from arangodb-persist -p 8529:8529 arangodb/arangodb

If want to save a few bytes for you can alternatively use [tianon/true][3] or [progrium/busybox][4] for creating the volume only containers. For example

    unix> docker run -d --name arangodb-persist -v /data tianon/true true

###Building an image

Simple clone the repository and execute the following command in the arangodb-docker folder

    unix> docker build -t arangodb .

This will create a image named arangodb.

##Comment

A good explanation about persistence and docker container can be found here: 

* [Docker In-depth: Volumes][1]
* [Why Docker Data Containers are Good Using host directories][2]

**Author:** [Frank Celler](https://github.com/fceller)

**Tags:** #docker #howto

[1]: http://container42.com/2014/11/03/docker-indepth-volumes/
[2]: https://medium.com/@ramangupta/why-docker-data-containers-are-good-589b3c6c749e
[3]: https://registry.hub.docker.com/u/tianon/true/
[4]: https://registry.hub.docker.com/u/progrium/busybox/


