Fullstack Deployment: From Containers to production AWS
=======================================================
https://frontendmasters.com/workshops/modern-deployment/#player  
https://github.com/ALT-F4-LLC/fem-fd-service  
stage-01-start-up  
stage-02-growth  
stage-03-scale

Erik Reinert

Prerequisites Setup:  
https://gist.github.com/dtauer/2fb90b55865cbb3e8a7c9b198ab19650

Workshop perspective:  
Focused on evolution of application.  
Seeing application grow and scale.

Consider watching also:  
https://frontendmasters.com/courses/fullstack-v3/

Manage growing infrastructure  
Learn when to make changes  
Balance pros and cons of solutions  
Improve hands-on experience

#### Neovim markdown plugin
render markdown

#### Linking in Docker
legacy way for one container to communicate with another.  
It works by exposing environemtn variables and updating /etc/hosts  
As for today, use networks instead

```bash
docker network create mynet
docker run -d --network=mynet --name db postgres
docker run -d --network=mynet --name web my-web-app
```

#### Start-up Phase
Just get it working

#### Growth Phase
We gotta go fast!

#### Scale Phase
Plan for the future  
How much traffic we will have in year.

### Start-up Phase
looking for biggest wins with minimal setup

we are learning about what needs to be maintained  
don't optimize in that phase

#### VPS
Virtual Private Server  
AWS EC2  
Rackspace uses EC2

in "good old days" everybody would get VPS  
linux instace that you ssh into  
vps is great but it's outdated

arguments for VPS (arguments against containers)  
majority of development today is not k8s and terraform  
it's coolocated or vps

instead we prefer to use containers,  
so that in future it will be easier to grow

#### Containers
Our first decision, to use

#### Target Setup
Client browser will call AWS App Runner

AWS App Runner
- deploy image (AWS container image)
- fetch data (supabase database)
- get config (AWS parameter store)

Developer (command line)
- push image (AWS container image)
- Update schema (supabase database)
- Update config (AWS parameter store)
- deploy changes (AWS App Runner)

#### Database on AWS?
raw RDS sucks  
(AWS is generally great)

DBA: database administrator  
people which are hired to just work on database

having a public database url connection string  
makes things very, very easy on start

we will not stay with supabase  
because at some point it will start to cost a lot

#### Parameter store
Easy way to store secrets  
And it comes with some UI

Vault would be alternative,  
but it would require another deployment

AWS Secrets Manager?  
like a book with ton of pages  
good if you have a ton of secrets  
in comparison, parameter store is  
more like real key-value store

Also service discovery works better  
with approach of parameter store  
and accessing one value

#### CLI
on startup phase you will run cli tools a lot  
a lot of manual running commands

#### push to main
you can, but better not  
get used 

#### git worktrees
with normal git you have repo  
and switch between branches

with worktrees you have folder for each branch,  
it's great if you need to switch branches a lot

#### Google Cloud, GCP
Google Cloud Platform (GCP)  
we are using it so that Google handles authentication  
http://cloud.google.com

do I want to roll my own Auth?  
how do I know who is who, you can use Google/Apple/MS/...  
rolling your own auth is not that complicated  
just get it started and not overcomplicate

but for simplicity you can also use something ready  
something like GCP  
fem-fd-service

http://cloud.google.com  
GCP > Api and Services > Credentials > Create OAuth client ID  
(before there is a wizard to setup up OAuth)(before there is a wizard to setup up OAuth)(before there is a wizard to setup up OAuth)(before there is a wizard to setup up OAuth)(before there is a wizard to setup up OAuth)(before there is a wizard to setup up OAuth)(before there is a wizard to setup up OAuth)(before there is a wizard to setup up OAuth)(before there is a wizard to setup up OAuth)  
click + create client

#### OAuth redirect flow
Authorized redirect URIs  
OAuth is about handshaking  
We will go somewhere, we will validate  
and we will go back  
the provider needs to know who you are  
and how to redirect user back?

user goes to 3rd party  
3rd party generates token  
goes back to your site with that token

users go to 3rd party api  
before visiting your application

type  
http://localhost:8080/auth/google/callback

not https  
but its localhost  
(no one will get to localhost)  
(unless your computer is being hacked)

#### .env
possible to have export statements

```.env
export GOOGLE_CLIENT_ID=<google-client-id>
export GOOGLE_CLIENT_SECRET=<google-client-secret>
export GOOGLE_REDIRECT_URL=http://localhost:8080/auth/google/callback
export POSTGRES_URL=postgresql://postgres:password@localhost:5432/postgres?sslmode=disable
```

and with that its possible to   
load and export environment variables  
(for duration of terminal session)

```bash
source .env
```

(under the hook source runs bash)

#### run containers
```bash
docker compose up --detach
```

#### OrbStack
Alternative to Docker desktop  
That can also create VMs Virtual Machines

Seed database schema

```bash
source .env
docker compose exec postgres psql -U postgres -d postgres
```

Start app
```bash
source .env
go run main.go
```

#### ECR
Elastic Container Registry

To run containers  
first step is to put images something

create repository  
all resources are separated by account id

730...337.dkr.ecr.eu-central-1.amazonaws.com

#### Add a Dockerfile

```Dockerfile
FROM public.ecr.aws/docker/library/golang:1.24.2-alpine

# Set the working directory
WORKDIR /app

# Copy the go.mod and go.sum files
COPY go.mod go.sum ./

# Download the dependencies
RUN go mod download

# Copy the source code
COPY . .

# Build the Go application
RUN go build -o main .

# Expose the port the app runs on
EXPOSE 8080

# Command to run the application
CMD ["./main"]
```

#### Work with ECR
Push Commands  
help for cli commands   
it's on a ECR page  
copy paste them

**1 Sign in**  
Retrieve an authentication token and authenticate your Docker client to your registry.
```bash
aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 730...837.dkr.ecr.eu-central-1.amazonaws.com
```

**2 Build Docker image**
```bash
docker build -t fem-fd-service .
docker buildx build --platform linux/amd64 --tag fem-fd-service:latest .
```

AMD 64  
Second command `buildx build` is neccessary  
becuase on first we were running on new ARM MacBooks  
and build would work only on ARM MacBook  
the second command is needed to make it multiplatform

**3 Tag**  
tag your image so you can push the image to this repository:
```bash
docker tag fem-fd-service:latest 730...837.dkr.ecr.eu-central-1.amazonaws.com/fem-fd-service:latest
```

**4 Push image to ECR**
```bash
docker push 730...837.dkr.ecr.eu-central-1.amazonaws.com/fem-fd-service:latest
```

When this is done and ECR page is refreshed  
then image tag "latest" should be visible on list

#### Supabase
Has many alternatives

Create schema from sql

And then click "Connect"  
Direct connection is not IPv4 compatible  
Transaction pooler is more for lambda case  
Session pooler - we will use that one

#### Parameter store
AWS console  
SSM > Parameter Store  
click create

name:  
/fem-fd-service/google-client-id

select SecureString

key-value store  
key is a bit like url

you can later see these  
and use Parameter Store as a documentation place  
for yourself, in case you need to restore secret

