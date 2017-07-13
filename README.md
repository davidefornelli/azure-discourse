# Discourse on Azure
Notes on how to create a [Discourse](https://www.discourse.org/) forum on [Azure](https://azure.microsoft.com/).

## Deploy Discourse VM
The Discourse VM is offered by [Bitnami](https://bitnami.com/):

Intall the VM on Azure's [Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/bitnami.discourse?tab=Overview).

-[Documentation](https://docs.bitnami.com/azure/apps/discourse/)

[Retrieve the password](https://docs.bitnami.com/azure/faq/#how-to-find-application-credentials)

### Create a Domain
Create a Azure webapp.
Under custom domain, buy a new domain with the illustrated configurations 

==========
--- CONFIGURING HOST ---

sudo /home/bitnami/apps/discourse/bnconfig --machine_hostname DOMAINNAME