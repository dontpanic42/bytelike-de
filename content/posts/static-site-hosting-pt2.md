+++
title = "Hosting a static site on AWS S3 (Part 2)"
date = 2020-02-16T10:32:58.029Z
author = "Daniel"
cover = ""
tags = []
keywords = ["Tech", "AWS", "Hosting on S3", "Terraform", "Infrastructure-as-Code"]
description = "In the second part of the series, we will look at the Infrastructure-as-Code paradigm, why it is a good idea and how to use terraform to implement it."
showFullContent = false
+++

In the [last installment]({{< ref "/posts/static-site-hosting.md">}}) of this series, we had a look at 
the architecture and all the services I used when creating the infrastructure for this site.

When trying things out and experimenting with AWS, it's perfectly fine to click things together in the AWS web
console. But in a more professional setting, where outages can cost millions in short time, this is usually
not an option. And especially when you work in a regulated industry (eg. finance), you will need to bea able 
to find out who did what when to which environment.

As you probably know, for software development we have git, a version controll system. Git enables us to 
collaboratively work on a piece of code, revert changes when they turn out a mistake, look through the editing
history and - most importantly - it lets us proove to our coworker that we are not in fact responsible for 
the horrible mess that caused the bug of the day :-). Wouldn't it be nice to have this for our infrastructure
too?

This is where Infrastructure-as-Code comes into play.

If you never used [Terraform](https://www.terraform.io/), here is a quick introduction. Terraform is a tool that
lets you describe your infrastructure in a pretty simple language called 
[HashiCorp Configuration Language (HCL)](https://www.terraform.io/docs/configuration/index.html). HCL is a
declarative language, meaning you just tell it what you want to have (a "desired state") and it will do its best
to transform your current infrastructure into what you want it to be. This can be a destructive process. Eg. when
you have a MySQL Database Server and want it to be an Oracle Database Server it won't convert schemas for you. 
Instead it will just delete the MySQL Database Server and create a new Oracle Database Server. But more often than 
not it will actually be able to modify your setup in a nondestructive way. Eg. when you change a new DNS record,
it will not delete your hosted zone but modify the existing record to your liking.

Working with Terraform is usually a three step process.

First you initialize your project with `terraform init`. This will look through your code and download all required
terraform plugins automatically. In our case the "AWS" plugin (oh yeah, did I mention terraform supports multiple
clouds like Azure and Google Cloud? And also On-Prem stuff like VMWare?). Next it will look through
your modules and initialize them. If you get errors during this step, a good first response ist to delete your 
`.terraform` folder and running `terraform init` again.

When `terraform init` has run sucessfully, next you will want to run `terraform plan --out mypaln`. This will 
show you the delta between the current state of your infrastructure and the desired state and also save it in a file called `myplan`. I.e. it will show 
you everything it will do when you actually apply your changes without touching anything yet. In this step it 
is very important to look through the changes and see if they make sense to you. Big red `-` blocks
indicate large destructive changes. Also look out for `-/+` blocks which are terraforms way to tell you it won't
be able to change a resource in place but will delete and recreate it instead. Ok for, e.g., a security group, bad
for a production database server. Also be on the lookout for typos. An erroneous `xxlarge` instead of an `xlarge`
might mean a few thousand bucks more than you expected on your bill.

When you are sure you will get the intended results, run `terraform apply myplan`. This will take the results from
your previous `terraform plan` run and apply them to your infrastructure. Note that this can take a while. A
CloudFront distribution for example takes up to 30 minutes to apply changes, a large Multi-AZ Database Cluster
can take up to a few hours. It's important to not interrupt terraform during this time, since it might corrupt
your *state file*.

The *state file* is Terraforms way to keep track of the current state of your infrastructure. Why does Terraform
not just fetch the complete state from an API instead? Well it actually tries before planning (and there is 
even a command for that, `terraform refresh`), but it won't be able to fetch everything. Imagine,
for example, you deploy an RDS database. During the creation you will have to provide a master user password. This
password - obviously - cannot be read from the API. So how should Terraform know what the current state of the
password is and if it changed in your desired state? It can't, so it keeps an inventory, which we call state file.
(By the way, there are ways to prevent Terraform from saving sensitive data in your state file since you probably
don't want to save passwords in plain text.)

There are mutliple ways to handle state files. The easiest one is the default, terraform will just create a file
`terraform.tfstate` in your local working directory. But this is usually not sufficient. What if your colleague
is working on the same infrastructure? Then you and him have different versions of the state file which will
inevitably lead to horrible things. Therefore most terraform projects create a backend configuration. The one for 
this site looks like this:

```hcl
terraform {
  backend "s3" {
    bucket = "bytelike.de-state"
    key    = "bytelike.de.tfstate"
    region = "eu-central-1"
  }

  ...
}
```

Here we tell Terraform that is should create a state file called `bytelike.de.tfstate` in an AWS S3 bucket called
`bytelike.de-state`. This way, when ever somebody tries to `terraform plan` or `terraform apply`, terraform will
always first fetch the latest state from S3. And also save the changed state back to S3 afterwards. 
There are more advanced mechanisms, like state locking with a DynamoDB table, but those are only really 
required when working with a larger team.

Summing up - you can follow the Infrastrucutre-as-Code paradigm to apply standard software development tools
and processes (like git and gitflow) to your (cloud-) infrastructure. You can use Terraform and HCL for
the implementation.

Next time we will have a look into the terraform code that makes up the infrastructure for this site.

- [Part 1: Introduction]({{< ref "/posts/static-site-hosting.md">}})
- Part 2: Infrastructure-as-Code and Terraform
- [Part 3: Project Structure and Modules]({{< ref "/posts/static-site-hosting-pt3.md">}})
