---
title: "Keepass to Vaultwarden"
subtitle: "A story of passwords managers"
summary: "Passwords managers are very important and useful and while a synchronized keepass database has its advantages, it has also some drawbacks that can be sorted out with vaultwarden"
date: 2021-10-03T00:09:00+02:00
lastmod: 2021-10-03T00:09:00+02:00
authors: 
  - nanoy
tags:
  - passwords
  - keepass
  - vaultwarden
categories:
  - programmation
image:
  caption: 'Image by <a href="https://pixabay.com/fr/users/geralt-9301/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=397652">Gerd Altmann</a> from <a href="https://pixabay.com/fr/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=397652">Pixabay</a>'
---

In this article we are going to review two major things : 

1. Why I move from keepass to vaultwarden
2. And the technical details of how I did that

But first we need a little introduction on password managers.

{{< toc >}}

## Password managers

Passwords are really old as they were used in the Ancient Times (by the Roman Empire for instance).

The computer use of password is almost as old as informatic itself and a password is, in informatic, a sequence of characters, that, when associated to a username, authenticate an user.

But the main point of a password is to be kept secret, and for this purpose, there are several conditions : 

* the password mustn't be a common password like 'password' or 'qwerty' or whatever common string. In fact, it's better if it's not a proper word. You can find some examples of common passwords here : https://en.wikipedia.org/wiki/List_of_the_most_common_passwords.
* the password mustn't be too simple, otherwise it could be broken by brute-force attacks (i.e trying all the passwords one by one)
* the password mustn't be written on post-its. In fact, it must not be written in a readable way.
* the password musn't be related to a personal thing, like a birthday or a pet name. Otherwise it could be guessed.
* passwords must be different for different services (i.e. you must not use the same password for every service you have).

### Common passwords

The issue with common passwords is pretty clear. It is quick to test all the passwords from a relatively short list. Of course, some websites now use techniques to prevent user to try a lot of differents passwords (you may have to wait between tries, or have to validate a captcha after a certain number of failures), but if your password is at the top of the list, or the website is not up to date with current security, your account will be open to every body.

Moreover, if you are using the same password everywhere, it only needs one not-so-safe website that allows to retry as much as we want to have access to all your accounts.

### Simple passwords

Imagine that you use a 1 digit password. There are only ten possible passwords (0 to 9). This means that it will be very easy to try all the possibilites (bruteforce) like in the previous case.

But if you add one more digit, there are now 100 possibles password (10 possible values for the first digit and 10 possible values for the second digit). They are in fact the numbers 0 to 99.

But let's go back to when you had one digit but now you allow the digit to also take alphabetical values. Let's say that you have one character that can be a digit or a lowercase letter (in the english alphabet). Now you have 36 passwords. Add one more character and it's 1296 passwords (still not enough).

In fact, if $L$ is the number of characters and $N$ the size of the characters set, you have $N^L$ possible passwords. Hold to this formula, we will go back to it later.

### Personnal passwords

Personal passwords are password based on / containing some of your personal information (name, birthday, cat's name, etc...). These passwords, even if they have no patterns, and are in no big tables of known password, are not safe because too easy to guess.

These passwords are attractive because it's easy to remember personnal information, but it's also easy to obtain someone's birthday.

Don't use personnal passwords. Use random passwords.

### Different passwords

It's good to have a long and complex password. It took you days to learn it by heart but finally you know it, and then you can use it everywhere. Bad idea...

It only needs that your passwords leaks from one place and everything is screwed. And you also would have to change it on every single service you ever used. And passwords leak... often... See : https://haveibeenpwned.com/

### Best practices

The best pratices are then as follows :

* have a master password that is randomly generated but that you can remember. This is the only one that you will need to know;
* generate a unique password for every service, with a high number of characters and a large character set. Don't worry, you will not have to remember those (if you can, well, it's impressive and the rest of this post is not for you);
* store those passwords in a password manager that you will encrypt with the master password.

You master password will also be used at other strategic places, like the password to access the backups of the password manager.

A password manager is like a giant postit, with all the passwords written on it, but there are not readable if you have not the master password. Some password managers can also be interfaced with browsers to autocomplete automatically the fields.

A password manager often have a random password generator.

Now remember the $N^L$ formula ? Let's give it an example. Imagine that are we allow :
* digits (10);
* lowercase characters (26);
* uppercase characters (26);


with a password length of $L=8$. In this case $N=62$ and you have 218 340 105 584 896 possible passwords. That is good, But not enough.

Now we add : 

* logograms #, $, %, &, @, ^. `, ~ (8)
* punctuation . , : ; (4)
* quotes " ' (2)
* dashes and slashes | \ / _ - (5)
* math symbols < > * + ! ? = (7)
* braces {[()]} (6)

