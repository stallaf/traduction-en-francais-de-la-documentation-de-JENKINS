# Autres Systèmes

## FreeBSD

Jenkins peut être installé sur FreeBSD à l'aide du gestionnaire de package FreeBSD standard, `pkg`.

### Version de support à long terme

Une [version LTS (support à long terme)](https://www.jenkins.io/download/lts/) est choisie toutes les 12 semaines à partir du flux de versions régulières en tant que version stable pour cette période. Elle peut être installée à partir du FreeBSD `pkg` Package Manager.

``` bash title="BASH"
# pkg install jenkins-lts
```

!!! warning "Avertissement"

    Le projet FreeBSD assure la maintenance du paquetage Jenkins pour FreeBSD. Le paquetage Jenkins pour FreeBSD n'est PAS officiellement pris en charge par le `projet Jenkins`, mais il est activement utilisé par le projet FreeBSD sur [ci.freebsd.org/](https://ci.freebsd.org/).

### Versions hebdomadaires

Une nouvelle version est produite chaque semaine pour fournir des corrections de bogues et des fonctionnalités aux utilisateurs et aux développeurs de plugins. Elle peut être installée à partir du FreeBSD `pkg` Package Manager.

``` bash title="BASH"
# pkg installer Jenkins
```

Le package d'assistance à long terme `jenkins-lts` et l'installation hebdomadaire du package `jenkins` seront : 

* Configurez Jenkins comme un démon qui peut éventuellement être lancé au début. Voir `/etc/rc.conf` pour plus de détails ;
* Créez un utilisateur «Jenkins» pour exécuter le service ; 
* Acheminez la sortie du journal de la console vers le fichier `/var/log/jenkins.log`. Vérifiez ce fichier lors du dépannage de Jenkins ;
* Définissez Jenkins pour écouter sur le port 8180 depuis le chemin `/jenkins`. Ouvrir http://localhost:8180/jenkins pour se connecter à Jenkins.

### Démarrer Jenkins

Vous pouvez démarrer le service Jenkins avec la commande :

``` bash title="BASH"
# service jenkins onestart
```

Vous pouvez vérifier l'état du service Jenkins à l'aide de la commande :

``` bash title="BASH"
# Service Jenkins Statut
```

Vous pouvez arrêter le service Jenkins avec la commande :

``` bash title="BASH"
# Service Jenkins Stop
```

### Activer Jenkins

Ajoutez ce qui suit à `/etc/rc.conf` pour démarrer automatiquement Jenkins au lancement du système :

``` bash title="BASH"
jenkins_enable="YES"
```

Une fois que Jenkins est activé, il peut être démarré avec :

``` bash title="BASH"
# service jenkins start
```

D'autres valeurs de configuration qui peuvent être définies dans `/etc/rc.conf` ou dans `/etc/rc.conf.d/jenkins` sont décrites dans `/usr/local/etc/rc.d/jenkins`. Reportez-vous à la [page Jenkins](https://wiki.freebsd.org/Jenkins) du [Wiki FreeBSD](https://wiki.freebsd.org/) pour plus d'informations spécifiques à Jenkins sur FreeBSD.

## OpenIndiana Hipster

Sur un système exécutant [OpenIndiana Hipster](https://www.openindiana.org/), Jenkins peut être installé dans la zone locale ou globale à l'aide du [système d'emballage d'image](https://en.wikipedia.org/wiki/Image_Packaging_System) (IPS).

!!! warning "Avertissement"

    Cette plateforme n'est PAS officiellement prise en charge par l'équipe Jenkins. Son utilisation se fait à vos propres risques. Le packaging et l'intégration décrits dans cette section sont gérés par l'équipe OpenIndiana Hipster, qui regroupe le fichier générique `jenkins.war` afin qu'il fonctionne dans cet environnement d'exploitation.

Dans le cas courant où vous exécutez la dernière version hebdomadaire packagée en tant que serveur autonome (Jetty), il suffit d'exécuter : 

``` bash title="BASH"
pkg install jenkins
svcadm enable jenkins
```

L'intégration de packaging commun pour un service autonome permettra de :

* Créez un utilisateur `jenkins` pour exécuter le service et pour posséder les structures de répertoire sous `/var/lib/ jenkins` ; 
* Tirez le package Java et les autres packages requis pour exécuter Jenkins, y compris le package `jenkins-core-weekly` avec le dernier `jenkins.war` ; 
* Configurez Jenkins en tant qu'instance de service SMF (`svc:/network/http:jenkins`) qui peut ensuite être activé avec la commande `svcadm` illustrée ci-dessus ;
* Configurez Jenkins pour écouter sur le port 8080 ;
* Configurez la sortie du journal à gérer par SMF à `/var/svc/log/network-http:jenkins.log`.

Une fois Jenkins en cours d'exécution, consultez le journal (`/var/svc/log/network-http:jenkins.log`) pour récupérer le mot de passe administrateur généré pour la configuration initiale de Jenkins, il sera généralement trouvé sur `/var/lib/jenkins/home/secrets/initialAdminPassword.`. Accédez ensuite à [localHost:8080](http://localhost:8080/) pour [terminer la configuration du contrôleur Jenkins](#assistant de configuration post-installation).

Pour modifier les attributs du service, tels que les variables d'environnement comme `JENKINS_HOME` ou le numéro de port utilisé pour le serveur Web Jetty, utilisez l'utilitaire `svccfg` :

``` bash title="BASH"
svccfg -s svc:/network/http:jenkins editprop
svcadm refresh svc:/network/http:jenkins
```

Vous pouvez également consulter le fichier `/lib/svc/manifest/network/jenkins-standalone.xml` pour plus de détails et de commentaires sur les paramètres actuellement pris en charge par le service SMF. Notez que le compte utilisateur `jenkins` créé par le packaging dispose de privilèges spéciaux lui permettant de se connecter à des numéros de port inférieurs à 1024.

L'état actuel des packages liés à Jenkins disponibles pour la version donnée d'OpenIndiana peut être interrogé avec :

``` bash title="BASH"
pkg info -r '*jenkins*'
```

Les mises à niveau du package peuvent être effectuées en mettant à jour l'intégralité de l'environnement d'exploitation avec `pkg update`, ou spécifiquement pour le logiciel Jenkins Core avec :

``` bash title="BASH"
pkg update jenkins-core-weekly
```

!!! danger "Attention"

    La procédure de mise à jour du package redémarrera le processus Jenkins en cours d'exécution. Assurez-vous de le préparer à la fermeture et terminez tous les travaux d'exécution avant de mettre à jour, si nécessaire.

## Solaris, Omnios, Smartos et autres systèmes apparentés

Généralement, il devrait suffire d'installer une [version Java prise en charge](./plateforme-java.md), de [télécharger](https://www.jenkins.io/download) le `jenkins.war` et de l'exécuter en tant que processus autonome.

Certaines mises en garde s'appliquent : 

* JVM sans affichage et polices : pour les versions OpenJDK sur des systèmes à empreinte minimale, il peut y avoir des [problèmes lors de l'exécution de la JVM sans affichage](https://wiki.jenkins.io/display/JENKINS/Jenkins+got+java.awt.headless+problem), car Jenkins a besoin de certaines polices pour afficher certaines pages.

## Assistant de configuration post-installation

Après avoir téléchargé, installé et exécuté Jenkins en utilisant l'une des procédures ci-dessus (à l'exception de l'installation avec l'opérateur Jenkins), l'assistant de configuration post-installation commence.

Cet assistant de configuration vous guide à travers quelques étapes "uniques" rapides pour déverrouiller Jenkins, la personnaliser avec des plugins et créer le premier utilisateur administrateur à travers lequel vous pouvez continuer à accéder à Jenkins.

### Déverrouiller Jenkins

Lorsque vous accédez à un nouveau contrôleur Jenkins pour la première fois, il vous est demandé de le déverrouiller à l'aide d'un mot de passe généré automatiquement. 

1. Parcourez `http://localhost:8080` (ou le port que vous avez configuré pour Jenkins lors de l'installation) et attendez que la page de **déverrouillage de Jenkins** apparaisse. 
![Déverrouiller la page Jenkins](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-01-unlock-jenkins-page.jpg)
2. À partir de la sortie du journal de la console Jenkins, copiez le mot de passe alphanumérique généré automatiquement (entre les 2 ensembles d'astérisques). 
![Copie de mot de passe d'administration initial](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-02-copying-initial-admin-password.png) 

    **Note :**

       * La commande : `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` imprimera le mot de passe à la console ; 
       * Si vous exécutez Jenkins dans Docker à l'aide de l'image officielle `jenkins/jenkins`, vous pouvez utiliser `sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword` pour imprimer le mot de passe dans la console sans avoir à exécuter dans le conteneur ; 

3. Sur la page **Déverrouiller Jenkins**, collez ce mot de passe dans le champ de **mot de passe de l'administrateur** et cliquez sur **Continuer**. 

    **Note :**

       * Le journal de la console Jenkins indique l'emplacement (dans le répertoire domestique Jenkins) où ce mot de passe peut également être obtenu. Ce mot de passe doit être entré dans l'assistant d'installation sur les nouvelles installations de Jenkins avant de pouvoir accéder à l'interface utilisateur principale de Jenkins. Ce mot de passe sert également de mot de passe du compte d'administrateur par défaut (avec le nom d'utilisateur "admin") s'il vous arrive de sauter l'étape de création utilisateur ultérieure dans l'assistant de configuration.

### Personnalisation de jenkins avec des plugins

Après avoir [déverrouillé Jenkins](#déverrouiller-jenkins), la page **_Customize Jenkins_** apparaît. Ici, vous pouvez installer n'importe quel nombre de plugins utiles dans le cadre de votre configuration initiale.

Cliquez sur l'une des deux options indiquées : 

* **Installez les plugins suggérés** - pour installer l'ensemble recommandé de plugins qui sont basés sur les cas d'utilisation les plus courants. 
* **Sélectionnez des plugins à installer** - pour choisir le jeu de plugins à installer initialement. Lorsque vous accédez d'abord à la page de sélection des plugins, les plugins suggérés sont sélectionnés par défaut. 


    !!! info

        Si vous ne savez pas de quel plugins vous avez besoin, choisissez **Installer les plugins suggérés**. Vous pouvez installer (ou supprimer) des plugins Jenkins supplémentaires à un moment ultérieur via la page [Manage Jenkins](./gestion-de-jenkins.md)> [Plugins](./gestion-de-jenkins.md#plugins) dans Jenkins.

L'assistant de configuration montre la progression de Jenkins en cours de configuration et votre ensemble choisi de plugins Jenkins en cours d'installation. Ce processus peut prendre quelques minutes.

### Création du premier utilisateur administrateur

Enfin, après avoir [personnalisé Jenkins avec des plugins](#personnalisation-de-jenkins-avec-des-plugins), Jenkins vous demande de créer votre premier utilisateur administrateur. 

1. Lorsque la page **Créer le Premier Utilisateur Administrateur** s'affiche, indiquez les informations relatives à votre utilisateur administrateur dans les champs correspondants, puis cliquez sur **Enregistrer et Terminer**.
2. Lorsque la page **Jenkins est prête** apparaît, cliquez sur **Démarrer à l'aide de Jenkins**. 

    **Notes :** 

    * Cette page peut indiquer que **Jenkins est presque prêt !** Au lieu de cela et si oui, cliquez sur **Redémarrer**. 
    * Si la page ne se rafraîchit pas automatiquement après une minute, utilisez votre navigateur Web pour actualiser la page manuellement. 

3. Si nécessaire, connectez-vous à Jenkins avec les informations d'identification de l'utilisateur que vous venez de créer et vous êtes prêt à commencer à utiliser Jenkins !
