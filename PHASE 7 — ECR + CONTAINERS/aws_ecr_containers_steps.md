# PHASE 7 — ECR + CONTAINERS
## Docker → ECR → EC2 → ECS → Fargate → ALB

> Goal: understand containers from the ground up and then deploy the same application first on EC2 and later on Amazon ECS with AWS Fargate.
>
> Main progression:
>
> ```text
> Dockerfile
>     ↓
> Docker Image
>     ↓
> Local Container
>     ↓
> Amazon ECR
>     ↓
> EC2
> ```
>
> Then:
>
> ```text
> Developer
>     ↓
> Docker Build
>     ↓
> ECR
>     ↓
> ECS
>     ↓
> Fargate
>     ↓
> ALB
>     ↓
> Internet
> ```
>
> The important mental model is:
>
> **Docker packages the application. ECR stores the image. ECS schedules/runs containers. Fargate provides the compute capacity without requiring you to manage EC2 container hosts. ALB exposes the application.**

---

# 1. What This Phase Is About

In previous phases you learned:

```text
EC2
RDS
VPC
Security Groups
ALB
IAM
S3
```

Now you are going to connect those concepts with containers.

Traditional deployment:

```text
Developer
    ↓
Application code
    ↓
EC2
    ↓
Install runtime
    ↓
Install dependencies
    ↓
Run application
```

Container deployment:

```text
Developer
    ↓
Dockerfile
    ↓
Docker image
    ↓
ECR
    ↓
ECS/Fargate
    ↓
Container
```

The container image becomes the deployable application artifact.

---

# 2. Why Containers Exist

Imagine your backend requires:

```text
Python 3.12
PostgreSQL client
specific OS libraries
specific Python packages
environment configuration
```

Developer machine:

```text
Works
```

Production:

```text
Different OS
Different Python version
Different dependencies
```

Result:

```text
"It works on my machine."
```

Containers reduce this environment mismatch by packaging the application and its runtime dependencies into an image.

Conceptually:

```text
Application
+
Runtime
+
Dependencies
+
Filesystem
+
Startup command
        ↓
     IMAGE
```

---

# 3. Docker

Docker is a platform/tooling ecosystem for building and running containers.

You should understand:

```text
Dockerfile
Image
Container
Registry
Volume
Network
Port mapping
Environment variables
```

For this phase, the most important are:

```text
Dockerfile
Image
Container
Registry
```

---

# 4. Image vs Container

This distinction is fundamental.

## Image

An image is a packaged, immutable artifact used to create containers.

Think:

```text
Image
=
Template
```

Example:

```text
my-api:1.0
```

## Container

A container is a running instance of an image.

Think:

```text
Image
   ↓
Container
```

One image can produce multiple containers:

```text
my-api:1.0
      │
      ├── container-1
      ├── container-2
      └── container-3
```

---

# 5. Image Mental Model

Suppose your application is:

```text
Node.js API
```

Docker image might contain:

```text
Linux base image
Node.js runtime
package.json
npm dependencies
application source
startup command
```

Then:

```text
Image
  ↓
Container
  ↓
Node.js API
```

---

# 6. Dockerfile

A Dockerfile is a text file containing instructions for building an image.

Example:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

Read it line by line.

---

# 7. FROM

```dockerfile
FROM node:22-alpine
```

This selects the base image.

Think:

```text
Base image
    ↓
Your image
```

The base image can provide:

```text
Operating system userspace
Runtime
Libraries
Tools
```

Use official/trusted base images and keep them updated.

---

# 8. WORKDIR

```dockerfile
WORKDIR /app
```

This sets the working directory inside the image/container.

Instead of:

```text
/
```

your application works from:

```text
/app
```

---

# 9. COPY

```dockerfile
COPY package*.json ./
```

Copies files from the build context into the image.

Then:

```dockerfile
COPY . .
```

copies the application source.

Be careful with:

```dockerfile
COPY . .
```

because without a proper `.dockerignore`, you might accidentally copy:

```text
.git
node_modules
.env
logs
temporary files
secrets
```

---

# 10. .dockerignore

Create:

```text
.dockerignore
```

Example:

```text
node_modules
.git
.env
.env.*
npm-debug.log
coverage
dist
```

This reduces:

```text
build context
image size
accidental secret exposure
```

---

# 11. RUN

Example:

```dockerfile
RUN npm ci --omit=dev
```

This executes a command during image build.

Conceptually:

```text
docker build
      ↓
Dockerfile
      ↓
RUN commands
      ↓
Image
```

---

# 12. EXPOSE

```dockerfile
EXPOSE 3000
```

This documents the container port.

Important:

```text
EXPOSE
```

does not automatically publish the port to the internet.

Actual networking depends on how the container is run.

---

# 13. CMD

```dockerfile
CMD ["node", "server.js"]
```

This specifies the default command used when the container starts.

Think:

```text
Container starts
       ↓
CMD
       ↓
Application starts
```

---

# 14. ENTRYPOINT vs CMD

You should understand the distinction.

```dockerfile
ENTRYPOINT
```

defines the executable behavior.

```dockerfile
CMD
```

provides default arguments/default command behavior.

For simple application containers:

```dockerfile
CMD ["node", "server.js"]
```

is often sufficient.

---

# 15. Build an Image

Suppose directory:

```text
my-api/
├── Dockerfile
├── .dockerignore
├── package.json
├── package-lock.json
└── server.js
```

Build:

```bash
docker build -t my-api:1.0 .
```

Meaning:

```text
docker build
    ↓
use Dockerfile
    ↓
build image
    ↓
tag image as my-api:1.0
```

---

# 16. List Images

```bash
docker images
```

You may see:

```text
REPOSITORY   TAG   IMAGE ID
my-api       1.0   abc123...
```

---

# 17. Run a Container Locally

Suppose application listens on:

```text
3000
```

Run:

```bash
docker run -p 3000:3000 my-api:1.0
```

Meaning:

```text
Host port 3000
      ↓
Container port 3000
```

Then:

```text
Browser
  ↓
localhost:3000
  ↓
Docker container
  ↓
Application
```

---

# 18. Port Mapping

This:

```bash
-p 3000:3000
```

means:

```text
HOST:CONTAINER
```

Example:

```bash
-p 8080:3000
```

means:

```text
localhost:8080
      ↓
container:3000
```

---

# 19. Test the Container

```bash
curl http://localhost:3000
```

or open:

```text
http://localhost:3000
```

If it works:

```text
Docker image
+
container
+
application
```

are functioning locally.

---

# 20. Inspect Containers

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

View logs:

```bash
docker logs <container-id>
```

Open a shell:

```bash
docker exec -it <container-id> sh
```

Stop:

```bash
docker stop <container-id>
```

Remove:

```bash
docker rm <container-id>
```

---

# 21. Container Lifecycle

Think:

```text
Image
  ↓
docker run
  ↓
Created
  ↓
Running
  ↓
Stopped
  ↓
Removed
```

The image remains:

```text
Image
```

while containers are created from it.

---

# 22. Image Tags

Example:

```text
my-api:1.0
my-api:1.1
my-api:2.0
```

Tag format:

```text
repository:tag
```

Examples:

```text
my-api:dev
my-api:staging
my-api:prod
my-api:2026-09-26
my-api:git-a1b2c3d
```

---

# 23. Do Not Rely Blindly on latest

You will often see:

```text
my-api:latest
```

This is convenient but can be ambiguous.

