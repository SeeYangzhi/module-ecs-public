**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-ecs>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-ecs/blob/master/examples/docker-service-with-public-discovery/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Docker Service with Service Discovery using a public DNS hostname

This example demonstrates how to set up an ECS cluster and then deploy an ECS service which can register
itself with [AWS Service Discovery](https://aws.amazon.com/blogs/aws/amazon-ecs-service-discovery/) using a public hostname.
This allows querying the hostname trough the public internet. It is important to notice that the service is still
private and the DNS query will return private IPs, only ECS Fargate supports auto assigning public IPs to your containers.


## How do you run this example?

To run this example, you need to do the following:

1. Have a registered domain on Route 53
1. Build the AMI
1. Apply the Terraform templates


### Have a registered domain on Route 53

You can either register a new domain or transfer an existing domain to AWS using [AWS Route 53](https://aws.amazon.com/route53/),
creating a public hosted zone for your domain.

### Build the AMI

See the [example-ecs-instance-ami docs](/examples/example-ecs-instance-ami).

### Apply the Terraform templates

To apply the Terraform templates:

1. Install [Terraform](https://www.terraform.io/).
1. Open `vars.tf`, set the environment variables specified at the top of the file, and fill in any other variables that
   don't have a default. This includes setting the `ecs_cluster_instance_ami` with the ID of the AMI you just built
   and `original_public_route53_zone_id` with the hosted zone id of the public hostname you want to use as a namespace for your service.
1. Run `terraform init`.
1. Run `terraform apply`, it will give you a plan and request if you wish to proceed.

#### Resources

1. [AWS ECS Service Discovery guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-discovery.html#create-service-discovery)
1. [ecs-service-with-discovery module](/modules/ecs-service-with-discovery)
