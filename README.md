# - Docker In Depth

### difference between running container in the background vs foreground
![](./static/background_foreground.png)

    + in the background you can run console else you couldn't.
    $ docker run --rm busybox:1.24 sleep 1
    # docker container will run for 1 sec and then exit and delete

    $ docker run busybox:1.24 ls /
    # list all container contents

    $ docker run -i -t busybox:1.24
    # ls

    # we can run any command because we are inside the container

### + Docker commit :

    $ Docker run -it debian:jessie
    $ apt-get update && apt-get install -y git

    # -y automaticly respond yes to prompts

    $ Docker commit c6e91fa21856 debian:v1
    # commit image in this state with git inside it

    $ docker run -it debian:v1
    $ git --version
    git version 2.1.4

    #first image
    $ Docker run -it debian:jessie
    $ git --version
    git not found

### + Dockerfile :

    1- FROM debian:jessie -> base image could be ubuntu, centos ..
    2- RUN -> commands to be ran inside the container to install tools
    3- CMD -> commands to be ran inside the container to run an app only after starting the container.
    4- COPY -> COPY A FILE/FOLDER FROM HOST MACHINE TO CONTAINER (Project ..)
    5- ADD -> can COPY files and folders and also download files from internet,
             also uncompress packed folders

    Note that each instruction in a Dockerfile create a layer so :
	to reduce number of layers

    + good :

    FROM debian:jessie
    RUN apt-get update && apt-get install -y git vim

    Bad :

    FROM debian:jessie
    RUN apt-get update
    RUN	apt-get install -y git
    RUN apt-get install -y vim


    Example :

    CMD ["ECHO", "hello world"]

    + Alter CMD FROM Dockerfile to shell

    $ Docker run 26a967fa5806 echo hello docker
    -> hello docker


    COPY abc.txt /src/abc.txt

    + Dockerfile :

    FROM debian:jessie
    RUN apt-get update
    COPY abc.txt /src/abc.txt
    CMD ["echo","done"]

    $ docker build -t debian:v1

    $ docker run -it 26a967fa5806 ls /src
    abc.txt

    + Cache
    $ docker build -t debian:v1 . no-cache=true

    docker exec -it bash

    + Docker-compose
    $ cd dockerapp
    $ docker-compose up -d
    $ docker-compose ps
    $ docker-compose logs
    $ docker-compose stop
    $ docker-compose rm

    #build docker-cmpose if you made a change
    $ docker-compose build


    + Docker networking :
    -> None network :
![](./static/none_network.png)
    $ docker run -d --net none busybox sleep 1000
    # sleep 1000 will keep the container running

    $ docker exec -it cf85d10cf484a7fc3781710f20e5d7f9ea4146368e14a8336cd01c73e2a85e87 /bin/ash
    $ ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8): 56 data bytes
    ping: sendto: Network is unreachable

    -> the container is isolated

    + Advantages :
    - maximum level of network protection

    + Disadvantages :
    - not good if internet connexion is required.

    -> Bridge network :
![](./static/bridge_network.png)

    $ docker network inspect bridge
    # "Subnet": "172.17.0.0/16",
    # addr are from 172.17.0.0 to 172.17.255.255

    $ docker run -d --name container_1 busybox sleep 1000
    $ docker exec -it container_2 ifconfig
    # check eth0 172.17.0.2 -> [ 172.17.0.0 to 172.17.255.255 ]

    #if i add another container `container_2` and try to ping container_1,
    #I could because they are in the same network. -> bridge

    # create a new network
    $ docker network create --driver bridge my_bridge_network
    $ docker run -d --name container_3 --net my_bridge_network busybox sleep 1000
    $ docker exec -it container_3 ifconfig
    #range [172.23.0.0 -> 172.23.255.255]

    #ping container_1 from container_3
    $ docker exec -it container_3 ping 172.17.0.2
    # failed because they are not in the same network

    $ docker network connect bridge container_3
    $ docker exec -it container_3 ping 172.17.0.2

    !SUCCESS

    $ docker exec -it container_3 ifconfig
    # you will find 3 interfaces

    $ docker network disconnect bridge container_3

     + Advantages :
    - containers have access to two interfaces -> loopback and private network
    - all containers in the same bridge could communicate with each others.

    -> Host network :

    $ docker run -d --name container_4 --net host busybox sleep 1000
    # open container have access to all interfaces of the host.
    # check
    $ docker exec -it container_4 ifconfig

    # overlay network
![](./static/overlay_network.png)
![](./static/network_isolation.png)