Production deployments benefit from immutable versioning such as:

```text
my-api:git-a1b2c3d
```

or a release identifier.

Even better, understand the image digest:

```text
sha256:...
```

A tag can move; a digest identifies a specific image manifest/content reference.

---

# 24. Image Digest

Conceptually:

```text
Tag:
my-api:1.0

Digest:
sha256:abc123...
```

A digest gives you content-addressed identification.

Deployment systems can use:

```text
repository@sha256:...
```

when you need exact image identity.

---

# 25. Docker Layers

Docker images are built in layers.

Example:

```text
FROM node
    ↓
COPY package.json
    ↓
RUN npm ci
    ↓
COPY source
```

Each instruction may contribute a layer.

This matters for:

```text
build speed
cache
image size
distribution
```

---

# 26. Docker Build Cache

Suppose:

```dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
```

If only:

```text
server.js
```

changes, Docker may reuse the dependency layers.

This is why Dockerfile instruction ordering matters.

A common pattern is:

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

instead of copying everything first.

---

# 27. Multi-Stage Builds

For production applications, multi-stage builds can reduce the final image.

Example:

```dockerfile
FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build


FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY --from=builder /app/dist ./dist

CMD ["node", "dist/server.js"]
```

Concept:

```text
Builder image
     ↓
Build application
     ↓
Final runtime image
     ↓
Only required artifacts
```

---

# 28. Container Security Basics

Do not put secrets inside images.

Bad:

```dockerfile
ENV DB_PASSWORD=my-secret-password
```

Bad:

```text
COPY .env .
```

Bad:

```text
COPY credentials.json .
```

Instead:

```text
Secrets Manager
Parameter Store
ECS secrets integration
runtime environment configuration
```

The image should be safe to distribute to the intended registry.

---

# 29. Run as Non-Root

Avoid unnecessarily running applications as root inside containers.

Example:

```dockerfile
USER node
```

when the selected image/user model supports it appropriately.

Principle:

```text
Least privilege
```

applies inside containers too.

---

# 30. Health Checks

A container can be:

```text
process running
```

but:

```text
application broken
```

Example:

```text
Node process = alive
Database connection = broken
```

A health check can test:

```text
GET /health
```

Expected:

```text
HTTP 200
```

This becomes especially important with:

```text
ECS
ALB
Auto Scaling
```

---

# 31. What Is ECR?

Amazon Elastic Container Registry (ECR) is AWS's managed container registry.

Think:

```text
Docker Hub
        ↓
Container image registry

Amazon ECR
        ↓
AWS-managed container image registry
```

ECR private repositories can store:

```text
Docker images
OCI images
OCI-compatible artifacts
```

AWS documents ECR as a managed registry with IAM-based access and support for private repositories. citeturn0search12turn0search10

---

# 32. ECR Mental Model

```text
AWS Account
    │
    ▼
ECR Registry
    │
    ├── backend-api
    │     ├── 1.0
    │     ├── 1.1
    │     └── prod-a1b2c3
    │
    └── worker
          ├── 1.0
          └── prod-d4e5f6
```

---

# 33. ECR Repository

Create:

```text
company-backend
```

This repository contains versions of your application image.

Example:

```text
company-backend:dev
company-backend:staging
company-backend:prod
```

For stronger release discipline:

```text
company-backend:git-a1b2c3d
company-backend:git-d4e5f6a
```

---

# 34. ECR Registry vs Repository

Do not confuse them.

## Registry

Account/Region-level container registry endpoint.

Example:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

## Repository

A named collection inside the registry.

Example:

```text
company-backend
```

Full image:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/company-backend:1.0
```

---

# 35. ECR Push Flow

The complete developer flow:

```text
Developer
   ↓
docker build
   ↓
Local image
   ↓
docker tag
   ↓
ECR registry
   ↓
docker push
```

AWS documents this exact authentication/tag/push workflow for private ECR repositories. citeturn0search1turn0search13

---

# 36. Create ECR Repository

Console:

```text
AWS Console
    ↓
Amazon ECR
    ↓
Private registry
    ↓
Repositories
    ↓
Create repository
```

Example:

```text
Repository name:
company-backend
```

Use a private repository for your company application.

---

# 37. ECR Image Immutability

A useful production control is:

```text
Tag immutability
```

Concept:

```text
company-backend:1.0
```

should identify one release rather than being overwritten repeatedly.

This helps prevent:

```text
same tag
different image
unexpected deployment
```

ECR supports preventing image tags from being overwritten. citeturn0search10

---

# 38. ECR Lifecycle Policies

Images accumulate:

```text
build-1
build-2
build-3
...
build-1000
```

Storage grows.

ECR lifecycle policies can automatically clean up images according to rules.

Example concept:

```text
Keep last 20 development images
Delete older unreferenced images
```

AWS documents ECR lifecycle policies as a way to manage unused images. citeturn0search12

---

# 39. Image Scanning

Container images can contain vulnerabilities.

Example:

```text
OS package
    ↓
CVE
    ↓
Image vulnerability
```

ECR provides:

```text
Basic scanning
Enhanced scanning
```

Enhanced scanning integrates with Amazon Inspector and can provide continuous scanning for OS and programming-language package vulnerabilities. Basic scanning focuses on OS vulnerabilities. citeturn0search2turn0search7

---

# 40. Basic vs Enhanced Scanning

Conceptually:

```text
Basic
 ↓
OS vulnerability scanning
```

Enhanced:

```text
Amazon Inspector
 ↓
OS packages
+
Programming language packages
+
Continuous scanning
```

For company workloads, understand what scanning mode is configured and how findings are handled.

A scan is not a guarantee that an image is safe.

It is one security control.

---

# 41. ECR Security Model

Developers need permission to:

```text
authenticate
push
pull
describe
list
```

Runtime systems need permission to:

```text
pull
```

This is where your Phase 1 IAM knowledge becomes important.

---

# 42. Developer IAM Permissions

A developer may need:

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:CompleteLayerUpload
ecr:InitiateLayerUpload
ecr:PutImage
ecr:UploadLayerPart
```

Do not automatically give developers:

```text
AdministratorAccess
```

Create least-privilege permissions appropriate to the workflow.

---

# 43. EC2 Pulling From ECR

First target architecture:

```text
Developer
    ↓
Docker build
    ↓
ECR
    ↓
EC2
    ↓
Container
    ↓
Internet
```

The EC2 instance needs permission to pull from ECR.

---

# 44. EC2 IAM Role

Attach an IAM role to the EC2 instance.

Conceptually:

```text
EC2
 │
 └── Instance Profile
        │
        └── IAM Role
              │
              └── ECR pull permissions
```

This is better than putting:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

inside the EC2 server.

---

# 45. EC2 ECR Pull Permissions

The EC2 role needs ECR permissions required to pull images.

AWS documents permissions such as:

```text
ecr:GetAuthorizationToken
ecr:BatchGetImage
ecr:GetDownloadUrlForLayer
```

for pulling from private ECR repositories. citeturn0search0

Use the appropriate AWS-managed policy or a narrower custom policy depending on your company IAM model.

---

# 46. ECR → EC2 Lab

Build:

```text
Developer
   ↓
Docker
   ↓
ECR
   ↓
EC2
   ↓
Docker
   ↓
Container
```

The EC2 server becomes the container runtime.

---

# 47. Step 1 — Build Local Application

Create:

```text
hello-api/
├── Dockerfile
├── .dockerignore
├── package.json
├── package-lock.json
└── server.js
```

