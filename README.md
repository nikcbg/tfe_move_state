# tfe_move_state

### Purpose of the repository
- This repository is an example on how to move items in Terraform state. 


### List of the files in the repository
File name |	File description
----------|--------------------
folder1/main.tf	| Terraform configuration code.
folder2/module.tf | Terraform configuration code.

### How to use this repository 
- Install `terraform` by following this [instructions](https://www.terraform.io/intro/getting-started/install.html).
- Clone the repository to your local computer: `git clone git@github.com:nikcbg/tfe_move_state`.
- Go into the cloned repo on your computer: `cd tfe_move_state`.
- Create a new directory(folder1) inside `tfe_move_state` and go into the new directory.
- Into the new directory folder1 create a `main.tf` file with the following code:

```
resource "random_pet" "name" {
 length    = "4"
 separator = "-"
}

resource "null_resource" "hello" {
  provisioner "local-exec" {
    command = "echo Hello ${random_pet.name.id}"
  }
}

```
- Next execute `terraform init` to download the necessary plugins.
- Next execute `terraform plan` to create execution plan.
- Next execute `terraform apply` to create the new resources.
- The output should display:
```
 null_resource.hello: Creating...
null_resource.hello: Provisioning with 'local-exec'...
null_resource.hello (local-exec): Executing: ["/bin/sh" "-c" "echo Hello sharply-only-positive-sunbeam"]
null_resource.hello (local-exec): Hello sharply-only-positive-sunbeam
null_resource.hello: Creation complete after 0s (ID: 1687949221779117505)

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```
### Module creation and move the state

- Create a new directory in `tfe_move_state` and name it `folder2`.
- Create a file `module.tf` in `folder2` directory with the following code:

  ```
   resource "random_pet" "name" {
   length    = "4"
   separator = "-"
   }

  output "display" {
  value = "${random_pet.name.id}"
  }

  ```
- Next edit the `main.tf` file located in `folder1` directory with the following code:

  ```
  module "example"{
  source = "../folder2"
  }
  resource "null_resource" "hello" {
  provisioner "local-exec" {
  command = "echo Hello ${module.example.display}"
  }
  }

  ```
- Next execute `terraform init` to to download the necessary plugins.
- Next move the `random_pet` resorce in `module.tf` file to the updated `main.tf` module file with the following command:
  - `terraform state mv random_pet.name module.example`, output shoudl display:
  ```
  Moved random_pet.name to module.example
  ```
- Next execute `terraform apply`, output should display:

  ```
  random_pet.name: Refreshing state... (ID: wrongly-daily-outgoing-camel)
  null_resource.hello: Refreshing state... (ID: 1929874788600161557)

  Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

  ```
### Move the state to Terraform Enterprise
- Next integrate your TFE with version control system (in this case GitHub) following this [instructions](https://www.hashicorp.com/resources/getting-started-with-terraform-enterprise#step-4-create-a-workspace).
- Next create a work space in TFE following this [instructions](https://www.hashicorp.com/resources/getting-started-with-terraform-enterprise#step-4-create-a-workspace).
- Next link your workspace with this repository by choosing GitHub as source and the repository name underneath it. 
- Next we need to authenticate to TFE so the existing state on your local computer gets copied to TFE:
  - go to TFE website and click on user settings in upper right corner.
  - then click on tokens on the left pane.
  - next name your token and click generate token.
  - make sure you copy and download your token and save it in secure place for future use.
- Next execute `export ATLAS_TOKEN=your_TFE_token_here` to authenticate with TFE.
- Next execute `terraform init` to initialize the new backend.
- Next execute `terraform apply`, teh output should display:

```
random_pet.name: Refreshing state... (ID: sharply-only-positive-sunbeam)
null_resource.hello: Refreshing state... (ID: 1687949221779117505)

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

```
- You can see that no resources have been created. That's becasue the terraform state remains the same and resources were already created when we ran `main.tf` file. 

 
### To Do: 
- Check if everything works as expected. 
