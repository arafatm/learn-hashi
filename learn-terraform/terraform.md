# Learn Terraform

- [ ] [learn-terraform](learn-terraform/terraform.md)
  - https://learn.hashicorp.com/terraform
  - https://console.aws.amazon.com/console/home

## Introduction to Infrastructure as Code with Terraform

> building, changing, and managing infrastructure in a safe, repeatable way.

### Infrastructure as Code

> the process of managing infrastructure in a file or files rather than
> manually configuring resources in a user interface. 

### Workflows

- **Scope** 
- **Author** 
- **Initialize** 
- **Plan & Apply** 

### Advantages of Terraform

- Platform Agnostic
- State Management
- Operator Confidence

## Installing Terraform

https://learn.hashicorp.com/terraform/getting-started/install

[Download & install](https://www.terraform.io/downloads.html)

### Quick start tutorial: Provision Nginx with docker

    mkdir terraform-docker-demo && cd $_

Install docker

:ship: install docker using snap
```bash
sudo snap install docker

 # Create and join the docker group.
sudo addgroup --system docker
sudo adduser $USER docker
newgrp docker

 # restart docker
sudo snap disable docker
sudo snap enable docker<Paste>

 # test docker
docker ps
```

:ship: Paste the following into a file named [main.tf](/code.terraform.docker.demo/main.tf)
```terraform
resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "tutorial"
  ports {
    internal = 80
    external = 80
  }
}
```

:ship: build the container
```bash
terraform init

terraform apply
```

[verify nginx is running](http://localhost) or `docker ps`

:ship: destroy the nginx container 
```bash
terraform destroy
```

### Getting Help

:ship: help commands
```bash
terraform -help

terraform --help <command>
```
## Build Infrastructure 

> Start creating some infrastructure.

### Overview

Terraform can 
[manage many providers](https://www.terraform.io/docs/providers/index.html)

Some example [use cases](https://www.terraform.io/intro/use-cases.html).

Signup for [free AWS account](https://aws.amazon.com/free/)

### Configuration

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

### Providers

The `provider` block is used to configure the named provider.

A provider is a plugin that Terraform uses to translate the API interactions
with the service e.g. aws.

### Resources

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

### Initialization


:ship: terraform init to initialize local settings and data. Including **plugins**
```bash
terraform init
```

### Formatting and Validating Configurations


:ship: `terraform fmt` to check language consistency
```bash
terraform fmt
```


:ship: `terraform validate` to check for errors
```bash
terraform validate
```

### Apply Changes


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

### Manually Managing State

:ship: For advanced state management
```bash
terraform state
```

[CLI state command documentation](https://www.terraform.io/docs/commands/state/index.html)

### Provisioning

At this point the AMI has not been provisioned.

### Configuration

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

### Apply Changes

:ship: apply the change
```bash
terraform apply
```
## Destroy Infrastructure 

### Destroy

:ship: Terminate resources defined 
```bash
terraform destroy
```
    
Just like with `apply`, Terraform determines the order in which things must be
destroyed, in a suitable order to respect dependencies.
## Resource Dependencies | Terraform - HashiCorp Learn

A basic example of multiple resources and how to reference the attributes of
other resources to configure subsequent resources.

### Assigning an Elastic IP

:ship: Assign an [elastic
IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)
```terraform
resource "aws_eip" "ip" {
    vpc = true
    instance = aws_instance.example.id 
               # :point_up: generated by resource "aws_instance" "example"
}
```

### Apply Changes


:ship: 
```bash
terraform apply
```

### Implicit and Explicit Dependencies

:exclamation: The `aws_instance` was created before the `aws_eip`. TF is able
to infer the **implicit** dependency due to the reference to
`aws_instance.example.ed`

When a dependency cannot be inferred, use `depends_on` to explictly create it.

For example, perhaps an application we will run on our EC2 instance expects to
use a specific Amazon S3 bucket, but that dependency is configured inside the
application code and thus not visible to Terraform. In that case, we can use
`depends_on` to explicitly declare the dependency:


:ship: Example explicit dependency: The `aws_s3_bucket` is configured in
application code and not visible to TF

```terraform
resource "aws_s3_bucket" "example" {
  bucket = "terraform-getting-started-guide"
  acl    = "private"
}

resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"
  
  depends_on = [aws_s3_bucket.example]
}
```
## Provision

Provisioners let you upload files, run shell scripts, or install and trigger
other software like configuration management tools, etc.

In general you manage image based infra using TF. Images can be built with
[Packer](https://www.packer.io/)), 

### Defining a Provisioner

To define a provisioner, modify the resource block defining the "example" EC2
instance to look like the following:

:ship: create `ip_address.txt` using `local-exec` provisioner
```terraform
resource "aws_instance" "example" {
  ami           = "ami-b374d5a5"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo ${aws_instance.example.public_ip} > ip_address.txt"
  }
}
```

:ship: verify `local-exec` ran

    cat ip_address.txt

Terraform supports [multiple
provisioners](https://www.terraform.io/docs/provisioners/index.html)

Another useful provisioner is `remote-exec` 


:ship: Create an ssh key with no passphrase with `ssh-keygen -t rsa` and use
the name `terraform.` Update the permissions of that key with `chmod 400
~/.ssh/terraform`.

:warning: This example is for reference and should not be used without testing.
If you are running this, create a new Terraform project folder for this
example.

    provider "aws" {
      profile = "default"
      region  = "us-west-2"
    }
    
    resource "aws_key_pair" "example" {
      key_name   = "examplekey"
      public_key = file("~/.ssh/terraform.pub")
    }
    
    resource "aws_instance" "example" {
      key_name      = aws_key_pair.example.key_name
      ami           = "ami-04590e7389a6e577c"
      instance_type = "t2.micro"
    
      connection {
        type        = "ssh"
        user        = "ec2-user"
        private_key = file("~/.ssh/terraform")
        host        = self.public_ip
      }
    
      provisioner "remote-exec" {
        inline = [
          "sudo amazon-linux-extras enable nginx1.12",
          "sudo yum -y install nginx",
          "sudo systemctl start nginx"
        ]
      }
    }
    
### Failed Provisioners and Tainted Resources

If a resource is created but fails at provisioning it is marked _tainted_

Terraform will remove any tainted resources and create new resources,
attempting to provision them again after creation.

### Manually Tainting Resources

In cases where you want to manually destroy and recreate a resource. 

Given this resource
:ship: 
```terraform
resource "aws_instance" "example" {
  ami           = "ami-b374d5a5"
  instance_type = "t2.micro"
}
```

:ship: manually taint a resource
```bash
    terraform taint aws_instance.example
```

### Destroy Provisioners

Provisioners can also be defined that run only during a destroy operation.
These are useful for performing system cleanup, extracting data, etc.

[see the provisioner documentation](https://www.terraform.io/docs/provisioners)
## Input Variables 

### Defining Variables

:exclamation: The file can be named anything, since Terraform loads all files
in the directory ending in `.tf`.

:ship:  Create `variables.tf`
```terraform
variable "region" {
  default = "us-east-1"
}
```

:ship: ship it 
```bash
terraform plan
terraform apply
```

### Using Variables in a Configuration

:ship: Use the defined `region` variable in `example.tf` 
```terraform
provider "aws" {
  region = var.region
}
```
    
### Assigning variables

There are multiple ways to assign variables. The order below is also the order
in which variable values are chosen.
- command-line flags
- from a file
- from env variables
- ui input

#### Command-line flags

:ship: using `-var`
```bash
terraform apply -var 'region=us-east-1'
```

#### From a file

To persist variable values, create a file and assign variables within this
file. Create a file named `terraform.tfvars` with the following contents:

:ship: `terraform.tfvars`
```terraform
region = "us-east-1"
```

TF loads `terraform.tfvars` or `*.auto.tfvars`

Can specify `-var-file` for custom files.

:ship: Use `secret.tfvars` for secrets like username/password
```bash
terraform apply \
  -var-file="secret.tfvars" \
```
    
:ship: Use `<env>.tfvars` to provision test/stage/prod
```bash
terraform apply \
  -var-file="production.tfvars"
```

#### From environment variables

TF can read `TF_VAR_<name>` environment variables

e.g. `TF_VAR_region`

#### UI input

If you execute `terraform apply` with any variable unspecified, Terraform will
ask you to input the values interactively. 

#### Variable defaults

If no value is assigned to a variable via any of these methods and the variable
has a `default` key in its declaration, that value will be used for the
variable.

### Rich data types

Data Types: strings, numbers, lists, maps (hashtable or dictionary)

#### Lists


:ship: 
```terraform

variable "cidrs" { default = [] }

variable "cidrs" { type = list }

cidrs = [ "10.0.0.0/16", "10.1.0.0/16" ]
```
    
#### Maps

:ship: example map 
```terraform
variable "amis" {
  type = "map"
  default = {
    "us-east-1" = "ami-b374d5a5"
    "us-west-2" = "ami-4b32be2b"
  }
}

resource "aws_instance" "example" {
  ami           = var.amis[var.region]
  instance_type = "t2.micro"
}
```

:ship: Map values can also be set using the `-var` and `-var-file` values.
```bash
terraform apply -var 'amis={ us-east-1 = "foo", us-west-2 = "bar" }'
```
    
### Next steps

For other examples, see the API documentation.

- [Input variables API doc](https://www.terraform.io/docs/configuration/variables.html)
- [Local variables API
  doc](https://www.terraform.io/docs/configuration/locals.html)> Organize your
  data for easier queries with outputs.
## Output Variables | Terraform - HashiCorp Learn

When building potentially complex infrastructure, Terraform stores hundreds or
thousands of attribute values for all your resources. But as a user of
Terraform, you may only be interested in a few values of importance, such as a
load balancer IP, VPN address, etc.

Outputs are a way to tell Terraform what data is important. This data is
outputted when `apply` is called, and can be queried using the `terraform
output` command.

### Defining Outputs

Let's define an output to show us the public IP address of the elastic IP
address that we create. Add this to any of your `*.tf` files:

:ship: This defines an output variable named `ip`. The `public_ip` attribute is
exposed by the resource. 
```terraform
output "ip" {
  value = aws_eip.ip.public_ip
}
```
    
[attributes
reference](https://www.terraform.io/docs/providers/aws/r/eip.html#attributes-reference)

### Viewing Outputs

Output will be displayed after `terraform apply` or can explicitly query with `terraform output`
## Modules | Terraform - HashiCorp Learn

_Modules_ are used to create reusable components, improve organization, and to
treat pieces of infrastructure as a black box.

[modules documentation](https://www.terraform.io/docs/modules/index.html).

:warning: The examples on this page are _**not** eligible_ for [the AWS free
tier](https://aws.amazon.com/free/). Do not try the examples on this page
unless you're willing to spend a small amount of money.

### Using Modules

:caution: If you have any instances running from prior steps in the getting
started guide, use `terraform destroy` to destroy them, and remove all
configuration files.

The [Terraform Registry](https://registry.terraform.io/) includes a directory
of ready-to-use modules for various common purposes, which can serve as larger
building-blocks for your infrastructure.

In this example, we're going to use [the Consul Terraform module for
AWS](https://registry.terraform.io/modules/hashicorp/consul/aws), which will
set up a complete [Consul](https://www.consul.io/) cluster. 

:ship: 
```terraform
terraform {
  required_version = "0.11.11"
}

provider "aws" {
  access_key = "AWS ACCESS KEY"
  secret_key = "AWS SECRET KEY"
  region     = "us-east-1"
}

module "consul" {
  source      = "hashicorp/consul/aws"
  num_servers = "3"
}
```

The `module` block begins with the example given on the Terraform Registry page
for this module, telling Terraform to create and manage this module. This is
similar to a `resource` block: it has a name used within this configuration --
in this case, `"consul"` -- and a set of input values that are listed in [the
module's "Inputs"
documentation](https://registry.terraform.io/modules/hashicorp/consul/aws?tab=inputs).


The `source` attribute is the only mandatory argument for modules. 

### Module Versioning

After adding a new module to configuration, it is necessary to run (or re-run)
`terraform init` to obtain and install the new module's source code:

:ship: Run `init` to install new modules
```bash
terraform init
```

`-upgrade` will check for any newer versions 

:flashlight: Explicitly constrain the acceptable version numbers for each
external module to avoid unexpected or unwanted changes.

:ship: Use the version attribute in the module block to specify versions:
```terraform
module "consul" {
  source  = "hashicorp/consul/aws"
  version = "0.7.3"

  servers = 3
}
```

### Apply Changes

:ship: Apply
```bash
terraform apply
```

The `module.consul.module.consul_clients` prefix shown above indicates not only
that the resource is from the `module "consul"` block we wrote, but in fact
that this module has its own `module "consul_clients"` block within it. Modules
can be nested to decompose complex systems into manageable components.

### Module Outputs

[The module's outputs
reference](https://registry.terraform.io/modules/hashicorp/consul/aws?tab=outputs)
describes all of the different values it produces. 

:ship: `asg_name_servers` is the name of the auto-scaling group that was
created to manage the Consul servers.
```terraform
output "consul_server_asg_name" {
  value = "${module.consul.asg_name_servers}"
}
```
    
:flashlight: If you look in the Auto-scaling Groups section of the EC2 console
you should find an autoscaling group of this name, and from there find the
three Consul servers it is running. (If you can't find it, make sure you're
looking in the right region!)

## Remote State Storage | Terraform

Terraform supports team-based workflows with a feature known as [remote
backends](https://www.terraform.io/docs/backends/index.html). 

Terraform has multiple remote backend options. HashiCorp recommends using
[Terraform
Cloud](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/terraform/cloud-gettingstarted/tfc_signup).

### How to Store State Remotely

 [sign up
here](https://app.terraform.io/signup) for this guide. 

For more information on Terraform Cloud, [view our getting started
guide](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/terraform/cloud-gettingstarted/tfc_signup).

When you sign up for Terraform Cloud, you'll create an organization. 

_Make a note of the organization's name._

:ship: Configure the backend in your configuration with the organization name,
and a new workspace name of your choice:
```terraform
terraform {
  backend "remote" {
    organization = "<ORG_NAME>"

    workspaces {
      name = "Example-Workspace"
    }
  }
}
```

You'll also need a user token to authenticate with Terraform Cloud. You can
generate one on the [user settings
page](https://app.terraform.io/app/settings/tokens):

![User
Token](https://d33wubrfki0l68.cloudfront.net/bf6f7a5a7cd9c90d976bc2139d2ad3c9e6a94c06/b53d3/img/tf-cloud-user-token.png
"User Token")

:ship: copy user token into `~/.terraformrc`  
```terraform
credentials "app.terraform.io" {
  token = "REPLACE_ME"
}
```
    
[CLI config doc](https://www.terraform.io/docs/commands/cli-config.html).

Now that you've configured your remote backend, run `terraform init` to setup
Terraform. It should ask if you want to migrate your state to Terraform Cloud.


:ship: init with new cloud setup
```bash
terraform init

terraform apply
```

_Terraform is now storing your state remotely in Terraform Cloud_

### Terraform Cloud

[Terraform Cloud](https://www.hashicorp.com/products/terraform/pricing) offers
commercial solutions which combines a predictable and reliable shared run
environment with tools to help you work together on Terraform configurations
and modules.


For a hands-on introduction to Terraform Cloud, [follow the Terraform Cloud
getting started
guides](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/terraform?track=cloud-gettingstarted#cloud-gettingstarted)
for our free offering as well as Terraform Cloud for
[Teams](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/terraform?track=enterprise#enterprise)
and
[Governance](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/terraform?track=sentinel#sentinel).

## Next Steps

- [Documentation](https://www.terraform.io/docs/index.html) - The documentation
  is an in-depth reference guide to all the features of Terraform, including
  technical details about the internals of how Terraform operates.
- [Examples](https://www.terraform.io/intro/examples/index.html) - The examples
  have more full featured configuration files, showing some of the
  possibilities with Terraform.
- [Import](https://www.terraform.io/docs/import/index.html) - The import
  section of the documentation covers importing existing infrastructure into
  Terraform.