Example Node server:

```javascript
const http = require("http");

const server = http.createServer((req, res) => {
  res.writeHead(200, {"Content-Type": "text/plain"});
  res.end("Hello from Docker on AWS");
});

server.listen(3000, "0.0.0.0", () => {
  console.log("Server listening on 3000");
});
```

Important:

```text
0.0.0.0
```

inside the container.

Do not bind only to:

```text
127.0.0.1
```

if you need traffic from outside the container.

---

# 48. Step 2 — Dockerfile

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

---

# 49. Step 3 — Build

```bash
docker build -t hello-api:1.0 .
```

Check:

```bash
docker images
```

---

# 50. Step 4 — Run Locally

```bash
docker run --rm -p 3000:3000 hello-api:1.0
```

Test:

```bash
curl http://localhost:3000
```

Expected:

```text
Hello from Docker on AWS
```

---

# 51. Step 5 — Create ECR Repository

Create:

```text
hello-api
```

Record:

```text
repositoryUri
```

Example:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/hello-api
```

---

# 52. Step 6 — Authenticate Docker to ECR

AWS's documented approach:

```bash
aws ecr get-login-password \
  --region ap-south-1 \
| docker login \
  --username AWS \
  --password-stdin \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

The ECR authorization token is valid for 12 hours. citeturn0search1

---

# 53. Step 7 — Tag Image

```bash
docker tag \
  hello-api:1.0 \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/hello-api:1.0
```

Now:

```text
Local
hello-api:1.0

ECR
123456789012.dkr.ecr.ap-south-1.amazonaws.com/hello-api:1.0
```

---

# 54. Step 8 — Push

```bash
docker push \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/hello-api:1.0
```

Then inspect:

```text
ECR
 ↓
Repositories
 ↓
hello-api
```

You should see:

```text
1.0
```

---

# 55. Step 9 — Prepare EC2

EC2 should have:

```text
Docker
AWS CLI
IAM role
network access to ECR
```

If using an AWS-provided Amazon Linux image, install/configure Docker according to the current distribution instructions.

---

# 56. Step 10 — Give EC2 ECR Pull Permissions

Attach an IAM role allowing ECR image pull.

The important idea is:

```text
EC2
  ↓
IAM Role
  ↓
ECR
  ↓
Pull image
```

Do not create a permanent AWS access key just to make the EC2 instance pull an image.

---

# 57. Step 11 — Login From EC2

```bash
aws ecr get-login-password \
  --region ap-south-1 \
| docker login \
  --username AWS \
  --password-stdin \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

---

# 58. Step 12 — Pull

```bash
docker pull \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/hello-api:1.0
```

Then:

```bash
docker images
```

---

# 59. Step 13 — Run

```bash
docker run -d \
  --name hello-api \
  -p 3000:3000 \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/hello-api:1.0
```

Test locally on EC2:

```bash
curl http://localhost:3000
```

---

# 60. Step 14 — Expose Through Security Group

If you want browser access directly to EC2:

```text
EC2 SG
Inbound:
TCP 3000
Source = your IP
```

For a production architecture, later use:

```text
ALB
 ↓
EC2
```

and keep application instances private where appropriate.

---

# 61. Better EC2 Architecture

Instead of:

```text
Internet
 ↓
EC2:3000
```

use:

```text
Internet
   ↓
ALB
   ↓
EC2
   ↓
Container
```

Security groups:

```text
ALB SG
  ↓
EC2 SG
```

EC2 should generally accept application traffic from the ALB SG rather than the entire internet.

---

# 62. ECR → EC2 Architecture

```text
Developer
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker push
    ▼
┌─────────────────┐
│      ECR        │
│  hello-api:1.0  │
└────────┬────────┘
         │
         │ docker pull
         ▼
┌─────────────────┐
│       EC2       │
│   Docker Host   │
└────────┬────────┘
         │
         ▼
     Container
         │
         ▼
        ALB
         │
         ▼
      Internet
```

---

# 63. What ECS Solves

Running Docker manually on EC2 works, but as the number of containers increases, operations become harder.

Suppose:

```text
EC2
 ├── container A
 ├── container B
 ├── container C
 ├── container D
```

You now need to manage:

```text
container placement
restarts
desired count
health
scaling
deployment
service discovery
load balancing
```

ECS provides orchestration.

---

# 64. Amazon ECS

Amazon Elastic Container Service is AWS's container orchestration service.

Core concepts:

```text
Cluster
Task Definition
Task
Service
Container
Capacity
```

Learn these deeply.

---

# 65. ECS Cluster

A cluster is a logical grouping for ECS tasks/services.

Conceptually:

```text
ECS Cluster
   │
   ├── Service A
   │     ├── Task
   │     └── Task
   │
   └── Service B
         ├── Task
         └── Task
```

A cluster is not itself the application.

It provides the logical environment in which ECS workloads run.

---

# 66. ECS Task Definition

This is one of the most important ECS concepts.

A task definition is the blueprint for running containers.

It describes things such as:

```text
container image
CPU
memory
ports
environment
logging
secrets
IAM roles
health checks
volumes
network mode
```

AWS describes a task definition as a blueprint that specifies the container image, resources, ports, and other task configuration. citeturn0search5turn0search4

---

# 67. Task

A task is a running instance of a task definition.

Think:

```text
Task Definition
      ↓
Task
```

Example:

```text
hello-api-task-definition:7
```

can produce:

```text
Task 1
Task 2
Task 3
```

---

# 68. Service

An ECS service maintains a desired number of tasks.

Example:

```text
Desired count = 2
```

ECS attempts to keep:

```text
Task 1
Task 2
```

running.

If one stops:

```text
Task 1
   X
```

the service scheduler can launch another task.

AWS documents ECS services as maintaining the desired number of tasks and replacing stopped/failed tasks. citeturn0search5

---

# 69. Task vs Service

Important:

## Task

```text
One running workload
```

## Service

```text
Keeps desired number of tasks running
```

Example:

```text
Service
desiredCount = 3

       ↓

Task 1
Task 2
Task 3
```

---

# 70. ECS Launch Options

Historically ECS can run containers on:

```text
EC2
Fargate
```

Your learning path:

```text
First:
ECR → EC2

Then:
ECR → ECS Fargate
```

This is a good progression because you first understand the container runtime, then let AWS manage the compute capacity.

---

# 71. Fargate

AWS Fargate is serverless compute for containers.

You do not manage:

```text
EC2 hosts
Docker daemon on hosts
host patching
container host capacity
```

Instead you specify:

```text
CPU
Memory
Network
Task definition
```

and ECS/Fargate runs the tasks.

---

# 72. EC2 vs Fargate

## ECS on EC2

You manage:

```text
EC2 instances
OS
capacity
container hosts
patching
```

ECS manages:

```text
container scheduling
service management
```

## ECS on Fargate

AWS manages:

```text
underlying compute infrastructure
```

You manage:

```text
task definition
container
networking
IAM
application
service
```

---

# 73. Fargate Mental Model

```text
You
 │
 ├── Docker image
 ├── Task definition
 ├── CPU
 ├── Memory
 ├── Network
 └── IAM
       ↓
     ECS
       ↓
   Fargate
       ↓
Container
```

---

# 74. ECS Fargate Architecture

Target:

```text
                    Internet
                       │
                       ▼
                      ALB
                       │
                ALB Security Group
                       │
                       ▼
              Fargate Tasks
              Private Subnets
                       │
             ┌─────────┴─────────┐
             │                   │
            Task               Task
             │                   │
             └─────────┬─────────┘
                       │
                      ECR
                       │
                Container Image
