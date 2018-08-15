**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-ecs>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-ecs/blob/master/modules/ecs-service-with-discovery/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# ECS Service with discovery

This Terraform Module creates an [EC2 Container Service (ECS) Service][1] with service discovery.
You can use an ECS service to run one or more related, long-running Docker containers, such as a web service.

Many services are not guaranteed to have the same IP address through their lifespan. They can, for example, be dynamically assigned to run on different hosts, be redeployed after a failure recovery or scale in and out. This makes it complex for services to send traffic to each other.

Service discovery is the action of detecting and addressing these services, allowing them to be found. Some of the ways of doing service discovery are, for example, hardcoding IP addresses, using a Load Balancer or using specialized tools.

ECS *Service Discovery* is an AWS feature allows you to reach your ECS services through a hostname managed by Route53. This hostname will consist of a service discovery name and a namespace (private or public), in the shape of `discovery-name.namespace:port`. For example, on our namespace `sandbox.gruntwork.io`, we can have a service with the discovery name `my-test-webapp` running on port `3000`. This means that we can `dig` or `curl` this service at `my-test-webapp.sandbox.gruntwork.io:3000`. For more information see the [related concepts](#related-concepts) section.

There are many advantages of using ECS Service Discovery instead of reaching it through a Load Balancer, for example:
* Direct communication with the container run by your service
* Lower latency, if using AWS internal network and private namespace
* You can do service-to-service authentication
* Not having a Load Balancer also means fewer resources to manage
* You can configure a Health Check and associate it with all records within a namespace
* You can make a logical group of services under one namespace

**If you wish to deploy instead an ECS Service with an Application Load Balancer (ALB), see the [ecs-service-with-alb module](../ecs-service-with-alb).**

## How do you use this module?

* See the [root README](/README.md) for instructions on using Terraform modules.
* See [vars.tf](./vars.tf) for all the variables you can set on this module.
* This module assumes you have already deployed:
  * An ECS Cluster: See the [ecs-cluster module](../ecs-cluster) for how to run one.
  * [A service discovery DNS namespace](#route-53-auto-naming-service)
* See the [examples](/examples) folder for example usage.


## Gotchas

* The ECS Service Discovery feature is not yet available in all regions.
For a list of regions where this feature is enabled, please see the [AWS ECS Service Discovery documentation][2].
* The discovery name is not necessarily the same as the name of your service. You can have a different name by which you want to discover your service.
* You can enable ECS Service Discovery only during the creation of your ECS service, not when updating it.
* The network mode of the task definition affects the behavior and configuration of ECS Service Discovery DNS Records.
    * Service discovery with `SRV` DNS records are not yet supported by this module. This means that tasks defined with with `host` or `bridge` network modes that can only be used with this type of record are also not supported.
    * For enabling service discovery, this module uses the `awsvpc` network mode. AWS will attach an Elastic Network Interface to your task, so you have to be aware that EC2 instance types have a [limit of how many ENIs can be attached to them][3].

## Related Concepts

### ECS clusters

See the [ecs-cluster module](../ecs-cluster).

### ECS services and tasks

See the [ecs-service module](../ecs-service).

### Route 53 Auto Naming Service

Amazon Route 53 auto naming service automates the process of:
* Creating a public or private namespace within a new or existing hosted zone
* Providing a service with the DNS Records configuration and optional health checks

The latter will be used in the Service Registry of your ECS Service Discovery, and it is the only type of service currently supported for this.

Important considerations:
* Public namespaces are accessible on the internet and need the domain to be registered already
* Private namespaces are accessible only within your VPC and can be queried immediately
* For cleaning up, deregistering the instances from the auto naming service will trigger an automatic deletion of resources in AWS. However, the namespaces themselves are not deleted. Namespaces must be deleted manually and that is only allowed once all services in that namespace no longer exist.

For more information on Route 53 Auto Naming Service, please see the AWS documentation on [Using Auto Naming for Service Discovery][4].

[1]:http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html
[2]:https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-service-discovery.html
[3]:https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI
[4]:https://docs.aws.amazon.com/Route53/latest/APIReference/overview-service-discovery.html
