#Terraform-example

This file contains an example on how to install Terraform on Linux and create an AWS VPC resource with Terraform.

## Table of Contents
1. <a href="#introduction">Introduction</a>
2. <a href="#installation">Installation</a>
3. <a href="#configuring-providers-in-terraform">Configuring Providers in Terraform</a>
4. <a href="#creating-an-aws-resource-with-terraform">Creating an AWS Resource with Terraform</a>

## Introduction

Terraform is an open source project maintained by HashiCorp for building, changing, and combining infrastructure safely and efficiently. Some of the benefits of using Terraform to provision and manage infrastructure include:
•	Version-controlled infrastructure as code
•	Multi-cloud support with over 100 providers including Amazon Web Services, Microsoft Azure, and Google Cloud Platform
•	Separate plan and apply stages so you can verify changes to your infrastructure before they happen
•	A public registry of modules that make provisioning common groups of resources easy
HashiCorp also provides enterprise versions of Terraform that provide additional features for collaboration, infrastructure policy, and governance. The enterprise versions are built on top of the open source version. What you learn in Cloud Academy Labs that use the open source version applies to enterprise versions as well.

Terraform allows you to safely manage infrastructure by modifying configuration files. Terraform will show you an execution plan that you must approve before any infrastructure changes are applied. This file highlights several capabilities of Terraform that make managing infrastructure easy. 

## Installation
1. Download a release package <a href=https://www.terraform.io/downloads.html> here</a>.
```
https://releases.hashicorp.com/terraform/0.13.5/terraform_0.13.5_linux_arm64.zip
```
2. Extract the zip archive containing the Terraform binary to the /usr/local/bin directory:
```
sudo unzip terraform_0.11.3_linux_amd64.zip -d /usr/local/bin/
```
3. Optional - Remove the release package:
```
rm terraform_0.13.5_linux_arm64.zip
```
4. Confirm Terraform is installed.
```
terraform version
```

## Configuring Providers in Terraform

Terraform uses a plugin architecture to manage all of the resource providers that it supports. No providers are included when you first install Terraform. You declare which providers you need to use in a configuration file. Terraform configuration files have a .tf file extension. The configuration is written using HashiCorp Configuration Language (HCL). HCL attempts to strike a balance between human- and machine-readability. JavaScript Object Notation (JSON) can also be used but is discouraged because it is less human-readable and doesn't support commenting.
Terraform includes an initialization command that reads the providers in a configuration and downloads the appropriate provider plugin. 

1. Make a directory for organizing the Terraform configuration, and change into it:
```
mkdir infra && cd infra
```
2. Create a Terraform configuration file declaring the provider in this case it is AWS.
```
cat > main.tf <<EOF
provider "aws" {
  version = "< 2"
  region  = "your-preferred-region"
  #example region  = "us-west-2"
  access_key = "my-access-key"
  secret_key = "my-secret-key"
 }
EOF
```
-	The provider keyword is followed by the name of the provider you want to use.
-	The name aws is a string and all strings must be enclosed in double quotes in HCL.
-	The braces surround the configuration arguments for the aws provider.
-	version is a special argument that all providers share. The value constrains which version of the provider plugin to use, in this case, a version less than 2.
-	Argument values are assigned using an equal sign =. 
-The region is the only required argument for the aws provider. An example of an optional argument for the AWS provider is access_key and secret_key credentials. If you use an EC2 instance to run Terraform, Terraform is able to use the IAM instance profile to authenticate any requests to AWS.

3. Initialize the working directory by running the init command:
```
terraform init
```
4. List all of the directories to see what the init command has created:
```
ls -A
```
The .terraform directory stores the downloaded AWS provider plugin in .terraform/plugins/linux_arm64/:

##Creating an AWS Resource with Terraform

The AWS Terraform provider allows you to manage resources in AWS. You can create resources provided by a provider by adding resource blocks to Terraform configuration files. The syntax of a resource block resembles the following:
```resource "resource_type" "identifier" {
  argument1 = value1
  argument2 = value2
}
```
The resource_type is a type of resource provided by the provider. The name of the provider of the resource type is always the first part of the type string. For example, AWS resources all have types beginning with aws_. A list of all supported resource types for the AWS provider is available in the sidebar of the AWS provider page. The identifier is used within Terraform to identify the resource.

1. Append an aws_vpc resource block to your main.tf Terraform configuration file:
```
cat >> main.tf <<EOF

resource "aws_vpc" "web_vpc" {
  cidr_block           = "192.168.100.0/24"
  enable_dns_hostnames = true

  tags {
    Name = "Web VPC"
  }
}
EOF
```
The cidr_block(Classless Inter-Domain Routing) is the only required argument for an aws_vpc resource. The enable_dns_hostnames and tags arguments are both optional and demonstrate a boolean value (true), and a mapping that includes the Name key

2. Issue the apply command to have Terraform generate a plan that you can review before actually applying:
```
terraform apply
```
3. Accept and apply the execution plan by entering yes at the prompt.
The following confirmation output is displayed:

4. Use the AWS command-line interface (CLI) to confirm that the VPC has been created with the arguments:
```
aws ec2 describe-vpcs -- <REGION> --filter "Name=tag:Name,Values=Web VPC"
```

5. Enter the following command to determine if DNS hostnames support is enabled for the VPC:
```
aws ec2 describe-vpc-attribute -- <REGION> --attribute enableDnsHostnames --vpc-id <VPC_ID>
```
Replace <VPC_ID> with the the value of the VpcId property in the output of the previous command. The value will resemble vpc-abcd1234. The output confirms that the DNS hostname support has been enabled.

