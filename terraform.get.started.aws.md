https://learn.hashicorp.com/terraform

## Introduction to Infrastructure as Code with Terraform

5 MIN | What is Infrastructure as Code and Why is Terraform Useful?

Terraform is the infrastructure as code offering from HashiCorp. It is a tool
for building, changing, and managing infrastructure in a safe, repeatable way.
Operators and Infrastructure teams can use Terraform to manage environments
with a configuration language called the HashiCorp Configuration Language
(HCL) for human-readable, automated deployments.

### Infrastructure as Code

If you are new to infrastructure as code as a concept, it is the process of
managing infrastructure in a file or files rather than manually configuring
resources in a user interface. A resource in this instance is any piece of
infrastructure in a given environment, such as a virtual machine, security
group, network interface, etc.

At a high level, Terraform allows operators to use HCL to author files
containing definitions of their desired resources on almost any provider (AWS,
GCP, GitHub, Docker, etc) and automates the creation of those resources at the
time of apply.

In this track, we will cover the basic functions of Terraform to create
infrastructure on AWS.

### Workflows

A simple workflow for deployment will follow closely to the steps below. We
will go over each of these steps and concepts more in-depth throughout this
track, so don't panic if you don't understand the concepts immediately.

- **Scope** - Confirm what resources need to be created for a given project.
- **Author** - Create the configuration file in HCL based on the scoped
  parameters
- **Initialize** - Run terraform init in the project directory with the
  configuration files. This will download the correct provider plug-ins for
  the project.
- **Plan & Apply** - Run terraform plan to verify creation process and then
  terraform apply to create real resources as well as state file that compares
  future changes in your configuration files to what actually exists in your
  deployment environment.

### Advantages of Terraform

While many of the current offerings for infrastructure as code may work in
your environment, Terraform aims to have a few advantages for operators and
organizations of any size.

- Platform Agnostic
- State Management
- Operator Confidence

### Platform Agnostic

In a modern datacenter, you may have several different clouds and platforms to
support your various applications. With Terraform, you can manage a
heterogenous environment with the same workflow by creating a configuration
file to fit the needs of your project or organization.

### State Management

Terraform creates a state file when a project is first initialized. Terraform
uses this local state to create plans and make changes to your infrastructure.
Prior to any operation, Terraform does a refresh to update the state with the
real infrastructure. This means that Terraform state is the source of truth by
which configuration changes are measured. If a change is made or a resource is
appended to a configuration, Terraform compares those changes with the state
file to determine what changes result in a new resource or resource
modifications.

### Operator Confidence

The workflow built into Terraform aims to instill confidence in users by
promoting easily repeatable operations and a planning phase to allow users to
ensure the actions taken by Terraform will not cause disruption in their
environment. Upon terraform apply, the user will be prompted to review the
proposed changes and must affirm the changes or else Terraform will not apply
the proposed plan.

## Installing Terraform

7 MIN | How to install Terraform and verify installation.

Terraform must first be installed on your machine. Terraform is distributed as
a binary package for all supported platforms and architectures. This page will
not cover how to compile Terraform from source, but compiling from source is
covered in the documentation for those who want to be sure they're compiling
source they trust into the final binary.

### Installing Terraform

To install Terraform, find the appropriate package for your system and
download it. Terraform is packaged as a zip archive.

After downloading Terraform, unzip the package. Terraform runs as a single
binary named terraform. Any other files in the package can be safely removed
and Terraform will still function.

The final step is to make sure that the terraform binary is available on the
PATH. See this page for instructions on setting the PATH on Linux and Mac.
This page contains instructions for setting the PATH on Windows.

### Verify the installation

Verify that the installation worked by opening a new terminal session and
running the terraform command.

```
$ terraform
Usage: terraform [--version] [--help] <command> [args]

The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you are just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.

Common commands:
    apply              Builds or changes infrastructure
    console            Interactive console for Terraform interpolations
    # ...
```

If you get an error that terraform could not be found, your PATH environment
variable was not set up properly. Please go back and ensure that your PATH
variable contains the directory where Terraform was installed.

### Quick start tutorial

Now that you've installed Terraform, you can provision an Nginx server in less
than a minute using Docker on Mac, Windows, or Linux. The next guide in this
track goes into more details and provisions live cloud resources but this
example can be run locally for free in barely any time at all.

