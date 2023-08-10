# AUTOMATE INFRASTRUCTURE USING TERRAFORM
Prerequisites
 - IAM
 - Visual

Create an IAM user, name it terraform (ensure that the user has only programatic access to your AWS account) and grant this user AdministratorAccess permissions. 

Copy the secret access key and access key ID. Save them in a notepad temporarily.

Create an S3 bucket to store Terraform state file. You can name it something like -dev-terraform-bucket (Note: S3 bucket names must be unique within a region partition.)

<img width="908" alt="image" src="https://github.com/Shubsdev/Devops-Projects/assets/102925329/2457f053-cadb-429a-a837-60a637b854ca">

## VPC | SUBNETS | SECURITY GROUPS
Let us create a directory structure

Open your Visual Studio Code and:

Create a folder called PBL

Create a file in the folder, name it main.tf

### Provider and VPC resource section
Set up Terraform on your PC and configure your AWS with the AWS toolkit extension

Add AWS as a provider, and a resource to create a VPC in the main.tf file.

Provider block informs Terraform that we intend to build infrastructure within AWS.

Resource block will create a VPC.

Note: You can change the configuration above to create your VPC in other region that is closer to you. The same applies to all configuration snippets that will follow.

According to our architectural design, we require 6 subnets:

2 public
2 private for webservers
2 private for data layer

Let us create the first 2 public subnets.

The main.tf file should look like this 

```
provider "aws" {
  region = "us-west-1"
  profile = "Your IAM User"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
}

# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-west-1a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-west-1b"
}

```



The next thing we need to do, is to download necessary plugins for Terraform to work. These plugins are used by providers and provisioners. At this stage, we only have provider in our main.tf file. So, Terraform will just download plugin for AWS provider.

Run `terraform init`

<img width="959" alt="image" src="https://github.com/Shubsdev/Devops-Projects/assets/102925329/9087a9c8-5804-47ea-af67-ae791b925db7">

Notice that a new directory has been created: .terraform.... This is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means that you must execute terraform init again, to download them.

we should check to see what terraform intends to create before we tell it to go ahead and create it.

Run `terraform plan`

<img width="935" alt="image" src="https://github.com/Shubsdev/Devops-Projects/assets/102925329/fc51b60a-70d2-4bc3-98d8-28b02328e600">


Then, if you are happy with changes planned, execute `terraform apply`

![image](https://github.com/Shubsdev/Devops-Projects/assets/102925329/745d160a-2523-4b7c-aae4-c2b7dad3c7b2)

Observations:

Hard coded values: Remember our best practice hint from the beginning? Both the availability_zone and cidr_block arguments are hard coded. We should always endeavour to make our work dynamic. Multiple Resource Blocks: Notice that we have declared multiple resource blocks for each subnet in the code. This is bad coding practice. We need to create a single resource block that can dynamically create resources without specifying multiple blocks. Now let us improve our code by refactoring it.

First, destroy the current infrastructure. Since we are still in development, this is totally fine. Otherwise, DO NOT DESTROY an infrastructure that has been deployed to production.

To destroy whatever has been created run terraform destroy command, and type yes after evaluating the plan.

<img width="928" alt="image" src="https://github.com/Shubsdev/Devops-Projects/assets/102925329/4afd99cd-afa8-4565-a135-7da769cf77d8">




## Fixing problems by code refactoring

Starting with the provider block, declare a variable named region, give it a default value, and update the provider section by referring to the declared variable.

```
variable "region" {
        default = "us-west-1"
    }

    provider "aws" {
        region = var.region
    }
```

We would do the same to cidr value in the vpc block, and all the other arguments.

```
variable "region" {
        default = "us-west-1"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_support

    }
```

Terraform has a functionality that allows us to pull data which exposes information to us. Let us fetch Availability zones from AWS, and replace the hard coded value in the subnet’s availability_zone section.

```
# Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}
```

To make use of this new data resource, we will need to introduce a count argument in the subnet block

```
# Create public subnet1
resource "aws_subnet" "public" { 
    count                   = 2
    vpc_id                  = aws_vpc.main.id
    cidr_block              = "172.16.1.0/24"
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]

}
```

But we still have a problem. If we run Terraform with this configuration, it may succeed for the first time, but by the time it goes into the second loop, it will fail because we still have cidr_block hard coded. The same cidr_block cannot be created twice within the same VPC. So, we have a little more work to do.

We will introduce a function cidrsubnet() to make this happen. It accepts 3 parameters.

```
  # Create public subnet1
  resource "aws_subnet" "public" { 
      count                   = 2
      vpc_id                  = aws_vpc.main.id
      cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
      map_public_ip_on_launch = true
      availability_zone       = data.aws_availability_zones.available.names[count.index]

  }
```
We can introuduce length() function, which basically determines the length of a given list, map, or string.

Since data.aws_availability_zones.available.names returns a list like ["us-west-1a", "us-west-1b", "us-west-1c"] we can pass it into a lenght function and get number of the AZs.

Now we can simply update the public subnet block like this

```
resource "aws_subnet" "public" { 
    count                   = length(data.aws_availability_zones.available.names)
    vpc_id                  = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]

}
```

What we have now, is sufficient to create the subnet resource required. But if you observe, it is not satisfying our business requirement of just 2 subnets. The length function will return number 3 to the count argument, but what we actually need is 2. Now, let us fix this.

Declare a variable to store the desired number of public subnets, and set the default value

```
variable "preferred_number_of_public_subnets" {
  default = 2
}
```

Next, update the count argument with a condition. Terraform needs to check first if there is a desired number of subnets. Otherwise, use the data returned by the lenght function. See how that is presented below.

```
# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}
```
The first part var.preferred_number_of_public_subnets == null checks if the value of the variable is set to null or has some value defined.

The second part ? and length(data.aws_availability_zones.available.names) means, if the first part is true, then use this. In other words, if preferred number of public subnets is null (Or not known) then set the value to the data returned by lenght function.

The third part : and var.preferred_number_of_public_subnets means, if the first condition is false, i.e preferred number of public subnets is not null then set the value to whatever is definied in var.preferred_number_of_public_subnets

Now the entire configuration should now look like this

```
# Get list of availability zones
data "aws_availability_zones" "available" {
state = "available"
}

variable "region" {
      default = "us-west-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

variable "preferred_number_of_public_subnets" {
      default = 2
}

provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support

}

# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}
```