```

ECR is the image source.

Fargate is the compute capacity.

ECS is the orchestration layer.

ALB is the traffic entry point.

---

# 75. ECS + ECR IAM Roles

This is another place where IAM matters.

There are two important concepts:

```text
Task execution role
Task role
```

Do not confuse them.

---

# 76. Task Execution Role

The task execution role is used by the ECS/Fargate infrastructure for operations such as:

```text
pulling ECR images
sending logs to CloudWatch
retrieving certain secrets/configuration
```

AWS documents the Fargate task execution role as the role providing required permissions for pulling private ECR images. citeturn0search0

---

# 77. Task Role

The task role is the IAM identity available to the application container.

Example:

```text
Application
   ↓
AWS SDK
   ↓
Task Role
   ↓
S3
```

Suppose your backend needs:

```text
S3 GetObject
```

Do not give the entire ECS service:

```text
AdministratorAccess
```

Give the application task role only what the application requires.

---

# 78. Execution Role vs Task Role

Memorize:

```text
Execution Role
    ↓
ECS/Fargate infrastructure needs permissions

Task Role
    ↓
Application container needs AWS permissions
```

Example:

```text
ECR pull
CloudWatch logs
        ↓
Execution Role

S3
DynamoDB
SQS
Secrets Manager
        ↓
Task Role
```

The exact permissions depend on the workload.

---

# 79. ECS Network Mode

Fargate tasks use:

```text
awsvpc
```

network mode.

This means each task gets its own network interface/IP-level networking in the VPC.

Conceptually:

```text
VPC
 │
 ├── Task ENI
 │
 ├── Task ENI
 │
 └── Task ENI
```

This makes Security Group design very important.

---

# 80. Fargate Task Security Group

Example:

```text
Fargate SG
```

Inbound:

```text
TCP 3000
Source:
ALB SG
```

Not:

```text
TCP 3000
0.0.0.0/0
```

The ALB is the public entry point.

---

# 81. ALB → Fargate

Architecture:

```text
Internet
   ↓
ALB
   ↓
Target Group
   ↓
ECS Service
   ↓
Fargate Task
   ↓
Container port 3000
```

Security:

```text
ALB SG
   ↓
Fargate SG
```

---

# 82. ECS Target Group

The ALB needs a target group.

For Fargate:

```text
Target type:
IP
```

because tasks have their own IP addresses under `awsvpc`.

Conceptually:

```text
ALB
 ↓
Target Group
 ↓
Task IP:3000
```

---

# 83. Health Checks

ALB can check:

```text
GET /health
```

Example:

```text
HTTP 200
```

If:

```text
Task 1
```

fails health checks:

```text
ALB stops sending traffic
```

ECS can also replace unhealthy/stopped tasks depending on service configuration and health mechanisms.

---

# 84. Why Health Checks Matter

Suppose:

```text
Container process running
```

but:

```text
database unavailable
```

The container may technically be alive.

A useful application health endpoint can represent meaningful application health.

Example:

```text
GET /health
```

Response:

```json
{
  "status": "ok"
}
```

For deeper readiness semantics, you might use:

```text
/liveness
/readiness
```

with appropriate behavior.

---

# 85. ECS Fargate Build

Now create:

```text
ECR
 ↓
ECS
 ↓
Fargate
 ↓
ALB
```

---

# 86. Step 1 — Push Image to ECR

Use the image from the previous lab:

```text
hello-api:1.0
```

Push:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/hello-api:1.0
```

---

# 87. Step 2 — Create ECS Cluster

Console:

```text
AWS Console
 ↓
ECS
 ↓
Clusters
 ↓
Create cluster
```

Example:

```text
company-app-cluster
```

For Fargate, you do not need to create/manage EC2 container instances.

---

# 88. Step 3 — Create Task Definition

Go to:

```text
ECS
 ↓
Task definitions
 ↓
Create
```

Choose:

```text
Fargate
```

Configure:

```text
CPU
Memory
Task execution role
Task role
Container
Image
Port
Logs
Environment
Secrets
```

AWS's Fargate getting-started workflow uses a task definition as the blueprint and specifies container image, port mappings, and resource configuration. citeturn0search4

---

# 89. Example Task Definition Concept

```json
{
  "family": "hello-api",
  "networkMode": "awsvpc",
  "containerDefinitions": [
    {
      "name": "hello-api",
      "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/hello-api:1.0",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ]
    }
  ]
}
```

This is a simplified learning example.

Production definitions will normally also include:

```text
logging
health check
CPU/memory
secrets
environment
roles
```

---

# 90. Step 4 — Configure Logging

Send container logs to:

```text
CloudWatch Logs
```

Conceptually:

```text
Container
   ↓
stdout/stderr
   ↓
awslogs
   ↓
CloudWatch Logs
```

This is important because you should not depend on SSHing into servers to inspect container logs.

---

# 91. Step 5 — Create Fargate Service

In the ECS cluster:

```text
Create service
```

Choose:

```text
Launch type:
Fargate
```

Task definition:

```text
hello-api
```

Desired count:

```text
2
```

---

# 92. Step 6 — VPC

Select:

```text
company-vpc
```

Subnets:

```text
Private-App-A
Private-App-B
```

Security group:

```text
company-fargate-sg
```

Public IP:

```text
Disabled
```

The ALB should be public; the Fargate tasks can remain private.

---

# 93. Step 7 — Fargate Security Group

Inbound:

```text
TCP 3000
Source:
ALB SG
```

Outbound:

```text
Allow required outbound traffic
```

Do not make the task directly public just because the application needs internet traffic.

---

# 94. Step 8 — Load Balancer

Create/use:

```text
Application Load Balancer
```

Public subnets:

```text
Public-A
Public-B
```

ALB security group:

```text
ALB SG
```

Inbound:

```text
80
443
Source:
Internet
```

---

# 95. Step 9 — Target Group

Create:

```text
hello-api-tg
```

Target type:

```text
IP
```

Port:

```text
3000
```

Health check:

```text
HTTP
Path:
/health
```

---

# 96. Step 10 — Connect ECS Service to Target Group

Configure:

```text
ALB
 ↓
Listener
 ↓
Target Group
 ↓
ECS Service
 ↓
Fargate Tasks
```

The service registers task IPs as targets.

---

# 97. Step 11 — Test

Open:

```text
ALB DNS name
```

Example:

```text
hello-api-alb-123456.ap-south-1.elb.amazonaws.com
```

Request:

```text
GET /
```

Expected:

```text
Hello from Docker on AWS
```

---

# 98. Final Fargate Architecture

```text
                         INTERNET
                            │
                            ▼
                    ┌─────────────┐
                    │     ALB     │
                    │  Public     │
                    └──────┬──────┘
                           │
                       HTTP/HTTPS
                           │
                    ┌──────▼──────┐
                    │ Target Group│
                    └──────┬──────┘
                           │
             ┌─────────────┴─────────────┐
             │                           │
       ┌─────▼─────┐               ┌─────▼─────┐
       │  Fargate  │               │  Fargate  │
       │   Task    │               │   Task    │
       │ Container  │               │ Container  │
       └─────┬─────┘               └─────┬─────┘
             │                           │
             └─────────────┬─────────────┘
                           │
                          ECR
                           │
                     Docker Image
```

