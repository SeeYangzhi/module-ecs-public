**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-ecs>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-ecs/blob/master/modules/ecs-scripts/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# ECS Scripts

This folder contains helper scripts for running an ECS Cluster, including:

* `configure-ecs-instance`: This script configures an EC2 Instance so it registers in the specified ECS cluster and
  uses the specified credentials for private Docker registry access. Note that this script can only be run as root on
  an EC2 instance with the [Amazon ECS-optimized AMI](https://aws.amazon.com/marketplace/pp/B00U6QTYI2/) installed.

## Installing the helpers

You can install the helpers using the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer):

```bash
gruntwork-install --module-name "ecs-scripts" --repo "https://github.com/gruntwork-io/module-ecs" --tag "0.0.1"
```

For an example, see the [Packer](https://www.packer.io/) template under
[examples/docker-service-with-elb/packer/build.json](/examples/docker-service-with-elb/packer/build.json).

## Using the configure-ecs-instance helper

The `configure-ecs-instance` script has the following prerequisites:

1. It must be run on an EC2 instance.
1. The EC2 instance must be running an [Amazon ECS-optimized AMI](https://aws.amazon.com/marketplace/pp/B00U6QTYI2/).
1. The EC2 instance must have the AWS CLI installed.

To run the script, you need to pass it the ECS cluster name and the type of Docker repo auth you are using
(one of `ecr`, `docker-hub`, `docker-other`, or `none`). If you are using docker-hub or docker-other, you need to pass
the auth details using the environment variables `DOCKER_REPO_URL`, `DOCKER_REPO_AUTH`, and `DOCKER_REPO_EMAIL`. See
[Docker Authentication
Formats](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/private-auth.html#docker-auth-formats) to learn
about how ECS handles Docker registry authentication.

For example, to use [Amazon's EC2 Container Registry (ECR)](https://aws.amazon.com/ecr/), you would run:

```bash
configure-ecs-instance --ecs-cluster-name my-ecs-cluster --docker-auth-type ecr
```

To use a private [Docker Hub](https://hub.docker.com/) repo, you would run:

```bash
export DOCKER_REPO_AUTH="(your Docker Hub auth value)"
export DOCKER_REPO_EMAIL="(your Docker Hub email)"
configure-ecs-instance --ecs-cluster-name my-ecs-cluster --docker-auth-type docker-hub
```

To use a private Docker registry other than Docker Hub, you would run:

```bash
export DOCKER_REPO_URL="(your Docker repo URL)"
export DOCKER_REPO_AUTH="(your Docker repo auth value)"
export DOCKER_REPO_EMAIL="(your Docker repo email)"
configure-ecs-instance --ecs-cluster-name my-ecs-cluster --docker-auth-type docker-hub
```

Run `configure-ecs-instance --help` to see all available options.
