---
description: create Azure Service principal
---

# Create Azure Service Principal

For authorize Terraform to manage Azure Resources, it's necessary to create Azure Service principal in Azure AD, and assign it Contributor role on desired parent resources \(subscription or resource group\)

For create this Azure Service Principal we can use several methods:

* From the Azure Portal, [see the documentation](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal)
* From Azure cli with the command \([doc](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)\):

```text
az ad sp create-for-rbac --name="<Sp name>" --role="Contributor" --scopes="/subscriptions/<subscription Id>"
```

This command create the Service principal and add the role contributor on specified subscription

Replace the **Sp name** with the name the service principal, and the **subscription Id** with the contribution role on subscription target.

At the end of this configuration, you will receive the following information from the Service principal:

* The client ID  \(or Application Id\)
* The Client Secret
* The Tenant ID

This information will allow Terraform to authenticate itself and manipulate resources in an Azure subscription.