---

# 99. ECR → ECS → Fargate Flow

Remember this exact flow:

```text
Developer
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker tag
    │ docker push
    ▼
Amazon ECR
    │
    │ image pull
    ▼
ECS Task Definition
    │
    ▼
ECS Service
    │
    ▼
Fargate
    │
    ▼
Container
    │
    ▼
Target Group
    │
    ▼
ALB
    │
    ▼
Internet
```

---

# 100. Deployment Model

Imagine a new release:

```text
Version 1
hello-api:1.0
```

Then developer changes code.

Build:

```text
hello-api:1.1
```

Push:

```text
ECR
```

Update:

```text
ECS Task Definition
```

Deploy:

```text
ECS Service
```

ECS launches tasks using:

```text
1.1
```

and gradually replaces old tasks according to deployment configuration.

---

# 101. ECS Task Definition Revisions

Do not think:

```text
Task definition = one file forever
```

Instead:

```text
hello-api:1
hello-api:2
hello-api:3
hello-api:4
```

Each revision represents a configuration version.

For example:

```text
Revision 1
image = hello-api:1.0

Revision 2
image = hello-api:1.1
```

The service chooses which task-definition revision to deploy.

---

# 102. Rolling Deployment

Suppose:

```text
desired count = 2
```

Current:

```text
Task A → v1
Task B → v1
```

New deployment:

```text
Task C → v2
Task D → v2
```

ECS can progressively replace the old tasks according to deployment settings.

Conceptually:

```text
v1 + v1
   ↓
v1 + v2
   ↓
v2 + v2
```

This helps avoid taking the entire service offline during a normal deployment.

---

# 103. Scaling

Suppose:

```text
2 Fargate tasks
```

Traffic increases:

```text
2
 ↓
4
 ↓
8
```

ECS Service Auto Scaling can adjust desired task count based on metrics/policies.

Examples:

```text
CPU utilization
Memory utilization
Application Load Balancer request count
custom CloudWatch metrics
```

---

# 104. Desired Count

If:

```text
desiredCount = 3
```

ECS attempts to maintain:

```text
3 running tasks
```

If one stops:

```text
Task 1
Task 2
Task 3
   X
```

ECS service scheduling can replace it:

```text
Task 1
Task 2
Task 4
```

This is one of the major differences between manually running Docker and using ECS Service.

---

# 105. ECS Service vs Docker Run

Manual EC2:

```bash
docker run ...
```

You must manage:

```text
restart
replacement
deployment
scaling
```

ECS:

```text
Service
desiredCount = 3
```

ECS manages the desired service state.

---

# 106. Container Environment Variables

Do not bake environment-specific values into the image.

Bad:

```dockerfile
ENV ENVIRONMENT=production
```

for every environment.

Prefer:

```text
same image
+
environment-specific runtime configuration
```

Example:

```text
dev
 ↓
same image
 ↓
dev environment

staging
 ↓
same image
 ↓
staging environment

prod
 ↓
same image
 ↓
prod environment
```

---

# 107. Immutable Artifact Principle

A strong deployment pattern:

```text
Build once
    ↓
Test
    ↓
Push image
    ↓
Promote same image
```

Avoid:

```text
Build separately for dev
Build separately for staging
Build separately for prod
```

if those builds can drift.

Prefer:

```text
One tested image
    ↓
dev
    ↓
staging
    ↓
prod
```

with environment-specific runtime configuration.

---

# 108. Image Tagging Strategy

A practical strategy:

```text
company-backend:git-a1b2c3d
```

where:

```text
a1b2c3d
```

is a commit identifier.

You can also maintain human-readable release tags:

```text
company-backend:1.4.0
```

Production systems should have a clear policy for:

```text
tag mutability
rollback
retention
promotion
```

---

# 109. Rollback

Suppose:

```text
v2
```

is broken.

You want:

```text
v1
```

Again.

If the image still exists:

```text
ECR
 ↓
hello-api:git-oldcommit
 ↓
ECS task definition revision
 ↓
Service
```

Rollback becomes a deployment operation rather than rebuilding an old application from memory.

This is another reason to retain release images appropriately.

---

# 110. ECR Image Lifecycle and Rollback

Do not delete old images immediately.

Balance:

```text
storage cost
```

against:

```text
rollback capability
```

A company should define an image retention policy.

Example:

```text
Keep production release images:
90 days

Keep latest 20 development images
```

The actual policy depends on business and compliance needs.

---

# 111. Secrets in ECS

Your container may need:

```text
DB_PASSWORD
API_KEY
JWT_SIGNING_SECRET
```

Do not put them inside:

```text
Dockerfile
Git
ECR image
```

Use:

```text
AWS Secrets Manager
```

and configure the ECS task to retrieve the required secret.

This connects:

```text
Phase 1 IAM
+
Phase 6 RDS
+
Phase 7 ECS
```

---

# 112. Task Role + Secrets

Conceptually:

```text
Fargate Task
    │
    ▼
Task Role
    │
    ▼
Secrets Manager
    │
    ▼
DB credentials
```

The task should receive only the secrets it needs.

---

# 113. ECS + RDS Architecture

Now combine Phase 6 and Phase 7:

```text
Internet
   │
   ▼
 ALB
   │
   ▼
ECS Fargate
   │
   ▼
RDS PostgreSQL
```

Security groups:

```text
ALB SG
   ↓
Fargate SG
   ↓
RDS SG
```

RDS:

```text
Private
No public access
```

---

# 114. Full Company Backend Architecture

A more complete architecture:

```text
                         INTERNET
                            │
                            ▼
                       CloudFront
                            │
                            ▼
                          ALB
                            │
                  ┌─────────┴─────────┐
                  │                   │
              Fargate              Fargate
                Task                 Task
                  │                   │
                  └─────────┬─────────┘
                            │
                     RDS PostgreSQL
                       Private DB
                            │
                     ┌──────┴──────┐
                     │             │
                    S3         Secrets Manager
                     │
                  Application
                     │
                    IAM
```

ECR:

```text
Developer
   ↓
ECR
   ↓
Fargate
```

---

# 115. ALB vs ECS vs Fargate vs ECR

Do not confuse these services.

## ECR

```text
Where is my container image?
```

## ECS

```text
How do I orchestrate my containers?
```

## Fargate

```text
Where does the container compute run without me managing EC2 hosts?
```

## ALB

```text
How does external HTTP/HTTPS traffic reach my tasks?
```

Mental model:

```text
ECR = image storage
ECS = orchestration
Fargate = compute
ALB = traffic distribution
```

---

# 116. EC2 vs ECS/Fargate

## EC2 Docker

```text
You manage:
EC2
OS
Docker
host capacity
container processes
```

## ECS Fargate

```text
AWS manages:
underlying compute

You manage:
container image
task definition
service
networking
IAM
application
```

This is why Fargate is attractive for teams that want containers without operating container hosts.

---

# 117. ECS Troubleshooting

When a Fargate task won't start:

Check:

```text
Task stopped reason
```

Then:

```text
Task definition
   ↓
Image URI
   ↓
ECR permissions
   ↓
Execution role
   ↓
Networking
   ↓
Security Groups
   ↓
Subnet
   ↓
CloudWatch logs
```

---

# 118. Error: Cannot Pull ECR Image

Check:

```text
ECR repository exists?
```

```text
Image tag exists?
```

```text
Correct region?
```

```text
Correct account?
```

```text
Execution role?
```

