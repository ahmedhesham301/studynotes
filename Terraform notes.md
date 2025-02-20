

# Terraform Provider Configuration
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
    }
  }
}
```
# Configure the AWS Provider
```hcl
provider "aws" {
  region = "eu-central-1"
}
```
# Launch an EC2 Instance
```hcl
resource "aws_instance" "name" {
  ami                    = "ami-0c8db01b2e8e5298d"
  instance_type          = "t2.micro"


  tags = {
    Name = "name"
  }
}
```
###### Run a Bash script on startup:
Add the following line to execute a script when the machine boots:
```hcl
user_data = file("script.sh")
```
###### Attach security groups to the instance:
```hcl
vpc_security_group_ids = [
  aws_security_group.groupName.id,
  aws_security_group.groupName.id
]
```

# Creating a Security Group
```hcl
resource "aws_security_group" "web-sg" {
  name = "http"
  ingress {
    from_port = 80
	to_port = 80
	protocol = "tcp"
	cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
	from_port = 0
	to_port = 0
	protocol = "-1"
	cidr_blocks = ["0.0.0.0/0"]
	ipv6_cidr_blocks = ["::/0"]
  }
}
```
# Generating a Random Name
```hcl
provider "random" {}
resource "random_pet" "name" {}
```
# variables 
It is recommended to add variables to `variables.tf`.
If you want to add a variable to a string use `${var.variableName}`
### To declare a variable:
```hcl
variable "variable_name" {
  description = "Description of the variable"
  type        = <type>
  default     = "default value"
}

```
### To refer to a variable:
```hcl
var.<variable_name>
```
### To slice a list:
```hcl
slice(var.listname, start, end)
```
### To set variable value in the command line:
```bash
terraform apply -var <variableName>=<value>
```
### To set variable values in a file:
make a file called `terraform.tfvars` or `*.auto.tfvars`. You can also use the `-var-file` flag to specify other files by name in command line then add the value of the variables in it.
### Environment Variables
Terraform searches the environment of its own process for environment variables named `TF_VAR_` followed by the name of a declared variable.
### Secret Variables
In declaring add the line
```hcl
sensitive = true
```
make a `secret.tfvars` file and set there values in it.(don't forget to use `-var-file`).