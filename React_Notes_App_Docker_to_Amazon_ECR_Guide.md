# React Notes App -- Docker to Amazon ECR Deployment Guide

Repository: https://github.com/kushguglani/react-note-docker

## Architecture

``` text
React Application
        │
        ▼
Dockerfile
        │
        ▼
docker build
        │
        ▼
Docker Image
        │
        ▼
Amazon ECR ✅
        │
        ▼
EC2 + Docker (Optional)
        │
        ▼
Live Application
```

------------------------------------------------------------------------

# 1. Clone the Repository

``` bash
git clone https://github.com/kushguglani/react-note-docker.git
cd react-note-docker
```

------------------------------------------------------------------------

# 2. Install Node.js

Use Node **16.20.2** and npm **8.19.4**

Verify:

``` bash
node -v
npm -v
```

Expected:

``` text
Node: v16.20.2
npm: 8.19.4
```

## Change Node Version Using nvm

### macOS/Linux

Install nvm if required:

``` bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

Reload terminal then:

``` bash
nvm install 16.20.2
nvm use 16.20.2
nvm alias default 16.20.2
```

Verify:

``` bash
node -v
npm -v
```

------------------------------------------------------------------------

# 3. Install Dependencies

``` bash
npm install
```

Run locally:

``` bash
npm start
```

Open:

http://localhost:3000

------------------------------------------------------------------------

# 4. Dockerfile

``` dockerfile
FROM node:16
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm","start"]
```

### One-line explanation

  Instruction             Purpose
  ----------------------- ------------------------------
  FROM node:16            Uses Node.js 16 base image
  WORKDIR /app            Sets the working directory
  COPY package\*.json .   Copies dependency files
  RUN npm install         Installs dependencies
  COPY . .                Copies project source
  EXPOSE 3000             Documents application port
  CMD \["npm","start"\]   Starts the React application

------------------------------------------------------------------------

# 5. Build Docker Image

``` bash
docker build -t notes-app:v1 .
```

Verify:

``` bash
docker images
```

Run locally:

``` bash
docker run -d -p 3000:3000 --name notes-app notes-app:v1
```

Visit:

http://localhost:3000

Stop:

``` bash
docker stop notes-app
docker rm notes-app
```

------------------------------------------------------------------------

# 6. Create Amazon ECR Repository

AWS Console

→ Elastic Container Registry (ECR)

→ Create Repository

Repository Type: **Private**

Repository Name:

``` text
notes-app
```

Example Repository URI:

``` text
397483228792.dkr.ecr.eu-north-1.amazonaws.com/notes-app
```

------------------------------------------------------------------------

# 7. Create IAM User

AWS Console

→ IAM

→ Users

→ Create User

Username:

``` text
docker-user
```

Attach policy:

``` text
AmazonEC2ContainerRegistryFullAccess
```

Create an **Access Key** (CLI).

Save:

-   Access Key ID
-   Secret Access Key

------------------------------------------------------------------------

# 8. Install AWS CLI

Verify:

``` bash
aws --version
```

Configure:

``` bash
aws configure
```

Example:

``` text
AWS Access Key ID: <YOUR_KEY>
AWS Secret Access Key: <YOUR_SECRET>
Default region: eu-north-1
Default output: json
```

Verify:

``` bash
aws sts get-caller-identity
```

------------------------------------------------------------------------

# 9. Login to Amazon ECR

``` bash
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 397483228792.dkr.ecr.eu-north-1.amazonaws.com
```

Expected:

``` text
Login Succeeded
```

------------------------------------------------------------------------

# 10. Tag Docker Image

``` bash
docker tag notes-app:v1 397483228792.dkr.ecr.eu-north-1.amazonaws.com/notes-app:v1
```

Verify:

``` bash
docker images
```

------------------------------------------------------------------------

# 11. Push Image to Amazon ECR

``` bash
docker push 397483228792.dkr.ecr.eu-north-1.amazonaws.com/notes-app:v1
```

Verify in AWS Console:

ECR → notes-app → Images → v1

------------------------------------------------------------------------

# 12. Pull Image from ECR

``` bash
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 397483228792.dkr.ecr.eu-north-1.amazonaws.com

docker pull 397483228792.dkr.ecr.eu-north-1.amazonaws.com/notes-app:v1
```

Run:

``` bash
docker run -d -p 3000:3000 397483228792.dkr.ecr.eu-north-1.amazonaws.com/notes-app:v1
```

Open:

http://localhost:3000

------------------------------------------------------------------------

# 13. Deploy Using EC2 + Docker

Launch a Free Tier EC2 instance.

Open Security Group:

-   SSH (22)
-   HTTP (80)

SSH:

``` bash
ssh -i your-key.pem ec2-user@<EC2-Public-IP>
```

Install Docker:

Amazon Linux:

``` bash
sudo yum update -y
sudo yum install docker -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user
newgrp docker
```

Configure AWS CLI:

``` bash
aws configure
```

Login:

``` bash
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 397483228792.dkr.ecr.eu-north-1.amazonaws.com
```

Pull:

``` bash
docker pull 397483228792.dkr.ecr.eu-north-1.amazonaws.com/notes-app:v1
```

Run:

``` bash
docker run -d -p 80:3000 397483228792.dkr.ecr.eu-north-1.amazonaws.com/notes-app:v1
```

Visit:

``` text
http://<EC2-Public-IP>
```

Your application is now live.

------------------------------------------------------------------------

# Summary

``` text
Clone Repository
      ↓
Install Node 16
      ↓
npm install
      ↓
npm start
      ↓
Create Dockerfile
      ↓
docker build
      ↓
Docker Image
      ↓
Amazon ECR
      ↓
docker pull
      ↓
EC2 + Docker
      ↓
Live Application
```
