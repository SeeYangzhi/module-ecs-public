**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-ecs>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-ecs/blob/master/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# ECS Modules

This repo contains modules for running Docker containers on top of [Amazon EC2 Container Service
(ECS)](https://aws.amazon.com/ecs/). The modules are:

* [ecs-cluster](/modules/ecs-cluster): Deploy a cluster of EC2 instances that ECS can use for running Docker
  containers.
* [ecs-service](/modules/ecs-service): Deploy an ECS Service, which is a way to run one or more related, long-running
  Docker containers, such as a web service. An ECS service can automatically deploy multiple instances of your Docker
  containers across an ECS cluster, restart any failed Docker containers, and route traffic across your containers
  using an optional Elastic Load Balancer (ELB).
* [ecs-scripts](/modules/ecs-scripts): Helper scripts you can run on the EC2 instances in your ECS cluster to
  initialize and configure them.

## What is a module?

At [Gruntwork](http://www.gruntwork.io), we've taken the thousands of hours we spent building infrastructure on AWS and
condensed all that experience and code into pre-built **packages** or **modules**. Each module is a battle-tested,
best-practices definition of a piece of infrastructure, such as a VPC, ECS cluster, or an Auto Scaling Group. Modules
are versioned using [Semantic Versioning](http://semver.org/) to allow Gruntwork clients to keep up to date with the
latest infrastructure best practices in a systematic way.

## How do you use a module?

Most of our modules contain either:

1. [Terraform](https://www.terraform.io/) code
1. Scripts & binaries

#### Using a Terraform Module

To use a module in your Terraform templates, create a `module` resource and set its `source` field to the Git URL of
this repo. You should also set the `ref` parameter so you're fixed to a specific version of this repo, as the `master`
branch may have backwards incompatible changes (see [module
sources](https://www.terraform.io/docs/modules/sources.html)).

For example, to use `v1.0.8` of the ecs-cluster module, you would add the following:

```hcl
module "ecs_cluster" {
  source = "git::git@github.com:gruntwork-io/module-ecs.git//modules/ecs-cluster?ref=v1.0.8"

  // set the parameters for the ECS cluster module
}
```

*Note: the double slash (`//`) is intentional and required. It's part of Terraform's Git syntax (see [module
sources](https://www.terraform.io/docs/modules/sources.html)).*

See the module's documentation and `vars.tf` file for all the parameters you can set. Run `terraform get -update` to
pull the latest version of this module from this repo before runnin gthe standard  `terraform plan` and
`terraform apply` commands.

#### Using scripts & binaries

You can install the scripts and binaries in the `modules` folder of any repo using the [Gruntwork
Installer](https://github.com/gruntwork-io/gruntwork-installer). For example, if the scripts you want to install are
in the `modules/ecs-scripts` folder of the https://github.com/gruntwork-io/module-ecs repo, you could install them
as follows:

```bash
gruntwork-install --module-name "ecs-scripts" --repo "https://github.com/gruntwork-io/module-ecs" --tag "0.0.1"
```

See the docs for each script & binary for detailed instructions on how to use them.

## What is EC2 Container Service?

EC2 Container Service (ECS) is the official AWS solution for running Docker containers on EC2 instances in a
fault-tolerant, scalable, and highly available way. Its primary advantage over alternatives like
[Mesos](http://mesos.apache.org/) and [Kubernetes](http://kubernetes.io/) is that it's much easier to set up,
understand, and integrate with other AWS services. Its primary downside is that it offers less powerful options for
"scheduling" containers across different hosts.

### Helpful Vocabulary

Amazon has its own vocabulary for ECS that can be confusing. Here's a helpful guide:

- **ECS Cluster:** One or more servers (i.e. EC2 instances) that ECS can use for deploying Docker containers.
- **Container Instance:** A single node (i.e. EC2 Instance) in an ECS Cluster.
- **ECS Task:** One or more Docker containers that should be run as a group on a single instance.
- **ECS Task Definition (AKA ECS Container Definition):** A JSON file that defines an ECS Task, including the
  container(s) to run, the resources (memory, CPU) those containers need, the volumes to mount, the environment
  variables to set, and so on.
- **Task Definition Revision:** ECS Tasks are immutable. Once you define a Task Definition, you can never change it:
  you can only create new Task Definitions, which are known as revisions. The most common revision is to change what
  version of a Docker container to deploy.
- **ECS Service:** A way to deploy and manage long-running ECS Tasks, such as a web service. The service can deploy
  your Tasks across one or more instances in the ECS Cluster, restart any failed Tasks, and route traffic across your
  Tasks using an optional Elastic Load Balancer.

## Developing a module

#### Versioning

We are following the principles of [Semantic Versioning](http://semver.org/). During initial development, the major
version is to 0 (e.g., `0.x.y`), which indicates the code does not yet have a stable API. Once we hit `1.0.0`, we will
follow these rules:

1. Increment the patch version for backwards-compatible bug fixes (e.g., `v1.0.8 -> v1.0.9`).
2. Increment the minor version for new features that are backwards-compatible (e.g., `v1.0.8 -> 1.1.0`).
3. Increment the major version for any backwards-incompatible changes (e.g. `1.0.8 -> 2.0.0`).

The version is defined using Git tags.  Use GitHub to create a release, which will have the effect of adding a git tag.

#### Tests

See the [test](/test) folder for details.