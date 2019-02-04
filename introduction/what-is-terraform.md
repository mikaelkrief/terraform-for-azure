---
description: Terraform presentation
---

# What is Terraform

Each cloud provider uses its own Infrastructure As Code language, for Azure it is a Json file called ARM Templates, for AWS it is CloudFormation, etc.

Terraform is a solution developed by HashiCorp. It is a command line tool that uses the same language, the same syntax to deploy on multiple cloud providers. This language is called HCL for HashiCorp Configuration Language. It is declarative and easy for a human to read.

Here HCL code sample

```text
resource "azurerm_resource_group" "rg" {
  name     = "rg_app"
  location = "northeurope"
}
```

With Terraform, all you have to do is declare the resources and how you want them to be configured, then it will set up the dependencies and create everything for you!

Terraform is an open source tool written in Go whose source code is available here [https://github.com/hashicorp/terraform](https://github.com/hashicorp/terraform), it consists of an executable that can be installed on all platforms \(Linux, Windows, etc.\). The solution is maintained by a very active community.

Terraform also allows:

* A dry run mode \(plan\) execution that allows you to visualize changes on the infrastructure before actually applying them. 
* Parallelization of operations. 
* The ability to generate a graph of the resources described in the code.

