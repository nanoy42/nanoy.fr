---
title: "De Keepass à Vaultwarden"
subtitle: "Une histoire de gestionnaire de mots de passe"
summary: "Passwords managers are very important and useful and while a synchronized keepass database has its advantages, it has also some drawbacks that can be sorted out with vaultwarden"
authors: 
  - nanoy
tags:
  - passwords
  - keepass
  - vaultwarden
categories:
  - programmation
date: 2021-10-03T00:09:00+02:00
lastmod: 2021-10-03T00:09:00+02:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: 'Image par <a href="https://pixabay.com/fr/users/geralt-9301/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=397652">Gerd Altmann</a> de <a href="https://pixabay.com/fr/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=397652">Pixabay</a>'
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

Dans ce billet, on va parler de deux gros points :

1. Pourquoi je suis passé de keepass à vaultwarden
2. Et les détails techniques de la migration

Mais avant, une petite introduction sur les gestionnaires de mots de passe.
{{< toc >}}

## Les gestionnaires de mots de passe

Les mots de passes sont très vieux : ils étaient par exemple utilisés durant l'Antiquité (Empire Romain par exemple).

L'usage informatique des mots de passe est quasiment aussi vieux que l'informatique elle même et, un mot de passe est, en informatique, une chaine de caractères qui, associée à un mot de passe, permet d'authentifier un utilisateur.

Mais le but d'un mot de passe est de rester secret, et pour cela, il y a plusieurs conditions :

* le mot de passe ne doit pas être un mot de passe connu comme "password" ou "azerty" ou n'importe quelle autre chaîne de caractère commune. En fait, c'est mieux si le mot de passe n'est pas un mot. Il ya des exemples de mots de passe commun ici : https://fr.wikipedia.org/wiki/Liste_des_mots_de_passe_les_plus_courants;
* le mot de passe ne doit pas être trop simple, sinon il peut être deviné avec un attaque de force brute (essayer tous les mots de passe un à un);
* le mot de passe ne doit pas être écrit sur des posts-its. En fait, il ne doit pas être écrit de façon lisible directement;
* le mot de passe ne doit pas avoir de lien personnel, comme une date de naissance ou le prénom d'un animal de compagnie, sinon il pourrait être deviné;
* les mots de passe doivent être différents pour chaque service (pas un seul mot de passe pour tous les services).

### Les mots de passe commun

Le problème avec les mots de passe communs est assez clair. C'est assez rapide et facile de tester tous les mots de passe d'une liste donné relativement courte. Certains sites internet ont des protections pour éviter qu'un utilisateur essaye un nombre trop grand de mots de passe différents (un temps d'attente entre les essais, ou la validation d'un captcha après un certain nombre d'erreurs), mais si le mot de passe est en haut de la liste, ou si le site internet n'a pas de protections, le compte sera alors ouvert à n'importe qui.

Aussi, si vous utilisez le même mot de passe partout, il suffit qu'un seul n'ait pas de protection pour avoir accès à tous vos comptes.

### Les mots de passe simples

Imaginez que vous utilisez un mot de passe consistant d'un seul chiffre, alors il y a dix mots de passe différents (de 0 à 9). Cela veut dire qu'il est très facile de le casser par la méthode d'attaque de force bruit.

Maintenant, ajoutons un chiffre de plus, il y a alors 100 possibilités (10 possibilités pour le premier chiffre et 10 autres pour le deuxième). Il s'agit en fait des nombres de 0 à 99.

Mais revenons au cas avec un seul chiffre, mais en autorisant maintenant aussi les caractères alphabétiques. Prenons l'exemple d'un seul caractère qui peut être un chiffre ou une lettre minuscule (de l'alphabet français). Il y a maintenant 36 mots de passe différents, et si on ajoute un autre caractère, on monte à 1296 mots de passe (toujours loin de ce qu'il faut).

En fait, si $L$ est la longueur du mot de passe et $N$ le nombre de caractères possibles, il y a $N^L$ mots de passe possibles. Gardons cette formule en tête, on y reviendra plus tard.

