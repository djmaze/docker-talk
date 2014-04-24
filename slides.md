## An introduction to Docker 

![Docker](https://www.docker.io/static/img/homepage-docker-logo.png)

### _NOT_ from a DevOps perspective

Martin Honermeyer 

Zweitag GmbH

---

## Survey

* Who has heard of Docker before this meetup? <!-- .element: class="fragment" -->
* Who has already tried out Docker? <!-- .element: class="fragment" -->
* Who is using Docker in production? <!-- .element: class="fragment" -->

Note: 

* Docker is ever changing

---

## TOC

* Docker intro
* VMs vs. Containers
* Containers & Images
* Dockerfile, Registry
* Further use cases, ecosystem
* Stuff

---

## Rise of the Docker

* Officially released by dotcloud Inc. in 3/13 (Jeróme Petazzoni)
* internal tool, open-sourced
* big interest in the DevOps community
* company renamed to *Docker Inc.* in the meantime

![Whales](/images/whales.jpg)

-

## Some figures

* 380 contributors, 95% non-Docker employees
* Meetups in >80 cities in 30 countries
* @Github:
  * \> 10k stars (4th)
  * \> 1.7k forks
  * \> 350 derivative projects

---

## In a nutshell: What is Docker?

* "an open source project to pack, ship and run any application as a lightweight container" <!-- .element: class="fragment" -->
* packaging applications including all dependencies <!-- .element: class="fragment" -->
  * libraries
  * dependent services
* written in Go (client-/server) <!-- .element: class="fragment" -->

## What is Docker NOT? <!-- .element: class="fragment" -->

* a PaaS <!-- .element: class="fragment" -->
* a configuration manager replacement (Chef, Puppet, Saltstack etc.) <!-- .element: class="fragment" -->

---

## There are many ways to use Docker!

This talk focuses on explaining:

* the technology
* the vision
* some best practices

---

## DevOps anyone?

![DevOps](/images/devops.jpg)

-

## DevOps explained

* no official definition, in my view:
  * "Automating operations and system administration using software development methodologies"
  * Infrastructure as code!

### 2 Aims

* applying development practices to operations (branching, auditing, review)
* getting developers to understand production more

---

## Virtual Machines vs. Containers for Server provisioning

#### Provisioning?

* Automated installation and configuration of:
  * servers
  * apps on servers

-

## Traditional approach: virtualization

* complete OS emulation:
  * kernel
  * peripherals

<!-- ![Virtual machines](/images/Virtual_machines.png) -->

> (Image not included because of copyright reasons. See the original image [here](http://de.slideshare.net/dotCloud/golub-ben-arevmspasse/9)!)

-

## Problems with virtualization

* emulation overhead (CPU, memory)
* managing a whole server
  * "dependency hell"
  * security updates
  * housekeeping (backups etc.)
* Provisioning is:
  * slow
  * OS dependent

---

## Containers FTW!

<!-- ![Containers](/images/Containers.png) -->

> (Image not included because of copyright reasons. See the original image [here](http://de.slideshare.net/dotCloud/golub-ben-arevmspasse/19)!)

* shared kernel, deploy to bare metal => (almost) no overhead <!-- .element: class="fragment" -->
* self contained => no dependency hell <!-- .element: class="fragment" -->
* isolating multiple apps on a single host <!-- .element: class="fragment" -->
* ..and more! <!-- .element: class="fragment" -->

-

## Technologies used (originally)

#### clever way of utilizing and building upon Linux features

* Linux containers (cgroups & namespaces)
* Copy-on-write filesystems

#### In the future <!-- .element: class="fragment" -->

* Code is already driver agnostic <!-- .element: class="fragment" -->
* Underlying technologies will be replaceable (-> VMs, Chroots) <!-- .element: class="fragment" -->

-

## Container isolation

* Processes
* Filesystem
* Network (IP address)

From inside the container, it looks a bit like a VM!

---

# Images

-

![Container
image](http://www.containercontainer.com/content/images/containers/15.jpg)

Container image.

-

## Images

Containers are built from images.

![Docker Images](http://www.roadside-developer.com/talks/2013-10-10_MelbDjango_dockbot/images/docker-filesystems-busyboxrw.png)

-

## Images (2)

* Images = file system snapshots
* can be "forked" from other ones
* layered using *Copy-on-Write filesystems*
  * base image contains a whole root filesystem
  * images contain only a diff to their parent

### Layering example

* Ubuntu base image
  * image with Ruby installed
    * image with Rails app installed

-

## Container advantages for provisioning

* no emulation overhead
* always start from the same base image => always idempotent (almost..)
* portability: "build once, run anywhere" (on Linux)
* uses COW snapshots at any point to allow getting "back in time"
* fast restart

Note:

* "almost": problem with external data (updates, downloads)
* "fast restart": no OS has to be booted

---

## Containers as black boxes - Example

![Applications as black boxes](/images/Applications_as_black_boxes.png)

-

## Containers as black boxes

*"12-factor app" approach - it's a Good Thing™*

* Define a container through its external behaviour:
  * *Ports* being exposed
  * *Services* needed (through *links*)
* Separate the app from its data:
  * through *Volumes*
  * directory which should be persisted independently of the container

#### That means:

* Containers are disposable
* only *images* and *volumes* needed to be kept
* volumes can be used from other containers

![Container as a bin](/images/Container_bin.jpg)

---

## Dockerfile

* recipe for building an image
* dumbed-down shell script with additional directives (evaluated line by line)
* starts from pre-existing root fs image
* (anonymous) layer image saved after every step (for caching)
* directives:
  * RUN
  * ADD
  * CMD
  * ..

-

## Dockerfile example

([orchardup/mysql](https://index.docker.io/u/orchardup/mysql/))

```bash
FROM stackbrew/ubuntu:12.04
MAINTAINER Ben Firshman <ben@orchardup.com>

RUN apt-get update -qq
RUN apt-get install -y mysql-server-5.5

ADD my.cnf /etc/mysql/conf.d/my.cnf
RUN chmod 664 /etc/mysql/conf.d/my.cnf
ADD run /usr/local/bin/run
RUN chmod +x /usr/local/bin/run

VOLUME ["/var/lib/mysql"]
EXPOSE 3306
CMD ["/usr/local/bin/run"]
```
-

## Running and linking containers

<!--<script type="text/javascript" src="http://asciinema.org/a/14.js" id="asciicast-14" async></script>-->

Build:

```bash
docker build -t mysql-server .
```

Run:

```bash
docker run -d -p 3306:3306 -name mysql mysql-server
```

Link to `my-app`:

```bash
docker run -link mysql:mysql my-app
```

my-app container gets some ENV variables when starting: <!-- .element: class="fragment" -->

<div class="fragment">

      MYSQL_PORT_3306_TCP_PORT*, *MYSQL_PORT_3306_TCP_ADDR, .. 

</div>

---

## Docker registry

http://index.docker.io

    # Example image from the registry
    FROM stackbrew/ubuntu:12.04

* central repository for public images
* simple push / pull
* build upon other's images
* trusted builds

### Additionally: <!-- .element: class="fragment" -->

* roll your own registry for private images <!-- .element: class="fragment" -->

---

# What else?

---

## What else is possible?

* scale out
* cross-cloud deployment
* continuous integration / deployment

Note:

scale out: rapidly scale up and down applications across 100s/1000s of servers

cross-cloud deployment: no provider-specific dependencies

---

## Docker ecosystem

Many tools have already been built around Docker.

![Container wall](/images/Container_wall.jpg)

-

## Development environment

* Building fast, isolated development environments
  * Fig
  * Bowery

### fig.yml sample

```yaml
web:
  build: .
  command: bundle exec rails server
  links:
   - db
   - redis
  ports:
   - "3000:3000"
db:
  image: orchardup/mysql
  environment:
    MYSQL_DATABASE: rails_db
    MYSQL_USER: rails_db_user
    MYSQL_PASSWORD: test123
redis:
  image: orchardup/redis
```

-

## Continuous integration & deployment

* Automatically run tests & deploy to production
  * Drone

![Drone.io](http://blog.drone.io/img/oss_screenshot_dashboard.png)

-

## Service discovery & orchestration

### Goal

* connecting containers across multiple servers
* moving containers to other servers
* scaling up with additional containers

### Solutions <!-- .element: class="fragment" -->

* Service announcement / discovery <!-- .element: class="fragment" -->
  * etcd
    * key-value store
  * Serf
    * gossip protocol
* Orchestration <!-- .element: class="fragment" -->
  * CoreOS (etcd, fleet)
  * Project Atomic (geard)

---

## Platforms as a service

#### How they work

* Heroku-style
* use Heroku buildpacks
* deploy almost anything

#### "Enterprise solutions"

* Flynn
* Voxoz
* Deis (with Chef)

#### Quick-and-dirty solutions

* Dokku (100-lines bash)
* peas
* spin-docker
* .. build your own one!

---

## Docker on OSX

Use Boot2docker.

* Tiny Linux VM with Docker
* use Docker like on localhost

![boot2docker Demo](https://camo.githubusercontent.com/fd2fda3c0d55a0a63873f4221ddbe2f1dda145c5/687474703a2f2f692e696d6775722e636f6d2f68497775644b332e676966)

Drawbacks:

* VM overhead
* port forwarding currently painful

---

## Is Docker ready for production?

* overall: pretty stable
* major users:

<!-- ![Major users](/images/Major_users.png) -->

> (Image not included because of copyright reasons. See the original image [here](http://de.slideshare.net/dotCloud/golub-ben-arevmspasse/30)!)

* but: highscore in RPR (Regressions per Release)

-

## Is Docker ready for production? (2)

### advice:

* follow updates <!-- .element: class="fragment" -->
* but be cautious for regressions; wait for point releases <!-- .element: class="fragment" -->

### Behold: <!-- .element: class="fragment" -->

*1.0 should be coming soonish!* <!-- .element: class="fragment" -->

---

## Why should you as a developer care?

* Easy, clean & reproducible dev env setup
* Run code on laptop in same environment as on server
* Docker *is* the future of app deployment & will be everywhere

## TL;DR <!-- .element: class="fragment" -->

<div class="fragment">
You **will** not get around Docker ;P
</div>

![Docker all the things](http://blogs.atlassian.com/wp-content/uploads/docker-all-the-things.png) <!-- .element: class="fragment" -->

---

# Thank you!

Run this presentation:

    sudo docker run -p 8000:8000 mazzolino/docker-talk

Then visit http://localhost:8000

### Resources

* [Slides from Ben Golub's talk "Are VM Passé?"](http://de.slideshare.net/dotCloud/golub-ben-arevmspasse)
* [What is Docker and when to use it](http://www.centurylinklabs.com/what-is-docker-and-when-to-use-it/)
* [Interview with @shykes](https://www.youtube.com/watch?v=r-XfAqbYtoU)
* many more..

#### Pictures

* [Tony Hisgett](https://www.flickr.com/photos/hisgett/400626710/in/photolist-Bpjn5-3b5sbX-5rLwqc-fAtcUT-fp53z7-7NLZiW-9dqhnn-9Jea1r-5mNuYs-8nX2iE-37fN4P-fjZPJx-6UNKcF-53LrD-SKbNE-3e3MmY-6bFzJL-a6xiV-4Hq1z-6PS6VX-31w5Sp-fAoDCM-3yCvSi-79iuto-3yD7Sx-52tAJH-6WAjGq-3b6SWg-3o1g6P-31ADeG-72jDAN-kcB7B-6MShLt-7PVmsq-6MWWcL-2YE35a-8nFfb5-6DVNkB-tifho-ffpD2x-2s6GL-8nX2dj-3yCubr-2YT6yC-5tuVTs-6PWdGs-8mR42X-2YReMj-3yGVoQ-5Z25h)
* [Stefan Goethals](https://www.flickr.com/photos/zipkid/5090236985/in/photolist-dXKitq-8KbZke-8KbZwe-8KpHyM-8KsS6J-8KNN7D-8KpJAP-8KpDR4-8KPb3i-8KsXmW-8KPbvD-8KPdeP-8KsUxq-8KqFRk-8KShks-8KtGCE-8KpQb4-8KqEuX-8KPaTP-8LAj3c-8Ke75s-8Kb45e-8KsUUd-8KSiof-8KpR6r-8KSidy-8KPf2F-8KSiUh-8KPeSt-8KSeT1-8KsPV5-8KsMsm-8KPdQH-8KsT1m-e5dbML-cnMUj7-cnMT2j-e9grLE-e8nsXM-e8vCqm-e9aHVK-e8uVCw-e8y2DN-e9aK1Z-e8srQJ-e9gsmf-e8soRk-e8peoe-e9aMa6-8KPbQM)
* [Dockbot talk](http://www.roadside-developer.com/talks/2013-10-10_MelbDjango_dockbot)
* [Container bin](https://www.flickr.com/photos/jellymc/7774898730/in/photolist-cR3ow7-aYqgo-adg96W-9tdiKg-5o1t5T-7Hvwj5-hD812L-4WgYF1-5iF3Pn-4uJbq5-6RbkZQ-inkvJ-a85qp6-deEhrd-6YXYGU-kKYocZ-5mDc7Z-958D3A-2zyPsW-9tktUm-4FCUC2-2zyPmL-5YfMKZ-fs2WCj-cNJHNA-9fJb3k-2TFPkt-kPaDUv-j7gdsP-hD7HeX-52Lc3H-j7gjCV-eohjcS-8f1Kg5-6aFmpQ-6BCmMw-e7pnoN-4iJF4W-5nUEdr-95pouQ-e8RuW9-7AUn5n-4QJFSh-2E9DhN-eZ31fq-cGmP7A-8QvqVa-8Bnvd-DAAZC-JthqE)
* [Container Wall](https://www.flickr.com/photos/joschmaltz/226001361/in/photolist-kYjkc-aXy6Ua-89mLXa-bmfv4k-bTkiRH-7VzYSt-2on4Mx-86bjz4-2orozC-2on3Ne-J6yhR-2on2og-9oEd2x-dH6sAU-jEGPn-4dgMja-6ardWM-6x8t8W-65Yujx-cbqeF9-6McJTv-8foPeg-9i1xfF-9i1M3P-f7t45E-6oy1w7-a34PBi-dPcqJ1-4ZfkcN-5kxw4o-dxJA5Z-cJYs5-hSo1p-koPMt-8aUV6k-7tnddP-2CqVL2-cjzFby-4ytPEq-AUWz7-eiL613-5QS3JQ-9Gin2f-65Yuoi-663KA3-663Kiu-65Yu9z-kbPvw-7U378N-8bgw1U)
