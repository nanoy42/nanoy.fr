---
title: Building a plugin system with Django
summary: Want to learn how to make a plugin system in Django ? Be sure to read this quick tutorial !
date: "2020-09-27T00:00:00Z"
lastmod: "2021-04-25T00:00:00Z"

authors:
  - nanoy
  - 
tags:
  - django
  - plugins

categories:
  - programmation
  - 
image:
  caption: 'Image credit: django'
---


Je vais présenter une méthode pour implémenter une infrastructure de plugin dans un environnement django. Cette méthod s'inspire en partie du travail fait par par pretix [https://github.com/pretix/pretix](https://github.com/pretix/pretix) et implémtentée sur barman [https://github.com/barmanaginn/barman](https://github.com/barmanaginn/barman).

## Cahier des charges

Avant de nous lancer tête baissée dans la conception de l'infrastructure, essayons de definir ce que l'on veut :

* le plugin doit être une app django
* l'installation du plugin se fait via setuptools/pip. Il n'y a pas besoin de modifier la configuration pour installer un plugin.
* La configuration du plugin se fait via le fichier de configuration globale
* Pas d'installation web
* Récuperation de configuration transparent pour le plugin
* Possibilité de renseigner des liens à mettre dans la barre de navigation

## Conception

### La classe principale

Le dossier pour notre plugin sera assez similaire à un dossier pour une app : locale, migrations, templates, __init__.py, models.py, urls.py, etc... Un fichier supplémetaire sera le fichier signals.py, qui ne sera pas obligatoire mais sera par défaut dans le template.

On va créer une classe qui va permettre de renseigner les informations du plugin. Cette classe héritera de `BarmanPlugin`

```python
class BarmanPlugin(AppConfig):
    IGNORE = False

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        if not hasattr(self, "BarmanPluginMeta"):
            raise ImproperlyConfigured(
                "A BarMan plugin config should have a BarmanPluginMeta inner class."
            )

```

Cette classe hérite de AppConfig. La fonction `init` vérifie que la classe a bien une sous classe `BarmanPluginMeta` dans laquelle sera définie tous les paramètres de notre plugin.

Voyons maintenant à quoi ressemble l'instanciation d'un plugin :

```python
from barman.plugin import BarmanPlugin

class PluginApp(BarmanPlugin):
    name = "barman_rankings"

    class BarmanPluginMeta:
        name = "rankings"
        author = "Yoann Pietri"
        description = _("Add rankings to BarMan")
        version = 0.1
        url = "https://github.com/barmanaginn/barman-rankings"
        email = "me@nanoy.fr"

    def ready(self):
        from . import signals

        return super().ready()

default_app_config = "barman_rankings.PluginApp"
```

On a donc ici la classe PluginApp, définie dans le `__init__.py` et qui définie donc un certain nombre de paramètres : le nom du plugin, l'identité de l'author,  la version, unlien et une description.

### Ajouter automatique les plugins installés

Nous allons maintenant voir comment les plugins sont automatiquement ajoutés et comment on gère les URLs.

Dans notre `settings.py` nous allons importer la fonction `iter_entry_points` et nous allons définir la variable `PLUGINS` grâce à elle. Tous les plugins devront avoir défini le groupe `barman.plugin`. Le plugin est alors rajouté dans la variables `PLUGINS` et dans la variable `INSTALLED_APPS`.

```python
from pkg_resources import iter_entry_points

PLUGINS = []

for entry_point in iter_entry_points(group="barman.plugin", name=None):
    INSTALLED_APPS.append(entry_point.module_name)
    PLUGINS.append(entry_point.module_name)
```

Dernière chose, il faut que les urls de chaque plugin soit ajouté aux urls du site.

On ajouter un namespace `plugins`

```python
    path("", include((plugins_urlpatterns, "plugins"), namespace="plugins")),
```

avec 

