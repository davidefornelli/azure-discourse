# Discourse on Azure
Notes on how to create a [Discourse](https://www.discourse.org/) forum on [Azure](https://azure.microsoft.com/).
This guide has been tested on
- OS: Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-123-generic x86_64)
- Bitnami Discourse: 1.8.1-0

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
```
sudo /home/bitnami/apps/discourse/bnconfig --machine_hostname DOMAIN
```

In /opt/bitnami/apps/discourse/htdocs/config/discourse.conf set
```
hostname = 'DOMAIN'
```

### Configure Let's Encrypt certificates

This steps are based on the apache
[Install certbot](https://docs.bitnami.com/azure/components/apache/#how-to-install-the-certbot-client-for-the-lets-encrypt-certificate-authority)

[Generate and install certificate](How to generate and install a Let's Encrypt certificate for your domain using the Certbot client?)

[Renew certificate](How to renew a Let's Encrypt certificate for your domain using the Certbot client?)

Install certbot
```
cd /tmp
git clone https://github.com/certbot/certbot
cd certbot
```

Download certificates
```
./certbot-auto certonly --webroot -w /opt/bitnami/apps/discourse/htdocs/ -d DOMAIN
```

Configure Apache tu use the new certificates
```
mv /opt/bitnami/apache2/conf/server.crt /opt/bitnami/apache2/conf/server.crt.bkp
sudo ln -s /etc/letsencrypt/live/DOMAIN/fullchain.pem /opt/bitnami/apache2/conf/server.crt
mv /opt/bitnami/apache2/conf/server.key /opt/bitnami/apache2/conf/server.key.bkp
sudo ln -s /etc/letsencrypt/live/DOMAIN/privkey.pem /opt/bitnami/apache2/conf/server.key
```

### Enable HTTPS
Force https url
```
vi /home/bitnami/apps/discourse/conf/httpd-prefix.conf

RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^/(.*) https://%{SERVER_NAME}/$1 [R,L]
```
## EMAIL
### Create SMTP
Use [SendGrid](https://azuremarketplace.microsoft.com/it-IT/marketplace/apps/SendGrid.SendGrid) on Azure, whith 25k free emails per month.
Generate a API Key: SendGrid -> Manage -> Settings -> API Keys
### Set Email SMTP
```
vi /home/bitnami/apps/discourse/htdocs/config/discourse.conf

#Add these lines

smtp_address = "smtp.sendgrid.com"
smtp_port = 587
smtp_domain = 'mcsdatasciencecommunity.com'
smtp_user_name = 'apikey' #Actually write aipkey as username
smtp_password = 'SENDGRID_KEY'
```
### Creae a new app on Azure
[Azure app](https://apps.dev.microsoft.com)
Generate a Key and save it
Add as Redirect URL https://DOMAIN/auth/oauth2_basic/callback
Add as Homepage https://DOMAIN
### Set Oauth2 for Microsoft