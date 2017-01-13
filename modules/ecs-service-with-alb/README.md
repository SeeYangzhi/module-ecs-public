**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-ecs>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-ecs/blob/master/modules/ecs-service-with-alb/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# ECS Service with ALB

This Terraform Module creates an [EC2 Container Service
Service](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) that you can use to run one or
more related, long-running Docker containers, such as a web service, fronted by an [Application Load 
Balancer](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) as created by the 
Gruntwork [alb module](https://github.com/gruntwork-io/module-load-balancer-public/tree/master/modules/alb). An ECS service can 
automatically deploy multiple instances of your Docker containers across an ECS cluster (see the [ecs-cluster module]
(../ecs-cluster)), and restart any failed Docker containers.

This module also supports [canary deployment](http://martinfowler.com/bliki/CanaryRelease.html), where you can deploy a
single instance of a new Docker container version, test it, and if everything works well, deploy that version across
the rest of the cluster.

**If you wish to deploy an ECS Service with a Classic Load Balancer (ELB), or no load balancer at all, see the [ecs-service
module](../ecs-service).**

## How do you use this module?

* See the [root README](/README.md) for instructions on using Terraform modules.
* See the [examples](/examples) folder for example usage.
* See [vars.tf](./vars.tf) for all the variables you can set on this module.
* See the [ecs-cluster module](../ecs-cluster) for how to run an ECS cluster.
* See the [alb module](https://github.com/gruntwork-io/module-load-balancer-public/tree/master/modules/alb) for how to create 
  an ALB that will route requests to this ECS Service.

**This module expects that you have already created an instance of the [alb module](https://github.com/gruntwork-io/module-load-balancer-public/tree/master/modules/alb).**

## Common Questions

### See the ecs-service module.

See the [ecs-service module](../ecs-service) for additional information on what is an ECS Service, how to do canary
deployments, and more.

### How do you add additional IAM policies?

This module creates an [IAM Role for the ECS Tasks](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html)
run by the ECS Service. Any custom IAM Policies needed by this ECS Service should be attached to that IAM Role. 

To do this in Terraform, you can use the [aws_iam_role_policy](https://www.terraform.io/docs/providers/aws/r/iam_role_policy.html) or
[aws_iam_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_policy_attachment.html) resources, and
set the `role` property to the Terraform output of this module called `ecs_task_iam_role_name`. For example, here is how
you can allow the ECS Service in this cluster to access an S3 bucket:

```hcl
module "ecs_service" {
  # (arguments omitted)
}

resource "aws_iam_role_policy" "access_s3_bucket" {
    name = "access_s3_bucket"
    role = "${module.ecs_service.ecs_task_iam_role_name}"
    policy = "${aws_iam_policy_document.access_s3_bucket.json}"
}

data "aws_iam_policy_document" "access_s3_bucket" {
  statement {
    effect = "Allow"
    actions = ["s3:GetObject"]
    resources = ["arn:aws:s3:::examplebucket/*"]
  }
}
```

## Known Issues

### Switching the value of `var.use_alb_sticky_sessions`

If you switch `var.use_alb_sticky_sessions` from true to false or vice versa, Terraform will attempt to destroy and 
re-create the `aws_alb_target_group` which has a chain of dependencies that eventually lead to destroying and re-creating 
the ECS Service, which will lead to downtime. This is because we conditionally create Terraform resources depending on
the value of`var.use_alb_sticky_sessions`, and Terraform can't fully incorporate this concept into its dependency graph.
   
Fortunately, there's a workaround using manual state manipulation. We'll tell Terraform that the old resource is now 
the new one as follows.
   
```
# If you are changing var.use_alb_sticky_sessions from TRUE to FALSE:
terraform state mv module.ecs_service.aws_alb_target_group.ecs_service_with_sticky_sessions module.ecs_service.aws_alb_target_group.ecs_service_without_sticky_sessions

# If you are changing var.use_alb_sticky_sessions from FALSE to TRUE:
terraform state mv module.ecs_service.aws_alb_target_group.ecs_service_without_sticky_sessions module.ecs_service.aws_alb_target_group.ecs_service_with_sticky_sessions
```

Now run `terragrunt plan` to confirm that Terraform will only make modifications.