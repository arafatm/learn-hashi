# Modules | Terraform - HashiCorp Learn

_Modules_ are used to create reusable components, improve organization, and to
treat pieces of infrastructure as a black box.

[modules documentation](https://www.terraform.io/docs/modules/index.html).

:warning: The examples on this page are _**not** eligible_ for [the AWS free
tier](https://aws.amazon.com/free/). Do not try the examples on this page
unless you're willing to spend a small amount of money.

## Using Modules

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

## Module Versioning

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

## Apply Changes

:ship: Apply
```bash
terraform apply
```

The `module.consul.module.consul_clients` prefix shown above indicates not only
that the resource is from the `module "consul"` block we wrote, but in fact
that this module has its own `module "consul_clients"` block within it. Modules
can be nested to decompose complex systems into manageable components.

## Module Outputs

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

