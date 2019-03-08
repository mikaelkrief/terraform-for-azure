---
description: generate module doc
---

# Generate module documentation

Tools : terraform-docs [https://github.com/segmentio/terraform-docs](https://github.com/segmentio/terraform-docs)

my script:

**Download and generate doc**

{% code-tabs %}
{% code-tabs-item title="generate-doc.sh" %}
```text
#!/bin/bash

## Telechargement de terraform-docs
## https://github.com/segmentio/terraform-docs
wget https://github.com/segmentio/terraform-docs/releases/download/v0.6.0/terraform-docs-v0.6.0-linux-amd64 -O /usr/local/bin/terraform-docs && chmod 775 /usr/local/bin/terraform-docs &&

## generation du readme
echo "==> Generate the Readme.md" &&
rm readme.md &&
cat ./doc/intro.md > readme.md &&
echo "==> Call terraform-docs"
terraform-docs md table . >> Readme.md &&
cat ./doc/releasenote.md >> readme.md 
```
{% endcode-tabs-item %}

{% code-tabs-item title="intro.md" %}
```
# Module Azure App Service #

Ce module permet de provisionner les resources Azure suivantes:
- Une App Service qui est relié à un Service Plan déjà existant
- Une Application Insight
```
{% endcode-tabs-item %}

{% code-tabs-item title="releasenote.md" %}
```
## Release Note ##

**v1.0.0**
Ajout du module dans git avec :
- App service
- App Insight
```
{% endcode-tabs-item %}
{% endcode-tabs %}



