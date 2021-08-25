# Networking with Docker-Compose: A Simple Ping Example

https://grahambryan.medium.com/networking-with-docker-compose-a-simple-ping-example-7b6979dfc94b

Networking with Docker-Compose: A Simple Ping Example
By Graham Bryan https://gist.github.com/grahambryan

Sep 23, 2020·2 min read




TL;DR The networking capabilities Docker-Compose abstracts for docker users has significantly helped my mental state and should help yours too! Stop using bash scripts and hardcoding IP addresses for your docker container network. It leads to hacky solutions and can be prone to errors.

- Docker-Compose creates a single network for which all of its services can communicate over. 
- This feature makes docker-compose an incredibly powerful tool. 
- You will no longer have the self induced headache of remembering the IP address’ for all the containers you launched or creating your own docker network from scratch.

In this article, Ill show you a simple example of this awesome feature docker and docker-compose gives us.

For the example, I have put together a custom docker image based off the latest ubuntu with iputils-ping installed. The Dockerfile looks like this:

```java
FROM ubuntu
RUN apt-get update && \
    apt-get install -y iputils-ping
CMD bash
```

Below is a docker-compose.yml file I have put together in order to demonstrate the network. 

There are two services, service-1 and service-2, that both spawn from the image, my-image, based off the Dockerfile above.

```java
version: "3"
services:
  service-1:
    image: my_image
    build:
      context: .
    container_name: service-1
    command: "ping service-2 -c 3"
  service-2:
    image: my_image
    container_name: service-2
    command: "ping service-1 -c 3"
```    

The application of each service is to try and ping the other 3 times, ping <service name> -c 3.With docker-compose, it will automagically create a single network under the hood for you and map the containers IP directly to the service name so it can be resovled. Once you launch the services with docker-compose up the following is the result:

```java
root@docker-ubuntu-s-1vcpu-2gb-nyc1-01:~/ap/ping-docker-compose# docker-compose up
Creating network "ping-docker-compose_default" with the default driver
Building service-1
Step 1/3 : FROM ubuntu
latest: Pulling from library/ubuntu
16ec32c2132b: Already exists
Digest: sha256:82becede498899ec668628e7cb0ad87b6e1c371cb8a1e597d83a47fac21d6af3
Status: Downloaded newer image for ubuntu:latest
 ---> 1318b700e415
Step 2/3 : RUN apt-get update &&     apt-get install -y iputils-ping
 ---> Running in 055f1fab9601
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:3 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [30.6 kB]
Get:4 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [1034 kB]
Get:5 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [792 kB]
Get:6 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [488 kB]
Get:7 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:8 http://archive.ubuntu.com/ubuntu focal-backports InRelease [101 kB]
Get:9 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]
Get:10 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [177 kB]
Get:11 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [33.4 kB]
Get:12 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [1275 kB]
Get:13 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [33.8 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [1065 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [534 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [1471 kB]
Get:17 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 Packages [2668 B]
Get:18 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [6324 B]
Fetched 18.9 MB in 2s (7665 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  libcap2 libcap2-bin libpam-cap
The following NEW packages will be installed:
  iputils-ping libcap2 libcap2-bin libpam-cap
0 upgraded, 4 newly installed, 0 to remove and 4 not upgraded.
Need to get 90.5 kB of archives.
After this operation, 333 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu focal/main amd64 libcap2 amd64 1:2.32-1 [15.9 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal/main amd64 libcap2-bin amd64 1:2.32-1 [26.2 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal/main amd64 iputils-ping amd64 3:20190709-3 [40.1 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal/main amd64 libpam-cap amd64 1:2.32-1 [8352 B]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 90.5 kB in 0s (232 kB/s)
Selecting previously unselected package libcap2:amd64.
(Reading database ... 4127 files and directories currently installed.)
Preparing to unpack .../libcap2_1%3a2.32-1_amd64.deb ...
Unpacking libcap2:amd64 (1:2.32-1) ...
Selecting previously unselected package libcap2-bin.
Preparing to unpack .../libcap2-bin_1%3a2.32-1_amd64.deb ...
Unpacking libcap2-bin (1:2.32-1) ...
Selecting previously unselected package iputils-ping.
Preparing to unpack .../iputils-ping_3%3a20190709-3_amd64.deb ...
Unpacking iputils-ping (3:20190709-3) ...
Selecting previously unselected package libpam-cap:amd64.
Preparing to unpack .../libpam-cap_1%3a2.32-1_amd64.deb ...
Unpacking libpam-cap:amd64 (1:2.32-1) ...
Setting up libcap2:amd64 (1:2.32-1) ...
Setting up libcap2-bin (1:2.32-1) ...
Setting up libpam-cap:amd64 (1:2.32-1) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.30.0 /usr/local/share/perl/5.30.0 /usr/lib/x86_64-linux-gnu/perl5/5.30 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.30 /usr/share/perl/5.30 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up iputils-ping (3:20190709-3) ...
Processing triggers for libc-bin (2.31-0ubuntu9.2) ...
Removing intermediate container 055f1fab9601
 ---> b41a1cca1e58
Step 3/3 : CMD bash
 ---> Running in ebd1a5113d92
Removing intermediate container ebd1a5113d92
 ---> 34c1cd80b6e2

Successfully built 34c1cd80b6e2
Successfully tagged my_image:latest
WARNING: Image for service service-1 was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating service-1 ... done
Creating service-2 ... done
Attaching to service-1, service-2
service-1    | PING service-2 (172.20.0.3) 56(84) bytes of data.
service-1    | 64 bytes from service-2.ping-docker-compose_default (172.20.0.3): icmp_seq=1 ttl=64 time=133 ms
service-2    | PING service-1 (172.20.0.2) 56(84) bytes of data.
service-2    | 64 bytes from service-1.ping-docker-compose_default (172.20.0.2): icmp_seq=1 ttl=64 time=0.114 ms
service-1    | 64 bytes from service-2.ping-docker-compose_default (172.20.0.3): icmp_seq=2 ttl=64 time=0.131 ms
service-2    | 64 bytes from service-1.ping-docker-compose_default (172.20.0.2): icmp_seq=2 ttl=64 time=0.073 ms
service-1    | 64 bytes from service-2.ping-docker-compose_default (172.20.0.3): icmp_seq=3 ttl=64 time=0.080 ms
service-1    | 
service-1    | --- service-2 ping statistics ---
service-1    | 3 packets transmitted, 3 received, 0% packet loss, time 2010ms
service-1    | rtt min/avg/max/mdev = 0.080/44.543/133.419/62.844 ms
service-1 exited with code 0
service-2    | 
service-2    | --- service-1 ping statistics ---
service-2    | 3 packets transmitted, 2 received, 33.3333% packet loss, time 2036ms
service-2    | rtt min/avg/max/mdev = 0.073/0.093/0.114/0.020 ms
service-2 exited with code 0
```

## Results of running docker-compose up
- As you can see, the ping was successful! Both services were able to resolve and ping the other by the service name itself. Otherwise, we would need to know what the IP was for each container that was launched in order to do this simple execution.
- To see more about the networking services docker-compose has see their documentation. 
- This feature can easily be applied to more complex systems where messaging between services is essential. This makes the complicated network setup and tracking much easier.