### Les mots de passe personnels

Les mots de passe personnels sont des mots passes qui sont basés ou qui contiennent des informations personnelles (un nom, une date d'anniversaire, le nom d'un animal, ...). Ces mots de passes, même s'il n'y a pas de motif, ou qu'il ne se trouve pas dans une liste de passwords connus, ne sont pas sûrs car ils sont faciles à deviner.

Ces mots de passe sont souvent choisis car c'est facile de retenir des informations personnelles, mais c'est tout aussi facile de se procurer la date d'anniversaire d'une personne.

N'utilisez pas de mots de passe personnels. Utilisez des mots de passe aléatoires !

### Mots de passe différents

C'est bien d'avoir un long mot de passe difficile. Ça vous a pris des jours pour l'apprendre par coeur et maintenant vous le connaissez enfin, et vous pouvez l'utiliser partout. Mauvaise idée...

Si votre mot de passe fuit ne serait que d'un seul endroit, et il est foutu. Il faudra alors changer les mots de passe de tous les services affectés. Et les mots de passe fuitent... souvent... voir : https://haveibeenpwned.com/

### Bonnes pratiques

Les bonnes pratiques sont les suivantes :

* avoir un mot de passe maître, généré aléatoirement mais qu'il est possible de se souvenir. C'est le seul qu'il faudra connaître;
* générez un mot de passe unique pour chaque service, avec un grand nombre de caractères, parmi un choix très larges. Ne vous inquiétez pas, il ne faudra pas retenir ces mots de passe (si vous pouvez tous les retenir, c'est impressionnant et vous n'avez besoin du reste du billet);
* enregistrez ces mots de passe dans un gestionnaire de mot de passe qui sera chiffré avec le mot de passe maître.

Le mot de passe maître peut aussi être utilisé dans d'autres endroits stratégiques, comme pour accéder aux backups du gestionnaire de mot de passe.

Un gestionnaire de mot de passe est comme un postit géant avec tous les mots de passe écrits dessus, mais ils ne sont pas lisibles sans le mot de passe maître. Certains gestionnaires peuvent aussi êtres interfacés avec les navigateurs pour compléter automatiquement les champs.

Un gestionnaire de mots de passe a souvent un générateur de mot de passe aléatoire.

Vous vous rappelez de la formule $N^L$ ? Donnons un exemple, en imaginant qu'on autorise :

* chiffres (10);
* lettres minuscules (26);
* lettres majuscules (26);

avec un mot de passe de longueur $L=8$. Dans ce cas on a $N=62$ et il y a 218 340 105 584 896 mots de passe possibles. Mais toujours pas assez.

On ajoute maintenant : 

* les logogrammes #, $, %, &, @, ^. `, ~ (8)
* la ponctuation . , : ; (4)
* les guillemets " ' (2)
* les tirets et les slashs | \ / _ - (5)
* les symboles mathématiques < > * + ! ? = (7)
* les accolades {[()]} (6)

et maintenant on a $k = 94$. En utilisant 12 caractères (ce qui reste assez court), le nombre de mots de passe possibles :

$$475920314814253376475136$$

And on utilise pas tous les caractères ASCII (comme les caractères accentués).

Maintenant, parlons des gestionnaires des mots de passe. Si vous voulez en savoir sur l'estimation de la sécurité d'un mot de passe, je parle d'entropie ici : [Appendix A](#appendix-a--entropy-of-passwords).

## Ma solution actuelle

Avant d'expliquer ma solution actuelle, je dois expliquer mes besoins :

1. J'ai besoin d'un gestionnaire de mots de passe (je crois que c'est clair par rapport à ce que j'ai dit);
2. Les mots de passe enregistrés dans le gestionnaire doivent pouvoir être lus et écrits depuis mes différentes machines (deux ordinateurs, un téléphone et une tablette), en sachant que les systèmes d'exploitation sont différents entre les machines (Linux, Windows (malheuresement), Android et iOS).