#### IAM Role
To give permissions to execute actions  
which service needs to execute.

IAM is biggest point of pain for every developer

IAM > Policies  
Everthing with organge box is AWS managed  
try to use these AWS managed as much, as you can  
although sometimes you cannot  
and App Runner is a good example of that 

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": ["ssm:GetParameters"],
			"Resource": ["arn:aws:ssm<account-region>:<account-id>:parameter/fem-fd-service/*"]
		}
	]
}
```

get account id from right top menu

we also create role  
but we don't create a user  
because Amazon is user

Create role  
"Custom trust policy"

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "tasks.apprunner.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

Two types of policy on AWS  
Assume policy: I want to use that resource  
Just "policy"

#### IAM Policy
defines what actions can be performed  
once the role is assumed

#### Assume Role Policy
aka trust policy  
defines who can assume a role

#### App Runner
select ECR we crated

deployment settings: automatic  
everytime I push to ECR, automatically buildt

Create new service role  
(or reuse it if exists)  
AppRunnerECRAccessRole

select minimum
0.25vCPU
0.5GB

add 4 environemnt variables

SSM Parameter Store  
GOOGLE_CLIENT_ID  
arn:aws:ssm:eu-central-1:730...837:parameter/fem-fd-service/google-client-id

Plain text  
GOOGLE_REDIRECT_URL  
localhost:8080/auth/google/callback

SSM Parameter Store  
GOOGLE_CLIENT_SECRET  
arn:aws:ssm:eu-central-1:730...837:parameter/fem-fd-service/google-client-secret

SSM Parameter Store  
POSTGRES_URL  
arn:aws:ssm:eu-central-1:730...837:parameter/fem-fd-service/postgres-url

in AWS  
ARN: identifier of the resource

port 8080

in Security select role we created  
fem-fd-service

Auto scaling  
App runner supports  
Minimum size / Maxium size  
it scales automatically to your use

Concurrency  
100  
Minimum size  
1  
Maximum size  
25

these settings mean that when  
there are 100 concurrent requests

we will resize up, and then auto scale down  
when usage is back down below 100

create and deploy  
on the bottom we can see pending  
and see logs

#### Updating Parameter Store
to apply changes in parameter store  
redeploy in AppRunner after change

#### App Runner
It is that it can provide a bit of metrics,  
how many requests what CPU and memory consumption

#### Small summary
We had a lot of work done and imagine,  
we would have to deploy whole Kubernetes cluster like this

99% of the work is in the browser  
you didn't had to learn terraform

#### AWS calc runner
main cost will be Cpu  
AWS calc runner  
https://calculator.aws/  
https://calculator.aws/#/createCalculator/apprunner  
concurency: 20  
min prov containers 1  
peak traffic hours 8  
number of req in peak 10  
number of req off-peak 1

Cost: Total Monthly cost:
14.10 USD

### Growth Phase
Scenario
- we are shipping code to production
- we have no integration process
- we have no delivery process
- we don't want to replace anything

Goals
- improve data reliability
- improve developer experience
- create integration solution
- create delivery solution

Once you have more developers  
you want to separate them from resources  
because this is a point where things can break

#### Goose
migration tool in Go  
Create migrations in SQL

```bash
go install github.com/pressly/goose/v3/cmd/goose@latest
```

craete migration

```bash
mkdir -p migrations
goose -dir "migrations" create base_schema sql
```

it will create empty file for us to fill

migrations/20250507203205_base_schema.sql

```sql
-- +goose Up
-- +goose StatementBegin
SELECT 'up SQL query';
-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin
SELECT 'down SQL query';
-- +goose StatementEnd
```

add two new variables to .env  
this is a way to reuse same value  
in two env variables

```env
export GOOSE_DBSTRING=$POSTGRES_URL
export GOOSE_DRIVER=postgres
```

then test

```bash
source .env
echo $GOOSE_DBSTRING
```

restart: stop postgres and remove volume,  
because we will manage using migrations

```bash
docker compose down --remove-orphans --volumes
docker compose up --detach
```

check is goose working  
and run it

```bash
goose -dir "migrations" status
goose -dir "migrations" validate
goose -dir "migrations" up
```

#### Makefile
been for a long time  
it's a file for running commands  
so you can run much simpler commands

name of file is `makefile`

```makefile
DOCKERIZE_HOST := $(shell echo $(GOOSE_DBSTRING) | cut -d "@" -f 2 | cut -d ":" -f 1)
DOCKERIZE_URL := tcp://$(if $(DOCKERIZE_HOST),$(DOCKERIZE_HOST):5432,localhost:5432)
.DEFAULT_GOAL := build

build:
  go build -o ./goals main.go

build-image:
  docker buildx build \
    --platform "linux/amd64" \
    --tag "$(BUILD_IMAGE):$(GIT_SHA)-build" \
    --target "build" \
    .
  docker buildx build \
    --cache-from "$(BUILD_IMAGE):$(GIT_SHA)-build" \
    --platform "linux/amd64" \
    --tag "$(BUILD_IMAGE):$(GIT_SHA)" \
    .
```

in purest form it's for making files.

`make some-binary`: if that file exists,   
then makefile will not make it

but we will use makefile to run commands  
so this is a bit like scripts

first we will add bunch of variables  
makefile has its syntax, not really a language  
we are telling make, that there is a  
bunchof variables we want to use

```makefile
MIGRATION_DIR := migrations
AWS_ACCOUNT_ID := 677459762413
BUILD_TAG := $(if $(BUILD_TAG),$(BUILD_TAG),latest)
DOCKERIZE_HOST := $(shell echo $(GOOSE_DBSTRING) | cut -d "@" -f 2 | cut -d ":" -f 1)
```

in variables we can run bash commands  
by using `$(...)` and run conditionals

next we write "targets"  
which are list of commands  
make build => docker build ...

```makefile
build:
  go build -o ./goals main.go
```

anything you need to run  
and memorize ... put it into makefile

there is a way to use variable  
but if not present, use default value  
(few ways to do it)

```makefile
VARIABLE ?= default_value
```

Way to run list of targets

```makefiel
target1:
    @echo "Running target1"

target2:
    @echo "Running target2"

target3:
    @echo "Running target3"