we now have $k = 94$. Using 12 characters (which is still quite small), we have the number of possible passwords : 


$$475920314814253376475136$$

And we still are not using all of the ASCII characters (like accentuated characters).

But let's talk of password managers. If anyway you want to know more about that, and for instance how we use entropy to evaluate the strength of a password, go to [Appendix A](#appendix-a--entropy-of-passwords).

## Current solution

Before explaning what is my current solution, I need to explain my needs : 

1. I need a password manager (it's pretty clear from what I have written before I think);
2. The passwords stored in the password manager should be readable and writetable from my different devices (two computers, a smartphone and a tablet), knowing that the OS differs from device to device (Linux, Windows (unfortunately), Android and iOS).

And that's basically all. Well I mean, I would enjoy a nice integration with other tools (in particular the browers on computers and keyboards on the mobile devices) but the two main requirements are those one,

The solution that I had until now is this one : 

* I use a keepass database to store my passwords, encrypted using a master password (you can encrypt with a key file or and hardware key also);
* This database is kept synchronised using a nextcloud server.

Let's dig into it.

### keepass

[Keepass](https://keepass.info/) is an open source password manager distributed under the GNU General Public License version 2 (GPLv2). But I don't use the default keepass client. In fact I just use the format of the database, which is KDBX format, originally created by keepass.

Hence, I use as a primary client [keepassxc](https://keepassxc.org/) which is also an opensource project, distributed under the GPLv3 and has a nice interface (see below) and a nice integration with firefox : [see here for more details](https://addons.mozilla.org/en-US/firefox/addon/keepassxc-browser/).

{{< figure src="posts/keepass-to-vaultwarden/keepass.png" caption="Keepassxc interface" numbered="true" >}}

Keepassxc has other nice features I use, like :

* a password generator;
* an integration with SSH keys (unlocking the database also unlocks my SSH key and adds it to the agent);
* locking of the database when the user session is locked;
* multi-website integrations (this doesn't mean I use the same password on different services, but this useful in case of a centralised authentication scheme).

### Synchronisation

For the synchronisation, the keepass database is only a regular file, and I just needed to find a way to share it across devices. My solution was to use my nectcloud instance (that I also use for other things than keeping my keepass synchronised).

It is of course possible to use commercial clouds as well to share the database even if you have to trust the companies holding your data (the database is always stored encrypted though).

## Issues with the current solution

There are two main issues with the current solution : 

1. Synchronisation conflicts : a password, when created, is stored into the local and encrypted database file. When an internet connection is available, the client will attempt a synchronisation of the local file, but this synchronisation might fail, if other modifications were done at the same time or during the time where the client didn't have internet. As the database is encrypted, it also not possible to compare two versions.
2. The variety of clients : I use keepassxc on computers (linux or windows) but there is no official keepassxc app for mobile devices. Hence I use KeePassDx on android that can handle the database file and another client on iPad. KeePassDx has sometimes some issues with the synchronisation of the database and the client on iPad is the worst as it gets reset at each synchronisation.

## To Vaulwarden

The solution ? [Vaultwarden](https://github.com/dani-garcia/vaultwarden), which is a clients-server based password manager. But to understand why it's really interesting, we also need to talk a bit about [Bitwarden](https://bitwarden.com/).

Bitwarden is a passwords manager that is based on a clients-server infrastructure, and there is a lot of clients (web, windows, macos, linux, browser integration, android, appstore and even some cli tools). There is a strong ecosystem around bitwarden which is the very cool part. Also it's opensource 🥳.

You can use the vaultwarden servers for free or cheap for a personnal use (see below) and there are also a number of business plans.

{{< figure src="posts/keepass-to-vaultwarden/bitwarden-prices.png" caption="Prices of the personal bitwarden plans" numbered="true" >}}

You can also host your own server, but it is resources expensive. But another server implementation was made, far more less demanding. This was originally called Bitwarden_RS and is now called Vaultwarden, and the official bitwarden clients are compatible with the vaultwarden server.

Let's finally go technical !

The first step is to pull the docker image for vaultwarden :

```bash
docker pull vaultwarden/server:latest
```

Then I will generate an admin token for the initial configuration : 

```bash
openssl rand -base64 48
```

The output of this command is, for instance :

<center>
DBMmeR7o+ump0JSzsDTQJINozqkednNq55WXIFZvI+IapCR2MdyxiB8+QuHhTs1G
</center>

_This value is not used in production_.

Now I will start the docker container with the following command

```bash
docker run -d --name vaultwarden -e ADMIN_TOKEN=DBMmeR7o+ump0JSzsDTQJINozqkednNq55WXIFZvI+IapCR2MdyxiB8+QuHhTs1G -v /vw-data/:/data/ -p 8080:80 vaultwarden/server:latest
```

Here, the port 8080 of my server will be linked to the port 80 of the container, and I will handle the SSL/TLS layer with an apache2 proxy.

After this command is successful, you should have access to a vaultwarden instance on the port 8080 of your server (assuming no firewall blocks this port). This looks like this :

{{< figure src="posts/keepass-to-vaultwarden/vaultwarden_home.png" caption="First home of vaultwarden" numbered="true" >}}

Before doing anything else, I will configure a proxy to have secure connections. In `/etc/apache2/sites-available` I add the follwing file : 

```apache
<IfModule mod_ssl.c>
<VirtualHost *:80>
ServerName vaultwarden.nanoy.fr
Redirect / https://vaultwarden.nanoy.fr/
LogLevel warn
CustomLog /var/log/apache2/vaultwarden.access.log combined
ErrorLog /var/log/apache2/vaultwarden.error.log
</VirtualHost>
<VirtualHost *:443>
ServerName vaultwarden.nanoy.fr
CustomLog /var/log/apache2/vaultwarden.access.log combined
ErrorLog /var/log/apache2/vaultwarden.error.log
SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/nanoy.fr/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/nanoy.fr/privkey.pem
#Include /etc/letsencrypt/options-ssl-apache.conf
<Location />
  Require all granted
</Location>

RequestHeader setifempty X-Forwarded-Proto https

RewriteEngine On
RewriteRule ^$ / [R,L]
RewriteRule ^/(.*) http://localhost:8080/$1 [P,L]
</VirtualHost>
</IfModule>
```

(note that the ssl certificate is a DNS challenge valid on *.nanoy.fr), and then I run the command 

```bash
a2ensite vaultwarden; service apache2 restart
```

And the website is now securely accessible at `vaultwarden.nanoy.fr`.

As there is no account, we need to enter the admin mode by going to the `/admin/` URL and entering the admin code that we generated earlier.

{{< figure src="posts/keepass-to-vaultwarden/vaultwarden_admin.png" caption="Vaultwarden admin" numbered="true" >}}

and then there is a number of possible settings to change : 

{{< figure src="posts/keepass-to-vaultwarden/vaultwarden_admin2.png" caption="Vaultwarden admin once logged in" numbered="true" >}}

For my part, I have done the following : 

* create a personal account, and shut down the auto-registration (note that even if you disable the registration, the link on the login page and the form stay available, but an error is thrown when attempting to register);
* Invitation organization name;
* Disbale Yubikey;
* SMTP settings.

The configuration is pretty straightforward. Next I login with the user that I created on step 1 and I get the following : 

{{< figure src="posts/keepass-to-vaultwarden/vaultwarden_user.png" caption="User view" numbered="true" >}}

The next step was to install the clients. To use a custom server, you have to click on the cog in the top left corner and then set a custom URL. And then I started migrating my password. There is a specific import function to import a keepass database, but as my passwords were a bit of a mess and I decided to reogranize.

Unlike the app that I was using for keepass on my android, there is no virtual keyboard associated to the official bitwarden client. The virtual keyboard can help in some situations where the autofill API of android does not work. I will see out it goes. You can have a bit more information on these threads : https://community.bitwarden.com/t/android-virtual-keyboard-for-field-and-password-entry/3735 and https://github.com/bitwarden/mobile/issues/62

On the point of the Two-factor authentication on bitwarden, I think that in theory it could be a good idea. However : 
* it may be annoying if you have to enter the code each time you eant to open the vault. From what I saw until now, it's only needed for new devices, but that is maybe a setting;
* if you loose your smartphone, you should anyway be able to recover from that and get back your database.

The second point can be handled by two different considerations :
* first, you have a backup code. You must securely store it but you can store it encrypted with your master password/passphrase on several devices;
* secondly, you should also be able to recover from the loss of the vaultwarden server. In my case, I will probably try to set a backup system to regularly export my passwords or look at the backup solutions in vaultwarden.

On other points, I had to modify the default configuration to get some behaviors of keepass (auto-close the vault when going to sleep, automatic startup etc) but nothing too complicated.

## Conclusion

In conclusion, I would say that the shared keepass using a cloud (either self hosted or not) is a good idea and that is working up to the small issues I have discussed on the synchronisation on the non-uniformity of clients. Also, for most people, you can use a solution like bitwarden (or other password managers) with a free solution for individuals.

Anyway, the specific case where you want full control over your data and get rid of the small issues can be to install a server like vaultwarden, even if new issues arise.

The first one is the synchronisation : while my nextcloud was synchronising pretty often and hence, I had almost not wait time between the edition of the database and the sharing between devices and with vaultwarden, I sometimes have to trigger a manual sync. It could only be a configuration issue though.

Another potential issue is the lack of a virtual keyboard of android but I will have to use it more to see if it's a real issue or not.

For now, vaultwarden/bitwarden seem to work pretty well.

## Appendix A : Entropy of passwords

The notion of the _entropy of a password_  is equivalent to the combinatorial description made before (i.e. we counted the number of possible passwords given the length and an alphabet), but expressed in computer way.

In fact the idea is the following : it is to say "this password is as secure as a random lists of $H$ bits" where $H$ is the entropy of the password. As a bit can take two possible values (0 or 1), there are $2^H$ differents lists of $H$ bits.

But remember, before, we were saying that with if you have an alphabet with $N$ elements and a length of $L$ then there will be $N^L$ possible passwords. Hence if we want to have the comparison between the combinatorial way and the entropy way we want to have 

$$N^L = 2^H$$

Then we can apply the logarithm (an operation that transforms products into sums, and hence powers into products), we have 

$$\log_2(N^L) = \log_2(2^H)$$

$$L\log_2(N) = H$$

Hence the entropy of a password is the length of the password multiplied by the binary log of the alphabet size. In fact $\log_2(N)$ can be seen as the average entropy per character, and then multiplied by the length, we indeed have the entropy of the password.

Let's take some examples : 

| Name               |        Elements        | Size of alphabet | Average entropy per char. |
| ------------------ | :--------------------: | ---------------: | ------------------------: |
| Digits             |          0-9           |               10 |                3.322 bits |
| Lowercase          |          a-z           |               26 |                4.700 bits |
| Uppercase          |          A-Z           |               26 |                4.700 bits |
| Logograms          | #, $, %, &, @, ^. `, ~ |                8 |                    3 bits |
| Punctuation        |        . , : ;         |                4 |                    2 bits |
| Quotes             |          " '           |                2 |                     1 bit |
| Dashes and slashes |     &#124; \ / _ -     |                5 |                2.322 bits |
| Math symbols       |     < > * + ! ? =      |                7 |                2.807 bits |
| Braces             |         {[()]}         |                6 |                2.585 bits |
    

Let's finally take a concrete example, with the password :

<center>
RS=%Q@K2Ecxt2g
</center>

It has length 14 and the alphabet's size is 10 (digits) + 26 (lowercase) + 26 (uppercase) + 8 (logograms) + 7 (math symbols) = 77, and hence the entropy of the password is :

$$14\times\log_2(77) = 87.735 \text{ bits}$$

which is almost the answer given by keepass (91.16 bits) : 

{{< figure src="posts/keepass-to-vaultwarden/keepass_entropy.png" caption="Entropy of the password RS=%Q@K2Ecxt2g" numbered="true" >}}

How to explain this difference ? You can remark that if you generate several passwords with the same alphabet and the same size, you will end up with different entropies, which should not be the case with our model. In fact our model is valid if passwords were truly random, which is not often the case. Hence, many providers don't use this method to compute the entropy but other algorithms.

Keepass for example use the zxcvbn algorithm, that also tries to detect patterns and repetitions (see more information on this very interesting [blog post](https://dropbox.tech/security/zxcvbn-realistic-password-strength-estimation)).

You can fin some information on the [Wikipedia page](https://en.wikipedia.org/wiki/Password_strength).

## Appendix B : Multi factor authentication

In this short blog post, I have only spoken of passwords. However, nowadays, we also use other authentication methods and there are frequently combined.

The authentication methods can be grouped in 3 categories : 

* something you know (a password, a passphrase, answer to backup question, etc...);
* something you have (phone number, mobile app, email address);
* something you are (facial recognition, fingerprints, etc...).

When several authentication methods from at least two of the groups are combined, we speak of multi-factor authentication.

Here are several examples : 

* password and code with SMS/email;
* password and confirmation on app;
* password and fingerprint.

Often, password is one of the authentication method and the second one is often something you have (email or app or SMS).

Facial recognition and fingerprints are often used to unlock your phone (and can be considered as an additional security to the app or SMS method).

This issue with the "something you are" is that it's very difficult or impossible to change that something. If a password is corrupted, you can just generate a new one and it's okay. You can also change phone numbers. But you can't change your fingerprints and it's a hard process to change a face.

Anyway, multi-factor is a good thing and there are some cool apps (like [2FAS](https://2fas.com/)) and more and more services are proposing this option of 2 factor authentication. I strongly encourage you to use mutli-factor authentication.