Required ECR pull permissions include:

```text
ecr:GetAuthorizationToken
ecr:BatchGetImage
ecr:GetDownloadUrlForLayer
```

AWS documents these permissions for Fargate/ECS private ECR image pulls. citeturn0search0

---

# 119. Error: Task Starts Then Stops

Check:

```text
CloudWatch logs
```

Possible causes:

```text
Application crash
Missing environment variable
Missing secret
Wrong command
Wrong port
Database unavailable
Application binds to localhost
```

---

# 120. Error: ALB Health Check Fails

Check:

```text
Container port
Target group port
Health check path
Fargate SG
ALB SG
Application listening address
Application process
```

For example:

```text
App listens:
3000

Target group:
3000

Container:
3000

Fargate SG:
3000 from ALB SG
```

Everything must align.

---

# 121. Error: ALB Gives 502/503

Investigate:

```text
Target health
```

If:

```text
unhealthy
```

check:

```text
Health path
Port
Security group
Container
Application logs
```

If no healthy targets exist, ALB cannot successfully forward traffic.

---

# 122. Error: Container Works Locally But Not ECS

Classic problem.

Local:

```text
localhost:3000
```

ECS:

```text
fails
```

Check whether application is listening on:

```text
0.0.0.0
```

instead of:

```text
127.0.0.1
```

Also check:

```text
environment variables
secrets
network
port mappings
health check
IAM
```

---

# 123. Container Logs

Your application should log to:

```text
stdout
stderr
```

Then ECS can route those logs to CloudWatch.

Example:

```javascript
console.log("Application started");
console.error("Database connection failed");
```

Avoid relying on local files inside ephemeral containers for important logs.

---

# 124. Container Filesystem

Containers are generally treated as ephemeral.

Do not assume:

```text
container filesystem
```

is durable storage.

If your application needs persistent data, use an appropriate external storage service such as:

```text
RDS
S3
EFS
```

depending on the workload.

For databases:

```text
Do not store production database data inside the container filesystem.
```

---

# 125. Stateless Containers

A strong ECS architecture usually makes application containers stateless.

Example:

```text
Fargate Task 1
Fargate Task 2
Fargate Task 3
```

Any task can handle a request.

Persistent state goes elsewhere:

```text
Database
Object storage
Cache
Queue
```

This makes:

```text
scaling
replacement
deployment
```

much easier.

---

# 126. Session State

Suppose your backend stores user sessions only inside:

```text
Task 1 memory
```

Then:

```text
Request 1 → Task 1
Request 2 → Task 2
```

could fail because Task 2 does not have Task 1's memory.

Prefer an architecture where shared state is externalized.

Examples:

```text
Redis
Database
JWT
```

depending on your application design.

---

# 127. ECS Auto Scaling

You can scale based on:

```text
CPU
Memory
ALB request count
custom metrics
```

Example:

```text
CPU > 70%
   ↓
Increase tasks
```

When load falls:

```text
CPU < threshold
   ↓
Decrease tasks
```

But scaling policies should account for:

```text
startup time
database capacity
downstream services
traffic patterns
```

Scaling the application without scaling its dependencies can simply move the bottleneck.

---

# 128. Fargate + RDS Bottleneck Example

Suppose:

```text
Fargate tasks:
2 → 20
```

but:

```text
RDS:
small instance
```

Now 20 application tasks may create:

```text
many DB connections
```

Result:

```text
RDS connection pressure
```

Therefore:

```text
Application scaling
```

must be considered together with:

```text
Database scaling
Connection pooling
Caching
```

---

# 129. ECR Security Checklist

```text
[ ] Private repositories
[ ] IAM least privilege
[ ] Tag immutability considered
[ ] Image scanning enabled appropriately
[ ] Lifecycle policies configured
[ ] Old image retention defined
[ ] No secrets inside images
[ ] Trusted base images
[ ] Dockerfile reviewed
[ ] .dockerignore used
[ ] Non-root container where practical
[ ] Image provenance/release process understood
```

---

# 130. ECS Security Checklist

```text
[ ] Private Fargate subnets
[ ] No public IP unless specifically required
[ ] ALB public
[ ] Fargate SG accepts traffic only from ALB SG
[ ] Task execution role least privilege
[ ] Task role least privilege
[ ] Secrets Manager for sensitive values
[ ] CloudWatch logs
[ ] Health checks
[ ] ECR image scanning
[ ] Image tag strategy
[ ] HTTPS at ALB
[ ] RDS private
```

---

# 131. IAM Mental Model

Your Phase 1 IAM knowledge now becomes:

```text
Developer
   │
   ├── Push image
   ▼
ECR

ECS/Fargate
   │
   ├── Pull image
   ▼
ECR

Application
   │
   ├── Read secret
   ▼
Secrets Manager

Application
   │
   ├── Read object
   ▼
S3
```

Different identities should have different permissions.

---

# 132. Developer vs Runtime Permissions

A common mistake is:

```text
Give ECS task administrator access
```

because:

```text
"It needs AWS permissions."
```

Instead:

```text
Developer role
    ↓
Build/push permissions

Task execution role
    ↓
Pull/logging permissions

Task role
    ↓
Application-specific permissions
```

This separation is extremely important.

---

# 133. CI/CD Future Architecture

Eventually your deployment may become:

```text
Developer
    ↓
Git
    ↓
CI/CD
    ↓
Docker Build
    ↓
Security Scan
    ↓
ECR
    ↓
ECS Task Definition
    ↓
ECS Service
    ↓
Fargate
    ↓
ALB
```

Later you can learn:

```text
GitHub Actions
CodePipeline
CodeBuild
CodeDeploy
```

But first master the manual deployment path.

---

# 134. Why Learn Manual Deployment First?

If you immediately start with:

```text
GitHub Actions
```

you may not understand why:

```text
ECR push failed
ECS task stopped
ALB health failed
IAM permission denied
```

Manual deployment teaches the actual infrastructure.

Then CI/CD automates it.

---

# 135. Recommended Learning Sequence

Follow exactly:

```text
1. Docker fundamentals
       ↓
2. Dockerfile
       ↓
3. Build image
       ↓
4. Run container locally
       ↓
5. ECR repository
       ↓
6. Push image
       ↓
7. EC2 pulls image
       ↓
8. EC2 runs container
       ↓
9. ALB → EC2 → container
       ↓
10. ECS cluster
       ↓
11. Task definition
       ↓
12. Fargate task
       ↓
13. ECS service
       ↓
14. ALB → Fargate
       ↓
15. CloudWatch logs
       ↓
16. Health checks
       ↓
17. Scaling
       ↓
18. Deployment/rollback
```

Do not skip directly to step 18.

---

# 136. Hands-On Lab 1 — Docker Basics

Install Docker Desktop locally.

Verify:

```bash
docker version
```

Then:

```bash
docker run hello-world
```

Understand:

```text
Docker client
Docker engine
Image
Container
```

---

# 137. Hands-On Lab 2 — Build Your Own Image

Create:

```text
hello-api/
```

Create:

```text
Dockerfile
```

Build:

```bash
docker build -t hello-api:1.0 .
```

Run:

```bash
docker run --rm -p 3000:3000 hello-api:1.0
```

Test:

```bash
curl http://localhost:3000
```

---

# 138. Hands-On Lab 3 — Inspect Container

Run:

```bash
docker ps
```

Then:

```bash
docker logs <container>
```

Then:

```bash
docker inspect <container>
```

Then:

