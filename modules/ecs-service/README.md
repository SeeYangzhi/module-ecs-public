**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-ecs>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-ecs/blob/master/modules/ecs-service/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# ECS Service Module

This Terraform Module creates an [EC2 Container Service
Service](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) that you can use to run one or
more related, long-running Docker containers, such as a web service. An ECS service can automatically deploy multiple
instances of your Docker containers across an ECS cluster (see the [ecs-cluster module](../ecs-cluster)), restart any
failed Docker containers, and route traffic across your containers using an optional Elastic Load Balancer (ELB).

## How do you use this module?

* See the [root README](/README.md) for instructions on using Terraform modules.
* See the [examples](/examples) folder for example usage.
* See [vars.tf](./vars.tf) for all the variables you can set on this module.
* See the [ecs-cluster module](../ecs-cluster) for how to run an ECS cluster.

## What is an ECS Service?

To run Docker containers with ECS, you first define an [ECS
Task](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_defintions.html), which is a JSON file that
describes what container(s) to run, the resources (memory, CPU) those containers need, the volumes to mount, the
environment variables to set, and so on. To actually run an ECS Task, you define an ECS Service, which can:

1. Deploy the requested number of Tasks across an ECS cluster based on the `desired_number_of_tasks` input variable.
1. Restart tasks if they fail.
1. Route traffic across the tasks with an optional Elastic Load Balancer (ELB). To use an ELB, set `is_associated_with_elb`
   to `true` and pass in the ELB details using the `load_balancer_name`, `container_name`, and `container_port`
   input variables.

## How do you create an ECS cluster?

To use ECS, you first deploy one or more EC2 Instances into a "cluster". See the [ecs-cluster module](../ecs-cluster)
for how to create a cluster.

## How do ECS Services deploy new versions of containers?

When you update an ECS Task (e.g. change the version number of a Docker container to deploy), ECS will roll the change
out automatically across your cluster according to two input variables:

* `deployment_maximum_percent`: This variable controls the maximum number of copies of your ECS Task, as a percentage of
  `desired_number_of_tasks`, that can be deployed during an update. For example, if you have 4 Tasks running at version
  1, `deployment_maximum_percent` is set to 200, and you kick off a deployment of version 2 of your Task, ECS will
  first deploy 4 Tasks at version 2, wait for them to come up, and then it'll undeploy the 4 Tasks at version 1. Note
  that this only works if your ECS cluster has capacity--that is, EC2 instances with the available memory, CPU, ports,
  etc requested by your Tasks, which might mean maintaining several empty EC2 instances just for deployment.
* `deployment_minimum_healthy_percent`: This variable controls the minimum number of copies of your ECS Task, as a
  percentage of `desired_number_of_tasks`, must stay running during an update. For example, if you have 4 Tasks running
  at version 1, you set `deployment_minimum_healthy_percent` to 50, and you kick off a deployment of version 2 of your
  Task, ECS will first undeploy 2 Tasks at version 1, then deploy 2 Tasks at version 2 in their place, and then repeat
  the process again with the remaining 2 tasks. This allows you to roll out new versions without having to keep spare
  EC2 instances, but it also means the availability of your service is somewhat reduced during rollouts.

## How do you add additional IAM policies?

If you associate this ECS Service with an ELB (`is_associated_with_elb` is set to true), then we create an IAM Role and
associated IAM Policies that allow the ECS Service to talk to the ELB. To add additional IAM policies to this IAM Role,
you can use the [aws_iam_role_policy](https://www.terraform.io/docs/providers/aws/r/iam_role_policy.html) or
[aws_iam_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_policy_attachment.html) resources, and
set the IAM role id to the Terraform output of this module called `service_iam_role_id` . For example, here is how
you can allow the ECS Service in this cluster to access an S3 bucket:

```hcl
module "ecs_service_with_elb" {
  # (arguments omitted)
}

resource "aws_iam_role_policy" "access_s3_bucket" {
    name = "access_s3_bucket"
    role = "${module.ecs_service_with_elb.service_iam_role_id}"
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