# Discourse on Azure
Notes on how to create a [Discourse](https://www.discourse.org/) forum on [Azure](https://azure.microsoft.com/).

### Discourse Notes
-[Documentation](https://docs.bitnami.com/azure/apps/discourse/)


### Deploy Discourse VM
The Discourse VM is offered by [Bitnami](https://bitnami.com/):

Intall the VM on Azure's [Marketplace].

1. Install [Bitnami's Discourse VM](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/bitnami.discourse?tab=Overview)
2. [Retrieve the password](https://docs.bitnami.com/azure/faq/#how-to-find-application-credentials)

### Create a Domain
Create a Azure webapp.
Under custom domain, buy a new domain with the illustrated configurations:
image

### Assign the domain to the VM
Log into the VM through SSH.
sudo /home/bitnami/apps/discourse/bnconfig --machine_hostname DOMAINNAME

### Enable HTTPS

### Get SSH certificates from Let's Encrypt

```
cd / tmp
git clone https://github.com/certbot/certbot
cd certbot
```

./certbot-auto certonly --webroot -w /opt/bitnami/apps/discourse/htdocs/ -d dscommunitywebappdevetest.com -d www.dscommunitywebappdevetest.com -d forum.dscommunitywebappdevetest.com

sudo ln -s /etc/letsencrypt/live/dscommunitywebappdevetest.com/fullchain.pem /opt/bitnami/apache2/conf/server.crt
sudo ln -s /etc/letsencrypt/live/dscommunitywebappdevetest.com/privkey.pem /opt/bitnami/apache2/conf/server.key