We've created an interactive environment where you can run this Terraform
configuration in your web browser. Launch it here.

[Show Tutorial]()

Or on your local machine, create a directory named terraform-docker-demo.

```
$ mkdir terraform-docker-demo && cd $_
```

Paste the following into a file named main.tf.

```
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

Initialize the project with the `init` command.

```
$ terraform init
```

Provision the Nginx server with apply. You'll be asked to confirm by typing
yes and pressing ENTER.

```
$ terraform apply
```

Verify the existence of the Nginx container by visiting localhost in your web
browser or running docker ps to see the Nginx container.

![Nginx running in Docker via
Terraform](https://learn.hashicorp.com/img/terraform/getting-started/terraform-docker-nginx.png)

```
$ docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
1acc851196f9        ed21b7a8aee9        "nginx -g 'daemon of…"   30 seconds ago      Up 28 seconds       0.0.0.0:80->80/tcp   tutorial
```

To stop the container, run `terraform destroy`

```
$ terraform destroy
```

That's it! You've provisioned and destroyed an Nginx webserver with Terraform.

For a more detailed example on cloud resources, see the next guide in this
track.

### Getting Help

The Terraform CLI has a built-help function. If at any point during this guide
you are unsure of how to proceed, consider using the -help flag with any
command. For example:

```
$ terraform -help
Usage: terraform [-version] [-help] <command> [args]

The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you are just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.

Common commands:
    apply              Builds or changes infrastructure
    console            Interactive console for Terraform interpolations
    destroy            Destroy Terraform-managed infrastructure
    env                Workspace management
    fmt                Rewrites config files to canonical format
    get                Download and install modules for the configuration
    graph              Create a visual graph of Terraform resources
    import             Import existing infrastructure into Terraform
    init               Initialize a Terraform working directory
    output             Read an output from a state file
    plan               Generate and show an execution plan
    providers          Prints a tree of the providers used in the configuration
    refresh            Update local state file against real resources
    show               Inspect Terraform state or plan
    taint              Manually mark a resource for recreation
    untaint            Manually unmark a resource as tainted
    validate           Validates the Terraform files
    version            Prints the Terraform version
    workspace          Workspace management

All other commands:
    0.12upgrade        Rewrites pre-0.12 module source code for v0.12
    debug              Debug output management (experimental)
    force-unlock       Manually unlock the terraform state
    push               Obsolete command for Terraform Enterprise legacy (v1)
    state              Advanced state management
```

Any of these commands can be added to the --help flag to get more information. For example:

```
$ terraform --help plan

Usage: terraform plan [options] [DIR]

  Generates an execution plan for Terraform.

  This execution plan can be reviewed prior to running apply to get a
  sense for what Terraform will do. Optionally, the plan can be saved to
  a Terraform plan file, and apply can take this plan file to execute
  this plan exactly.

Options:

  -destroy            If set, a plan will be generated to destroy all resources
                      managed by the given configuration and state.

  -detailed-exitcode  Return detailed exit codes when the command exits. This
                      will change the meaning of exit codes to:
                      0 - Succeeded, diff is empty (no changes)
                      1 - Errored
                      2 - Succeeded, there is a diff

  -input=true         Ask for input for variables if not directly set.

  -lock=true          Lock the state file when locking is supported.

  -lock-timeout=0s    Duration to retry a state lock.

  -no-color           If specified, output will not contain any color.

  -out=path           Write a plan file to the given path. This can be used as
                      input to the "apply" command.

  -parallelism=n      Limit the number of concurrent operations. Defaults to 10.

  -refresh=true       Update state prior to checking for differences.

  -state=statefile    Path to a Terraform state file to use to look
                      up Terraform-managed resources. By default it will
                      use the state "terraform.tfstate" if it exists.

  -target=resource    Resource to target. Operation will be limited to this
                      resource and its dependencies. This flag can be used
                      multiple times.

  -var "foo=bar"      Set a variable in the Terraform configuration. This
                      flag can be set multiple times.

  -var-file=foo       Set variables in the Terraform configuration from
                      a file. If "terraform.tfvars" or any ".auto.tfvars"
                      files are present, they will be automatically loaded.
```

## Build Infrastructure 

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

```bash
$ terraform apply
```

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
again to see the new values associated with this instance.