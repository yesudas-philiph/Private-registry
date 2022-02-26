# Private-registry

## Private registry with Front end

![Languages used](https://img.shields.io/badge/Number%20of%20Languages-1-Green) ![Languages used](https://img.shields.io/badge/Languages-YAML-Green)

Assuming the fact the your domain’s ssl cert and key are present in cert directory and auth file for basic authentication is loacated in auth directory. Both these directories needs to be present in your current directory

 [root@ip-172-31-32-89 test]# ls
auth  certs  docker-compose.yml
[root@ip-172-31-32-89 test]# ls auth/
htpasswd
[root@ip-172-31-32-89 test]# ls certs/
domain.crt  domain.key

the original registry ui image used here is konradkleine/docker-registry-frontend:v2
it will help us to get a web view of your private regisry however the original image won’t allow you to push these images to the registry, current workaround is to comment out the following line in site configuration


----------------------------------------------------
When FRONTEND_BROWSE_ONLY_MODE is defined in envvars
we will only allow GET requests to the frontend. All other
HTTP requests will be aborted with a HTTP 403 Error.

  <IfDefine FRONTEND_BROWSE_ONLY_MODE>
    <Location />
      <LimitExcept GET>
        Order Allow,Deny
        Deny From All
      </LimitExcept>
    </Location>
  </IfDefine>
-------------------------------------------------------


I’ve made these changes to my image and using it here
docker compose file is updated down below

version: '3'
services:
  registry:
    container_name: registry
    image: registry:2
    ports:
    - "5000:5000"
    environment:
      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm
      - REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
      - REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data
      - REGISTRY_HTTP_ADDR=0.0.0.0:5000
      - REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt
      - REGISTRY_HTTP_TLS_KEY=/certs/domain.key
    volumes:
      - data:/data
      - ./certs:/certs
      - ./auth:/auth
    networks:
      - mynet

  frontend:
    container_name: frontend
    environment:
     - ENV_DOCKER_REGISTRY_HOST=registry
     - ENV_DOCKER_REGISTRY_PORT=5000
     - ENV_DOCKER_REGISTRY_USE_SSL=1
     - ENV_USE_SSL=yes
    volumes:
     - ./certs/domain.crt:/etc/apache2/server.crt:ro
     - ./certs/domain.key:/etc/apache2/server.key:ro
    ports:
     -  443:443
    image: yesudasphiliph/registry-ui:latest
    networks:
     - mynet

volumes:
  data:
networks:
  mynet:


After installing the following package you can write auth credentials to htpasswd file
yum install httpd-tools
htpasswd -Bc htpasswd username

SSL can be generated using certbot via the following docker command
Assuming the fact that the domain is already pointed to the server IP  and DNS propagation for the same is completed also, please verify the fact that you are running this command from the server that your domain is pointed to.
>docker container run -it --rm --name certbot -p 80:80 -v "$pwd/certs:/etc/letsencrypt" certbot/certbot certonly --standalone

after ssl creation certificate location will be displayed in the terminal and container will auto terminate

[root@ip-172-31-32-89 t]# docker container run -it --rm --name certbot -p 80:80 -v $(pwd)/certs:/etc/letsencrypt certbot/certbot certonly --standalone
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): email_here

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
Account registered.
Please enter the domain name(s) you would like on your certificate (comma and/or
space separated) (Enter 'c' to cancel): test.sync-tracker.tk
Requesting a certificate for test.sync-tracker.tk

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/test.sync-tracker.tk/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/test.sync-tracker.tk/privkey.pem
This certificate expires on 2022-05-27.
These files will be updated when the certificate renews.

NEXT STEPS:
- The certificate will need to be renewed before it expires. Certbot can automatically renew the certificate in the background, but you may need to take steps to enable that functionality. See https://certbot.org/renewal-setup for instructions.
We were unable to subscribe you the EFF mailing list because your e-mail address appears to be invalid. You can try again later by visiting https://act.eff.org.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
[root@ip-172-31-32-89 t]# ls certs/
accounts  archive  csr  keys  live  renewal  renewal-hooks
[root@ip-172-31-32-89 t]# ls certs/live/
README                test.sync-tracker.tk/ 
[root@ip-172-31-32-89 t]# ls certs/live/test.sync-tracker.tk/
cert.pem  chain.pem  fullchain.pem  privkey.pem  README
[root@ip-172-31-32-89 t]# 
