# Terraform CLI Cheat Sheet

## [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#about-terraform-cli)About Terraform CLI

Terraform, a tool created by  [Hashicorp](https://www.hashicorp.com/)  in 2014, written in Go, aims to build, change and version control your infrastructure. This tool have a powerfull and very intuitive Command Line Interface.

## [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#installation)Installation

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#install-through-curl)Install through curl

    $ curl -O https://releases.hashicorp.com/terraform/
    0.11.10/terraform_0.11.10_linux_amd64.zip
    $ sudo unzip terraform_0.11.10_linux_amd64.zip
     -d /usr/local/bin/
    $ rm terraform_0.11.10_linux_amd64.zip

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#or-install-through-tfenv-a-terraform-version-manager)OR install through tfenv: a Terraform version manager

First of all, download the tfenv binary and put it in your PATH.

    $ git clone https://github.com/Zordrak/tfenv.git
     ~/.tfenv
    $ echo 'export PATH="$HOME/.tfenv/bin:$PATH"'
     >> $HOME/.bashrc

Then, you can install desired version of terraform:

    $ tfenv install 0.11.10

## [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#usage)Usage

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#show-version)Show version

    $ terraform --version
     Terraform v0.11.10

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#init-terraform)Init Terraform

    $ terraform init

It’s the first command you need to execute. Unless, terraform plan, apply, destroy and import will not work. The command terraform init will install :

-   terraform modules
    
-   eventually a backend
    
-   and provider(s) plugins
    

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#init-terraform-and-dont-ask-any-input)Init Terraform and don’t ask any input

    $ terraform init -input=false

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#change-backend-configuration-during-the-init)Change backend configuration during the init

    $ terraform init -backend-config=cfg/s3.dev.tf -reconfigure

`-reconfigure`  is used in order to tell terraform to not copy the existing state to the new remote state location.

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#get)Get

This command is useful when you have defined some modules. Modules are vendored so when you edit them, you need to get again modules content.

    $ terraform get -update=true

When you use modules, the first thing you’ll have to do is to do a terraform get. This pulls modules into the .terraform directory. Once you do that, unless you do another  `terraform get -update=true`, you’ve essentially vendored those modules.

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#plan)Plan

The plan step check configuration to execute and write a plan to apply to target infrastructure provider.

    $ terraform plan -out plan.out

It’s an important feature of Terraform that allows a user to see which actions Terraform will perform prior to making any changes, increasing confidence that a change will have the desired effect once applied.

When you execute terraform plan command, terraform will scan all *.tf files in your directory and create the plan.

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#apply)Apply

Now you have the desired state so you can execute the plan.

    $ terraform apply plan.out

**Good to know:**  Since terraform v0.11+, in an interactive mode (non CI/CD/autonomous pipeline), you can just execute  `terraform apply`  command which will print out which actions TF will perform.

By generating the plan and applying it in the same command, Terraform can guarantee that the execution plan won’t change, without needing to write it to disk. This reduces the risk of potentially-sensitive data being left behind, or accidentally checked into version control.

    $ terraform apply

#### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#apply-and-auto-approve)Apply and auto approve

    $ terraform apply -auto-approve

#### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#apply-and-define-new-variables-value)Apply and define new variables value

    $ terraform apply -auto-approve
    -var tags-repository_url=${GIT_URL}

#### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#apply-only-one-module)Apply only one module

    $ terraform apply -target=module.s3

This  _-target_  option works with  _terraform plan_  too.

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#destroy)Destroy

    $ terraform destroy

Delete all the resources!

A deletion plan can be created before:

    $ terraform plan –destroy

`-target`  option allow to destroy only one resource, for example a S3 bucket :

    $ terraform destroy -target aws_s3_bucket.my_bucket

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#debug)Debug

The Terraform console command is useful for testing interpolations before using them in configurations. Terraform console will read configured state even if it is remote.

$ echo "aws_iam_user.notif.arn" | terraform console
arn:aws:iam::123456789:user/notif

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#graph)Graph

    $ terraform graph | dot –Tpng > graph.png

Visual dependency graph of terraform resources.

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#validate)Validate

Validate command is used to validate/check the syntax of the Terraform files. A syntax check is done on all the terraform files in the directory, and will display an error if any of the files doesn’t validate. The syntax check does not cover every syntax common issues.

    $ terraform validate

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#providers)Providers

You can use a lot of providers/plugins in your terraform definition resources, so it can be useful to have a tree of providers used by modules in your project.

    $ terraform providers
    .
    ├── provider.aws ~> 1.24.0
    ├── module.my_module
    │   ├── provider.aws (inherited)
    │   ├── provider.null
    │   └── provider.template
    └── module.elastic
        └── provider.aws (inherited)

## [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#state)State

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#pull-remote-state-in-a-local-copy)Pull remote state in a local copy

    $ terraform state pull > terraform.tfstate

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#push-state-in-remote-backend-storage)Push state in remote backend storage

    $ terraform state push

This command is usefull if for example you riginally use a local tf state and then you define a backend storage, in S3 or Consul…​

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#how-to-tell-to-terraform-you-moved-a-ressource-in-a-module)How to tell to Terraform you moved a ressource in a module?

If you moved an existing resource in a module, you need to update the state:

    $ terraform state mv aws_iam_role.role1 module.mymodule

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#how-to-import-existing-resource-in-terraform)How to import existing resource in Terraform?

If you have an existing resource in your infrastructure provider, you can import it in your Terraform state:

    $ terraform import aws_iam_policy.elastic_post
    arn:aws:iam::123456789:policy/elastic_post

## [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#workspaces)Workspaces

To manage multiple distinct sets of infrastructure resources/environments.

Instead of create a directory for each environment to manage, we need to just create needed workspace and use them:

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#create-workspace)Create workspace

This command create a new workspace and then select it

    $ terraform workspace new dev

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#select-a-workspace)Select a workspace

    $ terraform workspace select dev

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#list-workspaces)List workspaces

    $ terraform workspace list
      default
    * dev
      prelive

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#show-current-workspace)Show current workspace

    $ terraform workspace show
    dev

## [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#tools)Tools

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#jq)jq

jq is a lightweight command-line JSON processor. Combined with terraform output it can be powerful.

#### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#installation-1)Installation

For Linux:

    $ sudo apt-get install jq

or

    $ yum install jq

For OS X:

    $ brew install jq

#### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#usage-1)Usage

For example, we defind outputs in a module and when we execute  _terraform apply_  outputs are displayed:

    $ terraform apply
    ...
    Apply complete! Resources: 0 added, 0 changed,
     0 destroyed.
    
    Outputs:
    
    elastic_endpoint = vpc-toto-12fgfd4d5f4ds5fngetwe4.
    eu-central-1.es.amazonaws.com

We can extract the value that we want in order to use it in a script for example. With jq it’s easy:

    $ terraform output -json
    {
        "elastic_endpoint": {
            "sensitive": false,
            "type": "string",
            "value": "vpc-toto-12fgfd4d5f4ds5fngetwe4.
            eu-central-1.es.amazonaws.com"
        }
      }
    
    $ terraform output -json | jq '.elastic_endpoint.value'
    "vpc-toto-12fgfd4d5f4ds5fngetwe4.eu-central-1.
    es.amazonaws.com"

### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#terraforming)Terraforming

If you have an existing AWS account for examples with existing components like S3 buckets, SNS, VPC … You can use terraforming tool, a tool written in Ruby, which extract existing AWS resources and convert it to Terraform files!

#### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#installation-2)Installation

    $ sudo apt install ruby`  or  `$ sudo yum install ruby

and

    $ gem install terraforming

#### [](https://github.com/scraly/terraform-cheat-sheet/blob/master/terraform-cheat-sheet.adoc#usage-2)Usage

Pre-requisites :

Like for Terraform, you need to set AWS credentials

    $ export AWS_ACCESS_KEY_ID="an_aws_access_key"
    $ export AWS_SECRET_ACCESS_KEY="a_aws_secret_key"
    $ export AWS_DEFAULT_REGION="eu-central-1"

You can also specify credential profile in  _~/.aws/credentials_s and with _–profile_  option.

    $ cat ~/.aws/credentials
    [aurelie]
    aws_access_key_id = xxx
    aws_secret_access_key = xxx
    aws_default_region = eu-central-1
***
    $ terraforming s3 --profile aurelie

Usage

    $ terraforming --help
    Commands:
    terraforming alb # ALB
    ...
    terraforming vgw # VPN Gateway
    terraforming vpc # VPC

Example:

    $ terraforming s3 > aws_s3.tf

Remarks: As you can see, terraforming can’t extract for the moment API gateway resources so you need to write it manually.
