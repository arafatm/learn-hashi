# Provision

Provisioners let you upload files, run shell scripts, or install and trigger
other software like configuration management tools, etc.

In general you manage image based infra using TF. Images can be built with
[Packer](https://www.packer.io/)), 

## Defining a Provisioner

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
    
## Failed Provisioners and Tainted Resources

If a resource is created but fails at provisioning it is marked _tainted_

Terraform will remove any tainted resources and create new resources,
attempting to provision them again after creation.

## Manually Tainting Resources

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

## Destroy Provisioners

Provisioners can also be defined that run only during a destroy operation.
These are useful for performing system cleanup, extracting data, etc.

[see the provisioner documentation](https://www.terraform.io/docs/provisioners)
