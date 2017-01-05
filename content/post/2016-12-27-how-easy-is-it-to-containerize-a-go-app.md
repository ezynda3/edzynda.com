---
date: "2016-12-27T09:18:06-08:00"
title: "How Easy Is It To Containerize A Go App?"
tags:
- Golang
- Docker
- Containers
- Websockets
image: "/images/go-docker-header.png"
---

Recently, I wrote an article on how to [create a simple chat server using WebSockets and Go](https://scotch.io/bar-talk/build-a-realtime-chat-server-with-go-and-websockets). I made the [source code](https://github.com/scotch-io/go-realtime-chat) available and it's pretty simple to get the example up and running if you have a basic familiarization with Go. I wanted to see if I could make it easier for people to try out though. For this, I looked to the very popular [Docker](https://docker.io) project.

Docker has been around for quite some time but, I only recently began toying around with it more seriously. At first it seems pretty complicated but as it turns out, it's really simple to use once you get the hang of the basic commands and concepts.

This is not going to be a tutorial how to get started with Docker though, I'm just going to run through the steps of how I turned my chat application into a redistributable container.

The first step is obviously to have Docker installed for your specific OS. Head on over to the [Docker website](https://docker.io) to fetch that.

The next step is to create a Dockerfile in the same directory you have your app source code. This is just a file that will tell Docker how to build your container. The Dockerfile for my chat application looks just like this.

```
FROM golang:1.7

MAINTAINER Ed Zynda

RUN mkdir /app
COPY . /app

WORKDIR /app/src
RUN go get -u github.com/gorilla/websocket
RUN go build main.go

CMD ["./main"]
```

Let's have a look at this line by line.

```
FROM golang:1.7
```

This tells Docker that I want to start out with an official Docker container called Golang and use the tagged version 1.7. This container has all of the Golang build tools preinstalled and will provide a good base for us to build upon.

The Docker hub has thousands of containers provided by Docker and the community which you can download and build upon yourself. There are even containers that literally have nothing in them if you want to keep your final container as light as possible.

```
MAINTAINER Ed Zynda
```

This just says that I'm the maintainer of this container. I'm a poet and I didn't even know it!

```
RUN mkdir /app
COPY . /app
```

The `RUN` keyword allows us to run any command we wish inside our container. In this case I just want to create a new directory to hold our application. Next I use the `COPY` keyword to move my source code into the container.

```
WORKDIR /app/src
```

This tells Docker that I now want to run commands from within the `/app/src` directory. This is just like running `cd /app/src`.

```
RUN go get -u github.com/gorilla/webSocket
RUN go build main.go
```

In this step, I'm just running the normal Go commands to pull down dependencies and then compile the application.

```
CMD ["./main"]
```

The final step is to tell Docker which command to run when the container is started. In this case I just want to run the binary called `main` which is the result of compiling the Go source.

After creating my Dockerfile, I can now build the container by running the following command.

```bash
docker build -t ezynda3/go-chat
```

This will build the container, following the steps in the Dockerfile and then tag it with `ezynda3/go-chat`. Since I didn't provide a version number the version will automatically be set to `latest`.

This container only exists on my machine right now though. In order to distribute it so others can use it, I need to publish it to a hub. Docker provides a free hub for public containers at [hub.docker.com](https://hub.docker.com). To push the container up just run the following.

```bash
docker push ezynda3/go-chat
```

Docker may complain about invalid credentials. In that case run the following after ensuring you have created an account on Docker's hub site.

```bash
docker login
```

It will prompt you for your credentials and you should be set to run the previous command again.

Now on any machine that has Docker installed, we can run

```bash
docker pull ezynda3/go-chat
docker run -p 8000:8000 ezynda3/go-chat
```

This will pull the container and then run it mapping port `8000` from within the container to port `8000` on the host machine.

As you can see, it was pretty easy to create a nice simple container for a Go application.
