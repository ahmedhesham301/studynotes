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
# Plan
> **Warning**
> Terraform plan files can contain sensitive data. **Never** commit a plan file to version control, whether as a binary or in JSON format.

### To output a plan to a file:
```bash
terraform plan -out "tfplan"
```
use -destroy to get a destroy plan.
### To read a plan from a file:
```bash
terraform show "tfplan"
```
### To apply a plan from a file use:
```bash
terraform apply "tfplan"
```
> **Note**
> When you apply a saved plan file, Terraform will not prompt you for approval and instead immediately execute the changes. This workflow is primarily used in automation.
### Convert the saved plan into JSON:
```bash
terraform show -json "tfplan" | jq > tfplan.json
```
# Apply
### Replace Resources
```bash
terraform apply -replace "resource address"
```
> The `-replace` argument requires a resource address. List the resources in your configuration with `terraform state list`.

# Output
> You can add output declarations anywhere in your Terraform configuration files. However, it is recommend defining them in a separate file called `outputs.tf`
### Output structure
```hcl
output "name" {
  description = "description"   #optional
  value       = value
  sensitive   = true            #optional
}
```
you can use `${}` to put a variable in an output string.
### Get output in terminal
```bash
terraform output
```
add output name at the end to get the value of that output only.
to get a string output without the quotes use -raw
use -json to get output as json