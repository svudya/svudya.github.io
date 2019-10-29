---
layout: post
title:  "Install and run Docker Private Registry with Certbot"
date:   2019-10-28 12:37:51 +0530
categories: docker
---

Docker Private Registry is a registry service provided by Docker which can be self-hosted. The following steps helps you out in installing Docker private registry in any Ubuntu Machine.

### Prerequisites

1. The minimum requirements for creating a docker registry as per Docker officical documentation are as below. Create a server with the following requirements.

    | Property | value |
    |----------|-------|
    | Cores(CPU) | 2 |
    | Disk Space | 20 GB |
    | Memory (RAM) | 16 GB |

2. Install Docker in the above created server. The Installation steps for various OS can be found [here](https://docs.docker.com/install/).

3. (Optional) Install Docker Compose in the same server. The Installation steps for Docker compose can be found [here](https://docs.docker.com/compose/install/).

4. Login to the server and get sudo access by running the following command:
    {% highlight shell %}sudo su{% endhighlight %}

### Create a SSL certificate using Certbot

[Certbot](https://certbot.eff.org/) is a free, open source software tool for automatically using [Let’s Encrypt](https://letsencrypt.org/) certificates on manually-administrated websites to enable HTTPS.

Here are the steps to install Certbot and create a certificate for your Docker registry:

#### Install Certbot, an open source ssl certificate provider

{% highlight shell %}
apt-get update
apt-get install software-properties-common
add-apt-repository universe
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install certbot
{% endhighlight %}

#### Run Certbot in a interactive or non interactive standalone mode to generate a certificate
  
- To run Certbot in interactive mode, run the following command
{% highlight shell %}certbot certonly --standalone{% endhighlight %}
- To run Certbot in non interactive mode, run the following command. Make sure you change your email ID and Domain name in the below command
{% highlight shell %}certbot certonly --standalone --preferred-challenges http --non-interactive  --staple-ocsp --agree-tos -m <your-email-id> -d <your-domain-name>{% endhighlight %}

- Navigate to Root folder
{% highlight shell %}cd /root{% endhighlight %}

- (Optional)Certbot can be setup to auto renew the certificates.
{% highlight shell %}cat <<EOF > /etc/cron.d/letencrypt
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
30 2 * * 1 root /usr/bin/certbot renew >> /var/log/letsencrypt-renew.log && cd /etc/letsencrypt/live/<your-domain-name> && cp privkey.pem domain.key && cat cert.pem chain.pem > domain.crt && chmod 777 domain.*
EOF{% endhighlight %}

- (Optional)Rename the SSL certificates
{% highlight shell %}cd /etc/letsencrypt/live/<your-domain-name> && \
cp privkey.pem domain.key && \
cat cert.pem chain.pem > domain.crt && \
chmod 777 domain.*
{% endhighlight %}

> **Note:** Remember that the certificate names you give in both the above optional steps should be the same

### Create an admin user

Follow the steps to create an admin user for logging in to the docker registry:

- Create a directory for user credentials
{% highlight shell %}mkdir -p auth{% endhighlight %}
- Create an admin user using the following command
{% highlight shell %}docker run --entrypoint htpasswd registry:2 -Bbn admin secret > auth/htpasswd{% endhighlight %}

### Start the Docker Registry

All we have to do now is run this Docker command
{% highlight shell %}docker run -d -p 443:5000 \
  --restart=always \
  --name registry \
  -v "$(pwd)"/auth:/auth \
  -v /etc/letsencrypt/live/<your-doamin-name>:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -e REGISTRY_AUTH=htpasswd \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/var/lib/registry/passfile \
  registry:latest{% endhighlight %}
