Helm, the definitive guide from beginner to master
==================================================
https://www.udemy.com/course/definitive-helm-course-beginner-master/learn/lecture/47512429#overview

#### Helm Overview
Way to describe, version and parametrize groups of k8s resources.  
Chart: application or a group of templates  
Template: kubernetes manifests

Chart can be public or private,  
Helm will install it with its dependencies.

Release: single installation of helm chart.  
Helm stores release state in Kubernetes Secrets.  
(this can be a problem when removing these secrets)

```bash
k describe secrets sh.helm.release.v1.my-wordpress.v1
```

Repository: reusable charts stored as tar files  
May be private or public  
http://artifacthub.io

#### Helm public repositories
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo list
# update cache
helm repo update
helm repo remove bitnami

helm search repo wordpress
helm search repo wordpress --versions
helm show chart bitnami/wordpress
helm show readme bitnami/wordpress
```

#### helm install
```bash
helm install my-wordpress bitnami/wordpress
helm install my-wordpress bitnami/wordpress --version 23.1.20
watch kubectl get all
# from private folder
helm install my-app .
```

#### providing values
custom-values.yaml
```yaml
wordpressUsername: myuser
wordpressPassword: mypass
```

usage examples
```bash
helm install my-wordpress bitnami/wordpress -f custom-values.yaml
helm install my-wordpress bitnami/wordpress \
  --set "wordpressPassword=mypass"

# check values provided in release
helm get values my-wordpress
```

#### helm values
```bash
helm show values bitnami/wordpress
helm get values my-wordpress
helm get values --all my-wordpress
```

#### helm uninstall
```bash
helm list
helm uninstall my-wordpress
k get pv
k delete pvc data-my-wordpress-mariadb-0
```



Kubernetes for developers
=========================
Manning  
https://livebook.manning.com/book/kubernetes-for-developers  
Repo  
https://github.com/WilliamDenniss/kubernetes-for-developers

### Deploying to Kubernetes
#### Authenticate in Docker Hub
As alternative ECR or Artifact Registry (Google Cloud) can be used
```bash
docker login
docker login -u maciejka docker.io
```

#### Build and tag
```bash
cd Chapter02/timeserver
IMAGE_TAG=maciejka/timeserver:1
docker build . -t $IMAGE_TAG
```

#### Push docker image
```bash
docker push $IMAGE_TAG
```

#### Error getting credendials
```bash
security -v unlock-keychain ~/Library/Keychains/login.keychain-db
```

#### Minimal Deploy configuration
Configuration files  
YAML is more popular  
JSON is another option (usually for automated access)

deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: timeserver
spec:
  replicas: 3
  selector:
    matchLabels:
      pod: timeserver-pod
  # pod object template, "PodSpec"
  template:
    metadata:
      labels:
        pod: timeserver-pod
    spec:
      containers:
        - name: timeserver-container
          image: docker.io/maciejka/timeserver:1
```

Important parts are: name, replica count and image path.  
If image doesn't have domain, then it's by default hosted on docker hub.  
Labels and match-labels can be any key-value pair, even `foo: bar`.

#### Deploy
```bash
k create -f deploy.yaml
k apply -f deploy.yaml
```

#### kubectl CRUD
```bash
k get
k create
k apply
k delete
```

#### Observe state of deployment
```bash
k get deploy
k get deploy timeserver
k get pods
k get pods --watch
k get pods --selector=pod=timeserver-pod
watch kubectl get deploy
watch -d kubectl get deploy
```

#### Get all
```bash
k get all
k get deploy,svc
```

#### Expose with port forward
```bash
# deployment
k port-forward deploy/timeserver 8080:80

# with ssh port forwarding
ssh -L 8080:localhost:8080 maciejka@91.214.2.28
k port-forward deploy/timeserver 8080:80

# service
k port-forward service/timeserver 8080:80
```

visit http://localhost:8080  

more than one port pair can be provided in port-forward
```bash
k port-forward deploy/timeserver 8080:80 9090:90
```

to enable forwarded port from public,  
bind port on all network interfaces
```bash
k port-forward --address 0.0.0.0 deploy/timeserver 8080:80
k port-forward --address 0.0.0.0 service/timeserver 8080:80
```

there is also minikube forwarding  
however, it will use random high port
```bash
minikube service timeserver
```

#### Use local container registry
For faster development  
avoid pushing to docker hub.

Change container part
```yaml
spec:
  containers:
  - name: timeserver-container
    image: timeserver:latest
    imagePullPolicy: Never
```

To run
```bash
minikube image build -t timeserver:latest app
k apply -f .
k port-forward --address 0.0.0.0 services/timeserver 8081:80
```

Minikube cannot reload image when it's running
```bash
k delete -f .
minikube image build -t timeserver:latest app
k apply -f .
k port-forward --address 0.0.0.0 services/timeserver 8081:80
```

Get minikube image list and prune dangling
```bash
minikube image ls
minikube ssh -- docker image prune -f
```

