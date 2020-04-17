# Build Infrastructure 

> Start creating some infrastructure.

## Overview

Terraform can 
[manage many providers](https://www.terraform.io/docs/providers/index.html)

Some example [use cases](https://www.terraform.io/intro/use-cases.html).

Signup for [free AWS account](https://aws.amazon.com/free/)

## Configuration

:ship: new project
```bash

mkdir learn-terraform-aws-instance
    
cd learn-terraform-aws-instance
```

:ship: `example.tf` to configure aws instance
```terraform
provider "aws" {
  profile    = "default"
  region     = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}
```

:warning: The `profile` attribute here refers to the AWS Config File in
`~/.aws/credentials` 
    
[Complete configuration file
documentation](https://www.terraform.io/docs/configuration/index.html).

To verify an AWS profile and ensure Terraform has correct provider
credentials, install the [AWS
CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

[Create IAM User](https://console.aws.amazon.com/iam/home#/home)

:ship: Configure creds for AWS
```bash
aws configure
```

[AWS Console for
Credentials](https://console.aws.amazon.com/iam/home?#security_credential).

## Providers

The `provider` block is used to configure the named provider.

A provider is a plugin that Terraform uses to translate the API interactions
with the service e.g. aws.

## Resources

The `resource` block defines a piece of infrastructure. A resource might be a
physical component such as an EC2 instance, or it can be a logical resource
such as a Heroku application.

The resource block has two strings before the block: 
1. the resource type 
2. the resource name. 

In the example, the resource type is `aws_instance` and the name is `example`.

The prefix of the type maps to the provider. In our case "aws_instance"
automatically tells Terraform that it is managed by the "aws" provider.

See [providers reference](https://www.terraform.io/docs/providers/index.html)

## Initialization


:ship: terraform init to initialize local settings and data. Including **plugins**
```bash
terraform init
```

## Formatting and Validating Configurations


:ship: `terraform fmt` to check language consistency
```bash
terraform fmt
```


:ship: `terraform validate` to check for errors
```bash
terraform validate
```

## Apply Changes


:ship: `terraform version` to check we're using required 0.11+ for this tutorial
```bash
terraform version
```

:ship: `terraform plan` for a dry run without applying
```bash
terraform plan
```

:ship: `terraform apply` to show the execution plan
```bash
terraform apply
```

[Verify your running in the EC2
console](https://us-east-1.console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home:)
    
:exclamation: Terraform writes data ino `terraform.tfstate` file. 
- This file keeps track of IDs of created resources.
- Save this file and distribute to your team

See [doc: setup remote state](https://www.terraform.io/docs/state/remote.html)
to share state automatically.

:ship: To inspect the current state 
```bash
terraform show
```

## Manually Managing State

:ship: For advanced state management
```bash
terraform state
```

[CLI state command documentation](https://www.terraform.io/docs/commands/state/index.html)

## Provisioning

At this point the AMI has not been provisioned.

## Configuration

Let's modify the `ami` of our instance. Edit the `aws_instance.example`
resource under your provider block in your configuration and change it to the
following:

:ship: Modify `aws_instance.example` to used Ubuntu 16.10 instead of 16.04 AMI 
```
provider "aws" {
  profile    = "default"
  region     = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-b374d5a5"
  instance_type = "t2.micro"
}
```    

Find more [Public and Private AMIs
here](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Images:visibility=public-images;search=ubuntu,16.10,canonical;sort=name)

## Apply Changes

:ship: apply the change
```bash
terraform apply
```
