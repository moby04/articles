# Containerized app overview

## Abstract

First steps towards containerized application design might seem overwhelming. It requires to understand a few concepts and use a collection of tools that work together
to enable multiple layers of architecture. Understanding of how these layers work together and what is the exact purpose of each tool in the chain is crucial to make best use of the patterns.

This article tries to explain what are these layers on a high level without diving deeply into mechanisms _under the hood_.

## Distributed application concept

### Application architecture

As the complexity of IT system grown separation of concerns became crucial principle of designing application. In fact, any web based application is in fact distributed as it consists
of a number of subprocesses working together to fullfil certain task. In most basic setup PHP application consists of:

* Webserver responsible for handling requests, executing PHP scripts and serving static files
* PHP interpreter that gets triggered by webserver or other way (command line, cronjob)
* Database engine responsible for storing persistent data

Naturally most systems nowadays consist of many other processes like Redis, Memcache, Elasticsearch and much more. And each of these processes is in fact a building block of entire application.

In the past we have considered these building blocks a monolith structure - all of these processes were running in a single environment like LAMP (with many letters that could extend this
acronym in fact). This resulted with multiple issues like dependency handling or resource requirement estimation to name a few.

Many of these challenges can be addressed by splitting a monolith into multiple separate blocks treated independently. And at some point web architectures evolved so that RDBMS got moved away from
webserver to a separate machine. So did cache related apps like Redis, Memcache or Varnish.

At the same time it has become clear that there might be specific tasks that could be further separated within a single application. This could mean running separate PHP server for cronjobs or
introducing specialized workers that would do some operations in background.

### Scaling of the application

As the popularity of given system grows it needs to be able to handle requests from growing amount of users. And that means growing resource requirements. Since we already partitioned our
monolith into multiple separate machines, we could start scaling these blocks independently.

There are generally two directions of scaling: vertical and horizontal. First would mean adding more resources to given machine - putting better CPU, more RAM, etc. Horizontal scaling though
means adding another server with the same responsibility.

Vertical scaling has its limits. Undeniable one is the fact that we are simply limited by quarz based technology physical capabilities. Another important factor is that, the process of adding
or removing resources from a running instance is a complex process that requires a lot of time.

Horizontal approach helps overcoming these limits and in addition offers additional benefit of redundancy.

### 12 Factor App

As far as horizontal scaling is preffered way of handling growing popularity, it requires certain discipline in the way the application is designed. [12 Factor App](https://12factor.net/) 
is a set of principles originally suggested by creators of Heroku platform that describe how to desing a scalable application.

Each factor is described separately on the [The Twelve-Factor App](https://12factor.net) website and I strongly encourage reading all of them:

1. [Codebase](https://12factor.net/codebase)
2. [Dependencies](https://12factor.net/dependencies)
3. [Config](https://12factor.net/config)
4. [Backing Services](https://12factor.net/backing-services)
5. [Build, release, run](https://12factor.net/build-release-run)
6. [Processes](https://12factor.net/processes)
7. [Port binding](https://12factor.net/port-binding)
8. [Concurrency](https://12factor.net/concurrency)
9. [Disposability](https://12factor.net/disposability)
10. [Dev/prod parity](https://12factor.net/dev-prod-parity)
11. [Logs](https://12factor.net/logs)
12. [Admin processes](https://12factor.net/admin-processes)

### Containers as distributed application

You might have noticed that this article did not speak of containers at all. The reason is that distributed application that can be scaled horizontaly can be implemented without containerization.

Containers however help to address challenges related to distributed application design and you will find that Docker best practices actually follow 12 Factors above quite naturally. In fact
some of the principles are fullfiled pretty much _out of the box_.

At the same time, container is just a concept that helps dividing responsibilities and separate application logic from underlying architecture. It is a technology that enables responsibility
separation of concerns. Entire technology stack consists of more tools but containers are critical building block.

## Ecosystem overview

### Docker

Docker is a broad term that will be described in more details in separate article. For now it should be considered a building block that enables to encapsulate a single process along with all
required dependencies in a single box. Developer's responsibility is to prepare this box in a way that can be reused without worrying about where exactly it will be used. 

Looking from this perspective any process is just a container that gets some resources (CPU, Memory, disk space), receives some messages (requests), answers to them (responses) or it might
request some information from other processes. No matter what exact task is the process doing or how complex this process is - from this point of view this process is pretty much identical
with any other.

A good analogy here is spedition. Global companies like UPS or DHL are able to transport anything around the world. As long as it is packed into boxes compliant with their requirements. These
companies do not care if the box you want to send contains electronics, new tires or perfumes. It would be a problem if you requested a courier to take a pile of sand and deliver it to someone
but if you put all this sand into the box and label it properly - that will be fine.

Note that as a sender you also do not care how exactly you package will be transported. It might be traveling on a track, train, airplane... as far as you are interested the courier could go to
the other continent by foot - as long as it gets delivered.

The same way developers do not need to worry about entire infrastructure on which their application will run. It might be executed on a single machine or in complex system that involves 
tens or even hundreds of servers spread around the world. Exact configuration of this runtime environment is usually beyond developer's interest as it is set up by people in different departments
like SysOps or DevOps.

Docker serves as a communication layer between developer and devops. It is the container into which developer packs the application and which then gets shifted and put in place by devops.

### Docker Compose

As explained above, any application actually consists of multiple processes that work together to fulfill given task. These processes are encapsulated in separate containers that communicate with
each other in specific way. This means that they need to be connected with a network.

Also data processed by containers needs to be stored in a safe, persistent way. It will be explained later that containers themselves are ephemeral and no data that exists within them will be
saved once containers gets deleted. To overcome this, concept of volumes that can be attached to the container was introduced.

Now, developer could run his local dev environment by set of Docker commands. This would be daunting and error prone task. Application described above would require following commands as minimum:

1. Creation of virtual network for communication between containers (`docker network create`)
2. Creation of volume for database files (`docker volume create`)
3. Running webserver (`docker container run`)
4. Running of php-fpm (`docker container run`)
5. Running od mysql (`docker container run`)

Each of these command would come with complex list of parameters to connect everything correctly. And this minimum does not including building of the images yet which would require another
3 commands.

To overcome this, `Docker Compose` was introduced. It is a wrapper around Docker CLI that introduces declarative way to describe entire application environment in YAML format so that developers can run
entire local system with a single command (`docker-compose up`).

Clearly, it is a major facilitation in day to day work. Still, one need to understand basic concepts of Docker itself before learning how to write their own `docker-compose` files.

### Orchestration

At the other end of the, there are orchestration engines. It might mean Docker Swarm, Kubernetes, AWS ECS or Azure ServiceFabric - they all use Docker containers as a building blocks for the
applications.

They offer many complex mechanism for handling logs, provisioning persistent storage, distributing containers between available machines (nodes), etc. All this complexity however is abstracted
by Docker's plugin mechanisms so that they can run any Docker container in a way transparent for the application.
