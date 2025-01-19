## Excerpt from Nest.js fundamentals
run docker compose in detached mode  
(in background)
```bash
docker compose up -d
```

run only one service
```bash
docker compose up db -d
```

## Kubernetes in action
https://livebook.manning.com/book/kubernetes-in-action-second-edition/chapter-1/v-14/

### 1. Introducing Kubernetes
in greek: person who steers the ship  
also called Kates, k8s

#### What it is
software for automating deployment of complex systems

provides abstraction over hardware  
to both users and applications  
(hides hardware details for dev-ops and apps)

you describe components in manifest file  
kubernetes keeps app healthy  
by restarting or recreating parts as needed

when you change description  
kubernetes will transform running components  
(like stop removed components)

dev/ops can focus on big picture

#### History
originally developed by Google  
which had a lot of running containers  
first called Borg, then Omega  
after going open source Red Hat joined early

has many related open source projects  
most of which are called together  
CNCF: Cloud Native Computing Foundation  
*organizator of KubeCon conf*

#### Why Kubernetes is so popular
in past apps were monoliths  
deployment was fairly easy  
and required little configuration  
it was unpossible to scale  
*only vertically, on stronger machines*

in microservices each service is separate app  
applications need configuration to talk to each other  
it's easier to scale

#### what Kubernetes affected
devs need to know about infrastructure  
on which they will run  
they are closer to dev-ops

Kates makes changing cloud provider easier  
AWS, IBM Cloud, Google Cloud, Azure  
they all support Kubernetes

#### some of features
service discovery  
horizontal scaling  
load-balancing  
self-healing (restarting failed apps)  
leader election

#### control plane / workload plane
Control Plane  
has few master nodes  
it's the "brain" of cluster  
*highly available clusters use 3+ master nodes*

