# Output Variables | Terraform - HashiCorp Learn

When building potentially complex infrastructure, Terraform stores hundreds or
thousands of attribute values for all your resources. But as a user of
Terraform, you may only be interested in a few values of importance, such as a
load balancer IP, VPN address, etc.

Outputs are a way to tell Terraform what data is important. This data is
outputted when `apply` is called, and can be queried using the `terraform
output` command.

## Defining Outputs

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

## Viewing Outputs

Output will be displayed after `terraform apply` or can explicitly query with `terraform output`
