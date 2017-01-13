**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-ecs>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-ecs/blob/master/examples/docker-service-with-alb-autoscaling/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Docker Service with ALB Auto Scaling example

This folder shows an example of how to use the ECS modules to:

1. Deploy an ECS cluster
1. Deploy an ALB that can be shared among many ECS Services
1. Run a simple "Hello, World" web service Docker container as an ECS service
1. Automatically scale the number of instances of the ECS Service in response to load

## How do you run this example?

To run this example, you need to do the following:

1. Build the AMI
1. Apply the Terraform templates
1. Generate load

### Build the AMI

See the [example-ecs-instance-ami docs](/examples/example-ecs-instance-ami).

#### Apply the Terraform templates

To apply the Terraform templates:

1. Install [Terraform](https://www.terraform.io/)
1. Open `vars.tf`, set the environment variables specified at the top of the file, and fill in any other variables that
   don't have a default. This includes setting the `cluster_instance_ami` the ID of the AMI you just built.
1. Run `terraform get`.
1. Run `terraform plan`.
1. If the plan looks good, run `terraform apply`.

#### Generate load

A simple way to generate load is to use [ab](http://httpd.apache.org/docs/2.4/programs/ab.html):

```
ab -n 100000 -c 100 http://$(terraform output alb_dns_name)
```