# Remote State Storage | Terraform

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

## Prerequisites

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
    

## How to Store State Remotely

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

## Terraform Cloud

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
