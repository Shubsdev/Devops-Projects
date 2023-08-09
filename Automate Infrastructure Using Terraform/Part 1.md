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

The next thing we need to do, is to download necessary plugins for Terraform to work. These plugins are used by providers and provisioners. At this stage, we only have provider in our main.tf file. So, Terraform will just download plugin for AWS provider.
![image](https://github.com/Shubsdev/Devops-Projects/assets/102925329/c3073353-a792-4e79-9803-97f6772ab2ab)

Notice that a new directory has been created: .terraform.... This is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means that you must execute terraform init again, to download them.

Moving on, let us create the only resource we just defined. aws_vpc. But before we do that, we should check to see what terraform intends to create before we tell it to go ahead and create it.

Run terraform plan

Then, if you are happy with changes planned, execute terraform apply
<img width="935" alt="image" src="https://github.com/Shubsdev/Devops-Projects/assets/102925329/fc51b60a-70d2-4bc3-98d8-28b02328e600">
![image](https://github.com/Shubsdev/Devops-Projects/assets/102925329/745d160a-2523-4b7c-aae4-c2b7dad3c7b2)

<img width="928" alt="image" src="https://github.com/Shubsdev/Devops-Projects/assets/102925329/4afd99cd-afa8-4565-a135-7da769cf77d8">
