https://learn.hashicorp.com/terraform

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

:ship: Paste the following into a file named [main.tf](terraform-docker-demo/main.tf)
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

xxx

> Start creating some infrastructure.

### Overview

With Terraform installed, let's dive right into it and start creating some
infrastructure.

We'll build infrastructure on [AWS](https://aws.amazon.com/) for the getting
started guide since it is popular and generally understood, but Terraform can
[manage many providers](https://www.terraform.io/docs/providers/index.html),
including multiple providers in a single configuration. 

Some examples of this are in the [use cases
section](https://www.terraform.io/intro/use-cases.html).

If you don't have an AWS account, [create one
now](https://aws.amazon.com/free/). For the getting started guide, we'll only
be using resources which qualify under the AWS
[free-tier](https://aws.amazon.com/free/), meaning it will be free. If you
already have an AWS account, you may be charged some amount of money, but it
shouldn't be more than a few dollars at most.

:warning: **Warning!** If you're not using an account that qualifies under the
AWS [free-tier](https://aws.amazon.com/free/), you may be charged to run these
examples. The most you should be charged should only be a few dollars, but
we're not responsible for any charges that may incur.

### Configuration

The set of files used to describe infrastructure in Terraform is known as a
Terraform _configuration_. You'll write your first configuration now to launch
a single AWS EC2 instance.

A configuration should be in its own directory. Create a directory for the new
configuration.

    $ mkdir learn-terraform-aws-instance
    
Change into the directory.

    $ cd learn-terraform-aws-instance
    
Create a file for the configuration code.

    $ touch example.tf

The format of the configuration files is [documented
here](https://www.terraform.io/docs/configuration/index.html).

Paste the configuration below into `example.tf` and save it. Later in the
guide when you run Terraform, it will load all files in the working directory
that end in `.tf`.

    provider "aws" {
      profile    = "default"
      region     = "us-east-1"
    }
    
    resource "aws_instance" "example" {
      ami           = "ami-2757f631"
      instance_type = "t2.micro"
    }
    
**Note**: The above configuration is designed to work on most EC2 instances
with access to a default VPC. If your configuration fails to apply, refer to
the [troubleshooting](#troubleshooting) section at the bottom of this guide.

The `profile` attribute here refers to the AWS Config File in
`~/.aws/credentials` on MacOS and Linux or `%UserProfile%\.aws\credentials` on
a Windows system. It is HashiCorp recommended practice that credentials
_never_ be hardcoded into `*.tf` configuration files. We are explicitly
defining the `default` AWS config profile here to illustrate how Terraform
accesses sensitive credentials.

To verify an AWS profile and ensure Terraform has correct provider
credentials, install the [AWS
CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
and run `aws configure`. The AWS CLI will then verify and save your AWS Access
Key ID and Secret Access Key. Those credentials are found on [this
page](https://console.aws.amazon.com/iam/home?#security_credential).

**Note**: If you leave out your AWS credentials, Terraform will automatically
search for saved API credentials (for example, in `~/.aws/credentials`) or IAM
instance profile credentials. This option is much cleaner for situations where
tf files are checked into source control or where there is more than one admin
user. See details
[here](https://aws.amazon.com/blogs/apn/terraform-beyond-the-basics-with-aws/).
Leaving IAM credentials out of the Terraform configs allows you to leave those
credentials out of source control, and also use different IAM credentials for
each user without having to modify the configuration files.

This is a complete configuration that Terraform is ready to apply.

### Providers

The `provider` block is used to configure the named provider, in our case
"aws". A provider is responsible for creating and managing resources. A
provider is a plugin that Terraform uses to translate the API interactions
with the service. A provider is responsible for understanding API interactions
and exposing resources. Because Terraform can interact with any API, almost
any infrastructure type can be represented as a resource in Terraform.

Multiple provider blocks can exist if a Terraform configuration manages
resources from different providers. You can even use multiple providers
together. For example you could pass the ID of an AWS instance to a monitoring
resource from DataDog.

### Resources

The `resource` block defines a piece of infrastructure. A resource might be a
physical component such as an EC2 instance, or it can be a logical resource
such as a Heroku application.

The resource block has two strings before the block: the resource type and the
resource name. In the example, the resource type is "aws\_instance" and the
name is "example." The prefix of the type maps to the provider. In our case
"aws\_instance" automatically tells Terraform that it is managed by the "aws"
provider.

The arguments for the resource are within the resource block. The arguments
could be things like machine sizes, disk image names, or VPC IDs. Our
[providers reference](https://www.terraform.io/docs/providers/index.html)
documents the required and optional arguments for each resource provider. For
your EC2 instance, you specified an AMI for Ubuntu, and requested a "t2.micro"
instance so you qualify under the free tier.

### Initialization

The first command to run for a new configuration -- or after checking out an
existing configuration from version control -- is `terraform init`. Subsequent
commands will use local settings and data that are initialized by `terraform
init`.

Terraform uses a plugin-based architecture to support hundreds of
infrastructure and service providers. The `terraform init` command downloads
and installs providers used within the configuration, which in this case is
the `aws` provider.

    $ terraform init
    Initializing the backend...
    
    Initializing provider plugins...
    - Checking for available provider plugins...
    - Downloading plugin for provider "aws" (terraform-providers/aws) 2.10.0...
    
    The following providers do not have any version constraints in configuration,
    so the latest version was installed.
    
    To prevent automatic upgrades to new major versions that may contain breaking
    changes, it is recommended to add version = "..." constraints to the
    corresponding provider blocks in configuration, with the constraint strings
    suggested below.
    
    * provider.aws: version = "~> 2.10"
    
    Terraform has been successfully initialized!
    
    You may now begin working with Terraform. Try running "terraform plan" to see
    any changes that are required for your infrastructure. All Terraform commands
    should now work.
    
    If you ever set or change modules or backend configuration for Terraform,
    rerun this command to reinitialize your working directory. If you forget, other
    commands will detect it and remind you to do so if necessary.
    
Terraform downloads the `aws` provider and installs it in a hidden subdirectory
of the current working directory.

The output shows which version of the plugin was installed.

### Formatting and Validating Configurations

To follow style conventions, we recommend language consistency between files
and modules written by different teams. The `terraform fmt` command enables
standardization which automatically updates configurations in the current
directory for easy readability and consistency.

If you are copying configuration snippets or just want to make sure your
configuration is syntactically valid and internally consistent, the built in
`terraform validate` command will check and report errors within modules,
attribute names, and value types.

### Apply Changes

**Note:** The commands shown in this guide apply to Terraform 0.11 and above.
Earlier versions require using the `terraform plan` command to see the
execution plan before applying it. Use `terraform version` to confirm your
running version.

In the same directory as the `example.tf` file you created, run `terraform
apply`. You should see output similar to below, though we've truncated some of
the output to save space.

    $ terraform apply
    # ...
    
    An execution plan has been generated and is shown below.
    Resource actions are indicated with the following symbols:
      + create
    
    Terraform will perform the following actions:
    
      # aws_instance.example will be created
      + resource "aws_instance" "example" {
          + ami                          = "ami-2757f631"
          + arn                          = (known after apply)
          + associate_public_ip_address  = (known after apply)
          + availability_zone            = (known after apply)
          + cpu_core_count               = (known after apply)
          + cpu_threads_per_core         = (known after apply)
          + get_password_data            = false
          + host_id                      = (known after apply)
          + id                           = (known after apply)
          + instance_state               = (known after apply)
          + instance_type                = "t2.micro"
          + ipv6_address_count           = (known after apply)
          + ipv6_addresses               = (known after apply)
          + key_name                     = (known after apply)
          + network_interface_id         = (known after apply)
          + password_data                = (known after apply)
          + placement_group              = (known after apply)
          + primary_network_interface_id = (known after apply)
          + private_dns                  = (known after apply)
          + private_ip                   = (known after apply)
          + public_dns                   = (known after apply)
          + public_ip                    = (known after apply)
          + security_groups              = (known after apply)
          + source_dest_check            = true
          + subnet_id                    = (known after apply)
          + tenancy                      = (known after apply)
          + volume_tags                  = (known after apply)
          + vpc_security_group_ids       = (known after apply)
    
          + ebs_block_device {
              + delete_on_termination = (known after apply)
              + device_name           = (known after apply)
              + encrypted             = (known after apply)
              + iops                  = (known after apply)
              + snapshot_id           = (known after apply)
              + volume_id             = (known after apply)
              + volume_size           = (known after apply)
              + volume_type           = (known after apply)
            }
    
          + ephemeral_block_device {
              + device_name  = (known after apply)
              + no_device    = (known after apply)
              + virtual_name = (known after apply)
            }
    
          + network_interface {
              + delete_on_termination = (known after apply)
              + device_index          = (known after apply)
              + network_interface_id  = (known after apply)
            }
    
          + root_block_device {
              + delete_on_termination = (known after apply)
              + iops                  = (known after apply)
              + volume_id             = (known after apply)
              + volume_size           = (known after apply)
              + volume_type           = (known after apply)
            }
        }
    
    Plan: 1 to add, 0 to change, 0 to destroy.
    
This output shows the _execution plan_, describing which actions Terraform will
take in order to change real infrastructure to match the configuration. The
output format is similar to the diff format generated by tools such as Git. The
output has a `+` next to `aws_instance.example`, meaning that Terraform will
create this resource. Beneath that, it shows the attributes that will be set.
When the value displayed is `(known after apply)`, it means that the value
won't be known until the resource is created.

If `terraform apply` failed with an error, read the error message and fix the
error that occurred. At this stage, it is likely to be a syntax error in the
configuration.

If the plan was created successfully, Terraform will now pause and wait for
approval before proceeding. If anything in the plan seems incorrect or
dangerous, it is safe to abort here with no changes made to your
infrastructure. In this case the plan looks acceptable, so type `yes` at the
confirmation prompt to proceed.

Executing the plan will take a few minutes since Terraform waits for the EC2
instance to become available.

    # ...
    aws_instance.example: Creating...
    aws_instance.example: Still creating... [10s elapsed]
    aws_instance.example: Creation complete after 1m50s [id=i-0bbf06244e44211d1]
    
    Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
    
After this, Terraform is all done! You can go to the EC2 console to see the
created EC2 instance. (Make sure you're looking at the same region that was
configured in the provider configuration!)

Terraform also wrote some data into the `terraform.tfstate` file. This state
file is extremely important; it keeps track of the IDs of created resources so
that Terraform knows what it is managing. This file must be saved and
distributed to anyone who might run Terraform. It is generally recommended to
[setup remote state](https://www.terraform.io/docs/state/remote.html) when
working with Terraform, to share the state automatically, but this is not
necessary for simple situations like this Getting Started guide.

You can inspect the current state using `terraform show`.

    $ terraform show
    # aws_instance.example:
    resource "aws_instance" "example" {
        ami                          = "ami-2757f631"
        arn                          = "arn:aws:ec2:us-east-1:130490850807:instance/i-0bbf06244e44211d1"
        associate_public_ip_address  = true
        availability_zone            = "us-east-1c"
        cpu_core_count               = 1
        cpu_threads_per_core         = 1
        disable_api_termination      = false
        ebs_optimized                = false
        get_password_data            = false
        id                           = "i-0bbf06244e44211d1"
        instance_state               = "running"
        instance_type                = "t2.micro"
        ipv6_address_count           = 0
        ipv6_addresses               = []
        monitoring                   = false
        primary_network_interface_id = "eni-0f1ce5bdae258b015"
        private_dns                  = "ip-172-31-61-141.ec2.internal"
        private_ip                   = "172.31.61.141"
        public_dns                   = "ec2-54-166-19-244.compute-1.amazonaws.com"
        public_ip                    = "54.166.19.244"
        security_groups              = [
            "default",
        ]
        source_dest_check            = true
        subnet_id                    = "subnet-1facdf35"
        tenancy                      = "default"
        volume_tags                  = {}
        vpc_security_group_ids       = [
            "sg-5255f429",
        ]
    
        credit_specification {
            cpu_credits = "standard"
        }
    
        root_block_device {
            delete_on_termination = true
            iops                  = 100
            volume_id             = "vol-0079e485d9e28a8e5"
            volume_size           = 8
            volume_type           = "gp2"
        }
    }
    
You can see that by creating our resource, we've also gathered a lot of
information about it. These values can actually be referenced to configure
other resources or outputs, which will be covered later in the getting started
guide.

### Manually Managing State

Terraform has a built in command called `terraform state` which is used for
_advanced_ state management. In cases where a user would need to modify the
state file by finding resources in the `terraform.tfstate` file with `terraform
state list`. This will give us a list of resources as addresses and resource
IDs that we can then modify.

For more information about the `terraform state` command and subcommands for
moving or removing resources from state, see the [CLI state command
documentation](https://www.terraform.io/docs/commands/state/index.html). This
is outside the core Terraform workflow, but is worth noting as you learn how
state is managed.

### Provisioning

The EC2 instance we launched at this point is based on the AMI given, but has
no additional software installed. If you're running an image-based
infrastructure (perhaps creating images with [Packer](https://www.packer.io/)),
then this is all you need.

However, many infrastructures still require some sort of initialization or
software provisioning step. Terraform supports provisioners, which we'll cover
a little bit later in the getting started guide, in order to do this.

### Troubleshooting

* **If you use a region other than `us-east-1`**, please use an [AMI
  specific](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html#finding-quick-start-ami)
  to that region as AMI IDs are region specific.
* **If you do not have a default VPC in your AWS account in the `us-east-1`
  region**, create a new VPC in your VPC Dashboard in AWS. You will also need
  to associate a subnet and security group to that VPC. In your Terraform
  configuration, uncomment and modify the following two lines to your
  configuration for the rest of this track: `vpc_security_group_ids` (set as an
  array) and `subnet_id` with the corresponding information you just created.
  For more information, [review this
  document](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html)
  from AWS on working with VPCs.## Change Infrastructure 

> Modify existing resources.

In the previous page, you created your first infrastructure with Terraform: a
single EC2 instance. In this page, we're going to modify that resource, and see
how Terraform handles change.

Infrastructure is continuously evolving, and Terraform was built to help manage
and enact that change. As you change Terraform configurations, Terraform builds
an execution plan that only modifies what is necessary to reach your desired
state.

By using Terraform to change infrastructure, you can version control not only
your configurations but also your state so you can see how the infrastructure
evolved over time.

### Configuration

Let's modify the `ami` of our instance. Edit the `aws_instance.example`
resource under your provider block in your configuration and change it to the
following:

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

**Note:** EC2 Classic users please use AMI `ami-656be372` and type `t1.micro`

We've changed the AMI from being an Ubuntu 16.04 LTS AMI to being an Ubuntu
16.10 AMI. Terraform configurations are meant to be changed like this. You can
also completely remove resources and Terraform will know to destroy the old
one.

### Apply Changes

After changing the configuration, run `terraform apply` again to see how
Terraform will apply this change to the existing resources.

    $ terraform apply

    aws_instance.example: Refreshing state... [id=i-0bbf06244e44211d1]

    An execution plan has been generated and is shown below.
    Resource actions are indicated with the following symbols:
    -/+ destroy and then create replacement

    Terraform will perform the following actions:

     # aws_instance.example must be replaced
    -/+ resource "aws_instance" "example" {
      ~ ami                          = "ami-2757f631" -> "ami-b374d5a5" # forces replacement
      ~ arn                          = "arn:aws:ec2:us-east-1:130490850807:instance/i-0bbf06244e44211d1" -> (known after apply)
      ~ associate_public_ip_address  = true -> (known after apply)
      ~ availability_zone            = "us-east-1c" -> (known after apply)
      ~ cpu_core_count               = 1 -> (known after apply)
      ~ cpu_threads_per_core         = 1 -> (known after apply)
      - disable_api_termination      = false -> null
      - ebs_optimized                = false -> null
        get_password_data            = false
      + host_id                      = (known after apply)
      ~ id                           = "i-0bbf06244e44211d1" -> (known after apply)
      ~ instance_state               = "running" -> (known after apply)
        instance_type                = "t2.micro"
      ~ ipv6_address_count           = 0 -> (known after apply)
      ~ ipv6_addresses               = [] -> (known after apply)
      + key_name                     = (known after apply)
      - monitoring                   = false -> null
      + network_interface_id         = (known after apply)
      + password_data                = (known after apply)
      + placement_group              = (known after apply)
      ~ primary_network_interface_id = "eni-0f1ce5bdae258b015" -> (known after apply)
      ~ private_dns                  = "ip-172-31-61-141.ec2.internal" -> (known after apply)
      ~ private_ip                   = "172.31.61.141" -> (known after apply)
      ~ public_dns                   = "ec2-54-166-19-244.compute-1.amazonaws.com" -> (known after apply)
      ~ public_ip                    = "54.166.19.244" -> (known after apply)
      ~ security_groups              = [
          - "default",
        ] -> (known after apply)
        source_dest_check            = true
      ~ subnet_id                    = "subnet-1facdf35" -> (known after apply)
      ~ tenancy                      = "default" -> (known after apply)
      ~ volume_tags                  = {} -> (known after apply)
      ~ vpc_security_group_ids       = [
          - "sg-5255f429",
        ] -> (known after apply)

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + snapshot_id           = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_interface_id  = (known after apply)
        }

      ~ root_block_device {
          ~ delete_on_termination = true -> (known after apply)
          ~ iops                  = 100 -> (known after apply)
          ~ volume_id             = "vol-0079e485d9e28a8e5" -> (known after apply)
          ~ volume_size           = 8 -> (known after apply)
          ~ volume_type           = "gp2" -> (known after apply)
        }
    }

    Plan: 1 to add, 0 to change, 1 to destroy.
    
The prefix `-/+` means that Terraform will destroy and recreate the resource,
rather than updating it in-place. While some attributes can be updated in-place
(which are shown with the `~` prefix), changing the AMI for an EC2 instance
requires recreating it. Terraform handles these details for you, and the
execution plan makes it clear what Terraform will do.

Additionally, the execution plan shows that the AMI change is what required
your resource to be replaced. Using this information, you can adjust your
changes to possibly avoid destroy/create updates if they are not acceptable in
some situations.

Once again, Terraform prompts for approval of the execution plan before
proceeding. Answer `yes` to execute the planned steps:

    aws_instance.example: Destroying... [id=i-0bbf06244e44211d1]
    aws_instance.example: Still destroying... [id=i-0bbf06244e44211d1, 10s elapsed]
    aws_instance.example: Still destroying... [id=i-0bbf06244e44211d1, 20s elapsed]
    aws_instance.example: Still destroying... [id=i-0bbf06244e44211d1, 30s elapsed]
    aws_instance.example: Destruction complete after 31s
    aws_instance.example: Creating...
    aws_instance.example: Still creating... [10s elapsed]
    aws_instance.example: Still creating... [20s elapsed]
    aws_instance.example: Still creating... [30s elapsed]
    aws_instance.example: Creation complete after 38s [id=i-0589469dd150b453b]
    
    Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

As indicated by the execution plan, Terraform first destroyed the existing
instance and then created a new one in its place. You can use `terraform show`
again to see the new values associated with this instance.# Change Infrastructure | Terraform - HashiCorp Learn

> Modify existing resources.

In the previous page, you created your first infrastructure with Terraform: a
single EC2 instance. In this page, we're going to modify that resource, and see
how Terraform handles change.

Infrastructure is continuously evolving, and Terraform was built to help manage
and enact that change. As you change Terraform configurations, Terraform builds
an execution plan that only modifies what is necessary to reach your desired
state.

By using Terraform to change infrastructure, you can version control not only
your configurations but also your state so you can see how the infrastructure
evolved over time.

### Configuration

Let's modify the `ami` of our instance. Edit the `aws_instance.example`
resource under your provider block in your configuration and change it to the
following:

```terraform
    provider "aws" {
      profile    = "default"
      region     = "us-east-1"
    }
    
    resource "aws_instance" "example" {
      ami           = "ami-b374d5a5"
      instance_type = "t2.micro"
    }
```

**Note:** EC2 Classic users please use AMI `ami-656be372` and type `t1.micro`

We've changed the AMI from being an Ubuntu 16.04 LTS AMI to being an Ubuntu
16.10 AMI. Terraform configurations are meant to be changed like this. You can
also completely remove resources and Terraform will know to destroy the old
one.

### Apply Changes

After changing the configuration, run `terraform apply` again to see how
Terraform will apply this change to the existing resources.

    $ terraform apply
    aws_instance.example: Refreshing state... [id=i-0bbf06244e44211d1]
    
    An execution plan has been generated and is shown below.
    Resource actions are indicated with the following symbols:
    -/+ destroy and then create replacement
    
    Terraform will perform the following actions:
    
      # aws_instance.example must be replaced
    -/+ resource "aws_instance" "example" {
          ~ ami                          = "ami-2757f631" -> "ami-b374d5a5" # forces replacement
          ~ arn                          = "arn:aws:ec2:us-east-1:130490850807:instance/i-0bbf06244e44211d1" -> (known after apply)
          ~ associate_public_ip_address  = true -> (known after apply)
          ~ availability_zone            = "us-east-1c" -> (known after apply)
          ~ cpu_core_count               = 1 -> (known after apply)
          ~ cpu_threads_per_core         = 1 -> (known after apply)
          - disable_api_termination      = false -> null
          - ebs_optimized                = false -> null
            get_password_data            = false
          + host_id                      = (known after apply)
          ~ id                           = "i-0bbf06244e44211d1" -> (known after apply)
          ~ instance_state               = "running" -> (known after apply)
            instance_type                = "t2.micro"
          ~ ipv6_address_count           = 0 -> (known after apply)
          ~ ipv6_addresses               = [] -> (known after apply)
          + key_name                     = (known after apply)
          - monitoring                   = false -> null
          + network_interface_id         = (known after apply)
          + password_data                = (known after apply)
          + placement_group              = (known after apply)
          ~ primary_network_interface_id = "eni-0f1ce5bdae258b015" -> (known after apply)
          ~ private_dns                  = "ip-172-31-61-141.ec2.internal" -> (known after apply)
          ~ private_ip                   = "172.31.61.141" -> (known after apply)
          ~ public_dns                   = "ec2-54-166-19-244.compute-1.amazonaws.com" -> (known after apply)
          ~ public_ip                    = "54.166.19.244" -> (known after apply)
          ~ security_groups              = [
              - "default",
            ] -> (known after apply)
            source_dest_check            = true
          ~ subnet_id                    = "subnet-1facdf35" -> (known after apply)
          ~ tenancy                      = "default" -> (known after apply)
          ~ volume_tags                  = {} -> (known after apply)
          ~ vpc_security_group_ids       = [
              - "sg-5255f429",
            ] -> (known after apply)
    
          - credit_specification {
              - cpu_credits = "standard" -> null
            }
    
          + ebs_block_device {
              + delete_on_termination = (known after apply)
              + device_name           = (known after apply)
              + encrypted             = (known after apply)
              + iops                  = (known after apply)
              + snapshot_id           = (known after apply)
              + volume_id             = (known after apply)
              + volume_size           = (known after apply)
              + volume_type           = (known after apply)
            }
    
          + ephemeral_block_device {
              + device_name  = (known after apply)
              + no_device    = (known after apply)
              + virtual_name = (known after apply)
            }
    
          + network_interface {
              + delete_on_termination = (known after apply)
              + device_index          = (known after apply)
              + network_interface_id  = (known after apply)
            }
    
          ~ root_block_device {
              ~ delete_on_termination = true -> (known after apply)
              ~ iops                  = 100 -> (known after apply)
              ~ volume_id             = "vol-0079e485d9e28a8e5" -> (known after apply)
              ~ volume_size           = 8 -> (known after apply)
              ~ volume_type           = "gp2" -> (known after apply)
            }
        }
    
    Plan: 1 to add, 0 to change, 1 to destroy.
    

The prefix `-/+` means that Terraform will destroy and recreate the resource,
rather than updating it in-place. While some attributes can be updated in-place
(which are shown with the `~` prefix), changing the AMI for an EC2 instance
requires recreating it. Terraform handles these details for you, and the
execution plan makes it clear what Terraform will do.

Additionally, the execution plan shows that the AMI change is what required
your resource to be replaced. Using this information, you can adjust your
changes to possibly avoid destroy/create updates if they are not acceptable in
some situations.

Once again, Terraform prompts for approval of the execution plan before
proceeding. Answer `yes` to execute the planned steps:

    aws_instance.example: Destroying... [id=i-0bbf06244e44211d1]
    aws_instance.example: Still destroying... [id=i-0bbf06244e44211d1, 10s elapsed]
    aws_instance.example: Still destroying... [id=i-0bbf06244e44211d1, 20s elapsed]
    aws_instance.example: Still destroying... [id=i-0bbf06244e44211d1, 30s elapsed]
    aws_instance.example: Destruction complete after 31s
    aws_instance.example: Creating...
    aws_instance.example: Still creating... [10s elapsed]
    aws_instance.example: Still creating... [20s elapsed]
    aws_instance.example: Still creating... [30s elapsed]
    aws_instance.example: Creation complete after 38s [id=i-0589469dd150b453b]
    
    Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
    
As indicated by the execution plan, Terraform first destroyed the existing instance and then created a new one in its place. You can use `terraform show` again to see the new values associated with this instance.> How to completely destroy the Terraform-managed infrastructure.


## Destroy Infrastructure | Terraform - HashiCorp Learn

We've now seen how to build and change infrastructure. Before we move on to
creating multiple resources and showing resource dependencies, we're going to
go over how to completely destroy the Terraform-managed infrastructure.

Destroying your infrastructure is a rare event in production environments. But
if you're using Terraform to spin up multiple environments such as development,
test, QA environments, then destroying is a useful action.

### Destroy

The `terraform destroy` command terminates resources defined in your Terraform
configuration. This command is the reverse of `terraform apply` in that it
terminates all the resources specified by the configuration. It does _not_
destroy resources running elsewhere that are not described in the current
configuration.

    $ terraform destroy
    
    # ...
      # aws_instance.example will be destroyed
      - resource "aws_instance" "example" {
          - ami                          = "ami-b374d5a5" -> null
    # ...
    

The `-` prefix indicates that the instance will be destroyed. As with apply,
Terraform shows its execution plan and waits for approval before making any
changes.

Answer `yes` to execute this plan and destroy the infrastructure.

    # ...
    aws_instance.example: Destroying... [id=i-0589469dd150b453b]
    
    Destroy complete! Resources: 1 destroyed.
    # ...
    
Just like with `apply`, Terraform determines the order in which things must be
destroyed. In this case there was only one resource, so no ordering was
necessary. In more complicated cases with multiple resources, Terraform will
destroy them in a suitable order to respect dependencies, as we'll see later in
this guide.> Understanding and working with multiple resources and
dependencies.

## Resource Dependencies | Terraform - HashiCorp Learn

In this page, we're going to introduce resource dependencies, where we'll not
only see a configuration with multiple resources for the first time, but also
scenarios where resource parameters use information from other resources.

Up to this point, our example has only contained a single resource. Real
infrastructure has a diverse set of resources and resource types. Terraform
configurations can contain multiple resources, multiple resource types, and
these types can even span multiple providers.

On this page, we'll show a basic example of multiple resources and how to
reference the attributes of other resources to configure subsequent resources.

### Prerequisites

You should have completed the previous guides in this track, or use the
following configuration to start this guide.

Create a directory named `learn-terraform-aws-instance` and paste this code
into a file named `example.tf`.

    provider "aws" {
      profile    = "default"
      region     = "us-east-1"
    }
    
    resource "aws_instance" "example" {
      ami           = "ami-b374d5a5"
      instance_type = "t2.micro"
    }
    

### Assigning an Elastic IP

We'll improve our configuration by assigning an elastic IP to the EC2 instance
we're managing. Modify your `example.tf` and add the following to the end of
the file.

    resource "aws_eip" "ip" {
        vpc = true
        instance = aws_instance.example.id
    }
    

This should look familiar from the earlier example of adding an EC2 instance
resource, except this time we're building an "aws\_eip" resource type. This
resource type allocates and associates an [elastic
IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)
to an EC2 instance.

The only parameter for
[aws\_eip](https://www.terraform.io/docs/providers/aws/r/eip.html) is
"instance" which is the EC2 instance to assign the IP to. For this value, we
use an interpolation to use an attribute from the EC2 instance we managed
earlier.

The syntax for this interpolation should be straightforward: it requests the
"id" attribute from the "aws\_instance.example" resource.

### Apply Changes

Run `terraform apply` to see how Terraform plans to apply this change. The
output will look similar to the following:

    $ terraform apply
    # ...
    
      # aws_eip.ip will be created
      + resource "aws_eip" "ip" {
          + allocation_id     = (known after apply)
          + association_id    = (known after apply)
          + domain            = (known after apply)
          + id                = (known after apply)
          + instance          = (known after apply)
          + network_interface = (known after apply)
          + private_dns       = (known after apply)
          + private_ip        = (known after apply)
          + public_dns        = (known after apply)
          + public_ip         = (known after apply)
          + public_ipv4_pool  = (known after apply)
          + vpc               = (known after apply)
        }
    
      # aws_instance.example will be created
      + resource "aws_instance" "example" {
          + ami                          = "ami-b374d5a5"
    # ...
    

Terraform will create two resources: the instance and the elastic IP. In the
"instance" value for the "aws\_eip", you can see the raw interpolation is still
present. This is because this variable won't be known until the "aws\_instance"
is created. It will be replaced at apply-time.

As usual, Terraform prompts for confirmation before making any changes. Answer
`yes` to apply. The continued output will look similar to the following:

    # ...
    aws_instance.example: Creating...
    aws_instance.example: Creation complete after 36s [id=i-0214dc13c9f74e01b]
    aws_eip.ip: Creating...
    aws_eip.ip: Creation complete after 2s [id=eipalloc-007abcd5e15792497]
    
    Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
    
As shown above, Terraform created the EC2 instance before creating the Elastic
IP address. Due to the interpolation expression that passes the ID of the EC2
instance to the Elastic IP address, Terraform is able to infer a dependency,
and knows it must create the instance first.

### Implicit and Explicit Dependencies

By studying the resource attributes used in interpolation expressions,
Terraform can automatically infer when one resource depends on another. In the
example above, the reference to `aws_instance.example.id` creates an _implicit
dependency_ on the `aws_instance` named `example`.

Terraform uses this dependency information to determine the correct order in
which to create the different resources. In the example above, Terraform knows
that the `aws_instance` must be created before the `aws_eip`.

Implicit dependencies via interpolation expressions are the primary way to
inform Terraform about these relationships, and should be used whenever
possible.

Sometimes there are dependencies between resources that are _not_ visible to
Terraform. The `depends_on` argument is accepted by any resource and accepts a
list of resources to create _explicit dependencies_ for.

For example, perhaps an application we will run on our EC2 instance expects to
use a specific Amazon S3 bucket, but that dependency is configured inside the
application code and thus not visible to Terraform. In that case, we can use
`depends_on` to explicitly declare the dependency:

    resource "aws_s3_bucket" "example" {
      bucket = "terraform-getting-started-guide"
      acl    = "private"
    }
    
    resource "aws_instance" "example" {
      ami           = "ami-2757f631"
      instance_type = "t2.micro"
      
      depends_on = [aws_s3_bucket.example]
    }
    

### Non-Dependent Resources

We can continue to build this configuration by adding another EC2 instance:

    resource "aws_instance" "another" {
      ami           = "ami-b374d5a5"
      instance_type = "t2.micro"
    }

Because this new instance does not depend on any other resource, it can be
created in parallel with the other resources. Where possible, Terraform will
perform operations concurrently to reduce the total time taken to apply
changes.

Before moving on, remove this new resource from your configuration and run
`terraform apply` again to destroy it. We won't use this second instance any
further in the getting started guide.> Initialize instances when they're
created with provisioners.

## Provision | Terraform - HashiCorp Learn

You're now able to create and modify infrastructure. Now let's see how to use
provisioners to initialize instances when they're created.

If you're using an image-based infrastructure (perhaps with images created with
[Packer](https://www.packer.io/)), then what you've learned so far is good
enough. But if you need to do some initial setup on your instances, then
provisioners let you upload files, run shell scripts, or install and trigger
other software like configuration management tools, etc.

### Prerequisites

You should have completed the previous guides in this track, or use the
following configuration to start this guide.

Create a directory named `learn-terraform-aws-instance` and paste this code
into a file named `example.tf`.

    provider "aws" {
      profile    = "default"
      region     = "us-east-1"
    }
    
    resource "aws_instance" "example" {
      ami           = "ami-b374d5a5"
      instance_type = "t2.micro"
    }

### Defining a Provisioner

To define a provisioner, modify the resource block defining the "example" EC2
instance to look like the following:

    resource "aws_instance" "example" {
      ami           = "ami-b374d5a5"
      instance_type = "t2.micro"
    
      provisioner "local-exec" {
        command = "echo ${aws_instance.example.public_ip} > ip_address.txt"
      }
    }

This adds a `provisioner` block within the `resource` block. Multiple
`provisioner` blocks can be added to define multiple provisioning steps.
Terraform supports [multiple
provisioners](https://www.terraform.io/docs/provisioners/index.html), but for
this example we are using the `local-exec` provisioner.

Run `terraform init` and `terraform apply` and observe the `local-exec`
provisioner executing a command locally on your machine running Terraform.
We're using this provisioner versus the others so we don't have to worry about
specifying any [connection
info](https://www.terraform.io/docs/provisioners/connection.html) right now.
The `local-exec` provisioner you just ran created a file called
`ip_address.txt` on your local machine where you ran your `terraform apply`
command.

    $ cat ip_address.txt
    54.89.98.96

Another useful provisioner is `remote-exec` which invokes a script on a remote
resource after it is created. This can be used to run a configuration
management tool, bootstrap into a cluster, etc. In order to use a `remote-exec`
provisioner, you must choose an `ssh` or `winrm` connection in the form of a
`connection` block within the provisioner. Here is an example of how to use
`remote-exec` to install a specific package on a single instance at startup.
You should have an ssh key created with appropriate permissions to run the
example below.

Create an ssh key with no passphrase with `ssh-keygen -t rsa` and use the name
`terraform.` Update the permissions of that key with `chmod 400
~/.ssh/terraform`.

This example is for reference and should not be used without testing. If you
are running this, create a new Terraform project folder for this example.

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
    
This example has a few pieces to go over. The initial resource for the
`aws_key_pair` is required for SSH connections. You must create a keypair
locally to upload to AWS and the `aws_key_pair` resource is the function for
that. The `aws_instance` resource needs the `key_name` connected to it directly
as an attribute. Within the `aws_instance` resource, we create a connection
block which must define the connection type, the user, host, and private\_key
attributes.

The `private_key` attribute is necessary to successfully provision the host.
Once that connection is successful, the `remote-exec` provisioner will run on
the remote host to install, update, and start `nginx` in this example.

### Running Provisioners

Provisioners are only run when a resource is _created_. They are not a
replacement for configuration management and changing the software of an
already-running server, and are instead just meant as a way to bootstrap a
server. For configuration management, you should use Terraform provisioning to
invoke a real configuration management solution.

Make sure that your infrastructure is
[destroyed](https://www.terraform.io/intro/getting-started/destroy.html) if it
isn't already, then run `apply`:

    $ terraform apply
    # ...
    
    aws_instance.example: Creating...
      ami:           "" => "ami-b374d5a5"
      instance_type: "" => "t2.micro"
    aws_eip.ip: Creating...
      instance: "" => "i-213f350a"
    
    Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
    
Terraform will output anything from provisioners to the console, but in this
case there is no output. However, we can verify everything worked by looking at
the `ip_address.txt` file:

    $ cat ip_address.txt
    54.192.26.128
    
It contains the IP, just as we asked!

### Failed Provisioners and Tainted Resources

If a resource successfully creates but fails during provisioning, Terraform
will error and mark the resource as "tainted". A resource that is tainted has
been physically created, but can't be considered safe to use since provisioning
failed.

When you generate your next execution plan, Terraform will not attempt to
restart provisioning on the same resource because it isn't guaranteed to be
safe. Instead, Terraform will remove any tainted resources and create new
resources, attempting to provision them again after creation.

Terraform also does not automatically roll back and destroy the resource during
the apply when the failure happens, because that would go against the execution
plan: the execution plan would've said a resource will be created, but does not
say it will ever be deleted. If you create an execution plan with a tainted
resource, however, the plan will clearly state that the resource will be
destroyed because it is tainted.

### Manually Tainting Resources

In cases where you want to manually destroy and recreate a resource, Terraform
has a built in `taint` function in the CLI. This command will not modify
infrastructure, but does modify the state file in order to mark a resource as
tainted. Once a resource is marked as tainted, the next plan will show that the
resource will be destroyed and recreated and the next apply will implement this
change.

To taint a resource, use the following command:

`terraform taint resource.id`

`resource.id` refers to the resource block name and resource ID to taint.
Review the resource block we previously created:

    resource "aws_instance" "example" {
      ami           = "ami-b374d5a5"
      instance_type = "t2.micro"
    }

The correct resource and ID to taint this resource would be `terraform taint
aws_instance.example`.

### Destroy Provisioners

Provisioners can also be defined that run only during a destroy operation.
These are useful for performing system cleanup, extracting data, etc.

For many resources, using built-in cleanup mechanisms is recommended if
possible (such as init scripts), but provisioners can be used if necessary.

The getting started guide won't show any destroy provisioner examples. If you
need to use destroy provisioners, please [see the provisioner
documentation](https://www.terraform.io/docs/provisioners).> Parameterize your
configuration with input variables.

## Input Variables | Terraform - HashiCorp Learn

You now have enough Terraform knowledge to create useful configurations, but
we're still hard-coding access keys, AMIs, etc. To become truly shareable and
version controlled, we need to parameterize the configurations. This page
introduces input variables as a way to do this.

### The Initial Configuration

If you're starting this tutorial from scratch, create a directory named
`learn-terraform-aws-instance` and paste this code into a file named
`example.tf`.

    provider "aws" {
      profile    = "default"
      region     = "us-east-1"
    }
    
    resource "aws_instance" "example" {
      ami           = "ami-b374d5a5"
      instance_type = "t2.micro"
    }
    
### Defining Variables

Let's first extract our region into a variable. Create another file
`variables.tf` with the following contents.

**Note**: The file can be named anything, since Terraform loads all files in
the directory ending in `.tf`.

    variable "region" {
      default = "us-east-1"
    }
    
This defines the `region` variable within your Terraform configuration. There
is a default value which makes it optional. If no default is set, the variable
is required and must be set using one of the techniques mentioned in this
guide.

### Using Variables in a Configuration

Next, replace the AWS provider configuration with the following.

    provider "aws" {
      region = var.region
    }
    

This uses the variable named `region`, prefixed with `var.`. It tells Terraform
that you're accessing a variable and that the value of the `region` variable
should be used here. It configures the AWS provider with the given variable.

### Assigning variables

There are multiple ways to assign variables. The order below is also the order
in which variable values are chosen.

#### Command-line flags

You can set variables directly on the command-line with the `-var` flag. Any
command in Terraform that inspects the configuration accepts this flag, such as
`apply`, `plan`, and `refresh`.

    $ terraform apply \
      -var 'region=us-east-1'
    
Once again, setting variables this way will not save them, and they'll have to
be entered repeatedly as commands are executed.

### From a file

To persist variable values, create a file and assign variables within this
file. Create a file named `terraform.tfvars` with the following contents:

    region = "us-east-1"
    

Terraform automatically loads all files in the current directory with the exact
name of `terraform.tfvars` or any variation of `*.auto.tfvars`. If the file is
named something else, you can use the `-var-file` flag to specify a file name.
These files use the same syntax as Terraform configuration files (HCL). And
like Terraform configuration files, these files can also be JSON.

We don't recommend saving usernames and passwords to version control. You can
create a local file with a name like `secret.tfvars` and use `-var-file` flag
to load it.

You can use multiple `-var-file` arguments in a single command, with some
checked in to version control and others not checked in.

    $ terraform apply \
      -var-file="secret.tfvars" \
      -var-file="production.tfvars"
    

**Tip:** This is one way to provision infrastructure in a staging environment
or a production environment using the same Terraform configuration.

### From environment variables

Terraform will read environment variables in the form of `TF_VAR_name` to find
the value for a variable. For example, the `TF_VAR_region` variable can be set
in the shell to set the `region` variable in Terraform.

**Note**: Environment variables can only populate string-type variables. List
and map type variables must be populated via one of the other mechanisms.

### UI input

If you execute `terraform apply` with any variable unspecified, Terraform will
ask you to input the values interactively. These values are not saved, but this
provides a convenient workflow when getting started with Terraform. UI input is
not recommended for everyday use of Terraform.

**Note**: In Terraform versions 0.11 and earlier, UI input is only supported
for string variables. List and map variables must be populated via one of the
other mechanisms. Terraform 0.12 introduces the ability to populate complex
variable types from the UI prompt.

### Variable defaults

If no value is assigned to a variable via any of these methods and the variable
has a `default` key in its declaration, that value will be used for the
variable.

### Rich data types

Strings and numbers are the most commonly used variables, but lists (arrays)
and maps (hashtables or dictionaries) can also be used.

### Lists

Lists are defined either explicitly or implicitly.

    
    variable "cidrs" { default = [] }
    
    
    variable "cidrs" { type = list }
    

You can specify list values in a `terraform.tfvars` file.

    cidrs = [ "10.0.0.0/16", "10.1.0.0/16" ]
    

### Maps

A map is a key/value data structure that can contain other keys and values.

We've replaced our sensitive strings with variables, but we are still
hard-coding AMIs. Unfortunately, AMIs are specific to the geographical region
in use. One option is to ask the user to input the proper AMI for the region,
but Terraform can do better than that with a _map_.

Maps are a way to create variables that are lookup tables. Let's extract our
AMIs into a map and add support for the `us-west-2` region.

    variable "amis" {
      type = "map"
      default = {
        "us-east-1" = "ami-b374d5a5"
        "us-west-2" = "ami-4b32be2b"
      }
    }
    

A variable can be explicitly declared as a `map` type, or it can be implicitly
created by specifying a default value that is a map. The above demonstrates
both an explicit `type = "map"` and an implicit `default = {}`.

To use the `amis` map, edit `aws_instance` to use `var.amis` keyed by
`var.region`.

    resource "aws_instance" "example" {
      ami           = var.amis[var.region]
      instance_type = "t2.micro"
    }
    

The square-bracket index notation used here is an example of how the `map` type
expression is accessed as a variable, with `[var.region]` referencing the
`var.amis` declaration for dynamic lookup.

For a static value lookup, the region could be hard-coded such as
`var.amis["us-east-1"]`.

### Assigning maps

Map values can also be set using the `-var` and `-var-file` values.

    $ terraform apply -var 'amis={ us-east-1 = "foo", us-west-2 = "bar" }'
    

**Note**: Even if a map's data is set in a `tfvar` file, the variable must be
declared separately with either `type="map"` or `default={}`.

Here is an example of setting a map's keys from a file. This is the variable
definition in `example.tf`.

    variable "region" {}
    variable "amis" {
      type = "map"
    }
    

You can specify keys in a `terraform.tfvars` file.

    amis = {
      "us-east-1" = "ami-abc123"
      "us-west-2" = "ami-def456"
    }
    

Create an `aws_instance` with the `amis` and `region`.

    resource "aws_instance" "example" {
      ami           = var.amis[var.region]
      instance_type = "t2.micro"
    }
    

Read the selected AMI attribute from the `aws_instance` resource.

    output "ami" {
      value = aws_instance.example.ami
    }
    

Provision it by providing a region on the command line.

    $ terraform apply -var region=us-west-2
    
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
    
    Outputs:
    
      ami = ami-def456
    

### Next steps

For other examples, see the API documentation.

- [Input variables API
  doc](https://www.terraform.io/docs/configuration/variables.html)
- [Local variables API
  doc](https://www.terraform.io/docs/configuration/locals.html)> Organize your
  data for easier queries with outputs.

## Output Variables | Terraform - HashiCorp Learn

In the previous section, we introduced input variables as a way to parameterize
Terraform configurations. In this page, we introduce output variables as a way
to organize data to be easily queried and shown back to the Terraform user.

When building potentially complex infrastructure, Terraform stores hundreds or
thousands of attribute values for all your resources. But as a user of
Terraform, you may only be interested in a few values of importance, such as a
load balancer IP, VPN address, etc.

Outputs are a way to tell Terraform what data is important. This data is
outputted when `apply` is called, and can be queried using the `terraform
output` command.

### Prerequisites

You should have completed the previous guides in this track, or use the
following configuration to start this guide.

Create a directory named `learn-terraform-aws-instance` and paste this code
into a file named `example.tf`.

    provider "aws" {
      profile = "default"
      region  = "us-east-1"
    }
    
    resource "aws_instance" "example" {
      ami           = "ami-b374d5a5"
      instance_type = "t2.micro"
    }
    
    resource "aws_eip" "ip" {
      vpc = true
      instance = aws_instance.example.id
    }
    

### Defining Outputs

Let's define an output to show us the public IP address of the elastic IP
address that we create. Add this to any of your `*.tf` files:

    output "ip" {
      value = aws_eip.ip.public_ip
    }
    

This defines an output variable named "ip". The `public_ip` attribute is
exposed by the resource. Find the names of available attributes in the
[attributes
reference](https://www.terraform.io/docs/providers/aws/r/eip.html#attributes-reference)
section of the AWS provider documentation for the resource.

The name of the variable must conform to Terraform variable naming conventions
if it is to be used as an input to other modules. The `value` field specifies
what the value will be. It almost always contains one or more interpolations,
since the output data is typically dynamic. In this case, we're outputting the
`public_ip` attribute of the elastic IP address.

Multiple `output` blocks can be defined to specify multiple output variables.

### Viewing Outputs

Run `terraform apply` to populate the output. This only needs to be done once
after the output is defined. The apply output should change slightly. At the
end you should see this:

    $ terraform apply
    ...
    
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
    
    Outputs:
    
      ip = 50.17.232.209
    

`apply` highlights the outputs. You can also query the outputs after apply-time
using `terraform output`:

    $ terraform output ip
    50.17.232.209
    

This command is useful for scripts to extract outputs.> Refactor your existing
configuration into a module for reusability.

## Modules | Terraform - HashiCorp Learn

This guide is not currently updated for Terraform 0.12. For more information
about updating or writing modules, see the [configuration language
documentation](https://www.terraform.io/docs/configuration/modules.html)

Up to this point, we've been configuring Terraform by editing Terraform
configurations directly. As our infrastructure grows, this practice has a few
key problems: a lack of organization, a lack of reusability, and difficulties
in management for teams.

_Modules_ in Terraform are self-contained packages of Terraform configurations
that are managed as a group. Modules are used to create reusable components,
improve organization, and to treat pieces of infrastructure as a black box.

This section of the getting started will cover the basics of using modules.
Writing modules is covered in more detail in the [modules
documentation](https://www.terraform.io/docs/modules/index.html).

**Warning!** The examples on this page are _**not** eligible_ for [the AWS free
tier](https://aws.amazon.com/free/). Do not try the examples on this page
unless you're willing to spend a small amount of money.

### Using Modules

If you have any instances running from prior steps in the getting started
guide, use `terraform destroy` to destroy them, and remove all configuration
files.

The [Terraform Registry](https://registry.terraform.io/) includes a directory
of ready-to-use modules for various common purposes, which can serve as larger
building-blocks for your infrastructure.

In this example, we're going to use [the Consul Terraform module for
AWS](https://registry.terraform.io/modules/hashicorp/consul/aws), which will
set up a complete [Consul](https://www.consul.io/) cluster. This and other
modules can be found via the search feature on the Terraform Registry site.

Create a configuration file with the following contents:

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
    

The `module` block begins with the example given on the Terraform Registry page
for this module, telling Terraform to create and manage this module. This is
similar to a `resource` block: it has a name used within this configuration --
in this case, `"consul"` -- and a set of input values that are listed in [the
module's "Inputs"
documentation](https://registry.terraform.io/modules/hashicorp/consul/aws?tab=inputs).

(Note that the `provider` block can be omitted in favor of environment
variables. See the [AWS Provider
docs](https://www.terraform.io/docs/providers/aws/index.html) for details. This
module requires that your AWS account has a default VPC.)

The `source` attribute is the only mandatory argument for modules. It tells
Terraform where the module can be retrieved. Terraform automatically downloads
and manages modules for you.

In this case, the module is retrieved from the official Terraform Registry.
Terraform can also retrieve modules from a variety of sources, including
private module registries or directly from Git, Mercurial, HTTP, and local
files.

The other attributes shown are inputs to our module. This module supports many
additional inputs, but all are optional and have reasonable values for
experimentation.

### Module Versioning

After adding a new module to configuration, it is necessary to run (or re-run)
`terraform init` to obtain and install the new module's source code:

    $ terraform init
    # ...
    

By default, this command does not check for new module versions that may be
available, so it is safe to run multiple times. The `-upgrade` option will
additionally check for any newer versions of existing modules and providers
that may be available.

We recommend explicitly constraining the acceptable version numbers for each
external module to avoid unexpected or unwanted changes.

Use the version attribute in the module block to specify versions:

    module "consul" {
      source  = "hashicorp/consul/aws"
      version = "0.7.3"
    
      servers = 3
    }
    

### Apply Changes

With the Consul module (and its dependencies) installed, we can now apply these
changes to create the resources described within.

If you run `terraform apply`, you will see a large list of all of the resources
encapsulated in the module. The output is similar to what we saw when using
resources directly, but the resource names now have module paths prefixed to
their names, like in the following example:

      + module.consul.module.consul_clients.aws_autoscaling_group.autoscaling_group
          id:                                        <computed>
          arn:                                       <computed>
          default_cooldown:                          <computed>
          desired_capacity:                          "6"
          force_delete:                              "false"
          health_check_grace_period:                 "300"
          health_check_type:                         "EC2"
          launch_configuration:                      "${aws_launch_configuration.launch_configuration.name}"
          max_size:                                  "6"
          metrics_granularity:                       "1Minute"
          min_size:                                  "6"
          name:                                      <computed>
          protect_from_scale_in:                     "false"
          tag.#:                                     "2"
          tag.2151078592.key:                        "consul-clients"
          tag.2151078592.propagate_at_launch:        "true"
          tag.2151078592.value:                      "consul-example"
          tag.462896764.key:                         "Name"
          tag.462896764.propagate_at_launch:         "true"
          tag.462896764.value:                       "consul-example-client"
          termination_policies.#:                    "1"
          termination_policies.0:                    "Default"
          vpc_zone_identifier.#:                     "6"
          vpc_zone_identifier.1880739334:            "subnet-5ce4282a"
          vpc_zone_identifier.3458061785:            "subnet-16600f73"
          vpc_zone_identifier.4176925006:            "subnet-485abd10"
          vpc_zone_identifier.4226228233:            "subnet-40a9b86b"
          vpc_zone_identifier.595613151:             "subnet-5131b95d"
          vpc_zone_identifier.765942872:             "subnet-595ae164"
          wait_for_capacity_timeout:                 "10m"
    

The `module.consul.module.consul_clients` prefix shown above indicates not only
that the resource is from the `module "consul"` block we wrote, but in fact
that this module has its own `module "consul_clients"` block within it. Modules
can be nested to decompose complex systems into manageable components.

The full set of resources created by this module includes an autoscaling group,
security groups, IAM roles and other individual resources that all support the
Consul cluster that will be created.

Note that as we warned above, the resources created by this module are not
eligible for the AWS free tier and so proceeding further will have some cost
associated. To proceed with the creation of the Consul cluster, type `yes` at
the confirmation prompt.

    # ...
    
    module.consul.module.consul_clients.aws_security_group.lc_security_group: Creating...
      description:            "" => "Security group for the consul-example-client launch configuration"
      egress.#:               "" => "<computed>"
      ingress.#:              "" => "<computed>"
      name:                   "" => "<computed>"
      name_prefix:            "" => "consul-example-client"
      owner_id:               "" => "<computed>"
      revoke_rules_on_delete: "" => "false"
      vpc_id:                 "" => "vpc-22099946"
    
    # ...
    
    Apply complete! Resources: 34 added, 0 changed, 0 destroyed.
    

After several minutes and many log messages about all of the resources being
created, you'll have a three-server Consul cluster up and running. Without
needing any knowledge of how Consul works, how to install Consul, or how to
form a Consul cluster, you've created a working cluster in just a few minutes.

### Module Outputs

Just as the module instance had input arguments such as `num_servers` above, a
module can also produce _output_ values, similar to resource attributes.

[The module's outputs
reference](https://registry.terraform.io/modules/hashicorp/consul/aws?tab=outputs)
describes all of the different values it produces. Overall, it exposes the id
of each of the resources it creates, as well as echoing back some of the input
values.

One of the supported outputs is called `asg_name_servers`, and its value is the
name of the auto-scaling group that was created to manage the Consul servers.

To reference this, we'll just put it into our _own_ output value. This value
could actually be used anywhere: in another resource, to configure another
provider, etc.

Add the following to the end of the existing configuration file created above:

    output "consul_server_asg_name" {
      value = "${module.consul.asg_name_servers}"
    }
    

The syntax for referencing module outputs is `${module.NAME.OUTPUT}`, where
`NAME` is the module name given in the header of the `module` configuration
block and `OUTPUT` is the name of the output to reference.

If you run `terraform apply` again, Terraform will make no changes to
infrastructure, but you'll now see the "consul\_server\_asg\_name" output with
the name of the created auto-scaling group:

    # ...
    
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
    
    Outputs:
    
    consul_server_asg_name = tf-asg-2017103123350991200000000a
    

If you look in the Auto-scaling Groups section of the EC2 console you should
find an autoscaling group of this name, and from there find the three Consul
servers it is running. (If you can't find it, make sure you're looking in the
right region!)

### Destroy

Just as with top-level resources, we can destroy the resources created by the
Consul module to avoid ongoing costs:

    $ terraform destroy
    # ...
    
    Terraform will perform the following actions:
    
      - module.consul.module.consul_clients.aws_autoscaling_group.autoscaling_group
    
      - module.consul.module.consul_clients.aws_iam_instance_profile.instance_profile
    
      - module.consul.module.consul_clients.aws_iam_role.instance_role
    
    # ...
    

As usual, Terraform describes all of the actions it will take. In this case, it
plans to destroy all of the resources that were created by the module. Type
`yes` to confirm and, after a few minutes and even more log output, all of the
resources should be destroyed:

    Destroy complete! Resources: 34 destroyed.
    

With all of the resources destroyed, you can delete the configuration file we
created above. We will not make any further use of it, and so this avoids the
risk of accidentally re-creating the Consul cluster.> A brief introduction to
remote backends and Terraform Cloud for remote state management.

## Remote State Storage | Terraform

You have now seen how to build, change, and destroy infrastructure from a local
machine. This is great for testing and development, but in production
environments it is considered a best practice to store state elsewhere than
your local machine. The best way to do this is by running Terraform in a remote
environment with shared access to state.

Terraform supports team-based workflows with a feature known as [remote
backends](https://www.terraform.io/docs/backends/index.html). Remote backends
allow Terraform to use a shared storage space for state data, so any member of
your team can use Terraform to manage the same infrastructure.

Depending on the features you wish to use, Terraform has multiple remote
backend options. HashiCorp recommends using [Terraform
Cloud](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/terraform/cloud-gettingstarted/tfc_signup).

[Terraform
Cloud](https://www.hashicorp.com/products/terraform/?utm_source=oss&utm_medium=getting-started&utm_campaign=terraform)
also offers HashiCorp's commercial solutions and with a free version which acts
as a remote backend. Terraform Cloud allows teams to easily version, audit, and
collaborate on infrastructure changes. Each proposed change generates a
Terraform plan which can be reviewed and collaborated on as a team. When a
proposed change is accepted, the Terraform logs are stored, resulting in a
linear history of infrastructure states to help with auditing and policy
enforcement. Additional benefits to running Terraform remotely include moving
access credentials off of developer machines and freeing local machines from
long-running Terraform processes.

### Prerequisites

You should have completed the previous guides in this track, or use the
following configuration to start this guide.

Create a directory named `learn-terraform-aws-instance` and paste this code
into a file named `example.tf`.

    provider "aws" {
      profile    = "default"
      region     = "us-east-1"
    }
    
    resource "aws_instance" "example" {
      ami           = "ami-b374d5a5"
      instance_type = "t2.micro"
    }
    

### How to Store State Remotely

First, we'll use Terraform Cloud as our backend. Terraform Cloud offers free
remote state management. Terraform Cloud is the recommended best practice for
remote state storage.

If you don't have an account, please [sign up
here](https://app.terraform.io/signup) for this guide. For more information on
Terraform Cloud, [view our getting started
guide](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/terraform/cloud-gettingstarted/tfc_signup).

When you sign up for Terraform Cloud, you'll create an organization. Make a
note of the organization's name.

Next, configure the backend in your configuration with the organization name,
and a new workspace name of your choice:

    terraform {
      backend "remote" {
        organization = "<ORG_NAME>"
    
        workspaces {
          name = "Example-Workspace"
        }
      }
    }
    

You'll also need a user token to authenticate with Terraform Cloud. You can
generate one on the [user settings
page](https://app.terraform.io/app/settings/tokens):

![User
Token](https://d33wubrfki0l68.cloudfront.net/bf6f7a5a7cd9c90d976bc2139d2ad3c9e6a94c06/b53d3/img/tf-cloud-user-token.png
"User Token")

Copy the user token to your clipboard, and create a Terraform CLI Configuration
file. This file is This file is located at `%APPDATA%\terraform.rc` on Windows
systems, and `~/.terraformrc` on other systems.

Paste the user token into that file like so:

    credentials "app.terraform.io" {
      token = "REPLACE_ME"
    }
    

Save and close this file, we don't need it again. You can read more about the
CLI config file in [the
documentation](https://www.terraform.io/docs/commands/cli-config.html).

Now that you've configured your remote backend, run `terraform init` to setup
Terraform. It should ask if you want to migrate your state to Terraform Cloud.

    $ terraform init
    
    Initializing the backend...
    Do you want to copy existing state to the new backend?
      Pre-existing state was found while migrating the previous "local" backend to the
      newly configured "remote" backend. No existing state was found in the newly
      configured "remote" backend. Do you want to copy this state to the new "remote"
      backend? Enter "yes" to copy and "no" to start with an empty state.
    
      Enter a value:
    

Say "yes" and Terraform will copy your state:

    ...
    
      Enter a value: yes
    
    Releasing state lock. This may take a few moments...
    
    Successfully configured the backend "remote"! Terraform will automatically
    use this backend unless the backend configuration changes.
    
    ...
    

Now, if you run `terraform apply`, Terraform should state that there are no
changes:

    $ terraform apply
    
    
    No changes. Infrastructure is up-to-date.
    
    This means that Terraform did not detect any differences between your
    configuration and real physical resources that exist. As a result, Terraform
    doesn't need to do anything.
    

Terraform is now storing your state remotely in Terraform Cloud. Remote state
storage makes collaboration easier and keeps state and secret information off
your local disk. Remote state is loaded only in memory when it is used.

If you want to move back to local state, you can remove the backend
configuration block from your configuration and run `terraform init` again.
Terraform will once again ask if you want to migrate your state back to local.

### Terraform Cloud

[Terraform Cloud](https://www.hashicorp.com/products/terraform/pricing) offers
commercial solutions which combines a predictable and reliable shared run
environment with tools to help you work together on Terraform configurations
and modules.

Although Terraform Cloud can act as a standard remote backend to support
Terraform runs on local machines, it works even better as a remote run
environment. It supports two main workflows for performing Terraform runs:

- A VCS-driven workflow, in which it automatically queues plans whenever
  changes are committed to your configuration's VCS repo.
- An API-driven workflow, in which a CI pipeline or other automated tool can
  upload configurations directly.

For a hands-on introduction to Terraform Cloud, [follow the Terraform Cloud
getting started
guides](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/terraform?track=cloud-gettingstarted#cloud-gettingstarted)
for our free offering as well as Terraform Cloud for
[Teams](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/terraform?track=enterprise#enterprise)
and
[Governance](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/terraform?track=sentinel#sentinel).>
Additional resources for Terraform practitioners.

## Next Steps | Terraform - HashiCorp Learn

That concludes the getting started guide for Terraform. Hopefully you're now
able to not only see what Terraform is useful for, but you're also able to put
this knowledge to use to improve building your own infrastructure.

We've covered the basics for all of these features in this guide.

As a next step, the following resources are available:

- [Documentation](https://www.terraform.io/docs/index.html) - The documentation
  is an in-depth reference guide to all the features of Terraform, including
  technical details about the internals of how Terraform operates.
    
- [Examples](https://www.terraform.io/intro/examples/index.html) - The examples
  have more full featured configuration files, showing some of the
  possibilities with Terraform.
    
- [Import](https://www.terraform.io/docs/import/index.html) - The import
  section of the documentation covers importing existing infrastructure into
  Terraform.
    
Was this guide helpful?