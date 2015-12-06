---
title: "Configuring Let's Encrypt for nginx with Automatic Renewal"
date: "2015-12-05"
URL: "2015/12/configuring-nginx-lets-encrypt-automatic-renewal/"
---

[Let's Encrypt](https://letsencrypt.org) is a new Certificate Authority that offers free certificates and automatic renewal through a new standard known as ACME (Automated Certificate Management Environment.)

It recently entered public beta, but nginx support is somewhat lacking. However, it's really not that difficult to get started.

Note that these instructions are for Debian, but can probably be easily modified to suit your favorite distro.

First, you'll need to [install Let's Encrypt](https://letsencrypt.readthedocs.org/en/latest/using.html#installation). I downloaded the [v0.1.0 release from Github](https://github.com/letsencrypt/letsencrypt/releases) and ran `python setup.py install`, but you can use whatever method you prefer.

Next, let's create a couple helper scripts. The first one will help us to generate new certificates, while we can use the second one to renew our certificates. This will come in handy when we setup a cron job later. I saved these two scripts in `/usr/local/bin` as `letsencrypt_gen` and `letsencrypt_renew`.

```bash
#!/usr/bin/env bash
if [[ -z "$DOMAINS" ]]; then
    echo "Please set DOMAINS environment variable (e.g. \"-d example.com -d www.example.com\")"
    exit 1
fi

if [[ -z "$DIR" ]]; then
    export DIR=/tmp/letsencrypt-auto
fi

mkdir -p $DIR && letsencrypt certonly \
    --server https://acme-v01.api.letsencrypt.org/directory \
    --webroot \
    --webroot-path=$DIR \
    $DOMAINS
service nginx reload
```

The second script is almost identical to the first but passes a `--renew` flag.

```bash
#!/usr/bin/env bash
if [[ -z "$DOMAINS" ]]; then
    echo "Please set DOMAINS environment variable (e.g. \"-d example.com -d www.example.com\")"
    exit 1
fi

if [[ -z "$DIR" ]]; then
    export DIR=/tmp/letsencrypt-auto
fi

mkdir -p $DIR && letsencrypt --renew certonly \
    --server https://acme-v01.api.letsencrypt.org/directory \
    --webroot \
    --webroot-path=$DIR \
    $DOMAINS
service nginx reload
```

However, before we can use these scripts we need to add a `location` block in our nginx config. This will allow Let's Encrypt to verify that we own the domains we're trying to generate certificates for.

```
server {
	# ...

	location /.well-known/acme-challenge {
		default_type  "text/plain";
		root          /tmp/letsencrypt-auto;
	}

	# ...
}
```

The scripts we created earlier will automatically generate the files necessary to validate our identity in `/tmp/letsencrypt-auto`, and so the rest is taken care of for us. The only thing we need to do is reload the nginx configuration so the changes take effect. On Debian, run `service nginx reload`.

Now it's time to generate our SSL certificate! We can call the script we saved earlier like so: `DOMAINS="-d johnmaguire.me -d www.johnmaguire.me" letsencrypt_gen`.

I also created a DH parameter file inside my `/etc/nginx` directory using OpenSSL: `openssl dhparam -out dhparams.pem 2048`

At this point, we just need to configure SSL-enabled server block. Mine looks like this, but you can customize it to your needs:

```
server {
	listen       443 ssl;
	server_name  johnmaguire.me www.johnmaguire.me;

	# certificates from letsencrypt
	ssl_certificate      /etc/letsencrypt/live/johnmaguire.me/fullchain.pem;
	ssl_certificate_key  /etc/letsencrypt/live/johnmaguire.me/privkey.pem;
	ssl_session_timeout 1d;
	ssl_session_cache shared:SSL:50m;
	ssl_session_tickets off;

	# Diffie-Hellman parameter for DHE ciphersuites
	ssl_dhparam /etc/nginx/dhparam.pem;

	# modern configuration
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!3DES:!MD5:!PSK';
	ssl_prefer_server_ciphers on;

	# HSTS (ngx_http_headers required) - 6 months
	# add_header Strict-Transport-Security max-age=15768000;

	# OCSP stapling
	ssl_stapling on;
	ssl_stapling_verify on;

	# verify chain of trust of OCSP response
	ssl_trusted_certificate /etc/letsencrypt/live/johnmaguire.me/chain.pem;

	# ...
}
```

[Mozilla](https://mozilla.github.io/server-side-tls/ssl-config-generator/) has an excellent SSL configuration generator. You can verify that SSL is working properly [using this tool from Qualys](https://www.ssllabs.com/ssltest/).

Once everything is setup correctly, we can create a cron job to automatically renew our certificate. Run `crontab -e` and add the following:

```
@monthly DOMAINS="-d johnmaguire -d www.johnmaguire.me" /usr/local/bin/letsencrypt_renew
```

You can also test out certificate renewal by running the above command manually, and viewing the certificate as reported by your browser. It will only be valid beginning at the time you created the certificate.

Finally, if you'd like to send users who access your site via HTTP over to your HTTPS endpoint, you can create a catch-all location block in nginx like so:

```
server {
	listen       80;
	server_name  johnmaguire.me www.johnmaguire.me;

	# ...

	location / {
		return 301 https://$server_name$request_uri;
	}
}
```

If everything is working correctly, consider uncommenting the [HSTS](https://www.owasp.org/index.php/HTTP_Strict_Transport_Security) line in the above configuration. It will allow your users' browsers to remember that they've connected with SSL before, and refuse to use non-SSL connections. Please understand that this is a fairly permanent choice.

If you do decide to use HSTS, you can also add your website to browser vendors' HSTS preload list, which will let the browser know your site has SSL before a user even hits it. Use **extreme caution** with [this tool](https://hstspreload.appspot.com). Please read the **entire page** and make sure you understand what it means before submitting your domain.
