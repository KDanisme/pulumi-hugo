---
title: ECS Hosted Installer
menu:
    userguides:
        parent: self_hosted
        identifier: self_hosted_ecshosted_installer
        weight: 5
meta_desc: Installer for deploying the self-hosted Pulumi service in ECS.
---

# ECS-Hosted Installer

The "ECS-hosted" installer is used to deploy the self-hosted Pulumi service in Amazon Elastic Container Service.  

# Prerequisites

The customer is required to provide and manage the following:

* AWS VPC with
  * At least 2 public subnets available.
  * At least 2 private subnets available.
  * At least 2 isolated subnets available.
    * An "isolated" subnet means it can only route traffic within the subnet. So there is no NAT gateway.
* Route53 hosted zone.
* ACM Certiciate that covers FQDNs of the form (note `{subdomain}` is optional):
  * `{subdomain}.{zoneDomainName}` 
  * `api.{subdomain}.{zoneDomainName}` 
  * `app.{subdomain}.{zoneDomainName}` 
* KMS key to be used the self-hosted Pulumi Service for encryption/decryption purposes.

## ECS-Hosted Deployment

The ECS-hosted installation of Pulumi deploys the following services:

* ECS - Managed ECS Cluster
* Fargate - Managed Container Service
* RDS Aurora - Managed MySQL DB for persistent state and automated replication and snapshotting.
* S3 - Object storage for checkpoints and policy packs.
* CloudWatch Logs - Centralized logging for all cluster pods.
* Route53 - Managed DNS records.
* NLB - Managed L4/application traffic and TLS termination.
* ACM - Managed Public TLS certificates.

### Pulumi deploying Pulumi

This installer uses Pulumi to deploy the Pulumi service. 
In this case, one uses the pulumi CLI with a self-managed backend (e.g. S3) to deploy all services listed above to stand up the self-hosted Pulumi Service.  
The installation package includes Pulumi project code so that you can deploy the service by running `pulumi up`. 

To this end, you need to set up the following:
* [Download and install the Pulumi CLI]({{<relref "docs/get-started/install">}}) on the docker server. 
* [Login to S3-compatible backend]({{<relref "docs/intro/concepts/state#logging-into-the-aws-s3-backend">}})

### Deployment Steps

See the README.md file provided with the installer package for detailed deployment steps.

## ECS-Hosted System Management and Maintenance

### Pulumi Service Updates

When deploying the service, it is recommended to pin the Pulumi Service image tag to a specific version. See the installer's README file for how to set the `imageTag` configuration property for the installer to use.

When ready to update the Pulumi Sevice containers to use a different version, do the following:
* `pulumi login` to the self-managed (not self-hosted) backend as chosen above when installing the self-hosted service.
* `pulumi config set imageTag {image tag}` to set the version you want to use.
* `pulumi up` to deploy the updates. 

### Database Maintenance

The installer configures the RDS backend database for replication and checkpointing. So no additional maintenance is needed by the customer.

### Blob Storage Mataintenance

The service automatically creates backups of checkpoint files. However the customer may want to enable AWS Backup to periodically backup the S3 buckets created by the installer.  
The buckets will have names of the form:
* `pulumi-checkpoint-XXX`
* `pulumi-policy-XXX`