**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-ecs>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-ecs/blob/master/modules/ecs-cluster/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# ECS Cluster Module

This Terraform Module launches an [EC2 Container Service
Cluster](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_clusters.html) that you can use to run
Docker containers and services (see the [ecs-service module](../ecs-service)).

## How do you use this module?

* See the [root README](/README.md) for instructions on using Terraform modules.
* See the [examples](/examples) folder for example usage.
* See [vars.tf](./vars.tf) for all the variables you can set on this module.
* See the [ecs-service module](../ecs-service) for how to run Docker containers across this cluster.

## What is an ECS Cluster?

To use ECS, you first deploy one or more EC2 Instances into a "cluster". The ECS scheduler can then deploy Docker
containers across any of the instances in this cluster. Each instance needs to have the [Amazon ECS
Agent](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_agent.html) installed so it can communicate with
ECS and register itself as part of the right cluster.

## How do you run Docker containers on the cluster?

See the [service module](../service).

## How do you add additional security group rules?

To add additional security group rules to the EC2 Instances in the ECS cluster, you can use the
[aws_security_group_rule](https://www.terraform.io/docs/providers/aws/r/security_group_rule.html) resource, and set its
`security_group_id` argument to the Terraform output of this module called `ecs_instance_security_group_id`. For
example, here is how you can allow the EC2 Instances in this cluster to allow incoming HTTP requests on port 8080:

```hcl
module "ecs_cluster" {
  # (arguments omitted)
}

resource "aws_security_group_rule" "allow_inbound_http_from_anywhere" {
  type = "ingress"
  from_port = 8080
  to_port = 8080
  protocol = "tcp"
  cidr_blocks = ["0.0.0.0/0"]

  security_group_id = "${module.ecs_cluster.ecs_instance_security_group_id}"
}
```

**Note**: The security group rules you add will apply to ALL Docker containers running on these EC2 Instances. There is
currently no way in ECS to manage security group rules on a per-Docker-container basis.

## How do you add additional IAM policies?

To add additional IAM policies to the EC2 Instances in the ECS cluster, you can use the
[aws_iam_role_policy](https://www.terraform.io/docs/providers/aws/r/iam_role_policy.html) or
[aws_iam_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_policy_attachment.html) resources, and
set the IAM role id to the Terraform output of this module called `ecs_instance_iam_role_id` . For example, here is how
you can allow the EC2 Instances in this cluster to access an S3 bucket:

```hcl
module "ecs_cluster" {
  # (arguments omitted)
}

resource "aws_iam_role_policy" "access_s3_bucket" {
    name = "access_s3_bucket"
    role = "${module.ecs_cluster.ecs_instance_iam_role_id}"
    policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect":"Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::examplebucket/*"
    }
  ]
}
EOF
}
```

**Note**: The IAM policies you add will apply to ALL Docker containers running on these EC2 Instances. There is
currently no way in ECS to manage IAM policies on a per-Docker-container basis.

## How do you make changes to the EC2 Instances in the cluster?

While updating the Docker containers running in your cluster is easy and automatically rolled out by the ECS scheduler
(see the [service module docs](../service)), updating the actual instances in the ECS cluster is trickier. Currently,
**the instances in this ECS cluster will NOT update automatically** when you make a change to the launch configuration
of the ECS Instances. For example, if you change the instance type or change the AMI and run `terraform apply`, the
command will complete successfully, but nothing will automatically change with the instances in the cluster until you
manually deploy new ones.

That's because each instance may be running one or more Docker containers and you need to find a way to convince the
ECS scheduler to move those containers to the new instances. There are currently two options for doing this:

1. Temporarily double the size of your ECS cluster and the number of desired tasks for all of your ECS services. The
   ASG will roll out the new instances and the ECS scheduler will deploy the new tasks on top of those new instances.
   Once everything is up and running, you can reduce the size of the ECS cluster and the number of desired tasks back
   to normal. By default, the ASG should automatically remove the old instances, leaving you with a cluster with just
   the new ones.
1. Terminate one old instance at a time. This cluster is an Auto Scaling Group (ASG), so it will automatically detect
   that an old instance has been terminated and deploy a new one. Similarly, the ECS cluster will automatically detect
   that the Docker containers running on that instance were terminated and will automatically deploy them onto any
   available instances in the cluster (including the new one you deployed). Assuming you have at least two copies of
   every Docker container running in your cluster, terminating one instance at a time and waiting for the ASG and ECS
   cluster to automatically relaunch everything will allow you to deploy without downtime.

When Terraform adds support for [ECS auto scaling](https://github.com/hashicorp/terraform/issues/6763), we may be
able to automate option #1 by adding auto scaling policies (one doubling the size, one halving it) for the ASG and each
ECS service and trigger those policies to do a deployment.