#### Accessing Kubernetes from local Docker
If you have Docker running on some machine  
and want one container to access Kubernetes  
then that Docker can just reuse forwarded port  
Under the host address of: `host.docker.internal`

```bash
k port-forward service/timeserver 6379:6379
```

Configuration settings inside container app
```
redis.Redis(host='host.docker.internal', port= '6379')
```

#### Log and troubleshoot
For future troubleshooting it's good idea  
to start app with log statement about port it's running on  
(describe also includes environment variables)

```bash
k describe pod/timeserver-6ccf9cd79-b98h6
k logs timeserver-6ccf9cd79-b98h6
k logs timeserver-6ccf9cd79-b98h6 --previous
k events timeserver-6ccf9cd79-b98h6

# check 
k logs -f deploy/timeserver

# check memory and disk
minikube ssh -- free -m
minikube ssh -- df -h
minikube logs

# remove pod and let k8s recreate it
k delete pod timeserver-6ccf9cd79-b98h6
```

Any deployment that terminated is considered a crash.  
Even if it ended with a successful exit code.  
(for one shot tasks use "job" not "deployment")

#### Minikube reset
```bash
minikube stop
minikube delete
minikube start --nodes=3
minikube start --memory=4096 --cpus=2
```

#### Minikube virtualbox
```bash
minikube delete
minikube config set driver virtualbox
```

#### Service
Way to expose pods (which get internal IP)

It's possible to expose single Pod (using `hostPort`).  
Way often pods are aggregated behind load balancer.  
This makes even sense with one pod, to provide stable IP.

Service will route traffic only to running pods.

service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: timeserver
spec:
  selector:
    pod: timeserver-pod
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  type: LoadBalancer
```

it will enable outside port 80  
that is routed to pod targetPort 80  
(by default targetPort is same as port)

```bash
k apply -f service.yaml
k get service
```

to get external IP, k has to be run in cluster  
on local minikube, port-forward Service

#### Exec
Deployment will select a random pod
```bash
k exec -it pod/timeserver-6ccf9cd79-g98c4 -- sh
k exec -it deploy/timeserver -- sh
k exec -it deploy/timeserver -- echo "Testing exec"
```

#### Copy files from/to container
```bash
k cp timeserver-6ccf9cd79-g98c4:server.py server.py
k cp test.txt timeserver-6ccf9cd79-g98c4:.
```

#### Cleaning
Deleting deployment will delete all pods
```bash
k delete deploy timeserver
k delete service timeserver
k delete pod timeserver
```

or delete by referencing files or directory
```bash
k delete -f deployment.yaml
k delete -f .
```

#### Delete cluster itself
in GKE
```bash
gcloud container clusters delete $CLUSTER_NAME --region $REGION
```

#### Imperative Kubernetes commands
Create deployment, service and then update container version
```bash
k create deployment timeserver --image=docker.io/maciejka/timeserver:1
k expose deployment timeserver --type=LoadBalancer --port 80
k set image deployment timeserver timeserver=maciejka/timeserver:2
```

#### Export configuration from cluster
Exported files will have some extra fields that you will need to remove.

```bash
k get -o yaml $RESOURCE_TYPE $RESOURCE_NAME
```

#### Change connected k8s instance
```bash
k config get-contexts
k config use-context docker-desktop
```

there is also kubectx tool for that
```bash
kubectx
kubectx $CONTEXT
```

#### Check all listened ports
```bash
sudo lsof -iTCP -sTCP:LISTEN -n -P
```

#### Secrets
```bash
k get secrets
k describe secrets my-wordpress
k get secret my-wordpress -o jsonpath='{.data.wordpress-password}' | base64 -d

k create secret generic db-pass \
  --from-literal password=mypass

k get secret db-pass -o jsonpath='{.data.password}' | base64 -d
```

#### Storageclass
```bash
k describe storageclass standard
```

#### Minikube addons
```bash
minikube addons enable storage-provisioner
minikube addons enable default-storageclass
```

#### Persistent Volumes (Claims)
```bash
k get pv
k delete pvc data-my-wordpress-mariadb-0
```




Kubernetes in 4 Hours
=====================
https://learning.oreilly.com/live-events/kubernetes-in-4-hours/0636920056367/  
Sander van Vugt, O'Reilly

Service: exposing to outside  
Volume: making it persistent

#### What is Kubernetes
Platform for running container-based cloud-native applications.

Cloud are couple of servers  
that are not really visible for us.  
Around them you build cloud environment  
And then you run App on that

You don't run app on specific servers.  
It's decoupled from details of specific servers.

Cloud native is about decoupling

#### how to decouple?
This is the goal of Kubernetes.  
You can run your application.  
And it will be distributed on cloud.

#### resources
Kubernetes offers different resources that allow storing infroamtin in the cloud.

#### Pods
adds cloud properties to your containers  
its a set of properties that you add to container

basic unit in Kubernetes,  
represents one or more container  
that share common resource

```bash
kubectl get pods
kubectl get pods  -o yaml | less
```

output part  
pod can have one  
or more containers
```
  spec:
    containers:
    - image: nginx
      imagePullPolicy: Always
      name: nginx
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-zrxr5
        readOnly: true
