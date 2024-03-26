# Terraform Crash Course - Tulsa Web Devs
**Date of Event**: March 26th, 2024

## Introduction

### IaC? Terraform?
Infrastructure as Code (IaC) is the managing and provisioning of infrastructure through code instead of through manual processes. By using code and software to automate the infrastructure creation process, you can reduce errors in redeploying infrastructure, leverage tools like git for version control, and eliminate configuration drift. 

Just to note, Terraform is not the only Infrastructure as Code tool available. There are other existing tools such as Pulumi, Crossplane, Ansible, and others. All of them come with their own pros and cons. Some cloud providers have their own cloud-specific Infrastructure as Code tool. AWS has CloudFormation and Azure has Bicep (which is probably the coolest IaC tool name ever). Bt these tools only work within a specific cloud environment. If you are working with multi-cloud environments, you might be better off using an IaC tool like Ansible, Pulumi, or Terraform. 

### But What About Open Tofu?
OpenTofu is a fork of Terraform project maintained by The Linux Foundations Project since HashiCorp (the creator of Terraform) switched the license of Terraform from an open-source license to a source available license. If you'd rather use OpenTofu instead of Terraform, you should be able to follow along with this tutorial. Just replace `terraform` with `tofu` when running your terminal commands.

## Installing Terraform 
### MacOS
Installing Terraform on MacOS is very simple if you have the package manager Homebrew. Install it via homebrew by using the following command:
```
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

### Windows
Installing Terraform on Windows OS is also pretty simple if you have the package manager Chocolatey. Install it via chocolatey by using the following command:
```
choco install terraform
```

### Linux
If you are using a Linux distro, follow the directions at this [link here](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) to install.
*Note: These directions work for those who are using a Chromebook as well. Just follow the Debian installation instructions after activating Linux mode.*

You can check if the installation worked by running the `terraform -version` command in your terminal.


```python
!terraform -version
```

## Write the Plan

Before we can start deploying our infrastructure to cloud providers, we must write the plan. Every Terraform project needs to have a `main.tf` file. You can place all of your Terraform code in this file. But it is generally best practice to separate related infrastructure into separate files and within the main.tf file to only declare your required providers and backend references within the Terraform block.

With this in mind, there are two important parts of the Terraform block: the `backend` and `required providers`. The backend block contains information about where the state of your infrastructure is stored. The state is a document that contains a record of what infrastructure is currently deployed. This helps us see the discrepancies between what is written in the plan and what is currently deployed. For the purpose of this tutorial, we will be using the `local` backend, which is simply a reference to a file on your local computer. Tjere are many other backends you can use to store the state of your infrastructure, including: Azure Blob Storage (azurerm), Consul Key/Value store (consul), Tencent Cloud Object Storage (cos), Google Cloud Storage (gcs), REST API Client (http), Kubernetes secret (kubernetes), Alibaba Cloud Object Storage Service (oss), Postgres database (pg), and Amazon S3 (s3). To learn more about the available backend configurations, [click here](https://developer.hashicorp.com/terraform/language/settings/backends/configuration). 

The provider block allows you to include modules into your project. The most common use of modules is for provisioning resources in specific cloud providers. You can find the list of available modules at [registry.terraform.io](https://registry.terraform.io/). 



```python
%%writefile main.tf
terraform {
    required_providers {
        github = {
            source = "integrations/github"
            version = "6.1.0"
        }
    }

    backend "local" {
        path = "terraform.tfstate"
    }
}

provider "github" {
    token = var.token
    owner = var.github_organization
}
```

Terraform plans consist of five primary block types: `variable` blocks, `data` blocks, `resource` blocks, `local` blocks, and `output` blocks. We will explore four of the five block types in this tutorial. 

Variable blocks allow for you to customize the behavior of your modules without having to directly edit the plan. There are six optional parameters you can set when declaring a variable block: `type`, `description`, `default`, `validation`, `sensitive` and `nullable`.The `type` parameter indicates the type of data stored in the variable. There are six types in Terraform: `string`, `number`, `boolean`, `map` (or object), `list`, and `set`. The `description` is where you can provide a helpful description of what the variable is supposed to be for. The `default` parameter determines what the variable will be if the value is not defined elsewhere. The `validation` parameter allows you to define specific requirements a variable value must be. The `sensitive` parameter allows you to hide sensitive variables from being stored in the state. And `nullable` allows you to define whether the variable can be null or not.


```python
%%writefile variables.tf
variable "token" {
    type = string
    description = "The Github URL Token"
}

