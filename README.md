# secure-docker-registry

This repository is intended to provide a easy way to deploy a secure docker registry.
This docker registry will be deployed with Nginx and authentication by HTTP BASIC.

To have a secure docker registry, you'll need a SSL certificate. This repository explains how to generate your own SSL certificate with
letsencrypt (certbot).

Any suggestions or issues are welcome in the issue navigator.
No more bullshit, It's only five steps and I hope you enjoy the tutorial.

### Before start, we assume:
- You must have a domain. In this tutorial we'll name it `$YOUR_DOMAIN`;
- You have a remote machine that already is linked to your `$YOUR_DOMAIN`;
- You have `docker-compose` and `docker-machine` installed.


### Step 1: Clone and override

```bash
$(user)-$ git clone https://github.com/Tiago-Lira/secure-docker-registry.git
$(user)-$ cd secure-docker-registry/
```

Override $YOUR_DOMAIN in Nginx configuration file
```diff
# nginx/registry.conf

- line 9: server_name $YOUR_DOMAIN;
+ line 9: server_name yourdomain.com;

- line 101: server_name $YOUR_DOMAIN;
+ line 101: server_name yourdomain.com;
```


### Step 2: Create your own SSL Certificate

```bash
$(user)-$ ssh root@$YOUR_DOMAIN
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-45-generic x86_64)
$YOUR_DOMAIN~# sudo apt-get install letsencrypt
$YOUR_DOMAIN~# letsencrypt certonly --standalone -d $YOUR_DOMAIN -d www.$YOUR_DOMAIN
$YOUR_DOMAIN~# openssl dhparam -out /etc/letsencrypt/archive/$YOUR_DOMAIN/dhparam.pem 2048
$YOUR_DOMAIN~# exit
Connection to $YOUR_DOMAIN closed.
```

Select Nginx and your Operating System at https://certbot.eff.org/.
But **don't forget to run the openssl command** because it isn't in the certbot tutorial.

### Step 3: Download your certificate files to your registry folder
$(user)-$ scp root@$YOUR_DOMAIN:/etc/letsencrypt/archive/$YOUR_DOMAIN/* ./nginx/security/


### Step 4: Creating a docker-machine
```bash
$(user)-$ docker-machine create -d generic --generic-ip-address=$YOUR_DOMAIN registry
```

### Step 5: Deploying your registry

```bash
$(user)-$ eval $(docker-machine env registry)
$(user)-$ docker-compose up -d
registry uses an image, skipping
Building registry-nginx
Step 1 : FROM nginx:stable
 ---> 9bd6b3c63114
Step 2 : RUN rm /etc/nginx/conf.d/default.conf
 ---> 43b9ce488a59
Step 3 : COPY registry.conf /etc/nginx/conf.d/registry.conf
 ---> 72d64117c070
Step 4 : COPY registry.password /etc/nginx/conf.d/registry.password
 ---> f32da394c343
Step 5 : COPY ./security/dhparam.pem /etc/nginx/security/dhparam.pem
 ---> 945c50a54941
Step 6 : COPY ./security/fullchain.pem /etc/nginx/security/fullchain.pem
 ---> 8ac9194c0cf6
Step 7 : COPY ./security/privkey.pem /etc/nginx/security/privkey.pem
 ---> 6bff82c844e0
Successfully built 6bff82c844e0

```
