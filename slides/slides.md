



# Docker Workshop

## Boston Cloud Services Meetup

#### Dockerizing a Search Engine

---

## Acknowledgements

* IBM Contributors
	* Guillermo Cabrera
	* Dan Gisolfi
	* Peter Parente
	
* Workshop Technology
	* Presentation via [Reveal.js](https://github.com/hakimel/reveal.js/)
	* Cloud Hosting via [SoftLayer](https://www.softlayer.com)
 
---

## Goal

* Create a web application that searches semi-structured data 
* Use [Elasticsearch](http://www.elasticsearch.org/overview/elasticsearch) as the data store 
* Develop and distribute the solution using Docker Containers

---

##  Workshop Agenda

* Part 1: Containerizing Elasticsearch
* Part 2: Application assembly with linked containers

---

## Development Environment Prep

-

## Virtual Machine Decision

* SoftLayer VM
	* Default environment for workshop.
* [Vagrant  Setup](https://github.com/vinomaster/bcsm-dcw/blob/master/setup/vagrant-setup.md)
	* This option requires pre-workshop setup.
	
-

## SoftLayer Setup

### Obtain VM Credentials
* Docker, etcd and etcdctl preinstalled
* Obtain dedicated VM from workshop instructor
	* Instructor will use VM0 and VM1
* Obtain SSH Key for the Workshop
	* http://bit.ly/1ua2Kud
-

## SoftLayer VM

### Configure SSH

* Edit your SSH Config file  (~/.ssh/config)
	
```bash
	Host sl-bcsm-workshop
		HostName <ip_for_studentID>
		User root
		IdentityFile <your_path>/sl_bcs_meetup.pem
		IdentitiesOnly yes
```
- 

## SoftLayer VM

### Connect to VM

```bash
$ ssh sl-bcsm-workshop
```

-

## Vagrant Setup

* [Local Vagrant VM](https://github.com/vinomaster/bcsm-dcw/blob/master/vagrant-setup.md)
* Predefined Host IP, Ports and Mounted filesystem 
* Docker Preinstalled

-

## Vagrant Setup

### Connect to VM

```bash
$ vagrant ssh
$ cd /vagrant
```

-
##  Prepare Filesystem

```bash
$ git clone https://github.com/vinomaster/bcsm-dcw.git
# Create workarea for Workshop Part 1
$ mkdir -p  es
$ cp bcsm-dcw/elasticsearch/sample.jl es/
# Create workarea for Workshop Part 2
$ mkdir -p  es-webapp
$ cp -R bcsm-dcw/webapp es-webapp/
$ ls -l
```
-
##  View Slideshow

You may optionally view your own copy of the slideshow.
```bash
$ cd bcsm-dcw/slides
$ ./run.sh
$ docker ps
# Open your browser to http://<hostname>:8000/#/
```

---

## Part 1: Dockerizing Elasticsearch

1. Build an ES container image
2. Test it with some data
3. Deploy it to a target server

---

## 1. Build a container image

-

## Setup for dev

```bash
$ sudo apt-get -y install tmux
```

-

## Split the screen

```bash
$ cd es
$ tmux
# Ctrl-B, "
```

* We will use **"Ctrl-B, o"** to toggle between screens. 
* See [Tmux Cheat Sheet](https://gist.github.com/MohamedAlaa/2961058)

-

## Start a Dockerfile

```bash
$ vi Dockerfile
```

-

## Declare a base image

```bash
# Dockerfile
FROM ubuntu:14.04
```

-

## Update the package lists

```bash
# Dockerfile
FROM ubuntu:14.04
RUN apt-get update
```

-

## Build it

Toggle screen with **"Ctrl-B, o"**

```bash
$ docker build --rm -t bcsmdcw/es_<studentID> .
```

![](md/images/6-6.png)

-

## Install Java 7

```bash
# Dockerfile
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get -y install openjdk-7-jre-headless
```

-

## Build it

```bash
$ docker build --rm -t bcsmdcw/es_<studentID> .
```

![](md/images/6-8.png)

-

## Install Elasticsearch 1.0

```bash
# Dockerfile
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get -y install openjdk-7-jre-headless
RUN apt-get -y install wget
RUN wget https://ibm.biz/BdRSMs -O elasticsearch-1.0.0.deb
RUN dpkg -i elasticsearch-1.0.0.deb && \
    rm elasticsearch*.deb && \
    service elasticsearch stop
```

-

## Build it

```bash
$ docker build --rm -t bcsmdcw/es_<studentID> .
```

![](md/images/6-10.png)

-

## Explore the container

```bash
$ docker run -t -i bcsmdcw/es_<studentID> /bin/bash
```

![](md/images/6-11.png)

-

## Questions

1. Where's the config?
2. What's the HTTP port?
4. Where's the binary?
3. Where does data go?

-

## Declare the port

```bash
# Dockerfile
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get -y install openjdk-7-jre-headless
RUN apt-get -y install wget adduser
RUN wget https://ibm.biz/BdRSMs -O elasticsearch-1.0.0.deb
RUN dpkg -i elasticsearch-1.0.0.deb && \
    rm elasticsearch*.deb && \
    service elasticsearch stop
EXPOSE 9200
```
-

## Declare a working directory

```bash
# Dockerfile
FROM ubuntu:13.10
RUN apt-get update
RUN apt-get -y install openjdk-7-jre-headless
RUN apt-get -y install wget adduser
RUN wget https://ibm.biz/BdRSMs -O elasticsearch-1.0.0.deb
RUN dpkg -i elasticsearch-1.0.0.deb && \
    rm elasticsearch*.deb && \
    service elasticsearch stop
EXPOSE 9200
WORKDIR /usr/share/elasticsearch
```

-

## Declare the start command

```bash
# Dockerfile
FROM ubuntu:13.10
RUN apt-get update
RUN apt-get -y install openjdk-7-jre-headless
RUN apt-get -y install wget adduser
RUN wget https://ibm.biz/BdRSMs -O elasticsearch-1.0.0.deb
RUN dpkg -i elasticsearch-1.0.0.deb && \
    rm elasticsearch*.deb && \
    service elasticsearch stop
EXPOSE 9200
WORKDIR /usr/share/elasticsearch
CMD ["bin/elasticsearch", "-Des.path.conf=/etc/elasticsearch"]
```

-

## Build it

```bash
$ docker build --rm -t bcsmdcw/es_<studentID> .
$ docker images
```

![](md/images/6-17.png)

-

## Run it

```bash
$ docker run --name es-demo -p 9200:9200  \
	-d bcsmdcw/es_<studentID>
$ docker ps
```

![](md/images/6-18.png)

-

## Basic Container Management
Learn to halt, remove and rerun containers.
```bash
$ docker stop es-demo 
$ docker ps
$ docker ps -a
$ docker run --name es-demo -p 9200:9200  \
	-d bcsmdcw/es_<studentID>
# Name Conflict Issue; Remove Container
$ docker rm es-demo 
$ docker ps
$ docker ps -a
# Recreate Container
$ docker run --name es-demo -p 9200:9200 \
	-d bcsmdcw/es_<studentID>
```
-

## Try a HTTP GET

```
$ sudo apt-get -y install curl
$ curl -s -XGET http://localhost:9200
```
Sample Results
```
{
  "status" : 200,
  "name" : "Dr. Lemuel Dorcas",
  "version" : {
    "number" : "1.0.0",
    "build_hash" : "a46900e9c72c0a623d71b54016357d5f94c8ea32",
    "build_timestamp" : "2014-02-12T16:18:34Z",
    "build_snapshot" : false,
    "lucene_version" : "4.6"
  },
  "tagline" : "You Know, for Search"
}
```

---

## 2. Test it with some data

-

## Bulk load documents

```bash
$ curl -s -XPOST http://localhost:9200/demo/people/_bulk  \
	--data-binary @sample.jl
```

-

## Fetch a few documents

```bash
$ clear
$ curl -XGET http://localhost:9200/demo/people/10
$ curl -XGET http://localhost:9200/demo/people/1
$ curl -XGET http://localhost:9200/demo/people/42
```

-

## Nice, but how about a UI?

-

## Install a web dashboard

```bash
# Dockerfile
FROM ubuntu:13.10
RUN apt-get update
RUN apt-get -y install openjdk-7-jre-headless
RUN apt-get -y install wget adduser
RUN wget https://ibm.biz/BdRSMs -O elasticsearch-1.0.0.deb
RUN dpkg -i elasticsearch-1.0.0.deb && \
    rm elasticsearch*.deb && \
    service elasticsearch stop
RUN /usr/share/elasticsearch/bin/plugin \
    -install mobz/elasticsearch-head
EXPOSE 9200
WORKDIR /usr/share/elasticsearch
CMD ["bin/elasticsearch", "-Des.path.conf=/etc/elasticsearch"]
```

-

## Build it

```bash
$ docker build --rm -t bcsmdcw/es_<studentID> .
```

![](md/images/7-5.png)

-

## Run it

```bash
$ docker run --name es-demo-dash -p 9200:9200  \
	-d bcsmdcw/es_<studentID>
```

-

## KABOOM!

* Error: Port is already allocated.
* We didn't stop the old container.

![](md/images/7-7.png)

-

## Stop the old version

```bash
# Stop and unbind running container from port
$ docker ps
$ docker stop es-demo
# Container still remains available for restarting
$ docker ps -a
```

-

## Run it

```bash
$ docker run -name es-demo-dash -p 9200:9200 \
    -d bcsmdcw/es_<studentID>
```

-

## KABOOM!

The failed container is hogging the name `es-demo-dash`.

![](md/images/7-10.png)

-

## Remove the dead one

Do not remove es-demo, we will need that container in a future task.

```bash
$ docker rm es-demo-dash
```

-

## Run it

```bash
$ docker run -name es-demo-dash -p 9200:9200 \
    -d bcsmdcw/es_<studentID>
```

![](md/images/7-12.png)

-

## Hit the dashboard

SoftLayer Environment
```bash
http://<student_ip>:9200/_plugin/head/
```

Vagrant Environment
```bash
http://192.168.44.44:9200/_plugin/head
```
-

## Uh oh

* Q: Where's the data?
* A: In the old container. 

-

## Prime New Container

```bash
# Two containers are cached but only one is active on port 9200
$ docker ps -a
# Prime the new container with data
$ curl -s -XPOST http://localhost:9200/demo/people/_bulk  \
	--data-binary @sample.jl
```
-

## Hit the dashboard again

SoftLayer Environment
```bash
http://<student_ip>:9200/_plugin/head/
```

Vagrant Environment
```bash
http://192.168.44.44:9200/_plugin/head
```

---

## 3. Distribute the Container

-

## Workshop Options
We can copy a docker image to another Cloud Provider such as SoftLayer or AWS.

* Option 1: Simulated Approach
	* To avoid broadband issues, some students can work independently
* Option 2: Distribution Test 
	* Students can pair and share docker images
	* Students with even IDs send their tarballs to ID+1
	* Students with odd IDs send their tarballs to ID-1
	* Download/Upload may take 15mins
* Choose your option

-

## Save the repository

```bash
$ docker images
# Update the state of the image with container content
$ docker commit es-demo-dash bcsmdcw/esdash_<sid>
# Use your student ID to create image snapshot
$ docker save bcsmdcw/esdash_<sid> > esdash_<sid>.tar
$ ls -lh
```
where **sid** is Student ID
-

##  Distribute Container
### Option 1: Simulated Approach

The source and target environments are the same.

```bash
$ docker images
# Cleanup current environment.
$ docker ps -a
$ docker kill es-demo-dash
$ docker rm es-demo-dash
$ docker rmi bcsmdcw/esdash_<sid>
$ docker images
```
-

##  Distribute Container
### Option 2: Distribution Test 

On your local computer do the following:

```bash
# Create a local workspace
$ mkdir -p ~/Downloads/bcsm-dcw/es
$ cd ~/Downloads/bcsm-dcw/es
# Copy your tarball to local disk
$ scp -i <pemfile> root@<student_ip>:~/es/esdash_<sid>.tar .
# Copy your tarball to Partner VM
$ scp -i <pemfile> esdash_<sid>.tar root@<partner_ip>:~/es
```

-

## Load the Container Image

Use our classmate's image to create a new container.

```bash
# Load saved container into target environment
$ docker load <  esdash_<studentID>.tar
$ docker images
```

-

![](md/images/8-4.png)

-

## Run it

* Create a new container from the newly loaded image. 
* Create the new container using a different external port.

```bash
$ docker run --name es_dashboard -p 9300:9200  \
	-d bcsmdcw/esdash_<studentID>
$ docker ps
```
![](md/images/8-7.png)

-

## Test it

SoftLayer Environment
```bash
http://<student_ip>:9300/_plugin/head/
```

Vagrant Environment
```bash
http://192.168.44.44:9300/_plugin/head
```

-

## More? Sure!

Create more application instances and let Docker select external port and container name.

```bash
sl$ docker run -P -d bcsmdcw/esdash_<studentID>
sl$ docker run -P -d bcsmdcw/esdash_<studentID>
sl$ docker run -P -d bcsmdcw/esdash_<studentID>
sl$ docker ps
```

![](md/images/8-9.png)


-

## Test it

SoftLayer Environment
```bash
http://<student_ip>:<PORT>/_plugin/head/
```

Vagrant Environment
```bash
http://192.168.44.44:<PORT>/_plugin/head
```

-

## Whoa! How did we get master/slave?

1. All containers are on a private network 
2. Elasticsearch does subnet discovery 
3. Docker allows communication among containers by default 
4. EXPOSE plays no part in this! 

-

![](md/images/8-12.png)

---

## Part 2: Dockerizing Search Engine

* Build a docker image for a Sonnet Search App (sonnetsearch)
* Connect sonnetsearch container to ElasticSearch container

---

## Sonnet Search App

* Objective
	* Create a Python application to search Shakespeare sonnets.
* Required Technologies
	* bottle (web framework)
	* gunicorn (http server)
	* requests (HTTP lib)
	* ElasticSearch API
	
---

## Application Data

![](md/images/sonnets.png)

We will prime the pump with content from [Shakespeares Sonnets in Json Format](http://www.samdutton.com/sonnets.json).

---
## Review Source Code

* Take a peak at source code
	* ~/es-webapp/webapp/server.py
* Take note of the container environment variables
	* BCSMDCW_ES_ADDR
	* BCSMDCW_ES_PORT
	
---

## Build Dockerfile 

Prepare container environment.

```bash
# Change Project Workarea
$ cd  ~/es-webapp
$ vi Dockerfile
```
-

## Build Dockerfile

```bash
# Dockerfile
FROM ubuntu:14.04
RUN apt-get update
```

```bash
# Build Container
$ docker build -t bcsmdcw/sonnetsearch .
```
-

##  Build Dockerfile 

* Install Python and other Utilities.

```bash
# Dockerfile
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get -y install -y python wget git
RUN wget https://raw.github.com/pypa/pip/master/contrib/get-pip.py
RUN python get-pip.py
RUN pip install bottle==0.11.6 gunicorn==18.0 requests==2.1.0
RUN pip install elasticsearch
```

```bash
# Build Container
$ docker build -t bcsmdcw/sonnetsearch .
```
-

## Build Dockerfile

Finalize container.

```bash
# Dockerfile
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get -y install -y python wget git
RUN wget https://raw.github.com/pypa/pip/master/contrib/get-pip.py
RUN python get-pip.py
RUN pip install bottle==0.11.6 gunicorn==18.0 requests==2.1.0
RUN pip install elasticsearch
ENV BCSMDCW_ES_ADDR 192.168.1.10
ENV BCSMDCW_ES_PORT 9200
ADD webapp /webapp
EXPOSE 8080
WORKDIR /webapp
CMD python server.py
```

```bash
# Build Container
$ docker build -t bcsmdcw/sonnetsearch .
$ docker images
$ docker inspect bcsmdcw/sonnetsearch | more
```
---

## Run and Test

* Links containers using environment variables:
	* Create an ElasticSearch container with random port and random name.
	* Obtain public network address and port for ElasticSearch container
	* Create a Sonnet Search App container

```bash
$ ES=$(docker run -P -d --name \
	elasticsearch bcsmdcw/esdash_<studentID>)
$ docker ps
$ docker logs <random_container_name>
$ APP=$(docker run -d -p 8080:8080  --link elasticsearch:es \
	--name sonnetsearch -e BCSMDCW_ES_ADDR=184.173.163.109  \
	-e BCSMDCW_ES_PORT=49154 bcsmdcw/sonnetsearch)
$ docker logs sonnetsearch
$ docker ps
```
---

![](md/images/15.png)

---
## Thank you

