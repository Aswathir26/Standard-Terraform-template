# Standard-Terraform-template

**Introduction**

A Terraform template is a pre-created terraform code that already has some formatting. 
Rather than starting from scratch to format a terraform code, we can use the formatting of a terraform template to save yourself a lot of time. 
We can use a template that is already created and need to change the code according to our preference.

Here we created a template with two directories in it.
These are the directories and files holding inside the directories.

1. build_infra

	  -backend.tf
    
	  -inputs.tf

    -outputs.tf
	  
    -provider.tf
	  
    -random.tf

2. modules
 
      random_string (sub directory)
	    
      -main.tf
	    
      -outputs.tf
	    
      -variables.tf
      
The purpose of creating this template is to refer and also copy and change the code when a new terraform code was to build.
 
In this template it includes,

-   how to develop code as module.
- 	how to use multiple workspaces.
- 	how to declare variables.
- 	how to create storage in azure.

**Create and manage workspaces/environments**

workspaces are separate instances of state data that can be used from the same working directory. You can use workspaces to manage multiple non-overlapping groups of resources with the same configuration. Every initialized working directory has at least one workspace.

There will be a default workspace and also we can create and use workspaces as we needed.

Here including default workspace, we created and used 2 more workspaces. i.e. 'dev' and 'test'.

command to create and switch to a workspace:

    $ terraform workspace new dev
    $ terraform workspace new test

So here we add different values in both workspaces to test differently and obtain the output.
Here we can switch to different workspace considering which want to test.

example : terraform workspace select dev

**STEPS:**

**1.Create Directories**

Here we are creating a terraform code to get random string. So Create two directories one for main body(build_infra) and another for modules(modules). In modules directory we have to add directories as much as modules we using. here we only one module is using(random_string)

      $ mkdir build_infra
      $ mkdir modules 
      $ cd modules
      $ mkdir random_string


**2.Create Files**

- In build_infra directory we have to add files for main(here random), provider(azure), inputs and outputs
 
random.tf:

          module "random_string" {
            source = "../modules/random_string"

            length = local.random_string_input["length"]
          }
           This is the code to call module random_string. And defined variable length of string.

inputs.tf:

          locals {
            random_string_input_raw = {
              dev = {
                length = 10
              }
              test = {
                length = 11
              }
            }
            random_string_input = local.random_string_input_raw[terraform.workspace]
          }

Terraform variables can be defined within the infrastructure plan but are recommended to be stored in their own variables file. Input Variables serve as parameters for a Terraform module, so users can customize behavior without editing the source.

Here two different workspace is created and and different values are added to both. So data from these workspace are initialized into a random input variable using local. This value is initialized to the input variable using local.

outputs.tf:

          output "result" {
            description = "(String) The generated random string"
            value       = module.random_string.result
          }

Here we can define what are the things we want as a result. 

provider.tf:

          terraform {
            required_providers {
              azurerm = {
                source  = "hashicorp/azurerm"
                version = "=2.91.0"
              }
            }
          }
          provider "azurerm" {
            features {}
          }
          terraform {
            backend "azurerm" {
              resource_group_name  = "Backend-State"
              storage_account_name = "mystorageaccount"
              container_name       = "mybackend"
              key                  = "tf_template.tfstate"
            }
          }
          
          
 We can store this in a container in backed as we mention in provider.tf
 
- In random_string module we have to create files to provide variables(main.tf) to define variables(variables.tf) and for output(output.tf)
we will call this module in build_infra directory.  
 
**3.Build and Test**

commands:

        $ az login
        $ az account set --subscription <subscription id>
        $ terraform init
        $ terraform workspace new test
        $ terraform workspace new dev
        $ terraform plan
        $ terraform apply (inorder to initialize and apply the code)

commands to add repo in Azure DevOps:

        $ git init
        $ git remote add origin <link of repo>
        $ add . 
        $ git push  origin master(inorder to add file to the repository)
