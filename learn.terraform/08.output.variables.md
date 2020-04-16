# Output Variables | Terraform - HashiCorp Learn

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

## Prerequisites

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
    

## Defining Outputs

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

## Viewing Outputs

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
