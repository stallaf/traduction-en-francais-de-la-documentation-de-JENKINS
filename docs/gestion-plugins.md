# Gestion plugins

<div class="couleur-introduction">
Les plugins sont le principal moyen d'améliorer les fonctionnalités d'un environnement Jenkins afin de répondre aux besoins spécifiques d'une organisation ou d'un utilisateur. Il existe [plus d'un millier de plugins différents](https://plugins.jenkins.io/) qui peuvent être installés sur un contrôleur Jenkins et qui permettent d'intégrer divers outils de construction, fournisseurs de cloud, outils d'analyse, et bien plus encore.
</div>

Les plugins peuvent être téléchargés automatiquement, avec leurs dépendances, à partir du [Centre de mise à jour](./glossaire.md#centre-de-mise-a-jour). Le Centre de mise à jour est un service géré par le projet Jenkins qui fournit un inventaire des plugins open source développés et maintenus par divers membres de la communauté Jenkins.

Cette section couvre tout, depuis les bases de la gestion des plugins dans l'interface utilisateur web Jenkins jusqu'à la modification du système de fichiers du [contrôleur](./glossaire.md#controleur).

## Installation d'un plugin

Jenkins propose deux méthodes pour installer des plugins sur le contrôleur :

* À l'aide du « Plugin Manager » dans l'interface utilisateur Web ;
* À l'aide de la [commande CLI](#installation-en-ligne-de-commande) `install-plugin` de Jenkins.

Chaque approche permet à Jenkins de charger le plugin, mais peut nécessiter différents niveaux d'accès et des compromis pour pouvoir être utilisée.

Les deux approches nécessitent que le contrôleur Jenkins soit capable de télécharger des métadonnées à partir d'un centre de mise à jour, qu'il s'agisse du centre de mise à jour principal géré par le projet Jenkins [^1] ou d'un centre de mise à jour personnalisé.

Les plugins sont conditionnés sous forme de fichiers `.hpi` autonomes, qui contiennent tout le code, les images et les autres ressources nécessaires au bon fonctionnement du plugin.

### À partir de l'interface utilisateur Web

La méthode la plus simple et la plus courante pour installer des plugins consiste à utiliser la vue **Manage Jenkins > Plugins** (Gérer Jenkins > Plugins), accessible aux administrateurs d'un environnement Jenkins.

Sous l'onglet **Available** (Disponible), vous pouvez rechercher et examiner les plugins disponibles au téléchargement à partir du centre de mise à jour configuré :

![Onglet Available (Disponible) dans le gestionnaire de plugins](https://www.jenkins.io/doc/book/resources/blueocean/intro/blueocean-plugins-filtered.png)

La plupart des plugins peuvent être installés et utilisés immédiatement en cochant la case à côté du plugin et en cliquant sur **Install without restart** (Installer sans redémarrer).
	
!!! danger "Attention"
    Si la liste des plugins disponibles est vide, il se peut que le contrôleur soit mal configuré ou qu'il n'ait pas encore téléchargé les métadonnées des plugins depuis le centre de mise à jour. Cliquez sur le bouton Vérifier maintenant pour forcer Jenkins à tenter de contacter le centre de mise à jour configuré.

### Utilisation de l'interface CLI Jenkins

_Installation des plugins Jenkins à l'aide de l'interface CLI Jenkins_

<iframe width="800" height="420" src="https://www.youtube.com/embed/bTFMvXIkNIg" title="How to Install Jenkins Plugins From Command Line Using the Jenkins CLI" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Les administrateurs peuvent également utiliser l'interface [CLI Jenkins](gestion-cli.md) qui fournit une commande permettant d'installer des plugins. Les scripts permettant de gérer les environnements Jenkins, ou le code de gestion de la configuration, peuvent nécessiter l'installation de plugins sans interaction directe de l'utilisateur dans l'interface utilisateur Web. L'interface CLI Jenkins permet à un utilisateur de ligne de commande ou à un outil d'automatisation de télécharger un plugin et ses dépendances.

``` bash
java -jar jenkins-cli.jar -s http://localhost:8080/ install-plugin SOURCE ...[-deploy] [-name VAL] [-restart]
```

``` cfg
Installe un plugin à partir d'un fichier, d'une URL ou du centre de mise à jour.

 SOURCE    : si cela pointe vers un fichier local, ce fichier sera installé. Si
             il s'agit d'une URL, Jenkins télécharge l'URL et l'installe en tant que
             plugin. Sinon, le nom est supposé être le nom court du
             plugin dans le centre de mise à jour existant (comme « findbugs »), et le
             plugin sera installé à partir du centre de mise à jour.
 -deploy   : Déploie les plugins immédiatement sans les reporter jusqu'au redémarrage.
 -name VAL : Si spécifié, le plugin sera installé sous ce nom court
             (alors que normalement, le nom est déduit automatiquement du nom de la source ).
 -restart  : Redémarre Jenkins une fois l'installation réussie.
```

### Installation avancée

Le Centre de mise à jour permet uniquement l'installation de la dernière version publiée d'un plugin. Si vous souhaitez utiliser une version plus ancienne du plugin, un administrateur Jenkins peut télécharger une archive `.hpi` plus ancienne et l'installer manuellement sur le contrôleur Jenkins.

Jenkins stocke les plugins qu'il a téléchargés dans le répertoire `plugins` avec une extension `.jpi`, que les plugins aient à l'origine une extension `.jpi` ou `.hpi`.

Si un administrateur copie manuellement une archive de plugin dans le répertoire `plugins`, celle-ci doit être nommée avec une extension `.jpi` afin de correspondre aux noms de fichiers utilisés par les plugins installés à partir du centre de mise à jour.

#### À partir de l'interface utilisateur Web

En supposant qu'un fichier `.hpi` ait été téléchargé, un administrateur Jenkins connecté peut télécharger le fichier à partir de l'interface utilisateur Web :

1. Accédez à la page **Manage Jenkins > Plugins** (Gérer Jenkins > Plugins) dans l'interface utilisateur Web.
2. Cliquez sur l'onglet **Advanced** (Avancé).
3. Sélectionnez le fichier `.hpi` sur votre système ou entrez l'URL du fichier d'archive dans la section Déployer le plugin.
4. **Deploy** (Déployez) le fichier du plugin.

![Onglet Avancé dans le gestionnaire de plugins](https://www.jenkins.io/doc/book/resources/managing/plugin-manager-upload.png)

Une fois le fichier du plugin téléchargé, le contrôleur Jenkins doit être redémarré manuellement pour que les modifications prennent effet.

#### Sur le contrôleur

En supposant qu'un fichier `.hpi` ait été explicitement téléchargé par un administrateur système, celui-ci peut placer manuellement le fichier à un emplacement spécifique du système de fichiers.

Copiez le fichier `.hpi` téléchargé dans le répertoire `JENKINS_HOME/plugins` du contrôleur Jenkins (par exemple, sur les systèmes Debian, `JENKINS_HOME` est généralement `/var/lib/jenkins`). Si un administrateur copie manuellement une archive de plugin dans le répertoire `plugins`, celle-ci doit être nommée avec une extension `.jpi` afin de correspondre aux noms de fichiers utilisés par les plugins installés à partir du centre de mise à jour.

Le contrôleur doit être redémarré avant que le plugin ne soit chargé et disponible dans l'environnement Jenkins.

!!! info " "
    Les noms des répertoires de plugins dans le site de mise à jour [^1] ne sont pas toujours les mêmes que le nom d'affichage du plugin. Une recherche du plugin souhaité sur [plugins.jenkins.io](https://plugins.jenkins.io/) fournira le lien approprié vers le fichier d'archive.

[^1]: [updates.jenkins.io](https://updates.jenkins.io/) 

## Mise à jour d'un plugin

Les mises à jour sont répertoriées dans l'onglet **Updates** (Mises à jour) de la page **Plugins** et peuvent être installées en cochant les cases des mises à jour souhaitées, puis en cliquant sur le bouton Télécharger maintenant et installer après redémarrage.

![Onglet Mises à jour dans le gestionnaire de plugins](https://www.jenkins.io/doc/book/resources/managing/plugin-manager-update.png)

Par défaut, le contrôleur Jenkins vérifie les mises à jour disponibles dans le centre de mise à jour toutes les 24 heures. Pour déclencher manuellement une vérification des mises à jour, il suffit de cliquer sur le bouton Vérifier maintenant dans l'onglet Mises à jour.

### Répertorier les plugins installés et leurs versions

La façon la plus simple de récupérer la liste des plugins installés et leurs versions est d'utiliser la console de script Jenkins.

Procédez comme suit :

1. Ouvrez la [console de script](#console-de-script) Jenkins :
    * Accédez à `Manage Jenkins` > `Script Console` ;
2. Exécutez le script suivant pour répertorier les plugins installés et leurs versions :

``` groovy
   Jenkins.instance.pluginManager.plugins.each {
      println("${it.getShortName()}: ${it.getVersion()}")
   }
```

Ce script parcourt chaque plugin installé et affiche son nom abrégé ainsi que sa version.

## Suppression d'un plugin

Lorsqu'un plugin n'est plus utilisé dans un environnement Jenkins, il est prudent de le supprimer du contrôleur Jenkins. Cela présente plusieurs avantages, notamment la réduction de la surcharge mémoire au démarrage ou lors de l'exécution, la réduction des options de configuration dans l'interface utilisateur Web et la suppression du risque de conflits futurs avec les nouvelles mises à jour du plugin.

### Désinstallation d'un plugin

_Cette vidéo passe en revue le processus de désinstallation d'un plugin de Jenkins._

<iframe width="800" height="420" src="https://www.youtube.com/embed/Keh6riX7574" title="Can We Uninstall Plugins From Jenkins?" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

La façon la plus simple de désinstaller un plugin est d'accéder à l'onglet **Installed** (Installés) de la page **Plugins**. À partir de là, Jenkins déterminera automatiquement quels plugins peuvent être désinstallés en toute sécurité, c'est-à-dire ceux qui ne sont pas dépendants d'autres plugins, et affichera un bouton permettant de le faire.

![Onglet Installed dans le gestionnaire de plugins](https://www.jenkins.io/doc/book/resources/managing/plugin-manager-uninstall.png)

Un plugin peut également être désinstallé en supprimant le fichier `.jpi` correspondant du répertoire `JENKINS_HOME/plugins` sur le contrôleur. Le plugin continuera à fonctionner jusqu'au redémarrage du contrôleur.

!!! info " "
    Si un fichier de plugin est supprimé mais requis par d'autres plugins, le contrôleur Jenkins peut ne pas démarrer correctement.

La désinstallation d'un plugin ne supprime pas la configuration que le plugin a pu créer. S'il existe des configurations de tâches/nœuds/vues/compilations/etc. qui font référence à des données créées par le plugin, Jenkins affichera un avertissement lors du démarrage indiquant que certaines configurations n'ont pas pu être entièrement chargées et ignorera les données non reconnues.

Comme les configurations seront conservées jusqu'à ce qu'elles soient écrasées, la réinstallation du plugin entraînera la réapparition de ces valeurs de configuration.

#### Suppression des anciennes données

Jenkins fournit une fonctionnalité permettant de purger les configurations laissées par les plugins désinstallés. Accédez à **Manage Jenkins** (Gérer Jenkins), puis cliquez sur **Manage Old Data** (Gérer les Anciennes Données) pour consulter et supprimer les anciennes données.

### Désactivation d'un plugin

La désactivation d'un plugin est une manière plus douce de le retirer. Jenkins continuera à reconnaître que le plugin est installé, mais il ne le lancera pas et aucune extension fournie par ce plugin ne sera visible.

Un administrateur Jenkins peut désactiver un plugin en décochant la case dans l'onglet **Installed** (Installé) de la page **Plugins** (voir ci-dessous).

![Onglet Installed dans le gestionnaire de plugins](https://www.jenkins.io/doc/book/resources/managing/plugin-manager-disable.png)

Un administrateur système peut également désactiver un plugin en créant un fichier sur le contrôleur Jenkins, tel que : `JENKINS_HOME/plugins/PLUGIN_NAME.jpi.disabled`.

Les configurations créées par le plugin désactivé se comportent comme si le plugin avait été désinstallé, dans la mesure où elles génèrent des avertissements au démarrage, mais sont par ailleurs ignorées.

#### Utilisation de l'interface CLI

Il est également possible d'activer ou de désactiver des plugins via [l'interface CLI de Jenkins](./gestion-cli.md) à l'aide des commandes `enable-plugin` ou `disable-plugin`.

!!! info " "
    La commande `enable-plugin` a été ajoutée à Jenkins dans la [version 2.136](https://www.jenkins.io/changelog/#v2.136). La commande `disable-plugin` a été ajoutée à Jenkins dans la [version 2.151](https://www.jenkins.io/changelog/#v2.151).

La commande `enable-plugin` reçoit une liste des plugins à activer. Tous les plugins dont dépend un plugin sélectionné seront également activés par cette commande.

``` bash
java -jar jenkins-cli.jar -s http://localhost:8080/ enable-plugin PLUGIN ... [-restart]
```

``` cfg
Active un ou plusieurs plugins installés de manière transitive.

 PLUGIN   : Active les plugins avec les noms courts donnés et leurs
            dépendances.
 -restart : Redémarre Jenkins après avoir activé les plugins.
```

La commande `disable-plugin` reçoit une liste des plugins à désactiver. La sortie affiche des messages pour les opérations réussies et échouées. Si vous souhaitez uniquement voir les messages d'erreur, vous pouvez spécifier l'option `-quiet`. L'option `-strategy` contrôle l'action à entreprendre lorsqu'un des plugins spécifiés est répertorié comme dépendance facultative ou obligatoire d'un autre plugin activé.

``` bash
java -jar jenkins-cli.jar -s http://localhost:8080/ disable-plugin PLUGIN ... [-quiet (-q)]
[-restart (-r)] [-strategy (-s) strategy]
```

``` cfg
Désactivez un ou plusieurs plugins installés.
Désactivez les plugins avec les noms courts indiqués. Vous pouvez définir comment procéder avec les
plugins dépendants et si un redémarrage doit être effectué après. Vous pouvez également activer le mode silencieux
pour éviter les informations supplémentaires dans la console.

 PLUGIN                  : Plugins à désactiver.
 -quiet (-q)             : Mode silencieux, n'affiche que les messages d'erreur.
 -restart (-r)           : Redémarre Jenkins après avoir désactivé les plugins.
 -strategy (-s) strategy : Comment traiter les plugins dépendants.
                           - none : si un plugin dépendant obligatoire existe et
                           qu'il est activé, le plugin ne peut pas être désactivé
                           (valeur par défaut).
                           - mandatory : tous les plugins dépendants obligatoires sont
                           également désactivés, les plugins dépendants facultatifs restent
                           activés.
                           - all : tous les plugins dépendants sont également désactivés,
                           que leur dépendance soit facultative ou obligatoire.
```

!!! info " "
    Tout comme l'activation et la désactivation des plugins depuis l'interface utilisateur nécessitent un redémarrage pour que le processus soit complet, les modifications apportées à l'aide des commandes CLI prendront effet une fois Jenkins redémarré. L'option `-restart` force un redémarrage sécurisé du contrôleur une fois la commande terminée avec succès, afin que les modifications soient immédiatement appliquées.

## Plugins épinglés

La fonctionnalité des plugins épinglés a été supprimée dans Jenkins 2.0. Les versions postérieures à Jenkins 2.0 ne regroupent pas les plugins, mais fournissent à la place un assistant pour installer les plugins les plus utiles.

La notion de **plugins épinglés** s'applique aux plugins fournis avec Jenkins 1.x, tels que le [plugin Matrix Authorization](https://plugins.jenkins.io/matrix-auth).

Par défaut, chaque fois que Jenkins est mis à niveau, ses plugins intégrés écrasent les versions des plugins actuellement installés dans `JENKINS_HOME`.

Cependant, lorsqu'un plugin intégré a été mis à jour manuellement, Jenkins marque ce plugin comme épinglé à la version particulière. Dans le système de fichiers, Jenkins crée un fichier vide appelé `JENKINS_HOME/plugins/PLUGIN_NAME.jpi.pinned `pour indiquer l'épinglage.

Les plugins épinglés ne seront jamais écrasés par les plugins intégrés lors du démarrage de Jenkins. (Les versions plus récentes de Jenkins vous avertissent si un plugin épinglé est plus ancien que celui qui est actuellement intégré.)

Il est possible de mettre à jour en toute sécurité un plugin intégré vers une version proposée par le Centre de mise à jour. Cela est souvent nécessaire pour bénéficier des dernières fonctionnalités et corrections. La version intégrée est parfois mise à jour, mais pas de manière régulière.

Le gestionnaire de plugins permet de désactiver explicitement les plugins. Le fichier `JENKINS_HOME/plugins/PLUGIN_NAME.hpi.pinned` peut également être créé/supprimé manuellement pour contrôler le comportement d'épinglage. Si le fichier `pinned` est présent, Jenkins utilisera la version du plugin spécifiée par l'utilisateur. Si le fichier est absent, Jenkins restaurera la version par défaut du plugin au démarrage.

## Qu'est-ce qu'une dépendance implicite ?

Les fonctionnalités sont parfois détachées (ou séparées) du noyau Jenkins et déplacées vers un plugin.

De nombreux plugins, tels que [Subversion](https://plugins.jenkins.io/subversion) et [JUnit](https://plugins.jenkins.io/junit), ont vu le jour en tant que fonctionnalités du noyau Jenkins. Lorsqu'un plugin était attaché au noyau Jenkins avant d'être détaché, il pouvait ou non utiliser toutes ses fonctionnalités avec d'autres plugins qui reposaient sur la même version de Jenkins. Pour garantir que les plugins ne tombent pas en panne lorsqu'une fonctionnalité dont ils dépendent est séparée du noyau Jenkins, il est nécessaire d'avoir une dépendance dans le plugin détaché, s'il spécifie une dépendance à une version du noyau Jenkins antérieure à la séparation. On suppose qu'il existe une dépendance au plugin détaché, même si cela n'est pas explicitement indiqué.

Les dépendances implicites s'accumulent au fil du temps pour les plugins qui ne mettent pas fréquemment à jour la version du noyau Jenkins dont ils dépendent.

### Pourquoi cette dépendance implicite existe-t-elle ?

Elle est « implicite » car elle n'est pas explicitement mentionnée dans le plugin, et il s'agit d'une dépendance car le noyau Jenkins ne sait pas si les API disponibles dans le noyau Jenkins, avant la séparation du plugin, sont nécessaires ou non.

Par exemple, comme le plugin Instance Identity a été séparé du noyau Jenkins, celui-ci ne sait pas si le plugin dépendant a besoin d'une fonctionnalité qui était auparavant présente dans le noyau Jenkins. Cela crée une dépendance implicite.

En tant qu'administrateur Jenkins, vous pouvez voir les plugins qui ont une dépendance implicite depuis la page du gestionnaire de plugins. Passez la souris sur le bouton « désinstaller » et une liste des plugins ayant une dépendance implicite s'affiche.
Passez la souris sur le bouton « désinstaller » pour voir les dépendances implicites

![Comment supprimer la dépendance implicite ](https://www.jenkins.io/doc/book/resources/managing/hover-for-implied-dependencies.png)

### Comment supprimer la dépendance implicite ?

La dépendance implicite peut être supprimée en publiant une nouvelle version du plugin qui dépend d'un noyau Jenkins minimum plus récent. Le[] tutoriel Améliorer un plugin](https://www.jenkins.io/doc/developer/tutorial-improve/) fournit des étapes qui peuvent aider les développeurs de plugins à mettre à jour leur plugin afin qu'il [dépende d'un noyau Jenkins minimum plus récent](https://www.jenkins.io/doc/developer/tutorial-improve/update-base-jenkins-version/).

## Qu'est-ce que le score de santé ?

Une métrique appelée « Score de santé » est disponible à côté des plugins dans le gestionnaire de plugins.

![Score de santé des plugins dans la liste des plugins disponibles du gestionnaire de plugins](https://www.jenkins.io/doc/book/resources/managing/plugin-manager-available-health-score.png)

Vous trouverez la même métrique dans la page des mises à jour disponibles et la page des plugins installés du gestionnaire de plugins.

!!! info " "
	La valeur du score est toujours basée sur les dernières données disponibles. Cela signifie qu'il n'y a pas de score pour une version spécifique d'un plugin. Ce qui est disponible sur la page des mises à jour et des installations du gestionnaire de plugins ne correspond pas à la version déjà installée.

Cette métrique provient du [Plugin Health Scoring](https://github.com/jenkins-infra/plugin-health-scoring).

Ce service recueille des données auprès de chaque plugin.

Ces données peuvent être liées au code source du plugin, au contenu du référentiel du plugin ou à des mesures externes concernant le plugin, telles que le nombre de demandes d'extraction ouvertes.

Ces données sont ensuite utilisées pour évaluer l'état du projet. Tous les plugins sont évalués de la même manière, ce qui signifie qu'il existe un moyen équitable de comparer les plugins.

Bien que seul le score soit affiché, les détails derrière cette valeur sont disponibles sur le[ site des plugins](https://plugins.jenkins.io/) dans la section « Health Score » (Score de santé) de chaque plugin :

![Score de santé des plugins sur le site des plugins](https://www.jenkins.io/doc/book/resources/managing/plugin-site-health-score.png)

Il est également possible de trouver les détails directement sur le service [Plugin Health Scoring](https://plugin-health.jenkins.io/). Pour y accéder, vous devez ajouter le nom du plugin à l'URL du score. Par exemple, l'URL du score de santé du plugin `Mailer` est [plugin-health.jenkins.io/scores/mailer](https://plugin-health.jenkins.io/scores/mailer).

Chaque section du score peut comporter un ou plusieurs éléments évalués. Chaque composant de la section peut avoir plus ou moins d'impact sur le score global.

### Comment améliorer les scores ?

Les données recueillies par le projet Plugin Health Scoring sur chaque plugin sont évaluées fréquemment. Consultez les recommandations sur le site des plugins pour améliorer le score du plugin.

L'adoption d'un plugin ou la contribution à chaque plugin est toujours la bienvenue.

