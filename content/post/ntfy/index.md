---
title: "ntfy: A simple notification system"
summary: 
date: "2025-06-07T00:00:00Z"
lastmod: "2025-06-07T00:00:00Z"

authors:
  - nanoy

tags:
  - notifications
  - api

categories:
  - programmation
   
image:
  caption: 'An example of a ntfy notification'
---

Recently, I decided to recreate from scratch my arXiv bot (see [the post on my arxiv update bot]({{< relref "/post/tired-of-not-getting-arxiv-updates" >}})). In particular, the goal was to allow for more flexibility on the rules and the notification mechanisms. Indeed, the first version of my bot relied solely on telegram, which I am trying to use less and less for a number of reasons, but in this specific instance, it is because of the somehow buggy notifications on my phone.

Hence, I started looking at options where I could trigger a notification on my phone from sending an HTTP request to some URL. The first solution I encountered was [simplepush](https://simplepush.io/) which seems that it was doing exactly what I wanted, but... is not free of charge. Technically, there is a free plan with 10 notifications a month (which might be fine for some application, but not mine) or $12.49 per year for unlimited notifications. Therefore, I started looking for alternative, and especially alternatives that would be opensource and I could host on my server.

And I found it: [ntfy](https://ntfy.sh/) (pronounced _notify_) which is exactly what I needed: it sends push notification to your phone (and computer) by send a PUT/POST HTTP request, and you can customize it (buttons, callbacks, etc...). Let's see how it works !

## Server side installation

The procedure to install and configure a ntfy instance is described in their documentation ([https://docs.ntfy.sh/install/](for installation) and [https://docs.ntfy.sh/config/](for configuration)). There is a number of ways to install ntfy including:

* Binaries
* Package managers
* Docker
* Kubernetes

Since was server is running on Debian and ntfy is available on the Debian repositories, I will use this method.

We start by following the commands of the guide.

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://archive.heckel.io/apt/pubkey.txt | sudo gpg --dearmor -o /etc/apt/keyrings/archive.heckel.io.gpg
sudo apt install apt-transport-https
sudo sh -c "echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/archive.heckel.io.gpg] https://archive.heckel.io/apt debian main' \
    > /etc/apt/sources.list.d/archive.heckel.io.list"  
sudo apt update
sudo apt install ntfy
sudo systemctl enable ntfy
sudo systemctl start ntfy
```

At this point the ntfy server is already running. We can then modify the configuration file located at `/etc/ntfy/server.yml`. We start by settings the `base-url` to the required url, and then modify the `listen-http` port to listen on another port than the default 80 one. In the following, I assume that the `base-url` will be `ntfy.domain.tld` and that the http port will be 8080. I also set the `behind-proxy` parameter to true so that it can be used with nginx, so that the modified parameters are as of now:

```yaml
base-url: https://ntfy.domain.tld
listen-http: ":8080"
behind-proxy: true
```

I then create a nginx configuration file, following the example given in the [ntfy documentation](https://docs.ntfy.sh/config/?h=nginx#nginxapache2caddy)

```nginx
# /etc/nginx/sites-*/ntfy
#
# This config allows insecure HTTP POST/PUT requests against topics to allow a short curl syntax (without -L
# and "https://" prefix). It also disables output buffering, which has worked well for the ntfy.sh server.
#
# This is pretty much how ntfy.sh is configured. To see the exact configuration,
# see https://github.com/binwiederhier/ntfy-ansible/

server {
  listen 80;
  server_name ntfy.domain.tld;

  location / {
    # Redirect HTTP to HTTPS, but only for GET topic addresses, since we want
    # it to work with curl without the annoying https:// prefix
    set $redirect_https "";
    if ($request_method = GET) {
      set $redirect_https "yes";
    }
    if ($request_uri ~* "^/([-_a-z0-9]{0,64}$|docs/|static/)") {
      set $redirect_https "${redirect_https}yes";
    }
    if ($redirect_https = "yesyes") {
      return 302 https://$http_host$request_uri$is_args$query_string;
    }

    proxy_pass http://127.0.0.1:8080;
    proxy_http_version 1.1;

    proxy_buffering off;
    proxy_request_buffering off;
    proxy_redirect off;

    proxy_set_header Host $http_host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_connect_timeout 3m;
    proxy_send_timeout 3m;
    proxy_read_timeout 3m;

    client_max_body_size 0; # Stream request body to backend
  }
}

server {
  listen 443 ssl http2;
  server_name ntfy.domain.tld;

  # See https://ssl-config.mozilla.org/#server=nginx&version=1.18.0&config=intermediate&openssl=1.1.1k&hsts=false&ocsp=false&guideline=5.6
  ssl_session_timeout 1d;
  ssl_session_cache shared:MozSSL:10m; # about 40000 sessions
  ssl_session_tickets off;
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers off;

  ssl_certificate /etc/letsencrypt/live/domain.tld/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/domain.tld/privkey.pem;

  location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_http_version 1.1;

    proxy_buffering off;
    proxy_request_buffering off;
    proxy_redirect off;

    proxy_set_header Host $http_host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_connect_timeout 3m;
    proxy_send_timeout 3m;
    proxy_read_timeout 3m;

    client_max_body_size 0; # Stream request body to backend
  }
}
```

In particular, one wants to change the `server_name`, `proxy_pass`, `ssl_certificate` and `ssl_certificate_key` to match their environment. Note also that the nginx configuration must be compatible with websockets as ntfy heavily relies on them.

After a `systemctl restart nfty`, the ntfy server shall now be accessible at `ntfy.domain.tld`.

Now, I want to put some access control. Right now, everyone can use my instance and I want it to be a private instance. The details for the configuration are explained [here](https://docs.ntfy.sh/config/#access-control).

We will first give a storage space for the user database, and then deny all non-connected users.

We hence set the `auth-file` parameter to `/var/lib/ntfy/user.db` as suggested in the documentation, and set the `auth-default-access` to `deny-all` (choices are `read-write`, `write-only` and `deny-all`). We also enable login so that we can login from the webpage:


```yaml
base-url: https://ntfy.domain.tld
listen-http: ":8080"
behind-proxy: true

auth-file: /var/lib/ntfy/user.db
auth-default-access: "deny-all"
enable-login: true
```

I also let `enable-signup` to false to prevent users from registering.

At this point the "sign in" button has appeared on the web interface and non-logged user cannot do anything. We now need to create a user. This can be done by using the `ntfy` command

```bash
ntfy user add --role=admin nanoy
```

This will prompt for a password (can also be given by adding `NTFY_PASSWORD=...` in front of the command). Here the newly created user is created with the admin role, but can also be created with the user role.

Now, you can login to the webpage and start using your ntfy server.

There are many other options that you can set and I recommend reading the documentation for this. I have laid a minimal example, that also fits my application.

## Client side installation

The client side installation is quite straightforward. Here I assume that your goal is to use ntfy on a phone, and you can easily download the app from the [play store](https://play.google.com/store/apps/details?id=io.heckel.ntfy) or [App store](https://apps.apple.com/us/app/ntfy/id1625396347). The app is also available on [F-droid](https://f-droid.org/en/packages/io.heckel.ntfy/).

When the app is installed, you might want to change the default server in the options to match `ntfy.domain.tld`.

## Sending our first notification

You can start playing from the web interface to send notification to your phone. However, I am more interested in scripting.

The first step is to create an authentication token to avoid passing the password through the HTTP request. This can be done via Account > Access tokens. I will assume that the token is tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2 (same as in the documentation). I will also assume that you have crated a `test` topic.

Let's start simple:

```bash
curl \
  -H "Authorization: Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2" \
  -d "Test message" \
  https://ntfy.domain.tld/test
```

The response looks something like this:

```
{"id":"5OWynelc0SkY","time":1748898575,"expires":1748941775,"event":"message","topic":"test","message":"Test message"}
```

If you have subscribed to the `test` topic on the client, you should see the notification. It should also appear on the web interface.

Many parameters can be customized on the message, see [this page of the ntfy documentation](https://docs.ntfy.sh/publish/) for more information.

## Using python

Python can then be used to publish messages, for instance using the [requests](https://pypi.org/project/requests/) package. The authorization bearer must be given as a header:

```python
import requests

requests.post("https://ntfy.domain.tld/test",
  data="Test message",
  headers={
    "Authorization": "Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2"
  }
)
```

The message can be customized by giving more header parameter or by giving a JSON array as the data (sse [here](https://docs.ntfy.sh/publish/#using-a-json-array) for more information):

```python
import json
import requests

requests.post("https://ntfy.domain.tld/",
  data=json.dumps({
      "topic": "test",
      "message": "Test message",
      "priority": "5"
      "actions": [
          {
              "action": "view",
              "label": "Open portal",
              "url": "https://portal.domain.tld",
              "clear": true
          },
      ]
  }),
  headers={
    "Authorization": "Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2"
  }
)
```

Several wrappers for ntfy are also available on [pypi](https://pypi.org/search/?q=ntfy).