all: target1 target2 target3
```

#### makefile for ci/cd
another use case ... github actions  
you can run exactly same commands  
that you run locally and in github actions

makefile alternatives (task runners)  
`just` is popular

#### check image sizes
docker images --format "{{.Repository}}:{{.Tag}} {{.Size}}"

we are having a lot of packages that we don't use  
we should switch to "alpine"

#### Dockerfile
you can have one stage

but you can also build many images in several steps  
and next step can use files from previous stage

#### Github actions
don't abuse them  
if you have unit tests, first run it locally  
this will save you a ton of money

yaml

we will use docker instead of calling go manually  
this way we prevent problems with changing go versions

check that build command works  
build-and-deploy.yml

```yaml
name: build-and-deploy

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make build-image
```

#### from ci/cd to aws
for github actions to talk to aws we need:  
cli and secrets

in IAM we will create user  
fem-fd-service-github-actions  
click attach policies directly  
click AdministratorAccess  
(in future better use granular)

AdministratorAccess  
will allow to do everything

Security credentils > create Access key > CLI > I understand  
copy paste access key (or download)

go to github actions and save this in  
secrets and variables > actions

#### Github Secrets vs Variables
difference is that first ones are not visible  
(you can put all into secrets but it's not recommended)

in github actions we can provide secrets and variables  
just for one selected action, with syntax like:

```yaml
name: build-and-deploy

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - run: make build-image

      - env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_DEFAULT_REGION: ${{ vars.AWS_DEFAULT_REGION }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: make build-image-promote
```

if in github actions you need some specific tools  
you can create docker image with them and then  
use that docker image as a worker

```yaml
runs-on: ubuntu-latest
```

#### sha as image version
tip: use sha as a version for docker image  
this way we don't build one code twice  
(build once, push many)

#### dependent github actions
one task in github actions can depend on another

#### deploy
make github actions migrate supabase schema  
we add for that another job in github action  
called "deploy"

```yaml
  deploy:
    concurrency:
      cancel-in-progress: false
      group: deploy-lock
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_ACCOUNT_ID: ${{ vars.AWS_ACCOUNT_ID }}
      AWS_DEFAULT_REGION: ${{ vars.AWS_DEFAULT_REGION }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      DOCKERIZE_URL: ${{ secrets.GOOSE_DBSTRING }}
      GOOSE_DBSTRING: ${{ secrets.GOOSE_DBSTRING }}
      GOOSE_DRIVER: ${{ vars.GOOSE_DRIVER }}
    if: github.ref == 'refs/head/main'
    needs:
      - test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - run: make build-image-pull
      - run: make build-image-migrate
      - run: make build-image-promote
```

will make sure that in case two actions trigger  
then only one will work at any point

```yaml
group: deploy-lock
```

only run it when on main branch

```yaml
if: github.ref = "refs/main/href"
```

#### Dockerize
a way to tell is container running inside ci/cd  
(because it's not really easy to tell)

#### "Closed loops" in CI/CD
We first test that migration is valid in ci/cd  
and then, when PR is merged, we apply it on real database

#### Future growth
these both allow for easy extension in future  
github actions  
makefile

Cons of growth phase  
no netowrk  
no way to build services indepedengly

#### lowest denominator: fastest way
generally try to find lowest denominator  
of many problems you want to solve

#### summary
before we were looking for minimal effort to get to cloud

we started with Dockerfile  
set up repository  
we added parameters and database and deployed to containers

and then in phase two  
we though: how to have better ci/cd  
how to be sure that we are deliverying correct thing  
we were cleaning a bit, so that we can keep moving at same pace

### Scale Phase
Scale is about concurent users,

#### organization scale
but it's even more about organization scale:  
how many teams you have  
you will have to navigate pretty efficient

more team members  
more services  
you want to support multiple teams

we want better environemnt

#### money: there is some budget
in scale phase we have some money to spend

in start and growth phase you look for  
cheap and free solutions

in this state you have some budget

#### example circle CI
it makes sense to spend some money to not think about something  
in ci/cd you are provisioning many workers on demand,  
and you don't have to think about it

it's about focusing on business  
and not thinking about some things

also vertical scalling: strong VPS may sometimes be better  
as it's generally easier to manage

#### promotion process
when you merge into main  
but then you want to merge that into staging  
and tag it and deploy in docker on staging  
and then promote to production

### Terraform
can get really big

#### plan
client calls Cloudfront CDN  
then call goes to AWS ALB (load balancer)  
then to AWS ECS (containers)

we will swap supabase to AWS RDS (database)

#### changes
we are only changing selected places  
not everything and these changes should be not visible  
to the end user (begind browser)

#### aws
we will stay on Amazon

#### CloudFront
Cloudfront CND is quite good solution  
it's all around the world  
and has good integrations with other solutions  
and this is important because we don't want to expose many things

#### Amazon accessible network
and with amazon we can create private subnetwork  
and load balancer will be private  
load balancer will be only available locally for cloudfront CDN

#### ECS
container orchestrator  
place which runs containers

it tries little bit to be Docker competitor

#### Kubernetes
if you want to have unified platform  
with which you can move everywhere...  
this is way another world  
in most cases you don't need it

as alternative you can also go for Kubernetes  
Kubernetes does it way better

it's actually suprising that there are many devs  
which got used to Kubernetes and didn't had experience with ECS  
and they have to learn ECS

biggest difference between Kubernetes and ECS  
is support of Open Source. Because if you need something  
that it's not there, then you have to wait for AWS to add it

With Kubernetes you have more options to do what you can need  
And you may like ArgoCD, but its entirelly Kubernetes thing

#### Evolution
Levels of complexity/control/abstraction  
App Runner  
ECS  
Kubernetes (probably you don't need it)

#### VPC
Virtual Private Cloud  
it just means private network  
in which you can run services

#### ownership
compared to previous step  
this step is much more about ownership  
so that you want want to run resources  
and not buy from other vendors

#### how many people to run?
DBA  
Devops more then one

#### RDS
If as organization you are in situation where you database secrets  
are becomming more and more important and crucial that they are not  
exposed, then you will be better using RDS, because this way secrets  
never leave Amazon Cloud.

It's using PostgreSQL  
which is great has a lot of features  
and plugins

#### General note about scale phase
It will take a lot of consideration about existing elements.  
And you can do it normally without a lot of pain

Apart from migration of Database  
for which you will have to find night with less traffic

#### Terraform
terraform state

terraform keeps data that is referential to things that are created  
so that this resource with this id is represented by this data in state  
it's very sensitive piece of data, that you have to make sure you have data

you have many options for setting it up and storing it  
you can store it in many places, we will choose S3  
because it has high avaibility, high redundancy and versioning if we would need it

#### S3
amazon s3 bucket names are globally unique  
this is quite annoying, because often you cannot use some name

#### terraform
add to gitignore

```
*.tfplan
.terraform
.overrides.txt
```

create folder `terraform`

#### Bastion Host
backdoor connection  
way to inspect your network  
old way is to only permit your ip  
backdoor to your network

we will make instance public  
of that private network  
and limit access to known IP  
Bastion Host

new better way is vpn  
but Amazon VPN are not cheap

terraform/locals.tf
```tf
locals {
  bastion_ingress = ["<ip-address>/32"]
}
```

#### Terraform Registry
https://registry.terraform.io/  
great docs  
has interesting one..

instruction how to controll Spotify with terraform  
(also potentially you can manage Kubernetes with terraform)

#### Terraform modules
Terraform provides normally "resources"

one unit, one resource that terraform  
will create, provision and manage

Terraform modules are  
community prebuild solutions

aws_vpc  
but what if you don't want to care for every resource  
and this is where aws module comes in  
https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest

you get a ton of value by using one of these  
instead than building all these resources for yourself  
resource by resource

#### Safety?
There are some famous modules, that can be trusted
well known modules in community and official ones.

To be sure about other modules, it makes sense
to look into their source code.

#### Terraform Kubernetes module
This module makes it very easy to get from nothing  
to having Kubernetes working, it's massivelly used  
https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest

#### State
instruction for terraform where to store state

terraform/backend.tf
```tf
terraform {
  backend "s3" {
    bucket = "fem-fd-service-altf4"
    key = "terraform.tfstate"
    region = "us-west-2"
  }
}
```

#### Terraform providers
terraform/providers.tf
```tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

#### Run Terraform
```bash
cd terraform
terraform init
```

it should result in something like:
```
Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.
Initializing provider plugins...

Terraform has been successfully initialized!
```

idemponent  
you can call terraform init many times,  
it will not break anything

#### .terraform.local.hcl
it's something like package-lock.json  
Terraform is worried that some of providers could become malicious  
so they started to record hashes of what you use  
all dependencies for hasicor aws 5.0

#### environment description
first problem is how to describe our environment  
and we want that description to be composable  
so that we don't have to copy a lot of staff

we wan't to organize our large module  
and inside of it have separated smaller modules

#### main.tf
you don't have to call it like this  
but this is an often convention  
and other files in folder are usually relative to this file

#### reuse existing in terraform
if something already exists  
you don't have to necessary create it

module/environment/data.tf
```tf
data "aws_caller_identity" "this" {}
data "aws_region" "this" {}
```

this is setting that we are expecting some things to exist  
we will use monorepo approach here for simplicity

#### terraform modules
have a separate module for each concern,  
separate for network, separate for database, separate for ECS

#### terraform variables
one example of using variables in terraform is:  
if you want to restrict who you want to access Bastion on dev  
and if you want to say that on production even no one should be able  
you can change these dynamically by tweaking one variable

module/environment/variables.tf
```tf
variable "bastion_ingress" {
  default     = []
  description = "CIDR blocks for bastion ingress"
  type        = list(string)
}

variable "name" {
  description = "Name of the cloud environment"
  type        = string
}
```

#### root main.tf
place where we define staging

benefit of using terraform is solving  
hard problems with easy solutions

IaC Infrastructure code

```tf
module "staging" {
  source = "./module/environment"

  bastion_ingress = local.bastion_ingress
  name            = "staging"
}
```

```bash
terraform plan -out "terraform.tfplan"
```

will not work, because module has to be initialized

```bash
terraform init
```

every time you create module, you have to call `terraform init`  
you don't have to run it for variables

#### Next step
we have environment, we can go with next step  
what is the first thing you create inside Amazon?  
Network

#### Network
In amazon world everthing is VPC  
VPC, Virtual Private Cloud  
we need network to provision everything  
we own the network

and network is first thing we want to have

we will use terraform module  
because we don't want  
to build own secure network  
that's the beauty of Open Source

```tf
module "subnets" {
  source  = "hashicorp/subnets/cidr"
  version = "1.0.0"

  base_cidr_block = var.cidr

  networks = flatten([
    for k, v in local.subnets : [
      for az in var.availability_zones : {
        name     = "${k}-${az}"
        new_bits = v
      }
    ]
  ])
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.21.0"

  azs                    = var.availability_zones
  cidr                   = var.cidr
  database_subnets       = [for az in var.availability_zones : module.subnets.network_cidr_blocks["database-${az}"]]
  elasticache_subnets    = [for az in var.availability_zones : module.subnets.network_cidr_blocks["elasticache-${az}"]]
  enable_nat_gateway     = true
  intra_subnets          = [for az in var.availability_zones : module.subnets.network_cidr_blocks["intra-${az}"]]
  name                   = var.name
  one_nat_gateway_per_az = false
  private_subnets        = [for az in var.availability_zones : module.subnets.network_cidr_blocks["private-${az}"]]
  public_subnets         = [for az in var.availability_zones : module.subnets.network_cidr_blocks["public-${az}"]]
  single_nat_gateway     = true

  default_security_group_ingress = [
    {
      self = true
    }
  ]
}
```

#### Cidr
Actuall network address space  
that you want to own
10.0.0.0/16  
...  
means I will have how many addresses?
10.0.0.0 - 10.100.0.0

and the lower the value  
the higher the address space

32bits in total  
16bits for network

here we define the size

we define variable cidr

terraform/module/network/locals.tf
```tf
locals {
  subnets = {
    "database"    = 6,
    "elasticache" = 6,
    "intra"       = 5,
    "private"     = 3,
    "public"      = 5,
  }
}
```

these are octets, how many octets  
private will have around 10-20k

we are dividing network into multiple spaces

who can calculate cidr (pronounced "siders")?  
we are using cird to calculate all the cidr

Classless Inter-Domain Routing.   
A method for allocating IP addresses and routing IP packets  
more efficiently than the older class-based system.

format of Cidr
```
192.168.0.0/24
```

192.168.0.0 is the IP address  
(often a network address).

/24 is the prefix length, meaning the first 24 bits  
are the network part, and the rest (8 bits) are available for hosts.

it reduces IP address waste

#### Intra network
what is the purpose?  
we had intranets and internet

intranet is completely isolated  
it's completely offline for the internet  
boxed internet

if you need subnet that is not connected to internet  
you can put databases there, this is one of candidates  
(if you want to)

#### Security groups
Place to set custom preferences  
to say that we want something to be able to connect to database

here is security group for database, here is for something  
its better than saying "this database can connect to this resource"  
it's higher level security groups that everything shares  
so if something is provisioned it gets one or many security groups  
it makes networking way simpler, avoiding definition of 1:1 connections

```security_group.tf
module "security_group_db" {
  source  = "terraform-aws-modules/security-group/aws"
  version = "5.3.0"

  description        = "Security group for db subnet"
  egress_cidr_blocks = ["0.0.0.0/0"]
  name               = "${var.name}-db"
  vpc_id             = module.vpc.vpc_id

  ingress_with_self = [
    {
      rule = "all-all"
    }
  ]

  egress_with_cidr_blocks = [
    {
      rule = "all-all"
    }
  ]
}

...

resource "aws_vpc_security_group_ingress_rule" "db_allow_private" {
  description                  = "Allow private subnet to access db"
  from_port                    = 5432
  ip_protocol                  = "tcp"
  referenced_security_group_id = module.security_group_private.security_group_id
  security_group_id            = module.security_group_db.security_group_id
  to_port                      = 5432
}

...
```

If we put some resources in one subnets  
by definition, Amazon will allow it to talk to everything else  
in the same subnet

we made small hole, but it's very small hole  
on 5432 port and we didn't complicate it with using random port

if we will change the port, you will get devs comming to you  
with question, why it's not available,  
because you will be just using the default port  
and that is easier to work with

we are putting some security, but not very granual

#### variables
if you have variable on package above  
you have to pass it down manually  
it's like React props

this way if you change in one place on top  
you want to propagate that change all the way down

terraform/module/network/variables.tf

```tf
variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

variable "bastion_ingress" {
  default     = []
  description = "List of CIDR blocks to whitelist for bastion host"
  type        = list(string)
}

variable "cidr" {
  description = "CIDR block"
  type        = string
}

variable "name" {
  description = "Name of the network"
  type        = string
}
```

#### AZ Avaibility Zone
Only scenario that AZ will go down  
is if the world goes down

You normally don't need 5 Avaibility zones  
usually just 2 are enough, one that you use  
and the another one as the failback

something you have two

It's very difficult to create a new Avaibility Zone  
if you don't use something like terraform.  
Because to add one, you would need to replicate everything

With IaC you can just create one variable  
and see all the resource being replicated in new place

#### Terraform outputs
What is output in terraform

If you group several resources in module  
how do you access them from outside?

Like how to do you tell that you want some security group or network  
as a value in other modules and resource

```tf
output "database_security_group" {
  value = module.security_group_db.security_group_id
}

output "database_subnets" {
  value = module.vpc.database_subnets
}

output "private_security_group" {
  value = module.security_group_private.security_group_id
}

output "private_subnets" {
  value = module.vpc.private_subnets
}

output "vpc_id" {
  value = module.vpc.vpc_id
}

output "vpc_name" {
  value = module.vpc.name
}
```

#### Depends_on
Terraform runs by building graph  
Directed Acyclic Graph  
and this is a way it build

there is also a way to write explicit `depends_on: [module_name]`  
which is what you sometimes do, when terraform has a problem  
to calculate the graph for some reason

#### Basion module
Example of how flexible terraform is

```tf
resource "tls_private_key" "bastion" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "bastion" {
  key_name   = "${var.name}-bastion"
  public_key = tls_private_key.bastion.public_key_openssh
}

module "bastion-private-key" {
  source  = "terraform-aws-modules/ssm-parameter/aws"
  version = "1.1.2"

  name        = "/${var.name}/bastion/private-key"
  secure_type = true
  value       = tls_private_key.bastion.private_key_pem
}

module "bastion" {
  source  = "terraform-aws-modules/ec2-instance/aws"
  version = "5.8.0"

  associate_public_ip_address = true
  instance_type               = "t3a.micro"
  key_name                    = aws_key_pair.bastion.key_name
  monitoring                  = true
  name                        = "${var.name}-bastion"
  subnet_id                   = module.vpc.public_subnets[0]

  vpc_security_group_ids = [
    module.security_group_bastion.security_group_id,
    module.security_group_private.security_group_id,
  ]
}
```

We want to use a lot of modules  
because you don't want to check each individual resource manually

here is place we use module
```tf
module "bastion" {
  source  = "terraform-aws-modules/ec2-instance/aws"
```

And above we are asking terraform to generate SSH keys  
so that you don't have to generate them manually  
here we are asking to create private and public RSA keys

```tf
resource "tls_private_key" "bastion" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "bastion" {
  key_name   = "${var.name}-bastion"
  public_key = tls_private_key.bastion.public_key_openssh
}
```

from that point, when developer wants ssh key  
you don't give it to him, you give him permission to get it

we also enabled monitoring  
and provided security groups  
(gave it two security groups)

basion and private because it has to connect to both of them  
we are using access via security group  
its giving bastion access to whole private network

And later if we tell private that it can access some new network  
then bastion will also be able to access that network  
which is again nice way to manage

```
subnet_id                   = module.vpc.public_subnets[0]
```

#### do we have per environment name space?
environment contains all it needs, including network  
so when creating environment, we are creating new network  
and production and staging will have separate VPC  
and this means there is no connection

In Amazon world VPC is also tool for telling what is isolated

But if in future we decide, that we want to have some things  
that were isolated, now merged into one network  
then we are risking that resources will have conflict of IP  
(this can happen when you want to refresh production with preproduction)  
for that reason we give something like

staging 10.0.0.0/16  
production 10.100.0.0/16


#### Three classes of private IP addresses
Class A 10.0.0.0 - 10.255.255.255  
Class B 172.16.0.0 - 172.31.255.255  
Class C 192.168.0.0 - 192.168.255.255

The Class B private IP address range 172.16.0.0 to 172.31.255.255  
uses the first two octets (172. and 16-31) to identify the network portion,  
and the last two octets for host addresses.

In binary, this range translates to
10101100.00010000.00000000.00000000 to
10101100.00011111.11111111.11111110  
, offering a large address space for medium-sized networks. 


#### Create resources in terraform
Provision everything  
and list what was created  
you will also see calculated IP addresses  
together with cidr_block

first we run plan  
then apply will really create and provision

```bash
terraform plan -out "terraform.tfplan"
terraform apply "terraform.tfplan"
```

output in console will be list of all resources  
that were linked, in some situations this can take  
even up to 30 minutes to synchronize

it's easy to duplicate everything in terraform  
configurations

#### database module
create database/main.tf  
main tf for out terraform module  
we need to access password

we create random string  
and we store it in the SSM  
and under that variable we are accessing it to futher

its possible to use VPC name as the network name

#### RDS can be very confusing
you normally hire a person DBA who can manage it  
and manage creating settings

#### Parameter Groups and Option Groups
Options that you provide and parameters to the Postgres  
Amazon provides defaults for each, so you don't have to create one  
And we are using the default groups here

it always worked for me,   
and I never needed to tweak anything about it

```tf
resource "random_string" "password" {
  length  = 32
  special = false
}

resource "aws_ssm_parameter" "password" {
  name  = "/${var.name}/database/password"
  type  = "SecureString"
  value = random_string.password.result
}

module "this" {
  source  = "terraform-aws-modules/rds/aws"
  version = "6.12.0"

  allocated_storage                   = 50
  create_db_option_group              = false
  create_db_parameter_group           = false
  create_db_subnet_group              = false
  create_monitoring_role              = false
  db_subnet_group_name                = var.vpc_name
  engine                              = "postgres"
  engine_version                      = "17.2"
  iam_database_authentication_enabled = false
  identifier                          = var.name
  instance_class                      = "db.t4g.micro"
  manage_master_user_password         = false
  max_allocated_storage               = 100
  option_group_name                   = "default:postgres-17"
  parameter_group_name                = "default.postgres17"
  password                            = random_string.password.result
  publicly_accessible                 = false
  skip_final_snapshot                 = true
  username                            = replace(var.name, "-", "_")
  vpc_security_group_ids              = var.security_groups
}
```

We use Bastion Host  
so this part is important that it stays like this

```tf
publicly_accessible                 = false
```

this one is up to you  
it will create snapshot in case you delete accidentally

```tf
skip_final_snapshot                 = true
```

### Terraform organization example
#### Reason for Terraform
To be able to fastly create environments

Terraform is about abstracting what you need for deployment  
and parametrizing that provisioning process  
(with variables)

#### Just convention
All these below are common convesion  
because terraform can get confusing.

But with other filenames,  
these things would still work.

(all could be merged into one main.tf)  
some examples in internet are doing that

#### Environment
this is a place, where modules are created  
and they receive variables

this is a create place,  
where variables are bounded to modules

terraform/module/environment/main.tf

```tf
// ...

module "database" {
  source = "../database"

  security_groups = [module.network.database_security_group]
  subnets         = module.network.database_subnets
  name            = var.name
  vpc_name        = module.network.vpc_name
}

module "service" {
  source = "../service"

  capacity_provider = "spot"
  cluster_id        = module.cluster.cluster_arn
  cluster_name      = var.name
  image_registry    = "${data.aws_caller_identity.this.account_id}.dkr.ecr.${data.aws_region.this.name}.amazonaws.com"
  image_repository  = "fem-fd-service-preview"
  image_tag         = var.name
  listener_arn      = module.cluster.listener_arn
  name              = "service"
  paths             = ["/*"]
  port              = 8080
  vpc_id            = module.network.vpc_id

  config = {
    GOOGLE_REDIRECT_URL = "https://${module.cluster.distribution_domain}/auth/google/callback"
    GOOSE_DRIVER        = "postgres"
  }

  secrets = [
    "GOOGLE_CLIENT_ID",
    "GOOGLE_CLIENT_SECRET",
    "GOOSE_DBSTRING",
    "POSTGRES_URL",
  ]
}
```

#### Define values
This is declaration of exports

terraform/module/network/outputs.tf

```tf
output "database_security_group" {
  value = module.security_group_db.security_group_id
}

output "database_subnets" {
  value = module.vpc.database_subnets
}

// ...

```

and this is place where variable possible values are held

terraform/module/database/main.tf

```
// ...

module "this" {
  source  = "terraform-aws-modules/rds/aws"
  version = "6.12.0"

  allocated_storage                   = 50
  create_db_option_group              = false
  create_db_parameter_group           = false
  create_db_subnet_group              = false
  create_monitoring_role              = false
  db_subnet_group_name                = var.vpc_name
  engine                              = "postgres"
  engine_version                      = "17.2"
  iam_database_authentication_enabled = false
  identifier                          = var.name
  instance_class                      = "db.t4g.micro"
  manage_master_user_password         = false
  max_allocated_storage               = 100
  option_group_name                   = "default:postgres-17"
  parameter_group_name                = "default.postgres17"
  password                            = random_string.password.result
  publicly_accessible                 = false
  skip_final_snapshot                 = true
  username                            = replace(var.name, "-", "_")
  vpc_security_group_ids              = var.security_groups
}
```

### Automation
Automation is beatifull  
because it does a lot for you  
its here to avoid repetitive tasks

to do the same manually it would take a lot  
lot of clicking in AWS interface for you

if you are self hosting, you don't get UI  
like you have in AWS interface  
(never mind the look, it gives a lot of functionality)

#### Roles
you can make Role for Plan  
and a Role for apply

#### Terraform Enterprise
Terraform enterprise is about not running  
apply and plan locally but running on the cloud

#### What Terraform Plan does?
It's a dry run that previews the changes  
terraform will make to your infrastructure.

#### Routing
destination and target
0.0.0.0/0 is from the NAT gateway  
and if you're local, then 

#### Grafana
Amazon is such is a big ecosystem  
Don't stop obervability in those tabs

There are solutions like Grafana  
where you can make customized alerts  
(and you can also use Grafana Cloud)

CloudWatch UI  
you may not like to work with it,  
and in that case,  
Grafana Cloud would be better option

#### Backups
You will probably configure this in terraform module  
that created an instance, but also make sure you have  
automated backups set in the AWS UI (web console)

Perhaps set them to 7 days retention  
There is a lot in terraform to find out

### Cluser module
We are creating cluster (EC2)  
that will host virtual machines  
that will run container cluster (ECS)

#### Cloud In it
you can give it scripts  
that load the moment instance starts to run

this way you can configure instance  
this is purpose of this file

terraform/module/cluster/user_data.tpl

```tf
#cloud-config
write_files:
  - path: /etc/ecs/ecs.config
    permissions: 0400
    owner: root
    content: |
      ECS_CLUSTER=${cluster_name}
      ECS_ENABLE_SPOT_INSTANCE_DRAINING=true
      ECS_ENABLE_TASK_IAM_ROLE=true
      ECS_LOGLEVEL=debug

# vim:ft=yaml sw=2 ts=2 et
```

compared to App Runner, this is huge difference

#### Amazon is still a bit like hobbyst tool
it's a kit to be glued by you  
and terraform is trying  
to reproduce that glueing for you

#### ECS cluster module?
there is no good terraform module for ECS  
so in the workshop definition of it is verbose

#### Instance Profiles
Like Users have Roles  
Instance have Instance Profiles

#### Launch template
initial values that ECS will be using  
for load balancer and scalling

in the end we would have several laucn templates  
depending on number of enrionments we will run on

#### Base64
it's a format that will solve problem of multiline  
and whitespaces, because it will convert to letters  
that are safe to pass trough network

### Scaling
#### criteria for scaling?
you want to scale in response to CPU utilization  
not the number of requests. Because with number of  
resources you risk having a lot of unused resources,  
for which you pay for.

#### Additional tools
We need some of things
that if we only stop on ECS cluster
without additional tools,
then its like we would just have
a network of computing onlyu nodes

ingress
load balancer
and firewall rules
and security groups

terraform/module/cluster/load_balancer.tf

```tf
resource "aws_security_group" "load_balancer" {
  name_prefix = var.name
  vpc_id      = var.vpc_id
}

resource "aws_vpc_security_group_ingress_rule" "http" {
  security_group_id = aws_security_group.load_balancer.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 80
  ip_protocol = "tcp"
  to_port     = 80
}

resource "aws_vpc_security_group_egress_rule" "all" {
  security_group_id = aws_security_group.load_balancer.id

  cidr_ipv4   = "0.0.0.0/0"
  ip_protocol = "-1"
}

resource "aws_lb" "this" {
  enable_deletion_protection = false
  idle_timeout               = 300
  internal                   = true
  load_balancer_type         = "application"
  preserve_host_header       = false
  subnets                    = var.subnets

  security_groups = concat(
    [aws_security_group.load_balancer.id],
    var.security_groups,
  )
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type = "fixed-response"

    fixed_response {
      content_type = "text/plain"
      message_body = "404 Not Found"
      status_code  = "404"
    }
  }
}
```

#### Ingress / Egress
refer to the direction of data or traffic flow.
ingress: data entering a system or network,
egress data leaving a system or network

we define what should happen if the load balancer
cannot resolve resource (respond with 404)

#### CDN for API
network performance of Amazon is unmatched
because of that it makes sense to put
CDN in front of load balancer

#### CDN handles the first layer of delivery,
By placing a CDN in front of a load balancer,
CDN handles the first layer of delivery,
caching content and delivering it to users closer
to the source, and then the load balancer directs
any traffic that cannot be served by the CDN
to the appropriate backend servers.

### CloudFront
CloudFront is a beast
and you can spend a lot of money
if you don't know what you're doing

#### Price class
setting price class is important
we bound load balancer to our cloud infrastructure

```tf
resource "aws_cloudfront_vpc_origin" "this" {
  vpc_origin_endpoint_config {
    arn                    = aws_lb.this.arn
    http_port              = 80
    https_port             = 443
    name                   = "cluster-${var.name}"
    origin_protocol_policy = "http-only"

    origin_ssl_protocols {
      items    = ["TLSv1.2"]
      quantity = 1
    }
  }
}

resource "aws_cloudfront_distribution" "this" {
  enabled     = true
  price_class = "PriceClass_100"

  origin {
    domain_name = aws_lb.this.dns_name
    origin_id   = "cluster-${var.name}"

    vpc_origin_config {
      vpc_origin_id = aws_cloudfront_vpc_origin.this.id
    }
  }

  default_cache_behavior {
    allowed_methods        = ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "cluster-${var.name}"
    viewer_protocol_policy = "redirect-to-https"

    forwarded_values {
      query_string = true

      cookies {
        forward = "none"
      }
    }
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }
}
```

#### GET HEAD
we only cache GET and HEAD
because all the others are modifying the data

#### Block countries?
If you know for sure, that your product
is not to be used from other countries,
then it makes sense to block all the other traffic
because this will prevent DDos and many other attacts.

You will frustrate some people,
but value gained may be worth it,
if this can save you.

#### container organization
We may have further granulality
of how we organize our containers in cluster

Backend Tier (does not have access to net)
Gateway Tier (has access to net)
+
Frontend Tier

#### It's compiling time
Some parts of `terraform apply` will take 20 minutes
this is equivalent of "its compiling time"

#### Terraform vs Pullumi
If you want to use programming language
you use Pullumi

There is a lot of similarity with Terraform.

#### OpenTofu, sell Terraform as Product
It seems a lot of people like to say they use OpenTofu
for no good reason. Hashicorp did something not nice
but it only affected companies that tried to sell
terraform as a product.

Its probable that Terraform will close OpenTofu

#### IBM bought Terraform
they asked who is you competitor?
they answered few names that IBM would not like
and they said "we are shutting it"

### Service Automation
This is part where



AWS For Front-End Engineers
===========================
https://frontendmasters.com/courses/aws-v2/  
https://github.com/stevekinney/aws-v2

docs after registering:  
https://aws.amazon.com/registration-confirmation/

### Free Tier
S3: 5GB of storage.  
Cloudfront: 1TB of data transfer.  
Lambda: 1 million free requests per month.  
Cognito: 50,000 monthly active users.  
AppSync: 250,000 GraphQL data modifications per month.  
Amazon Location Service: 3 months of location data.

IAM user / root user  
iam - administrative  
principle of least priviledged user

### IAM: manage users
root  
- setup it as secure as you can
- hope to not ever use it again  
administrative users  
- the ones to use daily  
Administrator  
- access all but billing (this is left for root)

- [ ] check how it works: IAM > user create > Identity center  
  before it was "create IAM user"

create new user:  
- add to group
- copy permissions from another user
- attach policies directly  
AdministratorAccess

AdministratorAccess Policy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```

Access key ID - like username  
Secret access key - like password

in decent company size use groups

### S3 Simple Storage Service
bucket with files  
infinitely scalable  
file 0B max 5TB  
- lifecycle management
- versioning
- encryption
- security

S3 is really a key-value store

adding to S3 is immediate  
updating and removing can take a moment  
  to have high durability files are duplicated  
  ... we can update object and query and get old one (for a moment)

prices  
free: put to S3  
paid: storage (above 5G) and requests  
tiers:  
normal stuff  
infrequent stuff: cheaper storage, more expensive requests  
intelligent tearing

bucket names need to be unique  
glacier/freezer

some of edge S3 functionality is only available in us-east-1

ACL: Access Controll List
```bash
aws s3 ls
aws s3 ls s3://maciejka.click

aws s3 help
aws s3 cp ./index.html s3://maciejka.click/index.html
aws s3 cp ./build s3://maciejka.click --recursive
aws s3 cp /tmp/foo/ s3://bucket/ --recursive --exclude "ba*"
aws s3 cp /tmp/foo/ s3://bucket/ --recursive --exclude "ba*"
```
bucket can redirect to another bucket

s3 static hosted redirect
```
[
    {
        "Condition": {
            "KeyPrefixEquals": "*"
        },
        "Redirect": {
            "ReplaceKeyPrefixWith": "index.html"
        }
    }
]
```

s3 static hosting:  
  redirect requests for an object

### Policies
https://awspolicygen.s3.amazonaws.com/policygen.html

S3 Policy
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Statement1",
      "Principal": {},
      "Effect": "Allow",
      "Resource": [],
      "Action": []
    }
  ]
```

use S3 as hosting  
Action: s3:GetObject  
Resource: arn:aws:s3:::maciejka.click/*  
keyName: optional, restrict to only one S3 key

### Route 53
can detect that you host in multiple places  
and when health check is down reroute

Records Type  
- [ ] NS
- [ ] SOA  
A - connect to resource (can be aliased to s3 hosted site)

### Certificates
Certificates > Request certificate  
(requires domain name)  
add two:  
maciejka.click  
www.maciejka.click  
remember to hit "Create records in Route 53"  
  (even though its showing "pending validation")

To be used in Cloudfront...  
Certificate has to be in us-east-1  
(that's where technically Cloudfront "lives")

### CLI
handle multiple accounts:  
.aws/credentials  
`aws s3 ls --profile dev ...`  
multiple profiles:  
https://www.linkedin.com/pulse/aws-cli-multiple-profiles-bachar-hakam/

- [ ] can User have a Policy?
- [ ] can Resource have a Policy?

### Cloudfront
Puts content onto web of CDNs  
Cloud9: IDE  
CloudWatch: logs

point domain to s3 internal url (when static hosting)

Cloudfront and S3 has some gotchas, if you click this and that you will never have it working

Origin Shield: cache origin to have less requests  
WAF Web Application Firewall:

redirect HTTP to HTTPS

point Default root object to "index.html" (without slash)

CDN latency test  
https://www.cdnperf.com/tools/cdn-latency-benchmark/

by default cloud front ignores request headers  
it can be configured to let some custom headers pass trough  
with some added custom headers:  
Cloudfront-Is-Desktop-Viewer  
Cloudfront-Is-Mobile-Viewer  
Cloudfront-Is-SmartTV-Viewer  
Cloudfront-Is-Tablet-Viewer  
Cloudfront-Viewer-Country

distribution domain name is actually a working link  
https://d2d5ffddufrqe4.cloudfront.net/

cloudfront can wrap anything  
s3 bucket,  
api endpoints  
  hosted on ec2  
  - [ ] check more about "aws api ec2"  
  hosted on gateways  
  - [ ] check more about "aws api gateways"

status: enabled  
find emotional patience  
if status: deploying don't adjust it

a lot of apps have error response setting:  
to treat all 404 as custom response  
  with page path /index.html and response code 200

Caching:  
CloudFront / Distributions / behaviors  
default settings:  
min: 1s, max 1y?  
default: 24h

on front to deal with this:  
make index.html short TTL, like 5m  
rest is fingerprinted (and changes on each reload)

or create new behavior (there can be many)  
path /index.html  
redirect HTTP to HTTPS  
set different policy just for that file

if fast reaction is needed use Invalidations  
way to say "wipe it from memory"  
create  
add paths (1000 for free, after that 0.005 usd per invalidation)  
u can use wildcards  
/*

if cost of invalidation is a problem in CI/CD  
... then experiment with TTL

aws cloudfront list-distributions

cloudfront functions:  
can do way less than lambda  
but for a way cheaper and faster

### Lambda@Edge
functions on cloudfront nodes: Lambda@Edge  
intercept request and do something  
(like: add a cookie to header)  
(a/b testing: randomly add a header)  
can call database

four places to place:  
every request:  
cache request, cache response  
cache miss:  
origin request, origin response

lambda functions allow to execute code in response to events

if script needs access to other resources:  
check Enable Network

if you don't see specyfic add trigger  
it may be because you're on wrong region  
also deploy lambda on edge may be missing  
(when in doubt, use us-east-1 for cloudfront stuff)

test lambda:  
event template / filter: request  
deploy: to lambda (to cloudfront is a separate deploy)  
... this will create a new role

deploy to edge@lambda  
error:  
your function's execution role must be assumable by the adgelambda.amazonaws.com service principal  
... need to edit policy  
go to iam > find role > trust relationships > edit
```
"Principal": {
    "Service": ["lambda.amazonaws.com", "edgelambda.amazonaws.com"]
},
```

add trigger:  
CloudFront  
warning about being distributed to several places  
select cloudfront event

function associations  
cloudfont > distributions > select > behaviours > edit  
to delete using lambda: set "no association"

lambda to fix security headers:
```javascript
'use strict';

export const handler = async (event, context) => {
  const response = event.Records[0].cf.response;
  const headers = response.headers;

  headers['strict-transport-security'] = [
    {
      key: 'Strict-Transport-Security',
      value: 'max-age=63072000; includeSubdomains; preload',
    },
  ];

  headers['content-security-policy'] = [
    {
      key: 'Content-Security-Policy',
      value:
        "default-src 'none'; img-src 'self'; script-src 'self'; style-src 'self'; object-src 'none'",
    },
  ];

  headers['x-content-type-options'] = [
    { key: 'X-Content-Type-Options', value: 'nosniff' },
  ];

  headers['x-frame-options'] = [{ key: 'X-Frame-Options', value: 'DENY' }];

  headers['x-xss-protection'] = [
    { key: 'X-XSS-Protection', value: '1; mode=block' },
  ];

  headers['referrer-policy'] = [
    { key: 'Referrer-Policy', value: 'same-origin' },
  ];

  headers['server'] = [{ key: 'Server', value: 'Erlang on Eels' }];

  return response;
};
```

use as a viewer response  
Erlang on Eels - made up, to avoid telling which techs are used to make a site

### CloudFront functions
can do less but fast and cheap  
limited subset of JavaScript  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/functions-javascript-runtime-features.html#writing-functions-javascript-features-builtin-modules
```javascript
function handler(event) {
    var request = event.request;
    if (request.uri === '/beatles.jpg') {
        request.uri = '/not-beatles.jpg';
    }
    return request;
}
```
then go to distribution > behaviours > edit > set as viewer request

### Origin Access Identity OAI
says that only cloudfront is allowed to selected s3  
allows to make s3 private and locked  
  watch out: changes in OAI take a moment (1 min)  
  and things can not work for that moment

origin access identity  
OAI is almost an IAM but not really  
check "update policy"  
if it fails:
```
{
  "Version": "2012-10-17",
  "Id": "Policy1639917562467",
  "Statement": [
    {
      "Sid": "Stmt1639917558597",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity EP6AJZA5FKG3P"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::superawesome.xyz/*"
    }
  ]
}
```

- [ ] check new "Origin access: Origin access control settings"

watch out to not confuse OAI user id with cloud front distribution id

after saving you will probably see access denied  
check with index.html on end, like https://maciejka.click/index.html

using it may feel like pushing a problem only a little further

### Pipeline
create a user with least permissions possible to deploy  
(other options: self host jenkins, have fun)

create unique policy for CI/CD process  
- [ ] cloud front: create invalidation
- [ ] s3: PutObject, ListBucket  
policy name: DeployStaticAssets

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:ListBucket",
                "cloudfront:CreateInvalidation"
            ],
            "Resource": [
                "arn:aws:cloudfront::730335435837:distribution/EEF0O4KEHX0TC",
                "arn:aws:s3:::maciejka.click/*",
                "arn:aws:s3:::maciejka.click",
            ]
        }
    ]
}
```

cannot find it?  
filter by Customer managed

create user  
BuildProcess  
should only have API key,  
not be able to sign in to console

create access key, programmatic access

### CI/CD Github actions
go to github > actions > setup a workflow yourself  
they live in .github directory  
and at least one has to be named main.yml

^space: show autocomplete

use template:  
Static HTML

```gitactions
# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false


# (working) example based on Static HTML template:

# (not working?) example from course:
name: Deploy Website

on:
  push:
    branches: [main]

  # allows to manualy trigger from actions tab
  workflow_dispatch:

jobs:
  # name of job
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: eu-central-1
      # like install but for optimized for ci
      - run: npm ci
      - run: npm run build
      - name: Deploy to S3
        run: aws s3 sync ./build s3://${{ secrets.BUCKET_NAME }}
      - name: Create Cloudfront Invalidation
        run: aws cloudfront create-invalidation --distribution-id $${{ secrets.DISTRIBUTION_ID }} --paths "/*"
```
set secret variables:  
settings > secrets > actions

better to name steps

### Idea
Point several cloudfronts to one bucket  
call one of them staging

### Future explore
Cognito  
  authentication, authorization, login with facebook  
  use with s3 for interesting stuff:  
    user uploaded photo, give back a signed url which is valid only for 24h  
    and only that user can see it

DynamoDB  
  noSQL database  
  extremely scalable  
  key-value store  
  you work with cursors  
  you can call it in edge@lambda

AppSync  
  spinup graphql server  
  write schema for data  
  then hit button to create dynamodb to store  
  and create subscription to be used in websockets  
  very good for real-time apps  
  and rapid prototyping

Amazon Location Service  
  add geolocation in service

Amplify  
  set of UI components for React  
  cli to spin Cognito, S3, AppSync  
  and code pipeline (codecommit)  
  a bit like firebase

### General
JAM stack: JavaScript, API i Markup  
ARN: Amazon Resource Name  
Brotli: loseless compression method, by Google  
Dynamo, Aurora: databases with distributed option, and limitations  
HTTP/2: has built in streaming (we will send you everything in a open connection)  
Mozilla Observatory: check security of web service  
- [ ] image resize: is there a service for that?



Lambdas
=============================
**do aws lambda start a container each time?**  
No, AWS Lambda does not start a new container each time a function is invoked.  
Instead, it reuses containers for multiple invocations to optimize performance.  
This reuse process is known as "container reuse" or "warm start."  
If a container is already running from a previous invocation, Lambda may reuse it,  
resulting in faster response times.

However, if no existing container is available, or after a period of inactivity,  
Lambda might start a new container, leading to a "cold start,"  
which is generally slower.

**what are good use cases for aws lambdas?**  
generally good as programmable hooks inside AWS world

S3 events: process files as they are uploaded to S3  
microservices, handle http requests  
scheduled tasks  
automated infrastructure tasks  
IoT Data processing

**what can be done using aws lambdas but there is a better way?**  
long running tasks (lambda has 15 minute limit), use EC2  
heavy tasks (lambda ahs CPU/GPU limitations), use EC2  
high volume streams