```

other interinst parts
```
nodeName: minikube
priority: 0
```

kubernetes managed pods, not containers

#### deployment
the application itself  
a standard entity of Kubernetes  
on how many pods your app is running

```bash
kubectl get deploy
kubectl get deploy myweb -o yaml | less
```

contains information of number of replicas


#### scalable
more or less application instances

#### availability
when your application crashes  
kubernetes will restart it  
will make sure it's running

#### orchestrator
Kubernetes is orchestrator for containers  
makes sure that services are where they are required  
Today it's de-facto standard of orchestrating containers in cloud native environment.

#### Google Borg
Kubernetes is based on Google tech used in data centers (Borg)  
Kubernetes is Open Source

#### image
Archive, that has all dependencies required to run application.  
They are fetched from registries, online repositories.  
Same image can be started locally and in cloud.

So you can test it reliably locally, and then push them.  
And they should behave in the same way.

#### container
A running image.  
When it's running, container needs to be managed.

#### container engine
aka "container runtime"  
Docker, Podman, Kryo, Container-d

Kubernetes adds cloud features to containers  
by managinng them in pod resources.

#### Container needs in Cloud
all are provided by Kubernetes  
- storage not bound to any specific physical environment
- cluster of hosts to run containers
- monitoring and self healing of containers
- solution for updated without downtime
- a flexible network that can self extend if that is needed

#### Kubernetes host platforms
- hosted service in public cloud: (azure) AKS, (elastic) EKS, GKE (and many more)
- on top of physicall cluster (on premise), you take care of your servers
- all-in-one solution, minikube (good for learning)

#### CNCF, standarization of K8
Cloud native computing fundation.  
Part of Linux fundation.  
Standarization of K8s.  
Google wanted Kubernetes to be Open Sourced.

https://cncf.io/projects  
Solutions ready to run  
like container-d a container runtime

Helm: package manager for Kubernetes  
Prometheus: observability tool (checking what your app is doing)  
and more ...

#### Kubernetes Distributions Alternatives
Amazon ECS  
Docker Swarm  
Hasicorp Nomad  
Amazon Fargate

#### Kubernetes Distributions
Rancher  
Red Hat OpenShift (this one has a lot of additions)  
Google Anthos  
EKS, AKS, GKS

#### sandbox
https://learning.oreilly.com/interactive-lab/kubernetes-sandbox/9781492062820/

#### Some commands
Health Check
```bash
kubectl cluster-info
```

View all nodes
```bash
kubectl get nodes
```

View all pods
```bash
kubectl get pods --all-namespaces
```

View local containers
```bash
crictl ps
```

SSH to the second node
```bash
ssh node01
```

#### How to run
this will create a website client that allow starting applications

```bash
minikube dashboard
```

however in dev-ops, it's more important to do configuration  
and it's not convienent that individual is doing configuration  
using the web clients like this one  
so yaml configuration files are way better

plus icon > create from form > deploy

then you see stats:  
deployments running: 1  
pods running 3  
replica sets running 1

#### Resources Kubernetes uses
Containers put into Pod  
Pod: minimal unit K8s works on, but this part is not managed  
Deployment: contains multiple pods

Kubernetes applications are started in isolated network  
To access them from external, you need Service

#### Service
Service: it's a misleading name  
It has load balancer  
and it makes your application accessible from outside

#### Ingress
What makes your http and https applicationis accessible  
it talks with Services

#### Storage
PVC: persistent volume claim  
persistent volume: most typically cloud storage

#### Configuration Storage
Configuration Map  
Secret

they contain:  
configuration files  
environment files

#### Run and deploy with cli
same as dashboard above

```bash
kubectl create deploy myweb --image=nginx --replicas=3
```

#### Managing kubernetes
cli (kubectl)  
API (calling rest api of controll plane directly)  
web dashboard (click ops)

#### kubectl help
command shell

```bash
kubectl --help
kubectl create --help
kubectl create deployment --help
kubectl create deployment -h
```

help has good documentation  
and usefull examples

#### get
```bash
kubectl get all
```
#### get api resources
show 
```bash
kubectl api-resources
```

has many apis  
v1  
apps/v1

in the beginning you had to run manually pods and replicationcontrollers  
(it was long time ago, deployments didn't exist then)

api is extenable, for example ippools is api from calico.org  
you can add new apis

#### How many containers in Pod?
Usually, by standard there is one container per one pod  
There are good cases for having more containers close together

#### Deployment
To run application create deployments
```bash
kubectl create deploy mynginx --image=nginx --replicas=3
```
deployment adds scalability, protection and zero-downtime upgrades to pods  
Do not run standalone pods (naked pods)

If you delete pod that is used  
by deployment, it will be recreated  
(new pod will be created in its place)  
because kubernetes thinks: I need three replicas, one is missing
```bash
kubectl delete pod myweb-574d7b7f-c55g4
```

problem with running naked pods  
is that they are not protected from accidental delete
```bash
kubectl run nginx --image=nginx
kubectl delete pod nginx
```

```bash
kubectl get all
kubectl get pods
kubectl get all --selector app=mynginx
```

create few more
```bash
kubectl create deploy second --image=nginx --replicas=5
```

#### labels
in some situations get all will be too long  
to be really usable  
in those cases use labels to subselect

```bash
kubectl get all --show-labels
kubectl get all --selector app-myweb
kubectl
```

#### describe
the way to inspect and troubleshoot  
also the way to see event logs

```bash
kubectl describe pod myweb-574d7b7f-l6zq2
```

if you see  
Exit code: 1  
then it means it couldn't start

#### logs
```bash
kubectl logs mydb
```

#### using k8s in declarative way
so far we are typing all the commands  
but if your project is larger you prefer standarization

#### yaml manifest
configuration as code  
defined in yaml manifest  
write this once and put in github repo

they can get quite complex

to see runinng yaml  
you can see it with flag
```bash
kubectl get pods  -o yaml | less
```

#### generate yaml manifest
generate yaml file with dry-run
```bash
kubectl create deploy mynginx --image=nginx --dry-run=client -o yaml
```

and then write it to file
```bash
kubectl create deploy mynginx --image=nginx --dry-run=client -o yaml > mynginx.yaml
```

deployment should have labels  
labels are quite important in kubernetes

apiVersion: apps/v1  
at some point we will have v2
```bash
kubectl api-resource  | grep -i deploy
```

#### what is available
check kubernetes docs  
https://kubernetes.io/docs/home

or run explain command  
what can I put in my specification
```bash
kubectl explain deploy.spec | less
```

#### yaml
in yaml indentation is important

```yaml
spec:
  replicas: 1
  minReadySeconds: 30
