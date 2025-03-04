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
