---
layout: post
title:  "Create a Lets Encrypt Certificate with Certbot"
date:   2019-10-24 10:40:50 +0530
categories: ssl
---

[Certbot](https://certbot.eff.org/) is a free, open source software tool for automatically using [Let’s Encrypt](https://letsencrypt.org/) certificates on manually-administrated websites to enable HTTPS.

Here are the steps to install Certbot and create a certificate:

#### Install Certbot, an open source ssl certificate generator

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

> **Note:** For more information on Certbot and Lets Encrypt, visit the official documentation [Certbot](https://certbot.eff.org/), [Let’s Encrypt](https://letsencrypt.org/)