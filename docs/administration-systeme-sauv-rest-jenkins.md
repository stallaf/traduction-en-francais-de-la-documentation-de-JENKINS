# Sauvegarde/Restauration de Jenkins

<div class="couleur-introduction">
Il est essentiel de disposer de bonnes sauvegardes de votre contrôleur Jenkins. Les sauvegardes sont utilisées pour :

<ul>
<li>La reprise après sinistre ;</li>
<li>La restauration d'une ancienne configuration (une modification accidentelle de la configuration peut ne pas être découverte avant un certain temps) ;</li>
<li>La restauration d'un fichier corrompu ou supprimé accidentellement.</li>
</ul>
</div>

Cette page aborde les points suivants :

* Comment créer une sauvegarde ;
* Fichiers à sauvegarder ;
* Comment valider une sauvegarde pour s'assurer qu'elle est utilisable.

## Création d'une sauvegarde

Différents schémas peuvent être utilisés pour créer des sauvegardes. Ceux-ci sont abordés dans cette section :

* Instantanés du système de fichiers ;
* Plugins pour la sauvegarde ;
* Écrire un script shell qui sauvegarde le contrôleur Jenkins.

### Instantanés du système de fichiers

Les instantanés du système de fichiers offrent une cohérence maximale pour les sauvegardes. Ils s'exécutent également plus rapidement que les sauvegardes en direct, ce qui réduit le risque de copier des données différentes à des moments différents. Ils sont pris en charge par :

* Le gestionnaire de volumes logiques Linux (LVM) ;
* Linux btrfs ;
* Solaris ZFS (qui prend également en charge les sauvegardes incrémentielles) ;
* FreeBSD ZFS ;
* OpenZFS sur Linux ;
* Certaines autres architectures de système de fichiers ;
* De nombreux fournisseurs de cloud ;
* Certains périphériques de stockage séparés vous permettent également de créer des instantanés au niveau du stockage.

### Plugins pour la sauvegarde