```bash
docker exec -it <container> sh
```

Learn what each command tells you.

---

# 139. Hands-On Lab 4 — ECR

Create:

```text
company-hello-api
```

Build:

```bash
docker build -t company-hello-api:1.0 .
```

Authenticate:

```bash
aws ecr get-login-password --region ap-south-1 \
| docker login --username AWS --password-stdin \
123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

Tag:

```bash
docker tag \
company-hello-api:1.0 \
123456789012.dkr.ecr.ap-south-1.amazonaws.com/company-hello-api:1.0
```

Push:

```bash
docker push \
123456789012.dkr.ecr.ap-south-1.amazonaws.com/company-hello-api:1.0
```

Verify in ECR.

---

# 140. Hands-On Lab 5 — EC2 Pull

Create an EC2 instance.

Attach:

```text
ECR pull IAM role
```

Install Docker.

Authenticate.

Pull:

```bash
docker pull \
123456789012.dkr.ecr.ap-south-1.amazonaws.com/company-hello-api:1.0
```

Run:

```bash
docker run -d \
--name company-hello-api \
-p 3000:3000 \
123456789012.dkr.ecr.ap-south-1.amazonaws.com/company-hello-api:1.0
```

Test from EC2.

---

# 141. Hands-On Lab 6 — ALB → EC2 Container

Architecture:

```text
Internet
   ↓
ALB
   ↓
EC2
   ↓
Docker Container
```

Configure:

```text
ALB SG
80/443 from Internet

EC2 SG
3000 from ALB SG
```

Do not expose EC2 port 3000 to the entire internet.

---

# 142. Hands-On Lab 7 — ECS Fargate

Create:

```text
ECS Cluster
```

Then:

```text
Task Definition
```

Then:

```text
Fargate Service
```

Then:

```text
ALB
```

Then:

```text
Target Group
```

Then:

```text
CloudWatch Logs
```

Test:

```text
ALB DNS
```

---

# 143. Hands-On Lab 8 — Scale to Two Tasks

Set:

```text
Desired count = 2
```

Observe:

```text
Task 1
Task 2
```

Then stop one task.

Observe ECS replace it.

Understand:

```text
desired state
```

---

# 144. Hands-On Lab 9 — Break Health Check

Change:

```text
/health
```

to:

```text
/not-found
```

Observe:

```text
Target unhealthy
```

Then restore:

```text
/health
```

This teaches ALB health checks.

---

# 145. Hands-On Lab 10 — Break IAM

Temporarily remove ECR pull permissions from the execution role.

Deploy a new task.

Observe:

```text
Task fails to start
```

Restore permission.

Deploy again.

This teaches the relationship:

```text
IAM
 ↓
ECR
 ↓
ECS
```

---

# 146. Hands-On Lab 11 — New Image Version

Build:

```text
hello-api:2.0
```

Change response:

```text
Hello from version 2
```

Push:

```text
ECR
```

Create:

```text
new task definition revision
```

Deploy.

Observe:

```text
v1
 ↓
v2
```

---

# 147. Hands-On Lab 12 — Rollback

Deploy:

```text
v2
```

Then simulate a broken version.

Rollback the ECS service to:

```text
previous task definition revision
```

Understand:

```text
deployment
rollback
task definition revision
image tag/digest
```

---

# 148. Hands-On Lab 13 — Secrets

Store:

```text
DB password
```

in:

```text
Secrets Manager
```

Give task role only the required access.

Inject/read it from ECS.

Never put the password into:

```text
Dockerfile
Git
ECR image
```

---

# 149. Hands-On Lab 14 — ECS → RDS

Final combined architecture:

```text
Internet
   ↓
ALB
   ↓
Fargate
   ↓
RDS PostgreSQL
```

Security groups:

```text
ALB SG
 ↓
Fargate SG
 ↓
RDS SG
```

RDS:

```text
Private
No public access
```

This combines:

```text
Phase 1 IAM
Phase 3 VPC
Phase 4 EC2/ALB
Phase 6 RDS
Phase 7 containers
```

---

# 150. Hands-On Lab 15 — S3 From Fargate

Give the task role:

```text
s3:GetObject
```

only for a specific bucket/prefix.

Then:

```text
Fargate
 ↓
Task Role
 ↓
S3
```

This is a real-world IAM use case.

---

# 151. Hands-On Lab 16 — CloudWatch Logs

Generate application logs:

```javascript
console.log("request received");
console.error("test error");
```

Configure ECS logging.

Then:

```text
ECS
 ↓
CloudWatch Logs
```

Find the log stream.

---

# 152. Hands-On Lab 17 — ECR Scan

Push a new image.

Open:

```text
ECR
 ↓
Repository
 ↓
Image
 ↓
Scan findings
```

Understand:

```text
package
CVE
severity
finding
```

Enhanced scanning may update findings as new vulnerabilities are discovered. citeturn0search2

---

# 153. Hands-On Lab 18 — Lifecycle Policy

Create an ECR lifecycle policy.

Example concept:

```text
Development images:
keep last 10
```

Then test the policy using the available preview/evaluation functionality before applying it broadly.

---

# 154. Container Troubleshooting Cheat Sheet

## Docker build fails

Check:

```text
Dockerfile
base image
COPY paths
package installation
build context
.dockerignore
```

---

## Container exits immediately

Check:

```bash
docker logs <container>
```

Then:

```text
CMD
ENTRYPOINT
application crash
environment variables
```

---

## Port unreachable

Check:

```text
Application listening port
Container port
Host mapping
Security Group
ALB target group
```

---

## ECR push denied

Check:

```text
AWS identity
ECR permissions
repository
region
Docker login
```

---

## ECR pull denied

Check:

```text
Task execution role / EC2 role
ECR permissions
repository
image tag
region
```

---

## Fargate task stopped

Check:

```text
Stopped reason
CloudWatch logs
Task definition
Execution role
Image
Secrets
Environment
Networking
```

---

## ALB target unhealthy

Check:

```text
Health path
Container port
Target group port
Fargate SG
ALB SG
Application bind address
Application logs
```

---

# 155. Production Container Security

A production container should be treated as an application artifact.

Review:

```text
Base image
Dependencies
OS packages
CVE findings
Secrets
User privileges
File permissions
Network access
IAM permissions
Logging
Health checks
Image provenance
```

---

# 156. Do Not Put Secrets in Images

This is worth repeating.

Bad:

```dockerfile
ENV AWS_SECRET_ACCESS_KEY=...
```

Bad:

```dockerfile
COPY .env .
```

Bad:

```text
password.txt
```

inside the image.

Why?

Because anyone who can access the image may potentially inspect its layers/content.

Use:

```text
Secrets Manager
Parameter Store
runtime configuration
```

instead.

---

# 157. Container Image Size

Smaller images generally help with:

```text
pull time
startup time
storage
attack surface
```

Use:

```text
slim/alpine/distroless
```

where appropriate and where compatibility/security trade-offs are understood.

Do not choose an image only because it is smaller.

Validate:

```text
libraries
debugging
security updates
runtime compatibility
```

---

# 158. Distroless Concept

A distroless image attempts to include mostly what the application needs to run rather than a full general-purpose userspace.

Concept:

```text
Full OS tools
     ↓
unneeded packages
     ↓
larger attack surface

Distroless
     ↓
