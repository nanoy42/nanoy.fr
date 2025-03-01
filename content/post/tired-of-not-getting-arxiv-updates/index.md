---
title: Tired of Not Getting arXiv Updates ?
summary: Take full control of your personal brand and privacy by migrating away from the big tech platforms!
date: 2022-12-22T01:08:50+01:00
lastmod: 2022-12-22T01:08:50+01:00

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: 'Image credit: arXiv and Cornell University'

authors:
  - nanoy

tags:
  - python
  - arxiv
  - telegram
  - bot
---

## The genesis

[arxiv](https://arxiv.org) is a great project for open science, but with the number of articles added everyday, it's easy to miss articles related to your domain. When I started my PhD, I was facing this issue: I wanted to be up-to-date with the current state-of-the-art and I didn't wanted to check manually the arxiv everyday.

My idea was to create a bot that would scrap the RSS feed from arxiv each morning, and, after matching keywords, send the article to me. As I was using a lot the [telegram](https://telegram.org/) messaging app, that has a great API for bots, I decided to go along with this.

The scheme is the following: I decided to code a python script, that will read a configuration file, and fetch the RSS feed for one given arxiv category. Categories are the way articles are grouped together on arxiv and the list can be found [here](https://arxiv.org/category_taxonomy). Then the script will go through every article and check the title against a list of "buzzwords" defined in the configuration file. If one of the buzzwords is in the title, the title, authors and link of the article is send by telegram message to the configured user or group.

The configuration file looks like this 

```ini
[bot]
token = 0000000000:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

[quant-ph]
chat_id = 0
category = quant-ph
buzzwords = cvqkd,continuous variable,continuous-variable,qkd,quantum key distribution,rfsoc,fpga
```

Note: this is neither a real token nor a real chat id.

As you see, it's possible to define as many updates as you want, with different categories, chat ids and buzzwords.

## Using the python script

Using the python script is pretty straightforward. The code can be found on the [github repo](https://github.com/nanoy42/arxiv-update-bot), and a pypi package is also [provided](https://pypi.org/project/arxiv-update-bot/)

```
pip install arxiv-update-bot
```

Then you have to create a configuration file. By default, the script will look for its configuration file in `/etc/arxiv-update-bot/config.ini` but it's possible to change this behaviour with a command line argument. An example configuration file is shipped with the package and corresponds to the one shown above.

Once this is done, you can call the script. If you cloned the repository, you can execute the `main.py` file in the `arxiv_update_bot` module. If you installed with pip, the command `arxiv-update-bot` should have been installed. Here is the documentation of this command:

```
usage: arxiv-update-bot [-h] [-c CONFIG_PATH] [-q] [-p]

Scrap the arXiv

options:
  -h, --help            show this help message and exit
  -c CONFIG_PATH, --config-path CONFIG_PATH
                        Path for configuration path. Replace default of /etc/arxiv-update-bot/config.ini
  -q, --quiet           If quiet is set, then the bot will not send message if no article are found.
  -p, --print-info      If print-info is set, then the bot will send its configuration instead of the updates.
```

Hence the path of the configuration file can be given with the `-c` file. You can check that everything is working well with the following command:

```
arxiv-update-bot -c /path/to/config.ini -p
```

If you type 

```
arxiv-update-bot -c /path/to/config.ini
```

you should receive the update. However is it not yet very practical as we still have to manually execute the command to get the updates.

## Automate with cron

If you have a server (or anything that just stays up, like a raspberry pi), it's possible to use cron, to schedule the tasks.

For me, here is the cron job I configured:

``
0 8 * * * /home/nanoy/bots/arxiv-update-bot/.venv/bin/arxiv-update-bot -c /home/nanoy/bots/arxiv-update-bot/config.ini
``

Hence, each morning at 8, the script is called and I received the updates.


## The docker image

As of version 0.8 of the module, I introduced a [docker image](https://hub.docker.com/r/nanoy/arxiv-update-bot) that can be used to run the bot with minimal commands. You can either configure it directly with the docker command line

```
docker run -d -t -i -e AUB_TOKEN=0000000000:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA -e AUB_CHAT_IDS=0 -e AUB_CATEGORIES=quant-ph -e AUB_BUZZWORDS='cvqkd,cv-qkd,continuous variable,continuous-variable,qkd,quantum key distribution,rfsoc,fpga' -e AUB_CRONTAB='0 10 * * *' --name arxiv-update-bot nanoy/arxiv-update-bot
```

or with docker-compose 

```yml
version: '3.6'

services:
  arxiv-update-bot:
    image: nanoy/arxiv-update-bot
    environment:
      - AUB_TOKEN=0000000000:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
      - AUB_CHAT_IDS=0;10
      - AUB_CATEGORIES=quant-ph;category2
      - AUB_BUZZWORDS=cvqkd,cv-qkd,continuous variable,continuous-variable,qkd,quantum key distribution,rfsoc,fpga;buzzword1, buzzword2
      - AUB_CRONTAB=0 10 * * *

```

## Conclusion

Don't hesitate to report any bug or improvement on the [issue tracker](https://github.com/nanoy42/arxiv-update-bot/issues).

Hopefully, this can be be useful to other people (software is release under the MIT license).

> Disclosure: Thank you to arXiv for use of its open access interoperability. This project was not reviewed or approved by, nor does it necessarily express or reflect the policies or opinions of, arXiv.