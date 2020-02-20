+++
title = "Hosting a static site on AWS S3 (Part 3)"
date = 2020-02-20T10:32:58.029Z
author = "Daniel"
cover = ""
tags = []
keywords = ["Tech", "AWS", "Hosting on S3", "Terraform", "Infrastructure-as-Code"]
description = "In the third part of this ongoing series, we will have a look at the project structure, how to structure your code and how to create modules to keep your code reusable."
showFullContent = false
+++

[Last time]({{< ref "/posts/static-site-hosting-pt2.md">}}) we had a look at infrastructure-as-code and
the terraform workflow as well as how to facilitate (remote) state handling.

With the basics now out of the way, lets look at this sites project structure. You can find the complete code
for this series on github: [bytelike-de-infra](https://github.com/dontpanic42/bytelike-de-infra/tree/v1.0)

```bash
├── modules
│   ├── acm
│   │   ├── acm.tf
│   │   ├── outputs.tf
│   │   ├── providers.tf
│   │   └── variables.tf
│   ├── cloudfront
│   │   ├── cloudfront.tf
│   │   └── variables.tf
│   ├── oai
│   │   ├── oai.tf
│   │   └── outputs.tf
│   └── s3
│       ├── outputs.tf
│       ├── s3.tf
│       └── variables.tf
├── README.md
├── backend.tf
├── main.tf
├── providers.tf
└── variables.tf
```

As you can see, we have a fairly simple structure. This is in no way the end-all way things have to be done,
but experience shows it works well for smaller projects.

In the root directory we find our base terraform scripts. We have the `backend.tf` which contains the 
backend configuration we discussed in [part 2]({{< ref "/posts/static-site-hosting-pt2.md">}}). Further more
we have a file called `providers.tf`, which contains our provider config:

```hcl
provider "aws" {
  alias  = "default"
  region = "eu-central-1"
}

provider "aws" {
  alias  = "us_east_1"
  region = "us-east-1"
}
```

Here we define our base aws region (eu-central-1) as well as a second provider (`us_east_1`), which we will
use to provision our certificates.

Going further through the root directory, we find another file called `variables.tf`. The variables file
contains the definition and default values for all the toplevel configuration options our script
provides. Note that you can override the value for all the variables later with a `.tfvars` file
if you want to roll out eg. different environments with different configuration. Looking at our
`variables.tf` reveals only two variables:

```hcl
variable "page_domain_name" {
  description = "Domain name of the webiste (eg. bytelike.de)"
  type        = string
  default     = "bytelike.de"
}

variable "resource_tags" {
  description = "Tags attached to all resources"
  type        = map(string)
  default = {
    "env" = "prod"
    "app" = "bytelike.de"
  }
}
```

Here we define two variables, one called `page_domain_name` which is a string that defaults to
the page name (`bytelike.de`), and another one called `resource_tags` which is a map that contains
all the tags that should be attached to all created resources. Usecase for default tags is to separate
different applications in the same account, differentiate between environments (eg. dev/qa/prod) or - in
larger organizations - for billing purposes.

Note that there is no requirement to definie variables in a separate file (or, while we are at it, to
split anything into different files). When Terraform looks at your project directory it merges all
files in one directory into one big file which then gets evaluated. But separating things into logical
units and different files is never the less makes everything much more readable, keeps file sizes
small and is something that just worked for me in the past.

The last file in the root directory, `main.tf` is our central entrypoint.

```hcl
module "oai" {
  source = "./modules/oai"
}

module "s3" {
  source           = "./modules/s3"
  page_domain_name = var.page_domain_name
  resource_tags    = var.resource_tags
  oai_iam_arn      = module.oai.iam_arn
}

...
```

(Rememver that you can find all the files discussed in this series [here](https://github.com/dontpanic42/bytelike-de-infra/tree/v1.0)).

The `main.tf` calls all our submodules, for example one called "oai" (short for Origin Access Identity) and
one for our source S3-Bucket, simply called "s3". Don't despair if you don't know what a "Origin Access
Identity" is - we will talk about that later.

Some of you might have noticed that the modules refer to the one folder in our directory structure, `modules`.
Terraform modules are pretty much what you expect - a way to write code that is (in theory) reusable. Every module
has input variables and ouputs. And if you look at the directory strucutre in the S3 module, you will see exactly that:

```bash
├── modules
...
│   └── s3
│       ├── outputs.tf
│       ├── s3.tf
│       └── variables.tf
```

We have again a file called `variables.tf` which serves the same purpose than the top level file with the same
name - it contains all the input variables that this module supports. Next to the variables file there is a file 
`s3.tf` which contains all the resources that are managed by this module. We will have a deeper look into
this file next time. At last there is a file called `outputs.tf` which contains all the outputs this module provides.

Outputs are a construct that is specific to modules. They do exactly what you would think they do - the allow you
to output certain values to be referenced by other code. Looking through our `main.tf`, you might have noticed
the following line in the "cloudfront" module:

```hcl
module "cloudfront" {
  ...
  bucket_domain_name = module.s3.bucket_domain_name
  ...
}
```

The "cloudfront" module expects an input variable with the name `bucket_domain_name`. Since we don't know the
bucket domain name before creating the bucket, we need to reference a value that is usually hidden inside the
"s3" module. 

```hcl
# Output definition in ./modules/s3/outputs.tf

output "bucket_domain_name" {
  description = "Regional domain name of this bucket"
  value       = aws_s3_bucket.site_file_bucket.bucket_regional_domain_name
}
```

But since we defined an output "bucket_domain_name" in the "s3" module, we can just reference it
using the `module.[NAME_OF_THE_SOURCE_MODULE].[NAME_OF_THE_REFERENCED_OUTPUT]` syntax.

In the next part we will have a deeper look into our modules and what actually makes our
infrastrukture tick.

- [Part 1: Introduction]({{< ref "/posts/static-site-hosting.md">}})
- [Part 2: Infrastructure-as-Code and Terraform]({{< ref "/posts/static-site-hosting-pt2.md">}})
- Part 3: Project Structure and Modules




