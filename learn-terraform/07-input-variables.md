# Input Variables 

## Defining Variables

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

## Using Variables in a Configuration

:ship: Use the defined `region` variable in `example.tf` 
```terraform
provider "aws" {
  region = var.region
}
```
    
## Assigning variables

There are multiple ways to assign variables. The order below is also the order
in which variable values are chosen.
- command-line flags
- from a file
- from env variables
- ui input

### Command-line flags

:ship: using `-var`
```bash
terraform apply -var 'region=us-east-1'
```

### From a file

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

### From environment variables

TF can read `TF_VAR_<name>` environment variables

e.g. `TF_VAR_region`

### UI input

If you execute `terraform apply` with any variable unspecified, Terraform will
ask you to input the values interactively. 

### Variable defaults

If no value is assigned to a variable via any of these methods and the variable
has a `default` key in its declaration, that value will be used for the
variable.

## Rich data types

Data Types: strings, numbers, lists, maps (hashtable or dictionary)

### Lists


:ship: 
```terraform

variable "cidrs" { default = [] }

variable "cidrs" { type = list }

cidrs = [ "10.0.0.0/16", "10.1.0.0/16" ]
```
    
### Maps

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
    
## Next steps

For other examples, see the API documentation.

- [Input variables API doc](https://www.terraform.io/docs/configuration/variables.html)
- [Local variables API
  doc](https://www.terraform.io/docs/configuration/locals.html)> Organize your
  data for easier queries with outputs.