Plusieurs plugins sont disponibles pour la sauvegarde. Dans le menu principal, sélectionnez _Manage Jenkins_, puis allez dans _Plugins>Available_ et recherchez **backup**. Notez que seul le [plugin thinBackup](https://plugins.jenkins.io/thinBackup/) parmi les plugins open source est actuellement maintenu. Vous pouvez essayer les autres plugins, mais vous risquez de rencontrer des problèmes.

### Écrire un script shell pour les sauvegardes

Vous pouvez écrire votre propre script shell qui copie les fichiers et répertoires appropriés vers un emplacement de sauvegarde. Utilisez [cron](https://man7.org/linux/man-pages/man8/cron.8.html) pour planifier l'exécution du script de sauvegarde.

Le script shell doit créer un répertoire tel que `/mnt/backup` dans lequel la sauvegarde sera écrite ; assurez-vous que vous disposez des droits d'écriture sur ce répertoire. Envisagez de créer `/mnt/backup` en tant que système de fichiers distinct avec son propre point de montage. Une autre méthode consiste à créer un sous-répertoire dans `/var`. Notez que si vous utilisez cette méthode, vous devrez peut-être utiliser la commande **sudo** pour exécuter l'opération de restauration. Il n'est pas recommandé d'effectuer la sauvegarde dans `/tmp`, car `/tmp` peut être nettoyé au redémarrage.

Créez un identifiant unique pour chaque sauvegarde (utilisez un horodatage, par exemple) afin de vous assurer que la sauvegarde d'aujourd'hui n'écrase pas celle d'hier.

L'écriture de fichiers sur un système de fichiers local est le moyen le plus rapide d'effectuer la sauvegarde. Envisagez de copier la sauvegarde terminée sur un serveur ou un périphérique de sauvegarde distant pour un stockage à long terme.

## Sauvegardez la clé du contrôleur séparément

**N'incluez jamais la clé du contrôleur dans votre sauvegarde Jenkins !**

La clé du contrôleur est utilisée pour chiffrer les données dans le répertoire secrets qui sécurise les informations d'identification. Elle est stockée dans le fichier _$JENKINS_HOME/secrets/hudson.util.Secret_ dans le répertoire _$JENKINS_HOME/secrets/_ et chiffrée avec `master.key`. Si vous devez restaurer un système à partir d'une sauvegarde, vous aurez besoin de ce fichier. Et si quelqu'un d'autre accède à vos sauvegardes et dispose de cette clé, il aura un accès complet à toutes vos informations.

Vous devez traiter votre clé de contrôleur comme vous traitez votre clé privée SSH et ne JAMAIS l'inclure dans une sauvegarde régulière. Sauvegardez plutôt le fichier `master.key` séparément et stockez-le dans un endroit très sécurisé, loin de vos autres sauvegardes. Il s'agit d'un très petit fichier qui est rarement modifié. Si vous devez effectuer une restauration complète du système, vous devrez restaurer le reste du système, puis appliquer la sauvegarde du fichier `master.key` séparément.

## Quels fichiers doivent être sauvegardés ?

Le nombre de fichiers que vous sauvegardez peut avoir une incidence sur le temps nécessaire à l'exécution de la sauvegarde et sur la taille de la sauvegarde obtenue. Il a également une incidence sur la complexité de la restauration du système à partir de la sauvegarde. Nous expliquons ici pourquoi divers fichiers doivent être sauvegardés et énumérons certains fichiers qui peuvent être exclus en toute sécurité d'au moins certaines sauvegardes.

### $JENKINS_HOME

La sauvegarde de l'intégralité du répertoire `$JENKINS_HOME` permet de conserver l'intégralité du contrôleur Jenkins. Pour restaurer le système, il suffit de copier l'intégralité de la sauvegarde vers le nouveau système.

Notez toutefois que `JENKINS_HOME` comprend un certain nombre de fichiers qui n'ont pas vraiment besoin d'être sauvegardés. La sélection de répertoires et de fichiers spécifiques à sauvegarder permet d'obtenir des sauvegardes plus petites, mais peut nécessiter un effort plus important pour restaurer un système. Une approche consiste à sauvegarder différents répertoires selon des calendriers différents.

### Fichiers de configuration

Les fichiers de configuration sont stockés directement dans le répertoire `$JENKINS_HOME`. `./config.xml` est le fichier de configuration principal de Jenkins. Les autres fichiers de configuration ont également l'extension` .xm`l. Spécifiez `$JENKINS_HOME/*.xml` pour sauvegarder tous les fichiers de configuration.

Les fichiers de configuration peuvent également être stockés dans un référentiel SCM. Cela permet de conserver des copies de toutes les versions précédentes de chaque fichier, qui peuvent être récupérées à l'aide des fonctionnalités SCM standard.

### Sous-répertoire ./jobs

Le répertoire `$JENKINS_HOME/jobs` contient des informations relatives à tous les travaux que vous créez dans Jenkins.

* **./builds** — Contient les enregistrements de compilation ;
* **./builds/archive** — Contient les artefacts archivés ;
    * Sauvegardez-les s'il est important de conserver ces artefacts à long terme ;
    * Ils peuvent être très volumineux et alourdir considérablement vos sauvegardes.
* **./workspace** — Contient les fichiers extraits du SCM ;
    * Il n'est généralement pas nécessaire de sauvegarder ces fichiers. Vous pouvez effectuer une extraction propre après avoir restauré le système.
* **./plugins/*.hpi** — Paquets de plug-ins avec des versions spécifiques utilisées sur votre système ;
* **./plugins/*.jpi** — Paquets de plug-ins avec des versions spécifiques utilisées sur votre système.

### Ce qui n'a pas besoin d'être sauvegardé

Les fichiers et répertoires suivants n'ont généralement pas besoin d'être inclus dans chaque sauvegarde de routine, car vous pouvez télécharger la dernière version lorsque vous restaurez un système. Cependant, certains experts en reprise après sinistre déconseillent d'effectuer des mises à niveau pendant la restauration du système, afin d'éviter les retards causés par des problèmes de compatibilité qui pourraient survenir. Si votre plan de reprise après sinistre spécifie que vous devez restaurer le système à l'aide des mêmes versions logicielles que celles qui étaient précédemment utilisées, vous pouvez effectuer une sauvegarde occasionnelle du système et de tous les outils téléchargés, puis l'utiliser pour restaurer le système.

* **./war** — Fichier war décompressé ;
    * Pour restaurer un système, téléchargez le dernier fichier war.
* **./cache** — Outils téléchargés ;
    * Pour restaurer un système, téléchargez la version actuelle des outils.
* **./tools** — Outils extraits ;
    * Pour restaurer un système, extrayez à nouveau les outils.
* **./plugins/xxx** — Sous-répertoires des plugins installés ;
    * Ceux-ci seront automatiquement remplis au prochain redémarrage.

## Validation d'une sauvegarde

Votre stratégie de sauvegarde doit inclure la validation de chaque sauvegarde. Vous ne voulez pas découvrir que votre sauvegarde n'est pas bonne lorsque vous en avez besoin !

Une façon simple de valider une sauvegarde complète consiste à la restaurer dans un emplacement temporaire. Créez un répertoire pour la validation du test (tel que **/mnt/backup-test**) et restaurez la sauvegarde dans ce répertoire.

Définissez $JENKINS_HOME pour pointer vers ce répertoire, en spécifiant un port HTTP aléatoire afin d'éviter toute collision avec le contrôleur Jenkins réel :

``` bash title="BASH"
export JENKINS_HOME=/mnt/backup-test
```

Exécutez maintenant le contrôleur Jenkins restauré :

``` bash title="BASH"
java -jar jenkins.war --httpPort=9999
```

## Résumé

* La création de sauvegardes est une bonne pratique Jenkins ;
* Les sauvegardes sont essentielles pour la reprise après sinistre ;
* Définissez toujours une politique de sauvegarde qui précise :
    * Les configurations et les enregistrements qui doivent être sauvegardés à partir du contrôleur;
    * La fréquence à laquelle les sauvegardes doivent être effectuées ;
    * L'emplacement où les sauvegardes doivent être stockées.
* Vérifiez vos sauvegardes ;
    * Vous devez vérifier régulièrement que vos sauvegardes sont intactes et peuvent être utilisées pour atteindre vos objectifs de reprise.

## Pour aller plus loin

Quelques lectures recommandées sur ce sujet :

* [Pourquoi des techniques de sauvegarde et de restauration intelligentes et efficaces sont essentielles avec le serveur de production Jenkins](https://www.cloudbees.com/blog/why-smart-efficient-backup-and-restore-techniques-are-essential-jenkins-production-server) ;
* [Plugin de sauvegarde](https://plugins.jenkins.io/backup/) ;
* [Plugin thinBackup](https://plugins.jenkins.io/thinBackup/).