Et c'est tout. Bien sûr, des petits bonus comme l'intégration avec les autres outils (les navigateurs sur les ordinateurs et les claviers sur les appareils mobiles) mais les deux gros points sont ceux au dessus.

La solution que j'avais jusqu'à maintenant :

* J'utilise une base de données keepass pour engresitrer mes mots de passe, chiffrées avec un mot de passe maître (il est aussi possible de chiffrer avec un fichier ou des clés physiques);
* Cette base de données est synchronisée entre mes machines avec un serveur nextcloud.
  

Entrons dans le détail.
### keepass

[Keepass](https://keepass.info/) est un gestionnaire de mots de passe open-source ditribué sous la license License Générale Publique GNU 2 (GPLv2) mais je n'utilise pas le client keepass par défaut. En fait, j'utilise juste le format de la base données, qui est le KDBX, originellement créé par keepass.

Ainsi, mon principal client est [keepassxc](https://keepassxc.org/) qui est ausi un projet open-source, distribué sous la license GPLv3 and qui a une belle interface (voir ci dessous) et une bonne intégration avec firefox : [plus de détails ici](https://addons.mozilla.org/en-US/firefox/addon/keepassxc-browser/).

{{< figure src="posts/keepass-to-vaultwarden/keepass.png" caption="L'interface de keepassxc" numbered="true" >}}

Keepassxc a d'autres fonctionnalités sympas que j'utilise, comme :

* un générateur de mots de passe;
* une intégration pour les clés SSH (déverrouiller la base de données déverrouille la clé SSH et l'ajoute à l'agent SSH);
* le verrouillage automatique de la base de données en cas de verrouillage de la session;
* une intégration multi-sites (je n'utilise pas le même mot de passe pour différents services mais c'est utile pour les authentifications centralisées/partagées).

### Synchronisation

Pour la synchronisation, la base de données keepass est juste un fichier ordinaire et donc je dois juste synchroniser un fichier entre différents machines. Ma solution était donc de partager ce fichier (que j'utilise pour d'autres choses que juste la synchronisation du keepass).

Il est aussi possible d'utiliser des clouds commerciaux pour partager le fichier, même s'il faut alors faire confiance à la société qui possède le cloud (la base de données est néanmoins chiffrée).

## Les problèmes avec la solution actuelle

Il y a deux principaux problèmes avec cette solution :

1. Les conflits de synchronisation : un mot de passe, quand il est crée, va être écrit dans la base de données locale qui est chiffrée. Quand une connexion internet est disponible, le client va tenter de synchroniser le fichier local avec le serveur, mais cette synchronisation peut échouer, notamment si d'autres modifications ont été effectuées entre temps. Comme la base de données est chiffrée, il est dur de comparer les versions;
2. La variété des clients : j'utilise keepassxc sur les ordinateurs (linux ou windows), mais il n'y a aucune application officielle keepassxc pour les appareils mobiles. Sur mon portable android j'utilise KeePassDx et un autre client sur iPad. KeePassDx a parfois des problèmes avec la synchronisation mais le pire est l'app sur iPad où la base doit être reconfigurée à chaque synchronisation.

## Vers Vaulwarden

La solution ?  [Vaultwarden](https://github.com/dani-garcia/vaultwarden), qui est un gestionnaire de mots de passe basé sur une infrastructure clients-serveur. Mais pour comprendre l'intérêtde vaultwarden, il faut parle de [Bitwarden](https://bitwarden.com/).

Bitwarden est un gestionnaire de mots de passe basé sur une infrastructure clients-serveur avec plein de clients disponibles (web, windows, macos, linux, navigateurs, android, appstore et même des outils en ligne de commande). L'écosystème bitwarden est assez complet. Et c'est opensource 🥳.

Il est possible d'utiliser les serveurs de vaultwarden gratuitement ou pour un faible prix (voir ci dessous) et il y a aussi des offres entreprises.

{{< figure src="posts/keepass-to-vaultwarden/bitwarden-prices.png" caption="Prix des offres bitwarden" numbered="true" >}}

Il est aussi possible d'héberger son propre serveur bitwarden mais il est assez gourmand en ressources. Mais une autre implémentation du serveur existe, qui est beaucoup moins gourmande. Elle s'appelait Bitwarden_RS et s'appelle maintenant vaultwarden, et tous les clients officiels sont compatibles avec vaultwarden.

Rentrons dans le coeur du sujet technique !

La première étape est de télécharger l'image docker de vaultwarden :

```bash
docker pull vaultwarden/server:latest
```

Ensuite je génère un jeton administrateur pour la configuration initiale :

```bash
openssl rand -base64 48
```

La sortie de cette commande peut être, par exemple :

<center>
DBMmeR7o+ump0JSzsDTQJINozqkednNq55WXIFZvI+IapCR2MdyxiB8+QuHhTs1G
</center>

_Cette valeur n'est pas utilisée en production_.

Ensuite je démarre le conteneur docker avec la commande suivante :

```bash
docker run -d --name vaultwarden -e ADMIN_TOKEN=DBMmeR7o+ump0JSzsDTQJINozqkednNq55WXIFZvI+IapCR2MdyxiB8+QuHhTs1G -v /vw-data/:/data/ -p 8080:80 vaultwarden/server:latest
```

Ici le port 8080 de mon serveur est lié au port 80 du conteneur, et je gérerais la partie SSL/TLS avec un proxy apache.

Si cette commande fonctionne, l'instance vaultwarden devrait être accessible sur le port 8080 du serveur (si aucune parfeu ne bloque le port). Ça ressemble à ça :

{{< figure src="posts/keepass-to-vaultwarden/vaultwarden_home.png" caption="Premier aperçu de vaultwarden" numbered="true" >}}

Avant de faire quoi que ce soit d'autre, je configure les connexions sécurisées. Dans le dossier `/etc/apache2/sites-available` j'ajoute le fichier suivant :

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

(le certificate ssl est un challenge DNS valide sur *.nanoy.fr), puis je lance la commande

```bash
a2ensite vaultwarden; service apache2 restart
```

Et le site web est maintenant accessible à l'adresse `vaultwarden.nanoy.fr`.

Il n'y a pas de compte, mais on peut entre dans le mode d'administration en allant à l'URL `/admin` et en entran le code généré plus tôt.

{{< figure src="posts/keepass-to-vaultwarden/vaultwarden_admin.png" caption="Administration de vaultwarden" numbered="true" >}}

et on peut changer un certain nombre de paramètres :

{{< figure src="posts/keepass-to-vaultwarden/vaultwarden_admin2.png" caption="Administration de vaultwarden une fois authentifié" numbered="true" >}}

Pour ma part, j'ai configuré :

* la création ce compte personnel, et j'ai désactivé l'auto-inscription (même si l'inscription est désactivée, le lien sur la page de connexion et le formulaire d'inscription sont toujours disponibles, mais une erreur empêchera la création à la fin);
* Le nom du site;
* désactiver yubikey;
* les paramètres SMTP.

La configuration est plutôt simple et directe. Ensuite, je me connecte avec l'utilisateur que j'ai créé avant de désactiver l'auto-inscription :

{{< figure src="posts/keepass-to-vaultwarden/vaultwarden_user.png" caption="Vue de l'utilisateur" numbered="true" >}}

L'étape suivante est d'installer les clients. Pour utiliser un serveur auto-hébergé, il faut cliquer sur l'engrenage en haut à gauche et mettre l'URL correspondante. Et ensuite, j'ai commencé à migrer mes mots de passe. Il y a une fonction spécifique pour importer directement une base de données keepass dans vaultwarden, mais ma base était désorganisée donc j'ai importé à la main en réorganisant.

Contrairement à l'application que j'utilisais pour keepass sur android, il n'y a pas de clavier virtuel associé à l'application officielle bitwarden. Le clavier virtuel peut aider dans certaines situations où l'API d'auto-complémention android ne fonctionne pas. je verrai comment ça fonctionne. Un peu plus d'informations sur ces threads : https://community.bitwarden.com/t/android-virtual-keyboard-for-field-and-password-entry/3735 et https://github.com/bitwarden/mobile/issues/62.

En ce qui concerne l'authentification à double facteur sur bitwardend, je pense qu'en théorie ça peut être une bonne idée. Néanmoins :

* ça peut être gênant de devoir entrer un code à chaque déverrouillage du coffre. De ce que j'ai vu jusquà maintenant, c'est seulement lors de l'ajout d'un nouvel appareil (mais c'est peut-être un paramètre);
* Si vous perdez votre téléphone, vous devriez quand même être capable de récupérer vos mots de passe.

Le deuxième point peut être résolu avec deux considérations : 

* tout d'abord, il y a un code de restauration, en cas de perte. Il faut le garder de manière sécurisé (par exemple chiffré avec votre mot de passe maître sur différentes machines);
* ensuite, il faut aussi être capable de récupérer ses mots de passe en cas de perte du serveur vaultwarden. Pour cela, je vais probablement faire un système de backup pour exporter régulièrement les mots de passe.

Sur d'autres points, j'ai du modifié la configuration par défaut pour retrouver le comportement de keepass (fermeture automatique en cas de veille, démarrage automatique) mais rien de bien compliqué.

## Conclusion

En conclusion, je dirais que la solution du keepass partagé en utilisant un cloud (auto-hébergé ou pas) est une bonne idée qui marche bien si on oublie les deux problèmes de synchronisation et de non uniformité des clients. Aussi, pour la plupart des gens, une solution gratuite comme bitwarden (ou d'autres gestionnaires de mots de passe) marche bien aussi.

Pour le cas spécifique où l'on veut un contrôle total sur nos données et éviter les deux problèmes, une solution comme vaultwarden est bien, même s'il amène des nouvelles potentielles difficultés.


La première est la synchronisation : mon nextcloud se synchronisait souvent, et il n'y avait quasiment aucun temps entre l'édition et le partage sur les autres appareils. Avec vaultwarnden, je dois parfois demander la synchronisation manuellement. C'est peut être juste un problème de configuration pour le coup.

Un autre problème potentiel est l'absence du clavier virtuel sur android mais je dois continuer d'utiliser l'app pour voir si c'est un réel problème ou pas.

Pour le moment, vaultwarden/bitwarden fonctionne bien.

## Annexe A : Entropie des mots de passe

La notion d'_entropie d'un mot de passe_ est équivalente à la description combinatoire d'avant (quand on a compté le nombre de mots de passe possibles), mais exprimé dans une version plus informatique.

L'idée est la suivante : on veut arriver à dire "ce mot de passe est aussi sécurisé qu'une liste alétoire de $H$ bits" où $H$ est appelée l'entropie du mot de passe. Un bit peut prendre deux valeurs possibles (0 ou 1), et il y a donc $2^H$ différents listes de $H$ bits.

Mais rappelez vous, on a dit que si on a un alphabet de $N$ éléments et une longueur de $L$ alors il y a $N^L$ mots de passe possible. Ainsi, si on veut que la compraison tienne entre l'entropie et la combinatoire, il faut : 

$$N^L = 2^H$$

Ensuite on applique le logarithme (une opération qui transforme les produits en sommes, et ainsi les puissances en produits), on a 

$$\log_2(N^L) = \log_2(2^H)$$

$$L\log_2(N) = H$$

Ainsi l'entropie d'une mot de passe est la longueur du mot de passe multipliée par le logarithme binaire de la taille de l'alphabet. En fait $\log_2(N)$ peut être vu comme l'entropie moyenne par caractère, et ensuite une fois multplié par la longueur, on obtient l'entropie du mot de passe.

Par exemple :

| Nom                    |        Éléments        | Taille de l'alphabet | Entropie moyenne par caractère |
| ---------------------- | :--------------------: | -------------------: | -----------------------------: |
| Chiffres               |          0-9           |                   10 |                     3.322 bits |
| Minuscules             |          a-z           |                   26 |                     4.700 bits |
| Majuscules             |          A-Z           |                   26 |                     4.700 bits |
| Logogrammes            | #, $, %, &, @, ^. `, ~ |                    8 |                         3 bits |
| Ponctuation            |        . , : ;         |                    4 |                         2 bits |
| Guillements            |          " '           |                    2 |                          1 bit |
| Tirets et slashs       |     &#124; \ / _ -     |                    5 |                     2.322 bits |
| Symboles mathématiques |     < > * + ! ? =      |                    7 |                     2.807 bits |
| Accolades              |         {[()]}         |                    6 |                     2.585 bits |
    

Prenons un exemple concret, avec le mot de passe :

<center>
RS=%Q@K2Ecxt2g
</center>

La longueur du mot de passe est 14 et la taille de l'alphabet est 10 (chiffres) + 26 (minuscules) + 26 (majuscules) + 8 (logogrammes) + 7 (symboles mathématiques) = 77, et l'entropie du mot de passe est :

$$14\times\log_2(77) = 87.735 \text{ bits}$$

ce qui est quasiement ce qui est donné par keepass (91.16 bits) :

{{< figure src="posts/keepass-to-vaultwarden/keepass_entropy.png" caption="Entropie du mot de passe RS=%Q@K2Ecxt2g" numbered="true" >}}

Comment expliquer cette différence ? On peut remarquer que si on génère plusieurs mots de passe avec le même alphabet et la même longueur, il y aura des differences d'entropies, ce qui ne devrait pas être le cas avec notre modèle. En fait notre modèle est valide si les mots de passe étaient vraiment alétatoires, ce qui n'est pas souvent le cas dans la vraie vie. Ainsi beaucoup d'estimations se basent sur d'autres algorithmes pour calculer l'entropie.

Keepass utilise par exemple l'algorithme zxcvbn, qui vérifie aussi pour la présence de motifs et répétitions (plus d'informations sur ce [billet de blog](https://dropbox.tech/security/zxcvbn-realistic-password-strength-estimation) très intéressant).

Plus d'informations sur la [page wikipedia](https://fr.wikipedia.org/wiki/Robustesse_d%27un_mot_de_passe).

## Annexe B : authentification multi-facteurs

Dans ce court billet, je n'ai parlé que de mots de passe. Aujourd'hui, il y a d'autres méthodes d'authenfication et elles sont généralement combinées.

Les méthodes d'authentification sont regroupées en trois catégories :

* quelque chose que l'on sait (un mot de passe, une phrase de passe, la réponse à une question de secours, etc...);
* quelque chose que l'on a (un numéro de téléphone, une application mobile, une adresse email);
* quelque chose que l'on est (reconnaissance faciale, empreintes digitales, etc...).

Lorsque des méthodes d'authentification d'au moins deux groupes sont combinées, on parle d'authentification à multiples facteurs.

Quelques exemples : 

* mot de passe et un code par SMS/email;
* mot de passe et confirmation sur une application;
* mot de passe et empreinte digitale.

Souvent, le mot de passe est l'une des méthodes et la seconde et quelque chose que l'on a (email ou application ou SMS).

La reonnaissance faciale et les empreintes digitales sont souvent utilisés pour déverouiller un téléphone (et peuvent dont être considérés comme une méthode d'authenfication supplémentaire au SMS ou à l'application).

Le problème avec le "quelque chose que l'on est" est qu'il est très compliqué voire impossible de changer quelque chose. Si un mot de passe est corrumpu on peut juste le remplacer. On peut aussi changer un numéro de téléphone. Par contre il est impossible de changer des empreintes digitales et très compliqué de changer de tête.

Dans tous les cas, l'authentification à multiples facteurs est une bonne chose et il y a des applications cools (comme [2FAS](https://2fas.com/) et de plus en plus de services propensent une double authentification. J'encourage fortement d'utiliser plusieurs facteurs d'authentification).