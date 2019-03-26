# Installing Terraform

## Manually

From the download page [https://www.terraform.io/downloads.html](https://www.terraform.io/downloads.html)

## On  Windows

use Chocolatey with the command `choco install terraform`

## On Linux

With the script

`RUN apt-get update && gem update --system && apt-get install unzip  && curl -Os`[`https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.z`](https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip)`ip && curl -Os` [`https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_SHA256SUMS`](https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_SHA256SUMS)  `&& curl` [`https://keybase.io/hashicorp/pgp_keys.asc`](https://keybase.io/hashicorp/pgp_keys.asc) `| gpg --import  && curl -Os` [`https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_SHA256SUMS.sig`](https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_SHA256SUMS.sig)  `&& gpg --verify terraform`_`${TERRAFORM_VERSION}_SHA256SUMS.sig terraform`_`${TERRAFORM`_`VERSION}_SHA256SUMS  && shasum -a 256 -c terraform`_`${TERRAFORM`_`VERSION}_SHA256SUMS 2>&1 | grep "${TERRAFORM_VERSION}_linux_amd64.zip:\sOK"  && unzip terraform`_`${TERRAFORM_VERSION}_linux_amd64.zip -d /usr/local/bin`

