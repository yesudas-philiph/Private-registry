# Private-registry with SSL encryption

## Private registry 

A Docker registry is a storage and distribution system for named Docker images. The same image might have multiple different versions, identified by their tags. A Docker registry is organized into Docker repositories , where a repository holds all the versions of a specific image.

Docker has a free public registry, Docker Hub, that can host your custom Docker images, but there are situations where you will not want your image to be publicly available. Images typically contain all the code necessary to run an application, so using a private registry is preferable when using proprietary software.





## Set-up
![Languages used](https://img.shields.io/badge/Number%20of%20Languages-1-Green) ![Languages used](https://img.shields.io/badge/Languages-YAML-Green)

Assuming the fact the your domain’s ssl cert and key are present in cert directory and auth file for basic authentication is loacated in auth directory. Both these directories needs to be present in your current directory

>[root@ip]# ls\
auth  certs  docker-compose.yml\
[root@ip]# ls auth\
htpasswd\
[root@ip]# ls certs\
domain.crt  domain.key

the original registry ui image used here is konradkleine/docker-registry-frontend:v2
it will help us to get a web view of your private regisry however the original image won’t allow you to push these images to the registry, current workaround is to comment out few lines in site configuration.


I’ve made these changes to my image and using it here\
image link [Docker image](https://hub.docker.com/repository/docker/yesudasphiliph/registry-ui) \
docker compose file is updated here



After installing the following package you can write auth credentials to htpasswd file
yum install httpd-tools
htpasswd -Bc htpasswd username

SSL can be generated using certbot via the following docker command
Assuming the fact that the domain is already pointed to the server IP  and DNS propagation for the same is completed also, please verify the fact that you are running this command from the server that your domain is pointed to.
>docker container run -it --rm --name certbot -p 80:80 -v "$pwd/certs:/etc/letsencrypt" certbot/certbot certonly --standalone

after ssl creation certificate location will be displayed in the terminal and container will auto terminate

>[root@ip]# docker container run -it --rm --name certbot -p 80:80 -v $(pwd)/certs:/etc/letsencrypt certbot/certbot certonly --standalone
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): email_here


>Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server. Do you agree?
(Y)es/(N)o: Y


>Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
(Y)es/(N)o: Y
Account registered.
Please enter the domain name(s) you would like on your certificate (comma and/or
space separated) (Enter 'c' to cancel): test.sync-tracker.tk
Requesting a certificate for test.sync-tracker.tk
Successfully received certificate.

>Certificate is saved at: /etc/letsencrypt/live/test.sync-tracker.tk/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/test.sync-tracker.tk/privkey.pem
This certificate expires on 2022-05-27.
These files will be updated when the certificate renews.


>[root@ip]# ls certs\
accounts  archive  csr  keys  live  renewal  renewal-hooks\
[root@ip]# ls certs/live\
README                test.sync-tracker.tk\
[root@ip]# ls certs/live/test.sync-tracker.tk\
cert.pem  chain.pem  fullchain.pem  privkey.pem  README