```python
for app in apps.get_app_configs():
    if hasattr(app, "BarmanPluginMeta"):
        if importlib.util.find_spec(app.name + ".urls"):
            urlmod = importlib.import_module(app.name + ".urls")
            plugins_urlpatterns.append(
                path("", include((urlmod.urlpatterns, app.label), namespace=app.label),)
            )
```

ce qui veut dire que les urls auront pour nom d'appel `plugins:nomduplugin:url`.

Ainsi, nous avons ici un exemple minimal de fonctionnement d'un système de plugin.

### Les liens pour la navigation

Ajouter des liens pour la navigation n'est pas très dur. On ajoute dans la docuement la possibilité d'ajouter la variable `nav_urls` dans `BarmanPluginMeta`, avec la forme

```python
# Define here urls for navbar. See documentation for more details.
        nav_urls = (
            {
                "text": _("Clubs"),
                "icon": "fas fa-user-friends",
                "link": reverse_lazy("plugins:barman_clubs:clubs-index"),
                "permission": "",
                "login_required": True,
                "admin_required": False,
                "superuser_required": False,
            },
            {
                "text": _("Distribution"),
                "icon": "fa fa-hand-holding-usd",
                "link": "",
                "permission": "",
                "login_required": True,
                "admin_required": False,
                "superuser_required": False,
            },
        )
```

puis on écrit un tag de template pour générer le html dans la navbar :

```python
@register.simple_tag
def plugins_nav_links_login(user):
    template = """
    <span class="tabulation2">
	    <i class="{}"></i> <a href="{}">{}</a>
    </span>
    """
    res = ""
    for app in apps.get_app_configs():
        if hasattr(app, "BarmanPluginMeta"):
            if hasattr(app.BarmanPluginMeta, "nav_urls"):
                for link in app.BarmanPluginMeta.nav_urls:
                    if (
                        link["superuser_required"]
                        and user.is_superuser
                        or not link["superuser_required"]
                    ):
                        if (
                            link["admin_required"]
                            and user.is_staff
                            or not link["admin_required"]
                        ):
                            if (
                                link["permission"]
                                and user.has_perm(link["permission"])
                                or not link["permission"]
                            ):
                                res += template.format(
                                    link["icon"], link["link"], link["text"]
                                )
    return res
```
avec une deuxième fonction pour les liens pour les utilisateurs qui ne sont pas authentifiés.

### La configuration

Dernière chose, on veut pouvoir génére rune configuration pour chaque plugin. L'idée de base est la même que pour les liens : le plugin doit définir dans sa classe un certains nombre de paramètres avec un nom, une descritpion et une valeur par défaut : 

```python
settings = (
            {
                "name": "RANKINGS_LENGTH",
                "description": _("Number of places to display in the ranking"),
                "default": 20,
            },
```

Il va falloir aussi modifier un peu la classe BarmanPlugin. On va définir une fonction qui ajoute tous les paramètres comme atribut de la classe `BarmanMetaPlugin` avec la valeur qui se trouve dans le `settings.py` et si aucune valeur ne s'y trouve, celle qui se trouve dans la définition du plugin

```python
class BarmanPlugin(AppConfig):
    IGNORE = False

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        if not hasattr(self, "BarmanPluginMeta"):
            raise ImproperlyConfigured(
                "A BarMan plugin config should have a BarmanPluginMeta inner class."
            )

    def get_settings(self):
        if hasattr(self.BarmanPluginMeta, "settings"):
            for setting in self.BarmanPluginMeta.settings:
                setattr(
                    self.BarmanPluginMeta,
                    setting["name"],
                    getattr(settings, setting["name"], setting["default"]),
                )

    def ready(self):
        self.get_settings()
```

La fonction ready est automatiquement lancée par django au chargement. Il faut bien que la classe du plugin appelle la methode ready du parent.

## Petit bonus : un template cookiecutter

Dernière petite chhose : il est intéressant de proposer un template cookiecutter pour les plugins, voir [https://github.com/barmanaginn/barman-plugin-cookiecutter](https://github.com/barmanaginn/barman-plugin-cookiecutter)