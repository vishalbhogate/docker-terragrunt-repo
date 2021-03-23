## Docker mounts

The working directory inside the Docker container is **`/data/`** and should be mounted to your local filesystem where your Terragrant project resides.
(See [Examples](#examples) for mount location usage.)


## Usage

```bash
docker run --rm -v $(pwd):/data cytopia/terragrunt terragrunt <ARGS>
docker run --rm -v $(pwd):/data cytopia/terragrunt terraform <ARGS>
```

## Examples

### 1. Simple: Provision single sub-project on AWS

#### 1.1 Project overview
Let's assume your Terragrunt project setup is as follows:
```bash
/my/tf                                              # Terragrunt project root
├── backend-app
│   ├── main.tf
│   └── terragrunt.hcl
├── frontend-app
│   ├── main.tf
│   └── terragrunt.hcl
├── mysql                                           # MySQL sub-project directory
│   ├── main.tf
│   └── terragrunt.hcl
├── redis
│   ├── main.tf
│   └── terragrunt.hcl
└── vpc
    ├── main.tf
    └── terragrunt.hcl
```
The **MySQL** sub-project you want to provision is at the releative path `mysql/`.

#### 1.2 To consider
1. Mount the terragrunt root project dir (`/my/tf/`) into `/data/` into the container
2. Use the workding dir (`-w` or `--workdir`) to point to your project inside the container
3. Add AWS credentials from your environment to the container

#### 1.3 Docker commands
```bash
# Initialize the MySQL project
docker run --rm \
  -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
  -e AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN \
  -u $(id -u):$(id -g) \
  -v /my/tf:/data \
  -w /data/mysql \
  cytopia/terragrunt terragrunt init

# Plan the MySQL project
docker run --rm \
  -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
  -e AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN \
  -u $(id -u):$(id -g) \
  -v /my/tf:/data \
  -w /data/mysql \
  cytopia/terragrunt terragrunt plan

# Apply the MySQL project
docker run --rm \
  -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
  -e AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN \
  -u $(id -u):$(id -g) \
  -v /my/tf:/data \
  -w /data/mysql \
  cytopia/terragrunt terragrunt --terragrunt-non-interactive apply
```
<!-- #### 1.4 Makefile integration -->

### 2. Complex: Provision single sub-project on AWS

#### 2.1 Project overview
Let's assume your Terragrunt project setup is as follows:
```bash
/my/tf                                              # Terragrunt project root
└── envs
    └── aws
        ├── dev
        │   ├── eu-central-1
        │   │   ├── infra
        │   │   │   └── vpc-k8s                     # VPC sub-project directory
        │   │   │       ├── terraform.tfvars
        │   │   │       └── terragrunt.hcl
        │   │   ├── microservices
        │   │   │   └── api-gateway
        │   │   │       ├── terraform.tfvars
        │   │   │       └── terragrunt.hcl
        │   │   └── region.tfvars
        │   ├── global
        │   │   └── region.tfvars
        │   └── terragrunt.hcl
        └── _provider_include
            └── include_providers.tf
```
The **VPC** sub-project you want to provision is at the relative path `envs/aws/dev/eu-centra-1/infra/vpc-k8s/`.

#### 2.2 To consider
1. Mount the terragrunt root project dir (`/my/tf/`) into `/data/` into the container
2. Use the workding dir (`-w` or `--workdir`) to point to your project inside the container
3. Add AWS credentials from your environment to the container

#### 2.3 Docker commands
```bash
# Initialize the VPC project
docker run --rm \
  -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
  -e AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN \
  -u $(id -u):$(id -g) \
  -v /my/tf:/data \
  -w /data/envs/aws/dev/eu-central-1/infra/vpc-k8s \
  cytopia/terragrunt terragrunt init

# Plan the VPC project
docker run --rm \
  -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
  -e AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN \
  -u $(id -u):$(id -g) \
  -v /my/tf:/data \
  -w /data/envs/aws/dev/eu-central-1/infra/vpc-k8s \
  cytopia/terragrunt terragrunt plan

# Apply the VPC project
docker run --rm \
  -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
  -e AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN \
  -u $(id -u):$(id -g) \
  -v /my/tf:/data \
  -w /data/envs/aws/dev/eu-central-1/infra/vpc-k8s \
  cytopia/terragrunt terragrunt --terragrunt-non-interactive apply
```