```

#### apply manifest
```bash
kubectl apply -f mynginx.yaml
```

it will check do some resources have to be created  
or updated, it will report:
```
deployment.apps/mynginx created
deployment.apps/mynginx configured
```

after you edit manifest, use `kubectl apply`

to remove all connected resources:
```bash
kubectl delete -f mynginx.yaml
```

you can use kubectl create/update  
but it's better to use apply  
because you don't have to distinguish is it created

#### add labels to manifest
to add more labels, go to template section
```yaml
template:
  metadata:
    creationTimestamp: null
    labels:
      app: mynginx
      time: break
```

#### scale running app
```bash
kubectl scale deploy mynginx --replicas=5
```

#### namespaces
isolated environment for running applications

get pods
```bash
kubectl get pods
```

get pods and namespaces  
there you can see dashboard and controll
```bash
kubectl get pods -A
```

each application will be installed  
in separate namespace

you can say that namespaces can be used  
to create virtual data centers

create a namespace
```bash
kubectl create ns secret
kubectl run secretpod --image=nginx -n secret
kubectl get pods -n secret
```

#### troubleshooting
```bash
kubectl get all
```
look for CrashLoopBackOff

tools to troubleshoot
```bash
kubectl explain
kubectl logs
```

as alternative
```bash
kubectl get pods -o yaml
```

you can also enter pod directly  
however this is not so usefull

```bash
kubectl create deploy mydb --image=mariadb --replicas=3
kubectl describe pod mydb-7c9ddb78dc-bs9qk
kubectl logs mydb-7c9ddb78dc-bs9qk
kubectl set env deploy/mydb MARIADB_ROOT_PASSWORD=secret
```

Last command will update ubernetes deployment named “mydb”  
by adding (or updating) the environment variable.

### Access from outside
#### External, physical network
Nodes are connected to external, physical network

#### Cluster network
but Nodes are also connected to software internal network  
this one is private

#### Pod network
Pods are connected to pod network  
it's behind firewall, and cannot be directly accesssed

This is where your pods will connect to  
private, internal software network

Some parts of you pod network you want to make visible  
Pod in pod network have port numbers

#### Sevice resource
Internal Kubernetes load balancer  
Connected to cluster network  
Translates request to Pod network

Service uses ClusterIP

every application should be exposed  
by it's own Service

As typically multiple instances of pods are started  
a load balancer is needed to connect incomming user requests  
to a specific pod

#### way to make service accessible
using Node Port  
its a Service type with port forwarding

user will access external port  
it's forwarded to service  
and that is forwarded to pods

#### Ingress
Additional resource that provides external access to http  
Using Ingress requires taht Ingress application is installed on cluster  
(Ingress controll), it's not added by default  
Different Ingress solutios are provided by Kubernate ecosystem

most applications today are http/https  
to make your http apps accessible, use Ingress  
Ingress is a reverse proxy  
and it's difectly connected to Services

#### Sevice Types
ClusterIP: default service, accessible only within cluster  
NodePort: exposes an external port on the cluster nodes (it's quite limited)  
Ingress: what should be used for http/https

because NodePort is limited, sometimes to get extra options path is  
Ingress is calling NodePort

#### Create Service, expose app
```bash
kubectl create deployment nginxsvc --image=nginx
kubectl scale deployment nginxsvc --replicas=3
kubectl expose deployment nginxsvc --port=80
kubectl describe svc nginxsvc
kubectl get svc
kubectl get endpoints
kubectl get pods -o wide --selector app=nginxsvc
```

services connect to pods by labels  
they know what to expose, because both use nginxsvc label

ports visible in kubectl get svc  
are ClusterIP

#### curl cluster network
check that cluster network is working  
but it will be visible from within cluster  
10.109.249.15 is read from `kubectl get svc`

```bash
minikube ssh
curl 10.109.249.15
```

#### change type of service
```bash
kubectl edit svc nginxsvc
```

change type from ClusterIP to NodePort  
kubectl edit svc nginxsvc
```
  ports:
  - nodePort: 30782
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginxsvc
  sessionAffinity: None
  type: NodePort