Workload Plane  
*(sometimes called Data Plane, although it's more Apps)*  
has many worker nodes  
which host applications

after setup of kubernetes on machines  
you work with kubernetes api  
provided by Control Plane

#### deployment
kubernetes chooses best node to run app  
based on resource requiremenets of application  
and resource available on each node

when deploying applications  
you no longer need to think about individual computers  
they all become single space  
but still, every application has to be small enough  
to fit on some machine (called worker node)

Kates can move running apps between nodes  
you shouldn't even notice that

#### reducing cost
via better infrastructure utilization  
also Kates can decide to run apps on same machine

#### auto scale
to react to load peaks  
when on cloud, it can provision new nodes

#### keep app running
automatically restart failed nodes

#### simplify deployment
discovery services  
leader election  
centralized application configuration

also apps can query kubernetes api  
to obtain details about their environment

#### kubernetes architecture
worker nodes communicate with master  
but never with each other directly

each worker node  
runs one or several applications  
also runs several kubernetes components  
that manage running apps

#### control plane/ master node
at least one  
can be replicated for high availability

Kubernetes API server  
RESTful kubernetes API  
bus of communication with workers and users

etcd: distributed datastore  
only API Server talks with it

scheduler: decides which node to run apps

controllers: execute commands  
most often they create objects

#### worker node
kubelet  
in practice: operates Docker  
_instructs container runtime_

kube proxy  
load-balances network traffic  
historically traffic flowed trough it  
but not anymore  
creates a load balancer for each service

#### kubernetes clusters
add-on components:  
DNS server  
network plugins  
logging agents

#### objects
everything in k8s is represented by an object.  
you create and retrieve these objects via k8s api  
several types of objects:  
- application deployment as whole
- running instance of application
- service provided by a set of instances to reach single IP
and more

defined in json / yaml manifest files

#### how it works
you submit manifest to k8s api server  
api server stores object definitions in etcd  
controller notices change in etcd and creates new objects  
scheduler assigns a node to each instance  
kubelet notices it was assigned and runs container  
kube proxy notices instances are ready and configures a load balancer for them

#### kubectl
called: kube-control, kube-cuddle, kube-c-t-l  
command line tool  
used to submit manifest(s)

kubectl splits manifest file into objects  
and sends each by HTTTP PUT/POST to k8s API  
api server validates and stores in etcd  
and notifies controllers

#### controllers
most object types have associated controller  
one of them is responsible for application deployments  
it creates as many deployments as specified in replica count

scheduler is special type of controller  
it assigns app to worker node by modyfing object via k8s API

kubelet is also type of controller  
it waits for apps to be assigned to worker node  
and runs docker  
also restarts application when it terminates

other controllers move apps to healthy node  
if their nodes fail

kube proxy: another controller  
it creates load balancer  
so that app instances are available under one IP address

#### introducing k8s to project
decide:  
run applications locally  
or one cloud provider  
(or hybrid)

**On-premises** infrastructure  
is physically located  
within an organization.

running locally  
on-premises  
may be only option if regulations require  
k8s can run on bare-metal  
or in virtual machines in data center  
won't be able to scale as easily as in cloud

deploying in the cloud  
only option if you have no on-premises  
adventage that it can scale  
provision additional virtual machines  
or destroy unused ones

hybrid  
run on-premises  
but also use cloud if capacity exceeded

#### manage k8s yourself?
probably not  
as it's very complex to manage yourself

using is ten times easier than managing it  
kubernetes-as-a-service

Google Kubernetes Engine (GKE)  
Azure Kubernetes Service (AKS)  
Amazon Elastic Kubernetes Service (EKS)  
IBM Cloud Kubernetes Service  
Red Hat OpenShift Online and Dedicated  
VMware Cloud PKS  
Alibaba Cloud Container Service for Kubernetes (ACK)

#### valinilla or enterprise-grade?
vanilla: cutting edge  
extra work to make it secure and tuned

better to use enterprise-quality distro  
such as OpenShift or Rancher  
they offer additional object types  
like: cluster users

#### should you even use kubernetes?
no if app is  
- monolith
- only few microservices, less than 5

yes if  
- app has over 20 microservices

### 2 Understanding containers
#### introducing containers
as microservices become smaller and their numbers grow  
you man not be able to give each its own VM

#### comparison to virtual machines
allow multiple services on the same host computer  
like VM's but with much less overhead

unline virtual machines  
container runs within host operating system  
but is isolated  
though not as well, as when you run in separate vms

#### comparing overhead
containers are much lighter  
they don't require additional resource pool  
it's just isolated process  
without overhead

many applications in same vm  
often multiple applications are grouped into each VM  
because of VMs overhead

one application per container  
you should never run multi apps in same container  
it makes managing processes way more difficult  
software for containers assumes there is one app

kubernetes provides a way to run applications together  
even though they are in separate containers

#### comparing startup time
containers start the application faster  
because no additional system processes need to be started

#### why virual machine isolation is stronger
VM's run "Hypervisors"  
which splits operating systems and their resources  
and applications in VMs make system calls to guest kernel

*(there is also a type of hyervisor that doesn't require host OS)*

kernel isolation (linux kernel vulnerabilities)  
if there is a bug in the kernel  
if kernel is compromised, all containers are at risk  
with kernel breach, containers may be able to see each other  
and one application can read memory of other containers

breach in network stack  
could expose network traffic of other containers

easier to attack  
namespaces, cgroups and otherare big surface for potential attack  
vm's run on "hypervisors" which are designed for minimal attack surface

malware  
malware within a container can expliot shared resources  
to persist or affect the host

#### 2.1.2 Introducing the Docker container platform
docker was first convienent container solution

Docker allows to distribute package via publich repository

image  
something you package you application and its environment  
contains app, whole filesystem and additional metadata

registry  
enables place to store and exchange images  
certain registries are public, while other are private only

container  
an started image  
a normal process running in host operating system  
but isolated  
container is usually resource restricted

#### process
1. image is build and stored locally  
2. image is uploaded from local to registry  
3. anyone with access can pull image from registry

application running in container will see same files  
no matter does it run locally or on server machine  
(apart from scenario of enabling host os files in container)

#### portability limitations
OS host is irrevelant for application running in container  
although kernel version may be important  
and loaded kernel modules

because containers dont have own kernel  
if application requires a partical kernel version  
it may not work on every computer  
or if computer doesn't load the required kernel modules

also containerized app built for a specific architecture  
can only run on computers with the same architecture  
you can't put an application compiled for x86 into container  
and run it on ARM based computer  
*you need VM for that*

#### layers
container images consist of layers  
which can be shared and reused in multiple images  
they help reduce storing size of images  
and make building more efficient

#### Copy-on-Write (CoW) mechanism
in running container layers are readonly  
apart from one, top layer  
whenever file is modified, its first copied to that top layer

deleting never reduces size of image  
when file is deleted, its marked as deleted on top layer  
but still present in layers below

#### Running container
`busybox` image: single command that combines echo, ls and gzip  
it's usefull for small, embedded situations  
where one command can be smaller than a collection of commands

```bash
docker run busybox echo "Hello World"
```

often running is even simpler  
because you don't have to provide command
```bash
docker run redis:alpine
```

to run an image from different registry  
*Quay.io is example of another public registry*  
provide registry with image name
```bash
docker run quay.io/fedora/fedora echo "Hello from Quay.io"
```

#### image tags
like: latest, buster, alpine, 5.0.7-buster...  
way to have multiple versions of same image under same name  
each variant has unique tag  
*buster is a Debian version from 2022*  
*alpine is only 6MB!*

if you refer to images without explicit tag  
then special tag `latest` is used

when uploading, authors usually give image a tag  
and also latest tag is given

however `docker run` will try to use latest from cache  
so implicitly asking for latest  
will make pull newer version if available

#### Open Container Initiative, OCI
born after success of Docker

formed two specifications  
OCI Image Format Specification  
OCI Runtime Specification

Kubernetest now suports more container runtimes  
using Container Runtime Interface (CRI)

One implementation of CRI is CRI-O  
lightweight alternative to Docker  
that works with any OCI compliant container runtime

examples of Open Container Initiative, OCI runtimes:  
rkt (pronounced Rocket)  
runC  
Kata Containers

## Complete Intro to Containers, v2
https://frontendmasters.com/workshops/complete-intro-containers-v2/

### 1. Welcome

#### Simple
Containers may be intimidating but in reality they are suprisingly simple.  
Just combination of three linux tools.

#### Windows containers
are basically VMs, virtual machines  
not very common today

### 2. Crafting Containers by Hand

#### What is container
It's three features combined:  
1. "linux jail" (called also "jailed process", cha-root)  
2. namespaces  
3. cgroups (control groups)

#### Why containers
"Bare metal"  
To spin new idea otherwise you would need to order real machine.  
And then worry about security, updates, power...  
It's not practical.  
Bare metal makes sense if you are huge corporation, like Microsoft.

Virtual Machines  
With virtual machines you can take server and subdivide it.  
But you need still to  
- care about updates
- care about your own cloud
- pay for each debian instance running all time

Public VM Cloud  
AWS, Azure, Google...  
you probably have to pay 50% more  
but it still makes sense, because you don't have to worry about a lot of things  
Terraform, Chef, Puppet, Salt: can help you spin vms

Container  
You're running a copy of something  
And you're running it on container demon  
you don't have a full cost of VM  
container is lighweight compared to VM  
and separation is almost as secure  
most revolutionary infrastructure innovation from last 10 years

There is a way to run container as VM.  
but we will not talk about it.

#### Chroot
Run container  
Normally you don't want privileged but here we will be using more system commands
```bash
docker run -it --name maciejka-host --rm --privileged ubuntu:jammy
```

Check version of linux
```bash
cat /etc/issue
```

Present working directory
```bash
pwd
```

Create folder and file
```bash
mkdir my-new-root
echo "my super secret thing" >> my-new-root/secret.txt
```
This fails (yet):
```bash
chroot /my-new-root bash
```
because it changes root successfully  
but then when it's runninng bash, it cannot find `bash` as almost nothing is there

copy bash and it's dependencies
```bash
mkdir my-new-root/lib/bin
cp /bin/bash my-new-root/bin/

mkdir my-new-root/lib
ldd /bin/bash
cp /lib/aarch64-linux-gnu/libtinfo.so.6 /lib/aarch64-linux-gnu/libc.so.6  /lib/ld-linux-aarch64.so.1 my-new-root/lib/
chroot /my-new-root bash
```
few commands come together with bash  
like `cd`, `pwd`

locking down filesystem  
process thinks that its in root directory  
it cannot use filesystem to navigate out of "jail"

#### Namespaces
allow to hide processes from other processes  
cannot even guess process id of other namespace  
UTS: Unix Timesharing namespace

a lot of depth here, we will skip details  
it's core concept for security and resource management

#### show list of running containers
```bash
docker ps
```

#### connect to the same container
(have another bash connected to same container)
```bash
docker exec -it maciejka-host bash
```

stop process id 71
```bash
kill -9 71
```

unshare

install debian
```bash
apt-get update -y
apt-get install debootstrap -y
debootstrap --variant=minbase jammy /better-root
```
*use unshare*
```bash
unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot /better-root bash
mount -t proc none /proc # process namespace
mount -t sysfs none /sys # filesystem
mount -t tmpfs none /tmp # filesystem
```
after that the unshared process cannot see other processes  
we may say we "contained" that process

#### CGroups (controll groups)
Prevent situation when other process would  
- eat all of the RAM
- pin the CPU

cgroups where invented by google  
to prevent situation when bad code in one process  
would handicap other services on the same machine

cgroups is a bit strange api  
it's a sudo filesystem  
you write configurations to files

check root cgroup  
this cgroup has all the capacity  
(normally it wouldn't make sense to limit here)
```bash
ls /sys/fs/cgroup/
```
cpu  
io: how fast you can read/write to the disc  
memory  
cgroup.threads: how many threads can be spawned

you create a new cgroup by making a subfolder  
and cgroups will autopopulate this new folder
```bash
mkdir /sys/fs/cgroup/sandbox
```
although it will not copy all of files  
for example `cgroup.controllers` will be missing  
this is controlled by `cgroup.subtree_control`  
which if empty, says don't move any controllers to sub folders

#### adding a process to cgroup
process can belong only to one cgroup  
change 8262 to your PID number (process id)
```bash
echo 8262 > /sys/fs/cgroup/sandbox/cgroup.proc
```
this will  
- add 8262 to sandbox
- automatically remove it from root cgroup: /sys/fs/cgroup/cgroup.proc

as long as cgroup has any procs you cannot edit controllers  
you have to temporarly move all procs to another cgroup  
you can move only one proc at a time
```bash
mkdir /sys/fs/cgroup/other-procs
cat /sys/fs/cgroup/cgroup.procs
echo 1 > /sys/fs/cgroup/other-procs/cgroup.procs
echo ... > /sys/fs/cgroup/other-procs/cgroup.procs
```
then add controlers to subtree config entry
```bash
echo "+cpuset +cpu +io +memory +hugetlb +pids +rdma" > /sys/fs/cgroup/cgroup.subtree_control
```
this will automatically populate existing cgroups:
```bash
ls /sys/fs/cgroup/sandbox/
```

command to pin around 1GB RAM and 100% CPU:
```bash
yes | tr \\n x | head -c 1048576000 | grep n
```

*limit RAM to 80MB:*
```bash
echo 83886080 > /sys/fs/cgroup/sandbox/memory.max
```

span "yes" into void
```bash
yes > /dev/null
```

*limit CPU to 5%*
```bash
echo '5000 100000' > cpu.max
```

fork bomb
```bash
:(){ :|:& };:
```

*limit number of processes*
```bash
echo 3 > pids.max
```

### 3. Docker
Image: premade container  
reusable structure that you can instantiate many times  
ubutu:jammy is an image

#### premade containers:
https://hub.docker.com/  
(like npm for containers)  
*search in cli*  
`docker search python`

run docker host in docker:
```bash
docker run -it -v /var/run/docker.sock:/var/run/docker.sock --privileged --rm --name docker-host docker:26.0.1-cli
```
-t --tty allocate pseudo terminal  
-i --interactive, keep stdin open even if not attached  
-v mount a volume (so we can use host containers in child container)  
--rm automatically remove container when exits (no stopped container)

start container in that container  
but because docker.sock was shared, it will be visible in host docker
```bash
docker run --rm -dit --name my-alpine alpine:3.19.1 sh
```

#### export container and inspect it
(tar is like zip, but not compressed)  
tar stands for "tape archive":  
format structures into linear so that they can be stored on tape
```bash
docker export -o dockercontainer.tar my-alpine
mkdir container-root
tar xf dockercontainer.tar -C container-root/
```
#### run docker container without docker
```bash
unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot $PWD/container-root ash
mount -t proc none /proc
mount -t sysfs none /sys
mount -t tmpfs none /tmp
```
... here you would need to config cgroups to have basic docker imitation

#### stop container
```bash
docker kill my-alpine
```

#### run latest
```bash
docker run -it alpine
```
same as
```bash
docker run -it alpine:latest
```

#### run specific version
docker run will always start a new container
```bash
docker run -it alpine:3.19.1
```

running without it will start container  
and in this case it will immediatelly close it:
```bash
docker run alpine
```
because this one doesn't have any commands

this will start container, run ls, and stop
```bash
docker run -it alpine ls
```

#### run in background
-d --detach
```bash
docker run -dit ubuntu:jammy
```

#### attach to running container
useful to inspect, see running logs
```bash
docker attach <ID or name>
```
exec: is going to execute something new  
attach: just attach me to whatever is running (this may be not interactive)

when attaching, watch out for exiting  
because it can close and stop container  
how to detach from inside of container?

show all images:
```bash
docker image ls
```

remove image after exiting  
`--rm`

#### remove all stopped containers
```bash
docker container prune
```
remove just one
```bash
docker rm <name or hash>
```

#### show all containers, also stopped
```bash
docker ps --all
```

have a named container, instead of random name  
`--name`

#### run node
```bash
docker run -it --rm node:20
```

get into container and play with it
```bash
docker run -it --rm node:20 bash
node -v
npm -v
```
check what os it runs on
```bash
docker run -it --rm node:20 cat /etc/issue
```
run on alpine instead of default debian
```bash
docker run -it --rm node:20-alpine
```

check all images for node:  
https://hub.docker.com/_/node

#### full / slim
debian  
full: 900MB  
slim: minimal debian ~200MB  
metals (iron...): names of stable

use full for development  
change to minimal when prod shipping

#### run deno
```bash
docker run -it denoland/deno:centos-1.42.4 deno
```
you don't have to understand how to setup it

container names:  
if docker maintains, it doesn't have slash:
```bash
docker run -it node
```
if 3rd party maintains, it has a slash:
```bash
docker run -it denoland/deno deno
```

#### run bun
```bash
docker run -it oven/bun:1.1.3 bun repl
```

alpine linux  
it used to be smallest by far  
now it has competition

pull image into cache, but run later
```bash
docker pull bcbcarl/hollywood
docker run -it -rm bcbcarl/hollywood
```
uses `screen` linux command to divide into splits

pause container  
probably not used often, but maybe you have a case
```bash
docker pause <name or hash>
docker unpause <name or hash>
```

show history (of image?)  
when it was created
```bash
docker history node:20
```

info, total memory, is it in debug mode
```bash
docker info
```

top, inspect processes in container
```bash
docker run -dit --name my-mongo --rm mongo
docker top my-mongo
```
#### restart container
```bash
docker restart <name or hash>
```

### 4. Dockerfiles

#### Dockerfile
a manifesto of what goes into container  
- take this image
- add these users
- copy these files
- run this command
you can build your own and run it over and over

name: Dockerfile  
no extension  
(or perhaps no basename)

most basic Dockerfile:
```Dockerfile
FROM node:20
CMD ["node", "-e", "console.log(\"hi lol\")"]
```
every line is called instruction

node -e "..."  
-e, --eval: run whatever is in string

#### build docker
```bash
docker build .
```

then copy hash from terminal and run
```bash
docker run 139055cade292280e47b2b5a15c3d18e95f4ae0ec764971240
```

--tag, -t: give a name to image
```bash
docker build -t my-node-app .
docker run my-node-app
```
by default tag has version 1
```bash
docker run my-node-app:1
```

build version 2
```bash
docker build -t my-node-app:2 .
docker run my-node-app:1
```

#### layers
reuse images  
layer is a unit of cache in docker  
we built our image on top of node image  
which is built on top of another image  
...  
which in end is built on top of base debian image  
each step of this is called layer  
but also each instruction creates layers

layers help to make whole solution cachable  
docker is very smart in detecting was there a change  
and skiping if not

**CMD**  
used in execute  
not in build

ENTRYPOINT  
not used so often  
CMD: you can override command that can be run  
ENTRYPOINT: cannot be overriden

**Minimal node server**  
index.js
```javascript
const http = require("http");
http
  .createServer(function (request, response) {
    console.log("request received");
    response.end("omg hi", "utf-8");
  })
  .listen(3000);
console.log("server started");
```
**Not able to close with ctrl+c**  
use --init
```bash
docker run --init my-node-app:3
```
normally you should handle shutdown signal in node server  
--init is a bit blunt

**FROM**  
you can use it with both docker hub and local ones
```Dockerfile
FROM my-node-app:3
```
although this way (local) is only usable to you  
so it's better to have a space at docker hub and publish there

**COPY**
```Dockerfile
COPY index.js index.js
```
source destination

better to put destination more organized
```Dockerfile
COPY index.js /home/node/code/index.js
```
if directory doesn't exist, it will be created

by default files will be owned by root  
to change this:
```Dockerfile
COPY --chown=node index.js /home/node/code/index.js
```
or (second part is a name of group)
```Dockerfile
COPY --chown=node:node index.js /home/node/code/index.js
```

**WORKDIR**  
change working directory  
like `cd` into directory
```Dockerfile
WORKDIR /home/node/code
```
paths in COPY can be shorter

**ADD**  
works a bit like COPY but does more  
- unzips if needed
- can add from url
in practice it's not commonly used

**PUBLISH**  
-p, --publish 3000:3000 (host-machine:inside-container)  
expose port  
by default containers don't expose any ports
```bash
docker run --init -it -p 3010:3000 my-node-app2
```

also there is EXPOSE inside Dockerfile  
but it's only suggestion
```Dockerfile
EXPOSE 3000
```
it needs a flag when running  
run only with -p flag (but no specifics)
```bash
docker run --init -it -p my-node-app2
```
however port is a random high port  
check it with
```bash
docker ps
```
and look for port section in table

often port that you have to use is given from outside  
AWS will give you port that you have to use

**USER**  
it's generally better to not use default user
```Dockerfile
USER node
```

with this all processes in container will belong to node  
without processes will belong to root

node container has user node  
but not all containers will have user provided  
its easy to add your own user  
`RUN useradd -ms /bin/bash myuser`  
-m, --create-home: create users home directory  
-s, --shell: which login shell to use

**RUN**  
runs at build time
```Dockerfile
RUN npm ci
```
ci: run a pure clean package-lock.json  
always follow package-lock.json  
if it has any issues, it gives up

#### .dockerignore
list of ignored files in COPY or ADD  
same format as `.gitignore`  
some defaults:
```gitignore
node_modules/
.git/
```
(generally don't copy any os dependent clang, sass, turbolinks)

#### caching
Docker is really smart about caching  
and it tries to skip every instruction  
it looks in Docker line for first situation something has changed  
but this can be a problem:
```Dockerfile
FROM node:20
WORKDIR /home/node/code
COPY . .
RUN npm ci
CMD ["node", "index.js"]
```
after docker detects that copy changed something  
it will always assume that next steps need to rerun  
so it will do full npm ci, even if not needed  
to solve this we have to reorder:
```Dockerfile
FROM node:20
WORKDIR /home/node/code
COPY package.json package-lock.json .
RUN npm ci
COPY . .
CMD ["node", "index.js"]
```
or shorter:
```Dockerfile
COPY package*.json .
```

and now more steps have CACHED  
however, this will only work if you're building in environment that has cache  
but in CI/CD github actions none of this caching matters  
becuse there is no docker cache there

### 5. Making Tiny Containers
let's "productionize" containers

#### smaller = more secure?
having smaller container is a bit more secure  
because there is smaller area  
but it's not strong corelation  
don't get obsessed with making container smaller

sometimes hosting will charge depending on the size of container

just by changing
```Dockerfile
FROM node:20
```
to
```Dockerfile
FROM node:20-alpine
```
our example image shrinks  
from 1.1GB to 143MB

#### alpine
small as possible to do specific task  
based on BusyBox

what is missing?  
(reasons to use, say, debian)  
apt-get  
linux utilities for diagnosing memory leaks  
man page (it's actually quite large)

```bash
apk add
```
alpine's version of apt get

*making own Node.js alpine container*
```Dockerfile
FROM alpine:3.19
RUN apk add --update nodejs npm
RUN addgroup -S node && adduser -S node -G node
```
last command will create user and group "node"  
it could be written as two instructions instead of &&  
but it's better if one thing is done by one instruciton

size: 82MB

not sure what we did cut out  
what was that 60MB difference?  
perhaps we shouldn't mess with it

#### multi stage builds

you don't need npm to run your project  
it's only needed to build

after we can populate node_modules we can copy them  
and not have npm anymore to run
```Dockerfile
# build step
FROM node:20 as node-builder
WORKDIR /build
COPY package*.json .
RUN npm ci
COPY . .

# production step
FROM alpine:3.19
RUN apk add --update nodejs
RUN addgroup -S node && adduser -S node -G node
USER node
WORKDIR /home/node/code
COPY --from=node-builder --chown=node:node /build .
CMD ["node", "index.js"]
```
essential line is in flag in COPY  
use previous step
```Dockerfile
COPY --from=node-builder --chown=node:node /build .
```
--from=...

size: 72MB

two step has a security use  
if there is something like pull secrets from some place  
so that later, production step doesn't have connection to that secret storage

we don't have to explicitly delete first step

more then two stages?  
it's possible if you need to build in very many steps:  
here is my astro builder  
here is my sass builder

#### Distroless

`musl` (alpine version of glibc)  
you may not use alpine if you use kubernetes: https://martinheinz.dev/blog/92  
it's a super edge case with using `musl`

alternatives:  
debian slim  
wolfi  
red hat micro  
google's distroless

google's distroless is not really distroless  
(not some kind of pure linux kernel)  
it's still debian, very "hacked"

```Dockerfile
FROM gcr.io/distroless/nodejs20
```
gcr: google container registry  
it allows only for node (uses ENTRYPOINT)

size (two step): 137MB

it's a choice of who do you want to trust  
(who did better at building secure container)  
- alpine
- google (distroless)
- debian (slim)

#### Astro project
```Dockerfile
FROM node:20 as node-builder
WORKDIR /app
COPY package*.json .
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:stable-alpine
WORKDIR /usr/share/nginx/html
COPY --from=node-builder --chown=nginx /app/dist .
```
and to run:
```bash
docker build -t test-astro . && docker run -p 8080:80 test-astro
```

### 6. Docker Features

#### scout
inspect container for vulnerabilities
```bash
docker scout quickview <image name or id>
```

cves: common vulnerabilities and exposures  
(published vulnerability)

#### mount / volume
bind mounts:  
files will live on docker host machine  
but use them inside container  
(for example have a local astro but serve it trough nginx)  
(useful to avoid rebuilding docker in development)  
... but also container can modify these files

volumes:  
files/data will live inside container  
and it will be kept after container is stopped  
(by default container loses all state at that point)

#### bind mounts
```bash
docker run --mount type=bind,source=./dist,target=/usr/share/nginx/html -p 8080:80 nginx
```

.dist may not work
```bash
source=./dist,target=/usr/share/nginx/html
```
in that case use absolute path:
```bash
source="$(pwd)"/dist,target=/usr/share/nginx/html
```

potential problem with npm i  
if you would bind whole dev folder on docker there may be mismatch in node_modules  
your local build will have node_modules for mac  
but inside container it needs ones for linux  
... so in that case make sure to run npm i inside container

binds are bidirectional  
read/write

#### volumes
database in container  
you can use bind mount (because it's read and write)

but let's say you want to:  
have backups of that database  
save it and ship it to prod as it is

```bash
docker run --env DATA_PATH=/data/num.txt --mount type=volume,src=incrementor-data,target=/data incrementor
```
--env DATA_PATH=/data/num.txt: ignore it, is just some env expected in our script  
src=incrementor-data: can be anything, we're giving a name to volume

volumes can be exported  
(this can be used for backup)

#### list volumes:
```bash
docker volume ls
```

#### remove volume
```bash
docker volume rm <name or hash>
```

#### inspect volume
```bash
docker volume inspect <name or hash>
```

#### tmpfs
a memory only volume  
works only on linux  
usefull: for keeping secrets for a moment

#### dev containers
MS feature (VS code)  
solving a problem with spending weeks to setup  
they include .devcontainer  
open in vscode, it will ask to open as dev container  
you can go back to this later by "> reopen in container"

.devcontainer/devcontainer.json  
dev container can be obviously different from production one

it's ok that your project has multiple Dockerfiles

dev container is visual studio acting like a SSH entered container  
it's creating a secure tunnel  
you can SSH into any box  
you can also SSH into AWS EC2 that you want to use as your "dev box"

#### networking
when it works, you don't think about it  
when it doesn't, you want to die

we have an app and a database  
networking is making them able to talk  
you can also put them into one container, but that's worse idea  
you want app containers and database container and scale them separatelly
```bash
docker network ls
docker network rm <name or id>
```
you can use the default one "bridge"  
but then every container will see every container  
but it's better to have bespoke for each concerns  
also docker doesn't recommend it

```bash
docker network create --driver=bridge app-net
```

there are other drivers, rarelly used  
bridge: default  
host: use network of host machine directly (don't isolate host network from container)  
overlay: for swarms  
ipvlan/macvlan: control of IP addressing  
none...

start mongo with network connected:
```bash
docker run -d --network=app-net -p 27017:27017 --name=db mongo:7
```
--name=db

write app using mongo, build it as my-app-with-mongo  
_at this point you can run app locally, connected to running mongo container to test_
```javascript
...
const url = process.env.MONGO_CONNECTION_STRING || "mongodb://localhost:27017";
...
```
```bash
docker run -p 8080:8080 --network=app-net --init --env MONGO_CONNECTION_STRING=mongodb://db:27017 my-app-with-mongo
```
--env pass environmental variable into container  
container name `db` is used here:  
MONGO_CONNECTION_STRING=mongodb://db:27017

how to make something more complicated?  
use composer or kubernetes instead  
if you have 4+ containers you need some orchestration  
most popular seems to be kubernetes

you could still use low level networking  
but there are better tools for that


### Multi Container Projects, Orchestration

#### Options
Docker Compose:  
perfect for local, start 4+ containers, connect them over network:  
they used to say don't use in production  
but now they say use it if production is simple enough  
compose is also good for CI/CD

Kubernetes:  
for more complex situations, like:  
- more than one instance of docker
- scale to zero
- native (functions running in cluster)
amazing but also complicated  
rarelly used by devs for running local

Kompose:  
takes docker compose files and turns them into kubernetes

#### Docker Compose
everything we do by hand in cli will end in configuration file

commands  
docker compose: v2  
docker-compose: v1

docker-compose.yml
```yml
services:
  api:
    build: api
    ports:
      - "8080:8080"
    links:
      - db
    environment:
      MONGO_CONNECTION_STRING: mongodb://db:27017
  db:
    image: mongo:7
  web:
    build: web
    environment:
      API_URL: http://api:8080
    ports:
      - "8081:80"
```
we will start three services  
this one is just a bare image:  
`image: mongo:7`  
because there is no port forward, this mongo is only available inside docker network, it's not public

ask to enter api directory and run docker build  
`build: api`

to start:
```bash
docker compose up --build
```
--build flag: watch out, easy to miss

docker compose will setup network for you

#### how to scale using compose
```bash
docker compose up --scale web=10
```
create 10 web instances  
atm. it won't work, because all web services are on the same port and there is nothing in front of these web containers (load balancer)

```bash
docker compose up --scale db=10
```

#### kubernetes, K8S
abbreviated as K8S "kates"  
next level of difficulty, a lot of moving parts  
build for larger scale

control plane:  
brain of cluster  
this crashed, restart  
deciding to scale up/down  
sometimes named as "master node"  
often not charged (free in azure, google)  
but in aws charged

nodes:  
a deploy target  
individual worker, can be many things  
can be one or many containers  
in llm training this is usually one powerful node

pod:  
a group of nodes  
things that you have to deploy together  
but also they will scale together  
for that reason db and app should be separate pods  
its very often that one pod has one container

service:  
group of pods that make up one backend  
examples: backend service, frontend service, data science service

deployment:  
update of several services

#### kubernetes
strongly linux, harder on windows  
`kubectl`: cli tool, kube controll (it can controll minikube or docker desktop builtin)  
switch between minikube and docker:
```bash
kubectl config use-cotext minikube
```
can also select azure, aws ...  
or shorter just "use"

if you use minikube, start it: `minikube start`

#### kates, k8s in docker:
settings > kubernetes > enable  
restart, and check status in mac systembar  
should be:  
"docker is running"  
"kubernetes is running"

install kubectl  
check https://kubernetes.io/docs/tasks/tools/
```bash
brew install kubectl
```
or curl

http://kompose.io  
official kubernetes project to get started with kubernetes  
takes docker-compose.yml and turns into kubernetes  
good first step after having compose working

*modify docker-compose.yml before running kompose*  
take old compose config and add a bit to it:
```docker-compose.yml
services:
  api:
    build: api
    ports:
      - "8080:8080"
    links:
      - db
    depends_on:
      - db
    environment:
      MONGO_CONNECTION_STRING: mongodb://db:27017
    labels:
      kompose.service.type: nodeport
      # kompose.service.nodeport.port: "8080"
      kompose.image-pull-policy: Never
  db:
    image: mongo:7
    ports:
      - "27017:27017"
  web:
    build: web
    links:
      - api
    depends_on:
      - api
    labels:
      kompose.service.type: LoadBalancer
      kompose.service.expose: true
      kompose.image-pull-policy: Never
    ports:
      - "8081:80"
```
#### depends_on
```
depends_on:
- db
```
inform that one service cannot start without another  
(kubernetes is very precise about order of starting services)

#### labels

additional information to trace
```
labels:
  kompose.service.type: nodeport
  kompose.image-pull-policy: Never
```
kompose.service.type: nodeport  
run all stuff and expose to rest of clusters is trough one port  
it basically does load balancing

kompose.image-pull-policy: Never  
by default kubernetes wants to pull from registry  
but because our images haven't been published set to never pull
```
labels:
  kompose.image-pull-policy: LoadBalancer
  kompose.service.expose: true
```
kompose.image-pull-policy: LoadBalancer  
used for nginx  
LoadBalancer is special case of nodeport  
inform that when deploying to Azure use "frontdoor"

kompose.service.expose: true  
inform that this service should be public

#### ports
for kompose you have to be more explicit on every service  
(cannot skip this part)

#### links
for web set that it both links and depends on api

#### run kompose
```bash
kompose convert --build local
```
creates two files for each service  
api-service.yaml (description of service)  
api-deployment.yaml (how to deploy)

#### kubernetes
```bash
kubectl apply -f '*.yaml'
```

kubernetes is harder to debug than docker compose  
because it's in more places

with kubernetes you can roll forward and backward

```bash
kubectl get all
```
show overview

```bash
kubectl cluster-info
```
should give info about control plane running  
there is also url to web dashboard  
(it's ok its unsafe)  
it requires a bit of setup otherwise it will message about forbidden

CoreDNS: kubernetes is running own DNS

`kubectl cluster-info dump`  
a lot of info on configuration, running status  
also logs of pods (and containers inside)

Rancher: on top of kubernetes  
its'a a nice dashboard for kubernetes

#### scalling
kubernetes provides a proper loadbalancer  
so that whole thing is ready for scalling

let's say api is getting hammered
```bash
kubectl scale --replicas=5 deployment/api
```

scale other services:
```bash
kubectl scale --replicas=5 deployment/web
kubectl scale --replicas=5 deployment/db
```

scale back down:
```bash
kubectl scale --replicas=1 deployment/db
```

delete all:
```bash
kubectl delete all --all
```

kubernetes can do a lot:  
secret manager  
internal networking

in the end you will probably use it trough cloud:  
KS: kubernetes service (KE: engine)  
- Azure AKS
- Amazon EKS (this one is very expensive)
- Google GKE

#### Manage secrets, tools
Docker secrets  
Vault (HashiCorp)  
Azure keyvault

### Other
#### Docker alternatives
Buildah: only for building, is compatible with Docker, can read Dockerfile  
Open Container Initiative: standard for interop containers  
Podman: for running, doesn't have a deamon like Docker does  
Colima: made for macOS  
rkt: deprecated  
containerd: actual engine for running that Docker uses  
gVisor: running containers with security as top goal  
Kata: running containers as VMs, more secure but heavier, not recommended for running everything, just most crucial services from security point

#### Kubernetes alternatives
mesos: older than Kubernetes, almost got deprecated, very complicated, apache project  
Docker Swarm: initially Docker fighted against Kubernetes, they made this tool but it lost, atm somewhere between compose and kubernetes  
OpenShift: on top of Kubernetes, expands Kubernetes with CI/CD  
Rancher: on top Kubernetes, provides GUI  
Nomad: "we think kubernetes is too complex, use nomad if you just want to run some containers on production", Hashicorp

#### fixed versions
using fixed version: good, because you avoid suprises from breaking changes  
but how often you update these manually typed versions?

#### a bit of fastify
solve cors issues in fastify:
```javascript
async function start() {
  await fastify.register(cors, {
    origin: "*",
  });
```
#### a bit of mongo
has an eventual consistency  
after some time

if there are several nodes, there is election mechanism and one of nodes is selected as primary

#### other
MonaLisa font: for code ligatures, italics  
Caddy: Nginx replacement  
Orchestration: how to make apps communicate with each other  
`ldd /bin/ldd`: not a dynamic executable  
htop: terminal tool to monitor memory, cpu ...  
Ubuntu: is a downstream debian, whatever change in debian it's pulled into ubuntu  
Railway.app: another simple AWS: build, ship and monitor apps  
Hollywood: `docker run -it -rm bcbcarl/hollywood`  
fastify: another node server, like express, hapi, ...  
% at the end of cat: if file is missing linux file end  
git codespace: editor, online vscode  
scale to zero: if something is not used, it will be "scaled to zero", not present at all  
docker swarm: not used, you can achieve this with kubernetes  
"master": polically not correct anymore :D, use "main"