minimal runtime
```

This is an advanced optimization. Learn normal Docker images first.

---

# 159. Container Image Vulnerability Does Not Automatically Mean Exploitation

A scan finding means:

```text
Known vulnerability detected
```

You still need to assess:

```text
severity
affected package
package actually used?
reachable?
runtime exposure?
available patch?
```

Security scanning should feed a remediation process.

---

# 160. ECR Image Lifecycle

Think:

```text
Build
 ↓
Push
 ↓
Scan
 ↓
Test
 ↓
Deploy
 ↓
Promote
 ↓
Retain
 ↓
Eventually clean up
```

Do not treat ECR as:

```text
permanent garbage storage
```

---

# 161. Deployment Promotion Model

A company can use:

```text
ECR
 │
 ├── image:git-a1b2c3d
 │
 └── image:git-d4e5f6a
```

Then:

```text
Development
    ↓
Test
    ↓
Staging
    ↓
Production
```

The same immutable image can be promoted across environments.

---

# 162. Container + Database Architecture

Your application may eventually look like:

```text
                        Internet
                           │
                           ▼
                         ALB
                           │
                  ┌────────┴────────┐
                  │                 │
               Fargate           Fargate
                  │                 │
                  └────────┬────────┘
                           │
                           ▼
                    RDS PostgreSQL
```

ECR:

```text
Fargate
   ↑
   │
  ECR
   ↑
   │
Developer/CI
```

Secrets:

```text
Fargate
   │
   ▼
Secrets Manager
```

Logs:

```text
Fargate
   │
   ▼
CloudWatch
```

---

# 163. Complete AWS Relationship

```text
                         Developer
                            │
                      Docker Build
                            │
                            ▼
                         Image
                            │
                            ▼
                          ECR
                            │
                            ▼
                         ECS
                            │
                            ▼
                         Fargate
                            │
                            ▼
                           ALB
                            │
                            ▼
                        Internet

Fargate
   │
   ├── Task Role ───────→ S3
   │
   ├── Task Role ───────→ Secrets Manager
   │
   ├── Execution Role ──→ ECR / Logs
   │
   └── Network ─────────→ RDS
```

This is the architecture you should be able to explain after Phase 7.

---

# 164. Key Differences to Memorize

```text
Docker Image
=
Packaged application artifact

Container
=
Running instance of an image

ECR
=
Container image registry

ECS
=
Container orchestration

Fargate
=
Serverless container compute

Task Definition
=
Blueprint

Task
=
Running instance of task definition

Service
=
Maintains desired task count

ALB
=
HTTP/HTTPS traffic distribution

Task Execution Role
=
ECS/Fargate infrastructure permissions

Task Role
=
Application container AWS permissions
```

---

# 165. Final Architecture

Memorize:

```text
                     DEVELOPER
                         │
                    docker build
                         │
                         ▼
                    DOCKER IMAGE
                         │
                    docker push
                         │
                         ▼
                  ┌──────────────┐
                  │     ECR      │
                  │  repository  │
                  └──────┬───────┘
                         │
                    image pull
                         │
                         ▼
                  ┌──────────────┐
                  │     ECS      │
                  │    Service   │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │   FARGATE    │
                  │    Tasks     │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │     ALB      │
                  └──────┬───────┘
                         │
                         ▼
                     INTERNET
```

With networking:

```text
ALB
 │
 ▼
Fargate SG
 │
 ▼
RDS SG
 │
 ▼
RDS PostgreSQL
```

---

# 166. Phase 7 Completion Checklist

You should be able to explain:

```text
[ ] What is Docker?
[ ] What is an image?
[ ] What is a container?
[ ] Image vs container?
[ ] What is a Dockerfile?
[ ] FROM?
[ ] WORKDIR?
[ ] COPY?
[ ] RUN?
[ ] EXPOSE?
[ ] CMD?
[ ] ENTRYPOINT?
[ ] .dockerignore?
[ ] Docker layers?
[ ] Docker build cache?
[ ] Multi-stage builds?
[ ] Why avoid secrets in images?
[ ] Why use non-root containers?
[ ] What is ECR?
[ ] Registry vs repository?
[ ] Image tag?
[ ] Image digest?
[ ] Tag immutability?
[ ] ECR lifecycle policy?
[ ] ECR image scanning?
[ ] Basic vs enhanced scanning?
[ ] How does Docker push to ECR?
[ ] How does EC2 pull from ECR?
[ ] What IAM role does EC2 need?
[ ] What is ECS?
[ ] Cluster?
[ ] Task definition?
[ ] Task?
[ ] Service?
[ ] Desired count?
[ ] What is Fargate?
[ ] ECS on EC2 vs Fargate?
[ ] Task execution role?
[ ] Task role?
[ ] awsvpc?
[ ] ECS security group?
[ ] ALB target group?
[ ] Why target type IP for Fargate?
[ ] Health checks?
[ ] CloudWatch logs?
[ ] ECS rolling deployment?
[ ] Task definition revisions?
[ ] Rollback?
[ ] ECS auto scaling?
[ ] How do you troubleshoot image pull failures?
[ ] How do you troubleshoot unhealthy ALB targets?
[ ] How do you keep RDS private behind Fargate?
```

---

# 167. Official AWS Documentation

## Amazon ECR

https://docs.aws.amazon.com/AmazonECR/latest/userguide/

## ECR Documentation

https://docs.aws.amazon.com/ecr/

## ECR Private Images

https://docs.aws.amazon.com/AmazonECR/latest/userguide/images.html

## Push Docker Image to ECR

https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html

## ECR + ECS

https://docs.aws.amazon.com/AmazonECR/latest/userguide/ECR_on_ECS.html

## ECR Image Scanning

https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html

## ECR Lifecycle Policies

https://docs.aws.amazon.com/AmazonECR/latest/userguide/lifecycle_policy.html

## Amazon ECS

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/

## ECS Architecture

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-configuration.html

## Creating Container Images for ECS

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html

## ECS Fargate Getting Started

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/getting-started-fargate.html

## ECS Task Execution IAM Role

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html

## ECS Task IAM Role

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html

## ECS Services

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html

## ECS Service Auto Scaling

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html

## Docker Documentation

https://docs.docker.com/

## Dockerfile Reference

https://docs.docker.com/reference/dockerfile/

---

# 168. Final Mental Model

If you remember only one thing from Phase 7:

```text
DOCKER
    ↓
Builds the application image

ECR
    ↓
Stores the application image

ECS
    ↓
Schedules/manages containers

FARGATE
    ↓
Runs containers without you managing EC2 container hosts

ALB
    ↓
Sends HTTP/HTTPS traffic to healthy tasks
```

And for IAM:

```text
Developer
    ↓
Can push
    ↓
ECR

ECS/Fargate
    ↓
Execution Role
    ↓
Can pull
    ↓
ECR

Application
    ↓
Task Role
    ↓
Can access only required AWS services
```

And for networking:

```text
Internet
    ↓
ALB
    ↓
Fargate
    ↓
RDS
```

not:

```text
Internet
    ↓
RDS
```

The complete production-style mental model is:

```text
Developer
   │
   ▼
Docker Build
   │
   ▼
ECR
   │
   ▼
ECS Service
   │
   ▼
Fargate Tasks
   │
   ▼
ALB
   │
   ▼
Internet

Fargate
   ├──→ Secrets Manager
   ├──→ S3
   ├──→ CloudWatch
   └──→ RDS PostgreSQL
```

Once this is clear, the next major step is to learn how to turn this manual architecture into a repeatable deployment pipeline and infrastructure-as-code workflow.