```

#### check service port redirect and ip of engine
*(this is minikube only command)*
```bash
minikube ip
kubectl get svc
```

80:30782/TCP  
192.168.49.2

visit  
192.168.49.2:30782

### Storage
Containers are ephemeral by nature.  
To prevent data loss, use volume.
```bash
kubectl explain pod.spec.volumes
```

many options  
we will use emptyDir and hostPath

shared storage between two containers
```yaml
volumes: 
  - name: test
    emptyDir: {}
```

and then on each container define
```yaml
volumeMounts:
  - mountPath: /centos1
    name: test
```

end for second container
```yaml
volumeMounts:
  - mountPath: /centos2
    name: test
```

test that volume is shared
```bash
kubectl create -f morevolumes.yaml
kubectl get pods morevol
kubectl describe pods morevol | less
kubectl exec -ti morevol -c centos1 -- touch /centos1/test
kubectl exec -ti morevol -c centos2 -- ls -l /centos2
```

#### Persistent storage
let's say you have preprod and production environment  
and let's say that you have two solutions for storage

if you wan't to be able to change it automatically  
then use storage provision  
StorageClass

pod -> pvc  
persistent volume claim  
pvc will talk with StorageClass  
StorageClass will respond with created persistent volume  
(that it creates by calling provisioner)

MiniKube also has a storage class  
so that it can be also used in that dynamic storage

```bash
kubectl get pvc,pv,storageclass
```

persistent storage can be only created by yaml file  
(not cli directly)  
pvc.yaml
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

```

run it
```bash
kubectl apply -f pvc.yaml
kubectl get pv
```

example of client  
pv-pod.yaml
```yaml
kind: Pod
apiVersion: v1
metadata:
   name: pv-pod
spec:
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
  containers:
    - name: pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-storage
```

run it
```bash
kubectl apply -f pv-pod.yaml
kubectl exec -it pv-pod -- touch /usr/share/nginx/html/HELLO
minikube ssh
cd /tmp
ls
cd hostpath-provisioner/default/pv-claim
ls
```

you should be able to see created file

### Config maps
When you need to provide configuration  
Somethimes you need more config for preprod than prod  
Then you need replication to pick configuration from some place  
And that's why you use Config maps

Commonly used for env vars  
There are also secrets  
They are not really secret, but more secure

```bash
kubectl create deploy mynewdb --image=mariadb --replicas=3
kubectl get pods --selector app=mynewdb
kubectl create cm mynewdbvars --from-literal=MARIADB_ROOT_PASSWORD=password
kubectl describe cm mynewdbvars
kubectl set env --from=configmap/mynewdbvars deploy/mynewdb
kubectl get all --selector app=mynewdb
```

in this example `kubectl describe cm mynewdbvars` will show that password  
this is not secure and for that there is better solution

```bash
kubectl get deployments.app mynewdb -o yaml | less
```

this will be visible as  
and this definition is flexible  
it tells to find config map with that value  
and if config maps can be just found, it will work
```yaml
spec:
  containers:
  - env:
    - name: MARIADB_ROOT_PASSWORD
      valueFrom:
        configMapKeyRef:
          key: MARIADB_ROOT_PASSWORD
          name: mynewdbvars
```

### Ingress
Ingress solves problem of resolving domain name,  
so that kubernetes NodePort can understand what IP is requested.

you need Ingress controll  
without it will not work

```bash
minikube addons list
minikube addons enable ingress
kubectl get pods -n ingress-nginx
kubectl create ing nginxsvc --rule="myapp.info/=nginxsvc:80"
kubectl describe ing nginxsvc
```

add minikube ip to /etc/hosts  
curl myapp.info

