FROM nginx:stable

RUN rm /etc/nginx/conf.d/default.conf
COPY registry.conf /etc/nginx/conf.d/registry.conf
COPY registry.password /etc/nginx/conf.d/registry.password

COPY ./security/dhparam.pem /etc/nginx/security/dhparam.pem
COPY ./security/fullchain1.pem /etc/nginx/security/fullchain.pem
COPY ./security/privkey1.pem /etc/nginx/security/privkey.pem
