# Discourse on Azure
Notes on how to create a [Discourse](https://www.discourse.org/) forum on [Azure](https://azure.microsoft.com/).
This guide has been tested on
- OS: Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-123-generic x86_64)
- Bitnami Discourse: 1.8.1-0

### Discourse Notes
-[Documentation](https://docs.bitnami.com/azure/apps/discourse/)

### Useful commands
```
#Restart all services
sudo /opt/bitnami/ctlscript.sh restart
#Restart Apache only
sudo /opt/bitnami/ctlscript.sh restart apache
```


### Deploy Discourse VM
The Discourse VM is offered by [Bitnami](https://bitnami.com/):

Intall the VM on Azure's [Marketplace].

1. Install [Bitnami's Discourse VM](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/bitnami.discourse?tab=Overview)
2. [Retrieve the password](https://docs.bitnami.com/azure/faq/#how-to-find-application-credentials)

#### Set Static IP

### Check if reachable
http://VMIP

### Create a Domain
Create a Azure webapp.
Under custom domain, buy a new domain with the illustrated configurations:
image
In the DNS Management portal add the record
image
### Assign the domain to the VM
Log into the VM through SSH.
```
sudo /home/bitnami/apps/discourse/bnconfig --machine_hostname DOMAIN

sudo nano /etc/hosts #Add the following line
 IP-ADDRESS DOMAIN
```

### Discourse
Login into Discourse with:
username:user
password: [PASSWORD AT STEP X]

Set the domain in the forum:
Admin -> Settings -> Required -> company domain -> DOMAIN

## EMAIL
### Azure - Create SMTP
Use [SendGrid](https://azuremarketplace.microsoft.com/it-IT/marketplace/apps/SendGrid.SendGrid) on Azure, whith 25k free emails per month.
Generate a API Key: SendGrid -> Manage -> Settings -> API Keys

### Test
'''
curl --request POST \
  --url https://api.sendgrid.com/v3/mail/send \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{"personalizations": [{"to": [{"email": "recipient@example.com"}]}],"from": {"email": "sender@example.com"},"subject": "Hello, World!","content": [{"type": "text/plain", "value": "Heya!"}]}'
'''

### VM - Set Email SMTP
```
vi /home/bitnami/apps/discourse/htdocs/config/discourse.conf

#Add these lines

smtp_address = "smtp.sendgrid.net"
smtp_port = 587
smtp_domain = 'DOMAIN'
smtp_user_name = 'apikey' #Actually write aipkey as username
smtp_password = 'SENDGRID_KEY'
```

## Test
Admin -> Emails -> Settings

### Configure Let's Encrypt certificates
This steps are based on the apache
[Install certbot](https://docs.bitnami.com/azure/components/apache/#how-to-install-the-certbot-client-for-the-lets-encrypt-certificate-authority)

[Generate and install certificate](How to generate and install a Let's Encrypt certificate for your domain using the Certbot client?)
If this doesn't work, use the DNS method:
```
 ./certbot-auto certonly --manual --preferred-challenges=dns -d DOMAIN
```
[Renew certificate](How to renew a Let's Encrypt certificate for your domain using the Certbot client?)

Install certbot
```
cd /tmp
git clone https://github.com/certbot/certbot
cd certbot
```

Download certificates
```
./certbot-auto certonly --webroot -w /opt/bitnami/apps/discourse/htdocs/public -d DOMAIN
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
## Active Directory Login
### Azure - Create a new App on the new portal
[Azure app](https://apps.dev.microsoft.com)
Applicaton Id
Application Secrets -> Generate New Password
Generate a Key and save it
Add as Redirect URL https://DOMAIN/auth/oauth2_basic/callback
Add as Homepage https://DOMAIN
### VM - Install Oauth plugin
```
cd /opt/bitnami/apps/discourse/htdocs
sudo RAILS_ENV=production bundle exec rake plugin:install repo=https://github.com/discourse/discourse-oauth2-basic.git
sudo RAILS_ENV=production bundle exec rake assets:precompile
```
### Discourse - Set the oauth urls
[Docs](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols)
Authorize url
https://login.windows.net/microsoft.onmicrosoft.com/oauth2/v2.0/authorize?scope=https://graph.microsoft.com/User.Read
Token Url
https://login.windows.net/microsoft.onmicrosoft.com/oauth2/v2.0/token
User Json
https://graph.microsoft.com/v1.0/me
oauth2 enabled: checked
oauth2 client id: from app
oauth2 client secret: app_key
oauth2 json user id path: id
oauth2 json username path: userPrincipalName
oauth2 json name path: displayName
oauth2 json email path: mail
oauth2 email verified: checked
oauth2 send auth header: checked
### Notes
Restart servers
sudo /opt/bitnami/ctlscript.sh restart