recently Ingress has gone to feature freeze  
and prefered recent way is `GateApi`  
(but it's more complicated and will take more time)

### Clear things we created
```bash
minikube stop
minikube delete
```



Excerpt from Nest.js fundamentals
=================================
run docker compose in detached mode  
(in background)
```bash
docker compose up -d
```

run only one service
```bash
docker compose up db -d
```



Kubernetes in action
====================
https://www.manning.com/books/kubernetes-in-action-second-edition  
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

becuse of that it's important that every docker RUN  
will clear any temporary files before it ends

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

#### Building the application
```bash
docker build -t kiada:latest .
```
-t specifies desired name and tag

list images
```bash
docker images
```

Build is actually done by Docker Deamon  
Docker CLI is just giving instruction to do it

dont' add unnecessary files to build directory  
as they will slow down the build process  
especially if Docker deamon is running on remote system

#### image layers
each individual directive in Dockerfile  
creates a one Docker layer

check image sizes by running
```bash
docker history kiada:latest
```

#### running container
```bash
docker run --name kiada-container -p 1234:8080 -d kiada
```
--name container name  
-d detach from console (run in background)

and then visit http://localhost:1234

#### Docker will install VM
if your container is linux  
but your local machine is macOS or Windows  
then Docker will install Linux virtual machine

#### list running containers
prints id, name, image and comamnd it executes  
also shows date created and ports mapping
```bash
docker ps
```

to show more about container, use inspect
```bash
docker inspect kiada-container
```

container output is stored by Docker in logs  
to access them use logs command
```bash
docker logs kiada-container
```

#### distributing container image
let's push image to public docker hub registry  
you can also use other registries, such as quay.io  
or Google Container Registry

#### Docker Hub image naming schema
before publishing, image has to be re-tagged  
according to Docker Hub image naming schema

image tag must include Docker Hub ID  
which you choose when registering to Docker Hub

give additional tag image
```bash
docker tag kiada maciejka/kiada:0.1
```

confirm that it has new name
```bash
docker images
```

#### Pushing to Docker Hub
before you must log in
```bash
docker login -u maciejka docker.io
```

send container to docker hub
```bash
docker push maciejka/kiada:0.1
```

after pushing, image is available to all  
you can run the image anywhere with command
```bash
docker run --name kiada-container -p 1234:8080 -d maciejka/kiada:0.1
```

#### Stopping and deleting container
first send termination signal to the main process to stop gracefully.  
if process doesn't respond or wait time ends, docker kills it
```bash
docker stop kiada-container
```

stopped container still exists  
in case you want to start it again  
see stopped containers
```bash
docker ps -a
```

start stopped container
```bash
docker start kiada-container
```

remove container  
all content is removed  
image is still there
```bash
docker rm kiada-container
```

to remove container and image, use `rmi`
```bash
docker rmi kiada:latest
```

remove all unused images
```bash
docker image prune
```

#### Linux Namespaces
ensure that each process will only see its:  
files, processes and network interface  
feature of linux kernel  
when creating a new process it can be assigned to namespace

there is one namespace for each resource  
mnt: mount points (file systems)  
pid: process id  
net: network devices, ports  
ipc: inter process communication (between processes, message queues, shared memory)  
uts: unix time sharing system, domain name, tool for multitasking  
user: users and grups  
time: allows each container to have its own system clock  
cgroups: limits in accessing resources

each network interface belongs to one network namespace  
but can be moved from one namespace to another

thanks to namespaces there can be many `eth0` interfaces  
as long as each is in different namespace  
(same for localhost `lo`)

by looking at interfaces, container can't tell whether it's in container  
or runing directly on a bare-metal

there is on default network namespace

#### sharig namespaces
sometimes you want to share certain resource  
like domain name or network interface

container is something that runs with namespaces assigned

it's possible to run a "debug container"  
which will share network of exising container  
this will allow to debug networking system  
with tools that may not be available in observed container

#### explore running container
```bash
docker exec -it kiada-container bash
```
-i run in interactive mod (allows to input commands)  
-t allocate psudo terminal (allows to have prompt and internal TERM variable that commands expect)

running this command in host OS **will** show contained processess  
host OS has to be linux  
although PID number will be different  
(process can belong to more than one PID namespace)
```bash
ps aux | grep app.js
```

on macOS Docker starts linux virual machine  
to enter it run
```bash
wsl -d docker-desktop
```
or
```bash
docker run --net=host --ipc=host --uts=host --pid=host -it --security-opt=seccomp=unconfined --privileged --rm -v /:/host alpine chroot /host
```

#### Linux Control Groups (cgroups)
limits system resources like CPU, memory, network and disk

without doing anything additional,  
Docker container has unrestricted access to all CPU of host

limit explictly which cores can be used
```bash
docker run --cpuset-cpus="1,2"
```

flags to limit availabe CPU time  
`--cpus`, `--cpu-period`, `--cpu-quota`, `--cpu-shares`  
allow the container to use only haof of a CPU core
```bash
docker run --cpus="0.5"
```

flags to limit memory  
`--memory`, `--memory-reservation`, `--kernel-memory`, `--memory-swap`  
set maximum memory size available in the container to 100MB
```bash
docker run --memory="100m"
```

#### strengthening isolation between containers
sys-calls enable to load/unload modules from kernel  
change system time, start a process...

most containers run with sys-call disabled  
to enable them use `--privileged` flag

if only some sys-calls are needed  
there is a way to give subset of them  
linux kernel divides privileges into units caalled capabilities  
CAP_NET_ADMIN allows network related operations  
CAP_NET_BIND_SERVICE allows to bind to port numbers less than 1024  
CAP_SYS_TIME allows to modify system clock

capabilities can be added or removed from container

follow principle of least privilege when runnin containers  
don't give them any capabilities they don't need

#### Secure Computing Mode, seccomp
used to give finer control over what sys-calls program can use  
seccomp definition is a JSON file that lists the system calls  
that container is allowed to make

and then that file is provided to Docker when container is created

#### AppArmour, SELinux (security enchanced linux)
More ways to control security  
both enable to mark files and system resources  
either by labels or defining paths

they allow to access only files and resources  
which labels match set of policy running application has

### 3. Kubernetes: Deploying your first application
Setting multi-node Kubernetes cluster isn't simple  
it requires proper network setup  
to allow all containers to communicate with each other

installing and managing kubernetes cluster  
is much more difficult than useing it  
for that reason there are several ready solutions:  
GKE: Google Kubernetes Engine  
EKS: Amazon Elastic Kubernetes Service  
AKS: Azure Kubernetes Service  
IBM Cloud Kubernetes Service  
Alibaba Colud Container Service

there is also an option to setup kubernetes cluster  
on virtual machines, using `kubeadm` tool

#### Kubernetes cluster in Docker Desktop
Docker Desktop contains single node Kubernetes cluster  
you can enable it in the settings  
it's a easy start, but it may be a older version

Reset Kubernetes Cluster  
use it whenever you want to remove all objects deployed

enabling "show system containers" will show list of k8s containers:
```
k8s_storage-provisioner_storage-provisioner_kube-system_e240b4ec-d2b8-4d53-ae62-8e7288bc02fe_1
k8s_coredns_coredns-7db6d8ff4d-z4fkt_kube-system_b6eb55f9-0524-4fd6-a1f9-e54854697860_0
k8s_coredns_coredns-7db6d8ff4d-5ztcj_kube-system_a57f2f36-8095-442b-88ef-1a114990349c_0
k8s_vpnkit-controller_vpnkit-controller_kube-system_564e1fbf-ddeb-4a8e-aefb-4618d5214fd5_0
k8s_kube-proxy_kube-proxy-np255_kube-system_9706d181-e312-4f37-bcb1-2142247fd211_0
k8s_POD_vpnkit-controller_kube-system_564e1fbf-ddeb-4a8e-aefb-4618d5214fd5_0
k8s_POD_coredns-7db6d8ff4d-5ztcj_kube-system_a57f2f36-8095-442b-88ef-1a114990349c_0
k8s_POD_kube-proxy-np255_kube-system_9706d181-e312-4f37-bcb1-2142247fd211_0
k8s_POD_coredns-7db6d8ff4d-z4fkt_kube-system_b6eb55f9-0524-4fd6-a1f9-e54854697860_0
k8s_POD_storage-provisioner_kube-system_e240b4ec-d2b8-4d53-ae62-8e7288bc02fe_0
k8s_kube-apiserver_kube-apiserver-docker-desktop_kube-system_3fe11f12ace7990088a27f5ac063bceb_0
k8s_kube-scheduler_kube-scheduler-docker-desktop_kube-system_aa7bbfbbe0588d06569a828ad4116992_0
k8s_kube-controller-manager_kube-controller-manager-docker-desktop_kube-system_2d884e49e38d30ded4b72cbfd8a93ca9_0
k8s_etcd_etcd-docker-desktop_kube-system_3436b10c8a489053c5f3ba0f32b03652_0
k8s_POD_kube-scheduler-docker-desktop_kube-system_aa7bbfbbe0588d06569a828ad4116992_0
k8s_POD_kube-controller-manager-docker-desktop_kube-system_2d884e49e38d30ded4b72cbfd8a93ca9_0
k8s_POD_etcd-docker-desktop_kube-system_3436b10c8a489053c5f3ba0f32b03652_0
k8s_POD_kube-apiserver-docker-desktop_kube-system_3fe11f12ace7990088a27f5ac063bceb_0
```

single node cluster:  
runs both control plane, kubelet and your applications  
kubelet is a kubernetes agent which manages the node

explore Docker Linux VM
```bash
docker run --net=host --ipc=host --uts=host --pid=host --privileged --security-opt=seccomp=unconfined -it --rm -v /:/host alpine chroot /host
```
-v mounts host root directory to the /host directory in the container

#### Minikube
safest way if you're new  
Tool maintained by Kubernetes community  
normally runs k8s in Linux VM

Minikube has an option to run on VM  
but you need hypervisor like VirtualBox

you then don't need Docker

```bash
brew install minikube
minikube start
minikube status
minikube stop
```

to log into minikube and explore it from inside
```bash
minikube ssh
```

while inside, try
```bash
ps uax
docker ps
```

#### Kind
Kubernetes in Docker  
multi node  
*opposite to Minikube and Docker Desktop which are single node*

kubelet in Kind use CRI-O  
instead of Docker

more complex and similar to real multi node scenario  
but still simple to debug, as all runs on host OS  
where its possible to use tools like Wireshark to debug  
or even browser inside cluster network

nsenter  
tool to run tools in the network of container

install
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-$(uname)-amd64
chmod +x ./kind
mv ./kind /some-dir-in-your-PATH/kind
```

start cluster
```bash
kind create cluster
```

kind is single-node by default  
to make it multi-node, create configuration file  
kind-multi-node.yaml
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

start multi-node
```bash
kind create cluster --config kind-multi-node.yaml
```

list nodes, use one of these
```bash
kind get nodes
docker ps
```

log into cluster nodes
```bash
docker exec -it kind-control-plane bash
```

worker nodes and kubelet use CRI-O  
to interact with it use crictl
```bash
crictl ps
```

#### Google Kubernetes Engine
on start you get 300$ credit  
and Google **does not** charge you automatically after passing limit

interact using cli tool `gcloud`  
https://cloud.google.com/sdk/docs/install

install gcloud tool  
install / remove components
```bash
gcloud components install COMPONENT_ID
gcloud components remove COMPONENT_ID
```

update your SDK
```bash
gcloud components update
```

login, set default zone and project
```bash
gcloud init
```

one region  
europe-west3

has three avaibility zones  
europe-west3-a, europe-west3-b, europe-west3-c

to create three nodes in one zone  
set one of zones as default  
and create clusters with three workers
```bash
gcloud config set compute/zone europe-west3-c
gcloud container clusters create kiada --num-nodes 3
```

to create three workers each in different zones  
select whole region as default  
and create cluster with one worker
```bash
gcloud config set compute/zone europe-west3
gcloud container clusters create kiada --num-nodes 1
```

list nodes
```bash
gcloud compute instances list
```

control nodes in GKE are internal to Google  
and are not accessible directly at all

to access worker nodes  
(and then try `docker ps`)
```bash
gcloud compute ssh gke-kiada-default-pool-9bba9b18-4glf
```

each worker node is a virtual machine that you have to pay for  
to cut costs, scale to zero  
when scaling to zero, none of the objects created is deleted  
as soon as you scale up, all is redeployed
```bash
gcloud container clusters resize kiada --num-nodes 0
```

#### Amazon Elastic Kubernetes Service
First, install `ekstl` command line tool.  
https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

create cluster
```bash
eksctl create cluster --name kiada --region eu-central-1 --nodes 3 --ssh-access
```

use SSH to connect to nodes  
while inside, list running containers with `docker ps`

#### Deploying from scratch
Proper management of k8s is difficult.  
but if you want to try, go with `kubeadm` tool.  
Follow [Kubernetes The Hard Way](https://github.com/kelseyhightower/Kubernetes-the-hard-way)

#### Interacting with Kubernetes
using `kubectl` which talks with Kubernetes API server  
called: kube control, kube-cuddle or kube-c-t-l

```bash
brew install kubectl
kubectl --help
```

set alias  
.bashrc  
`alias k=kubectl`

#### Configure kubectl to use specific cluster
`~/.kube/config` file contains configuration of cluster  
its autocreated by Minikube, GKE and other...  
when getting invited to new cluster, you will receive such file

point kubectl by exporting env variable
```bash
export KUBECONFIG=/path/to/custom/kubeconfig
```

*(its also possible to use kubectl in more than one context)*
```bash
kubectl cluster-info --context kind-kind
```

#### Using kubectl
check connection to cluster
```bash
kubectl cluster-info
```

list nodes
```bash
kubectl get nodes
```

everything in Kubernetes is represented by an object  
and can be manipulated by API

get more info about node  
or about all nodes
```bash
kubectl describe node kind-worker
kubectl describe node
```

#### Web Dashboards
Not always available.  
On Docker Kubernetes there is a bit of [installation](https://livebook.manning.com/book/kubernetes-in-action-second-edition/chapter-3/v-14/210) needed.

```bash
minikube dashboard
```

#### Running first application, Imperative way
without a deployment definition (that would be declarative)

```bash
kubectl create deployment kiada --image=maciejka/kiada:0.1
```

by default image is pulled from Docker Hub  
but you can also specify image registry  
like: `quay.io/maciejka/kiada:0.1`

This creates deployment and stores it in Kubernetes API  
(it calls POST /deployments)  
It's a desired state.  
Kubernetes must now ensure that actual state matches it.

list deployments
```bash
kubectl get deployments
```

#### pods
one worker can have many pods  
one pod has one IP address and can have many containers

kubectl is all about creating and modifying objects  
there is no list containers  
container is not smallest unit of deployment

pod is group of co-located containers  
group of containers running on same worker node  
and sharing same network namespace (and other linux namespaces)  
which means: network interface, IP, port space

list pods
```bash
kubectl get pods
```

show details of pod  
(show it event log like pulling image)
```bash
kubectl describe pod
```

term scheduler  
actually it runs immediatelly  
scheduling in k8s means assigning containers to worker nodes

once pod is assigned to node it's never moved  
even when it fails, it's recreated on same node

#### exposing app to the world



Complete Intro to Containers, v2
================================
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
} 
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
