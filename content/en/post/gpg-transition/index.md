---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Transition to a near-perfect GPG key pair"
subtitle: ""
summary: ""
authors: [nanoy]
tags: []
categories: []
date: 2021-12-27T21:07:54+01:00
lastmod: 2021-12-27T21:07:54+01:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false
---

GPG keys are super useful, but as of December 2021, I need to change mine, to reach the recommendations for GPG key pairs. This blog post will explain what is a GPG key, why it is useful, why I need to change it and how I did that in the most preferable way.

{{<toc>}}

## What is a GPG key why is it useful and why do I need to change it ?

## Practical transition

### Clean the GnuPG installation

gpg --export -a > all_publickeys
gpgconf --kill gpg-agent
mv .gnupg .gnupg_old
mkdir .gnupg
chmod 700 .gnupg

https://www.gnupg.org/
https://wiki.archlinux.org/title/GnuPG

cd .gnupg
vim dirmngr.conf
nanoy@azilis  ~/.gnupg  cat dirmngr.conf 
keyserver hkps://keys.openpgp.org

vim gpg-agent.conf
nanoy@azilis  ~/.gnupg  cat gpg-agent.conf 
default-cache-ttl 600
pinentry-program /usr/bin/pinentry-gnome3
default-cache-ttl-ssh 600

vim gpg.conf


### Create the master key pair

Disconnect computer from the internet

### Create subkeys

### Splitting the master keys and subkeys

### Transition statement

### Wrap up

## Further reading and references