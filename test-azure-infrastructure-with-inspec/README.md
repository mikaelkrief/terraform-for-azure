---
description: What is Inspec
---

# Inspec tests for Azure

With the rapid development of DevOps processes and IaC \(Infrastucture As Code\) tools that are becoming more and more diverse Cloud infrastructures can be deployed and provisioned very quickly and with optimal automation.

The problem that often occurs in these deployments is that automation does not guarantee compliance with requirements and safety and functional compliance.  
And so, just as in applications unit tests and integration tests are coded and automatized, it is also necessary to code and automate tests on the infrastructure in order to verify that:

* The infrastructure deployed corresponds well to the architecture and company specifications
* That security policies are properly applied

To write these tests, there are many tools and you can use all the development languages that allow you to interact with your cloud provider.  
For Azure you can use for example [Azure Cli](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) or Powershell Azure commandLet.  
For PowerShell there is a library dedicated to these Azure tests which is [Pester](https://github.com/pester/Pester).

The problem \(in my opinion\) with these tools is that they often require a lot of code lines and are agnostic of the cloud provider \(here Azure\) and of the language.

So I became interested in another tool that is **Inspec** and we will see how to use it to test an Azure infrastructure.

## What is Inspec

[Inspec](https://www.inspec.io/) is an command line, open source tool, provided by [Chef ](https://www.chef.io/)witch audit and automated testing framework for integration, compliance and security.  
It does not require learning a new language, just knowing how to write the desired state of infrastructure resources.

With Inspec we can test the compliance of remotes machines OS , data and since the inspec 2.0 cloud infrastructure like Azure and AWS \(with theses API\) and since the version 3, GCP resources.

## Inspec installation and Azure Cloud shell

**Requirement**: The only requirement for Inspec is to have Ruby \( &gt;2.3 \) installed.

The installation of Inspec can be done by multiple way:

* Manually from the [download page](https://downloads.chef.io/inspec)
* By gem manager or by script , all scripts are located in the [GitHub](https://github.com/inspec/inspec)

inspec command line

![](https://cdn-images-1.medium.com/max/800/1*13d76ycLhUyfCfSAECrQlw.png)

Since May 2018 Microsoft Azure had integrated Inspec in the Azure Cloud Shell that provide the direct connection with your authentication.inspec in Azure Cloud Shell

![](https://cdn-images-1.medium.com/max/800/1*uhu0yRF52oHwSziSZdsKRw.png)

## The InSpec Azure Resource Pack

Since the v2, Inspec introduce the Cloud testing , by adding some libraries Azure for test Virtual Machine, Disk and all others resources had be tested by azure\_generic\_resource library.

Since the version 2.2.7 of Inspec, we have the feature to use a package of Inspec libraries that use Azure RM API and provide the possibility to any Azure resources.

Based on this improvement, the inspec team create a Azure resource pack that contains a lot of libraries for test a large panel of Azure resources like as :

* Azure users
* Azure Key Vault
* Azure Monitor
* Azure network \(Vnet, Subnet\)
* Azure Sql Server
* Azure Storage
* Azure Virtual Machine
* ….

The complete list of available resources is in the I[nspec documentation](https://www.inspec.io/docs/reference/resources/#azure-resources).



## Configure Inspec for Azure

Before to write tests for your Azure infrastructure we need to create an Azure Service Principal that have the reader permission to the Azure resources.

For create this Azure Service Principal we can use the Azure portal, or the AzureCli command:

```text
az ad sp create-for-rbac — name="<Sp name> — role=”Reader” — scopes=”/subscriptions/<subscription Id>”
```

This command return your 3 credentials information:

* The client ID \(Application Id\)
* The client Secret
* The Tenant ID

Then, use and export this values in specified variables environments on your machine

* AZURE\_SUBSCRIPTION\_ID =&gt; Subscription ID
* AZURE\_CLIENT\_ID =&gt; SP Client ID
* AZURE\_CLIENT\_SECRET =&gt; SP Client Secret
* AZURE\_TENANT\_ID =&gt; SP Tenant ID

By Script with export command or by using .envrc file

```text
export AZURE_SUBSCRIPTION_ID="<Subscription ID"
export AZURE_CLIENT_ID="<Client Id>"
export AZURE_CLIENT_SECRET="<Client Secret>"
export AZURE_TENANT_ID="<Tenant Id>"
```