variable "github_organization" {
    type = string
    description = "The name of the organization you want to make the Github Pages for"
}

variable "github_repository" {
    type = string
    description = ""
}

variable "forked_repository" {
    type = string
    description = "The name of the repository that we're forking files from"
    default = "EternalLuxury/basic-html-website"
}

variable "favorite_number" {
    type = number
    description = "A simple variable for storing my favorite number"
    default = 8
}

variable "toggle_index" {
    type = bool
    description = "Conditionally include the index file"
    default = true
}

variable "additional_files" {
    type = list(string)
    description = "A list of files to include in repository"
    default = [
        "assets/css/bulma.min.css",
        "assets/css/style.css",
        "assets/js/jquery-3.6.0.js",
        "assets/js/script.js"
    ]
}

variable "example_map" {
    type = map
    description = "A list of files to include in repository"
    default = {
        "name" = "Nile Dixon",
        "age" = 26,
        "tall" = true
    }
}
```

Data blocks are "read-only" resources that can pull information from a provider. The information being read into the plan can be used to help create resources or be outputs for the module. To see what information can be read, check the Terraform registry for the specific provider you're using.

```
#Example structure of data block.
data "type" "name" {}
```


```python
%%writefile data.tf
```

The resource block allows you to create resources from a specific module, which will usually correspond to a resource provided by a provider downloaded from the registry. You create resources by first referencing the type and then creating a user-defined name. Resource-specific parameters are passed within the block. To see what types are available and what their required and optional parameters are, check out the Terraform registry.

```
#Example structure of resource block.
resource "type" "name" {}
```


```python
%%writefile github.tf
```

In some cases, when a resource is created within a module, the output will sometimes need to be an input for another module. The output block allows you to pass a variable from the current module to another module. When you see values of resource blocks being accessed, it is because the module creator made those values outputs for us to import. 

```
#Example structure of resource block.
output "name" {}
```


```python
%%writefile outputs.tf
output "url" {
    value = github_repository.example_repository.html_url
}
```

To help better explain when you would use certain block types, look at the following table for an OOP equivalent mapping:


| Variable Type | OOP Equivalent |
| -----------   | -----------    |
| local         | private        |
| resource      | protected      |
| variable      | parameters     |
| output        | super-public   |

Earlier we created a list of variable blocks. There are a few ways to change the defaults. One way is by passing the variable values into your `terraform plan` and `terraform apply` commands using the `-var` argument. Another way is by creating a JSON file containing the arguments. You would pass that JSON file into your `terraform plan` and `terraform apply` command using the `-var-file` argument. However, if your JSON file is named `variables.auto.tfvars.json`, Terraform will automatically pick it up. There is a hierarchy of the order Terraform reads variables, so just know that when storing default values and creating JSON documents for Terraform to reads as variable files.



```python
%%writefile variables.auto.tfvars.json
{
    "token" : "",
    "github_organization" : "",
    "forked_repository" : "EternalLuxury/basic-html-website",
    "github_repository" : ""
}
```

## Execute the Plan

Before you push your Terraform code to a git repository or share it with others, you might want to make sure your code follows the styling guide. Terraform makes this easy with the `terraform fmt` command. Using the `-recursive` flag applies the formatting to all files in the folder and subfolder.


```python
!terraform fmt -recursive
```

Now that your code is formatted, we're going to initialize our Terraform plan by using the `terraform init` command. Our local configuration referenced in the `main.tf` file is statically referenced in the file. If you want to dynamically inject different configurations, you would use the `-backend-config` flag. The `-backend-config` parameter could be a file path to an HCL file, or it could be a key/value pair string.


```python
!terraform init
```

After initializing the Terraform plan, you can optionally run the `terraform validate` command to check the correctness of the Terraform code. This is more than just simply checking the syntax of the code, as this command checks to see if the appropriate references exist and other code checks. 


```python
!terraform validate
```

After validating your Terraform code, you can optionally run the `terraform plan` command. When running this command, Terraform will compare the state of the infrastructure with the state of the plan. If a resource does not exist in the state but does in the plan, the resource will be created. If a resource does exist in the state but not in the plan, the resource will be deleted. 


```python
!terraform plan
```

Now that the plan has passed, we can run the `terraform apply` command. This will actually deploy the changes. When running this command in the terminal, you will be asked to approve the plan changes by typing yes. You can use the `-auto-approve` flag to bypass the manual approval. But I don't recommend this.


```python
!terraform apply -auto-approve
```

We can delete everything we've made by running the `terraform destroy` command. Running this command can be helpful of clearing out an account without having to delete the code for the plan. Similar to the `terraform apply` command, it will require a confirmation if not using the `-auto-approve` flag. 


```python
!terraform destroy -auto-approve
```

## Have some fun with Terraform

If you are interested in experimenting with Terraform, try using the following Terraform providers. Some of the providers might interface with paid resources. Those resources are denoted below.

- [AWS (Paid)](https://registry.terraform.io/providers/hashicorp/aws/latest)
- [Azure (Paid)](https://registry.terraform.io/providers/hashicorp/azurerm/latest)
- [Docker (Free if run locally)](https://registry.terraform.io/providers/kreuzwerker/docker/latest)
- [GCP (Paid)](https://registry.terraform.io/providers/hashicorp/google/latest)
- [Github (Free)](https://registry.terraform.io/providers/integrations/github/latest)
- [Gitlab (Free)](https://registry.terraform.io/providers/gitlabhq/gitlab/latest)
- [Heroku (Paid)](https://registry.terraform.io/providers/heroku/heroku/latest)
- [Kubernetes (Free if run locally)](https://registry.terraform.io/providers/hashicorp/kubernetes/latest)
- [Snowflake (Paid)](https://registry.terraform.io/providers/Snowflake-Labs/snowflake/latest)
- [Vercel (Paid)](https://registry.terraform.io/providers/vercel/vercel/latest)


## Want to learn more about Terraform?
If you are interested in learning more about Terraform's syntax and how it can be incorporated into your workflow, look at the following books, courses, and videos:
- **Books**
  - [Terraform in Action by Scott Winkler](https://www.amazon.com/Terraform-Action-Scott-Winkler/dp/1617296899/ref=asc_df_1617296899/?tag=hyprod-20&linkCode=df0&hvadid=459709175715&hvpos=&hvnetw=g&hvrand=11464051453519057678&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=1024352&hvtargid=pla-932338106285&psc=1&mcid=6da05ddb860a317eb8edc8c45b7d3955&gclid=EAIaIQobChMIqcTx_57bgwMVOyvUAR0nsgnXEAQYBSABEgJ3rvD_BwE)
  - [Terraform: Up and Running: Writing Infrastructure as Code by Yevgeniy Brikman](https://www.amazon.com/Terraform-Running-Writing-Infrastructure-Code/dp/1098116747/ref=asc_df_1098116747/?tag=hyprod-20&linkCode=df0&hvadid=564675582183&hvpos=&hvnetw=g&hvrand=11464051453519057678&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=1024352&hvtargid=pla-1646950848405&psc=1&mcid=a915c98cefdc3e7598efec21012828c1&gclid=EAIaIQobChMIqcTx_57bgwMVOyvUAR0nsgnXEAQYASABEgJqWPD_BwE)
- **Courses**
  - [HashiCorp Terraform Associate Certification Course by FreeCodeCamp](https://www.youtube.com/watch?v=SPcwo0Gq9T8&pp=ygUJdGVycmFmb3Jt)
  - [Terraform explained in 15 Minutes | Terraform Tutorial for Beginners by TechWorld with Nana](https://www.youtube.com/watch?v=l5k1ai_GBDE)
