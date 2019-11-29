# Docker

## Abstract

Ever since Docker was released in 2013 it has becoming a new industry standard for running applications both in dev as well as production environments. Docker images are quite simple to build but, as any other powerfull tool, they have a few concepts that are easy to overlook.

This document aims to describe basic concepts of Docker and suggest a few good practices. It should be considered as a collection of various hints rather than comprehensive guide.

## Basics

### What is Docker

Docker is a broad term that actually can mean something totally different depending on exact context. It can mean Docker Image or Container running a specific application workload but this term is also used to refer to the actual applications responsible for running these workloads. Finally Docker might refer (though it's the least commonly used meaning) to a language used to write `Dockerfile`s to define Images.

It is mentioned in previous paragraph that there are multiple applications responsible for running Containers. These are:

1. Docker Daemon (or Engine). It is a process that manages various mechanisms and features of OS to ensure that Containers are running on the host operating system in a fully separated environment.
2. Docker CLI. A command line client that communicates with Docker Daemon.

Usually both these applications run on the same machine but that does not have to be the case every time. In certain situation it is possible to configure Docker CLI so that it will communicate with a Daemon running on a different machine - both virtual and remote.

### About Docker Daemon

Exact implementation details underlying containers concept is out of this article's scope but it is important to understand that Deamon actually needs to run as a root to make use of multiple kernel level concepts. This means that it is Docker's responsibility to restrict permissions of processes running within containers and improperly designed Images can be a serious threat to the system security.

### Images vs Containers

Main terms related to Docker are Images and Containers. Sometimes these terms are being mixed but in fact there's important difference between them and this should be clear.

Docker Image is a result of executing `docker image build` command. It is actually a definition of a workload and defines the environment in which given application workload will run. `Dockerfile` is used to define this environment. Every Image consists of multiple layers - one for each command within `Dockerfile` that are being cached and reused by Docker Engine. All image layers are read only.

Docker Container is actual running instance of the workload. If you're a developer you might think of it that in a way Container is a running instance of an Image in similar way like object is an instance of a specific class. Container adds a writable layer on top of Image's read-only ones.

This layer contains changes made while container is running and they can be saved as a new read-only layer to create new Docker Image.

## Implementation nuances

Exact implementation details underlying containers are way out of scope of this article but some concepts are worth mentioning. Check [this article](https://www.littleman.co/articles/what-is-a-container/) for more detailed explantion of containerization concepts.

### Image Layers

As mentioned above Docker Images consist of multiple layers that are put on top of previous ones. Thanks to Union File System each layer can change or add files in the filesystem that will be seen by resulting application. Old files however will be only hidden, not replaced - original files from previous layers will be still stored in resulting Image. 

For instance we check `mysql:5.7` Image to see how exactly it was built:

```
╰⎼$ docker image history mysql:5.7
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
1e4405fe1ea9        6 days ago          /bin/sh -c #(nop)  CMD ["mysqld"]               0B                  
<missing>           6 days ago          /bin/sh -c #(nop)  EXPOSE 3306 33060            0B                  
<missing>           6 days ago          /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B                  
<missing>           6 days ago          /bin/sh -c ln -s usr/local/bin/docker-entryp…   34B                 
<missing>           6 days ago          /bin/sh -c #(nop) COPY file:b3081195cff78c47…   12.7kB              
<missing>           6 days ago          /bin/sh -c #(nop)  VOLUME [/var/lib/mysql]      0B                  
<missing>           6 days ago          /bin/sh -c {   echo mysql-community-server m…   321MB               
<missing>           6 days ago          /bin/sh -c echo "deb http://repo.mysql.com/a…   56B                 
<missing>           6 days ago          /bin/sh -c #(nop)  ENV MYSQL_VERSION=5.7.28-…   0B                  
<missing>           6 days ago          /bin/sh -c #(nop)  ENV MYSQL_MAJOR=5.7          0B                  
<missing>           6 days ago          /bin/sh -c set -ex;  key='A4A9406876FCBD3C45…   30.2kB              
<missing>           6 days ago          /bin/sh -c apt-get update && apt-get install…   44.8MB              
<missing>           6 days ago          /bin/sh -c mkdir /docker-entrypoint-initdb.d    0B                  
<missing>           6 days ago          /bin/sh -c set -x  && apt-get update && apt-…   4.44MB              
<missing>           6 days ago          /bin/sh -c #(nop)  ENV GOSU_VERSION=1.7         0B                  
<missing>           6 days ago          /bin/sh -c apt-get update && apt-get install…   10.2MB              
<missing>           6 days ago          /bin/sh -c groupadd -r mysql && useradd -r -…   329kB               
<missing>           6 days ago          /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           6 days ago          /bin/sh -c #(nop) ADD file:2c1f5e08834f13ccb…   55.3MB 
```

This approach allows Docker to cache and reuse parts of the data in multiple Images. We could for instance change `groupadd` command in `Dockerfile` above. This would result with Image using exact same first 16 layers and introduce 3 new layers in the new branch - one for the command that was modified and one for each command that gets executed afterwards.

### Containers vs Virtual Machines

Both containers and VMs enable to encapsulate given workload from other processes running on the same hardware. They are not the same and it is important to understand the difference.

Virtual Machine is a full emulated computer running within host system. Through virtualization features it creates entire environment with separate CPU, memory and other hardware resources like graphic card. Within such virtualized hardware we can run any operating system we want. This virtualization layer provides much higher level of independence from host's hardware and OS but it comes for a price of high resource requirments used for the virtualization only.

Containers on the other hand are just isolated branches of host operating system process and filesystem trees. At the end of the day they use the same OS kernel and underlying hardware as host environment. This means that developer is limited to a single operating system (usually Linux but there are also Windows based containers nowadays) but can differentiate all the packages used within container (even different Linux distribution). In practical perspective resource cost of running container is much smaller than Virtual Machine.

As menetioned all the processes running in the host operating system which means they are visible in host's process list but with different PID:

```bash
╭tkaplonski@tkaplonski-XPS-15-9560:~
╰⎼$ docker container run -d --rm busybox sh -c "while [ 1 ] ; do echo "foo"; sleep 5; done;"
784d5d22439063ca1eef2b230f6581063a94658b0ae48ebe5e89ef927e06dd2e
╭tkaplonski@tkaplonski-XPS-15-9560:~
╰⎼$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
784d5d224390        busybox             "sh -c 'while [ 1 ] …"   4 seconds ago       Up 3 seconds                            dreamy_boyd
╭tkaplonski@tkaplonski-XPS-15-9560:~
╰⎼$ docker container exec -it dreamy_boyd sh
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sh -c while [ 1 ] ; do echo foo; sleep 5; done;
    9 root      0:00 sh
   14 root      0:00 sleep 5
   15 root      0:00 ps aux
/ # exit
╭tkaplonski@tkaplonski-XPS-15-9560:~
╰⎼$ ps aux | grep "sh -c while"
root      9545  0.1  0.0   1300     4 ?        Ss   08:02   0:00 sh -c while [ 1 ] ; do echo foo; sleep 5; done;
tkaplon+  9778  0.0  0.0   6208   892 pts/1    S+   08:03   0:00 grep --color=auto sh -c while
```

### Docker under various operating systems

As mentioned above, most Docker Images are Linux based and as such they require Linux kernel to run. For this reason Docker on Linux is "just" another process. In case of other operating systems however like OSX and Windows it requires virtual machine running to run Docker Daemon on Linux kernel. There are official applications that handle this virtualization (Docker for Mac and Docker for Windows) so that the behavior is almost seemless. However there is certain performance taxation on those operating systems.

Another thing worth mentioning is that Docker for Windows requires Windows 10 Professional because it uses HyperV, In case you are using older version of Windows or Win10 Home you can alternatively use Docker Toolbox that uses Oracle's VirtualBox for running Docker Daemon.

## Good practices

### Alpine vs Debian

There are two main distributions used as base for most Docker Images in the community: Alpine and Debian. Most official images by default use Debian as a preferred flavor but they are available also in Alpine version.

Most important difference is the size: Alpine is minimalistic so images based on it are smaller than those Debian based. In recent months however basic Debian distribution for containers got greatly reduced and the difference is not as big as it was in 2017 or earlier (around 100MB vs around 25MB usually).

THis minimalizm comes for a price of various nuances that might result with application misbehaving (i.e. Alpine uses buggy flavor of `awk` that can truncate big numeric values) so you should be very careful while deciding to go with Alpine distribution.

Also, from security perspective it's importatn to know that Alpine has issues with most CVE scanners.

Still, in certain scenarios small size of Alpine based images is a great advantage. It's also critical to realize risks related to this distribution in comparison to Debian ones.

### ADD vs COPY

Both `ADD` and `COPY` commands in `Dockerfile` can be used to add certain file or directory to the Image. The difference between them is that `ADD` has some extra capabilities that can result with side effects. For this reason it is considered best practice to use `COPY` unless those extra capabilities are needed in particular scenario.

Capabilities of `ADD`:

1. `ADD` can be used to download files from the internet while `COPY` works with local file system only.
2. `ADD` will automatically extract TAR archives and add the result of decompression as a folder within the Image.

### Docker CLI syntax

There are two flavors of Docker CLI commands:

1. Single level commands that existed from the very beginning of Docker
2. Management groups introduced in Docker June 2017 with version 1.13

Since Docker tries to keep backwards compatibility both versions are supported but managemenr groups offer much more capabilities. For consistency I strongly recommend management group approach, i.e. use `docker container ls` instead of `docker ps` or `docker image rm` instead of `docker rmi`.

### Order of commands in Dockerfile

Layered composition of Docker Images described above means that Deamon will try to reuse as much cached layers as possible but once certain Image diverge from existing order all following commands will actually create new layers. For this reason you should try to put commands that are most likely to change in the future at the end of Dockerfile.

### Multistage builds

TODO

## Docker Compose

TODO

## Learning materials

TODO