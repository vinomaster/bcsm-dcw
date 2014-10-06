# Docker Workshop

This repo provides material for a [Boston Cloud Services Meetup](http://www.meetup.com/Boston-cloud-services/) on [Docker](https://www.docker.com). 

While it serves is intended for use by meetup participants, it also serves as a basic tutorial or introduction to Docker containers.

## Acknowledgements

The material used within this repo is derived from prior work by [Peter Parente](https://github.com/parente) and [Guillermo Cabrera](http://github.com/gcabrera).

## Workshop Preparation

Choice one of the following approaches to establish a Docker environment:
  * [Local Vagrant](https://github.com/vinomaster/bcsm-dcw/blob/master/setup/vagrant-setup.md)
  * SoftLayer VM (to be provisioned and assigned during workshop)

## Tutorial
  
The material in this repo can be used in support of a hands-on workshop covering the steps involved in getting [Elasticsearch](http://www.elasticsearch.org/overview/elasticsearch) (i.e., a search engine based on Lucene) up and running in a Docker container. 

Using the creation of an Elasticsearch application as a devops goal, the material in this repo will provide a basic introduction to the use of Docker containers for application development. 

The topics to be covered in the workshop include:

* Environment setup
* Developing containers
* Configuring Docker ports and how networking is handled between containers
* Distributing containers

The workshop will follow a build, test, explore, repeat pattern, common to the "Dockerization" process.

## Slideshow

This repo contains a slide presentation based on [reveal.js](http://lab.hakim.se/reveal-js/#/). Assuming accessing to a Docker environment, the slides can be viewed by doing the following on your Docker enabled VM:

```
$ git clone https://github.com/vinomaster/bcsm-dcw.git
$ cd bcsm-dcw/slides
$ ./run.sh
$ docker ps
# Open browser to http://<hostname>:8000/#/
```

