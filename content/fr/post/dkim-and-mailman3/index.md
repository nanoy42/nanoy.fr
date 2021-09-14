---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Dkim and Mailman3"
subtitle: ""
summary: "In this short post, we review how to use DKIM and mailman3"
authors:
  - nanoy
tags:
  - mail
categories:
  - programmation
date: 2020-06-07T22:33:49+02:00
lastmod: 2020-06-07T22:33:49+02:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
## Mailman3

Mailman3 is mailing-list software.

## DKIM

DKIM (DomainKeys Identified Mail) is a norm that permits to verify the authenticity of the sender's name domain.

It works like this : you add a signature to outgoing emails and you also add a txt dns record with the public key in it. Then when an email is received, the client will check the DKIM signature in the email with the one in the dns record.

## Problem

Mailman3 modify headers and append text at end of emails. This breaks DKIM signature and you will likely have the error DKIM: Invalid (email was modified). 

## Proposed solution

A possible solution to this issue is the following:

1. Delete DKIM headers from incoming emails
2. Replace sender with list address
3. Sign email with the DKIM key of mail server

To delete the DKIM header from incoming emails, you can set the parameter `remove_dkim_headers` to `yes` in `cat /etc/mailman3/mailman.cfg` in the `[mta]` section. It will cause mailman to remove DKIM headers from any incoming email.

But some of you will already have spotted the problem. What if the sender uses a DMARC mitigation record. The email may be rejected if the DKIM header is not here.

So we can change the from header and sign the mail with our DKIM key.

To change the from header you can go to the list settings and then to DMARC mitigations, then change  "DMARC mitigation action"  to "Replace from: woth list address". If you want all messages to have the header replace, change " DMARC Mitigate unconditionally" from "no" to "yes".

Finally, you have to configure postfix (or any other software) to sign outgoing emails. For postfix, it will looks like 

```
#openDKIM
milter_default_action = accept
smtpd_milters = unix:/var/run/opendkim/opendkim.sock
non_smtpd_milters = unix:/var/run/opendkim/opendkim.sock
```
You then have to create a key, set up opendkim and add a txt dns record. Take a look at [http://opendkim.org/](http://opendkim.org/).

By default, mailman3 add a reply-to field ot the original poster. It can be changed to have a reply-to filed to the list address in the alter messages settings tab.