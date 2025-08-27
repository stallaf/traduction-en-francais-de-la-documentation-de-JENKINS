# Installations hors ligne

Cette section décrit comment installer Jenkins sur une machine qui n'a pas de connexion Internet.

Pour installer Jenkins lui-même, téléchargez le fichier de guerre approprié et transférez-le dans votre machine.

## Condition préalable

Exigences matérielles minimales : 

* 256 Mo de RAM  ;
* 1 Go d'espace disque  (bien que 10 Go soit le minimum recommandé si Jenkins est exécuté en tant que conteneur Docker).

Configuration matérielle recommandée pour une petite équipe : 

* 4 Go + de RAM ;
* 50 Go + d'espace disque.

Recommandations matérielles complètes : 

Matériel : voir la page des [recommandations matérielles](./mise-a-echelle-recommandations-materielles.md)

Exigences logicielles : 

* Java: Voir la page des [exigences Java](./plateforme-java.md) ; 
* Navigateur Web: Voir la page de [compatibilité du navigateur Web](./plateforme-navigateurs.md) ;
* Pour le système d'exploitation Windows : [politique de support Windows](./plateforme-windows.md) ; 
* Pour le système d'exploitation Linux: [Politique de support Linux](./plateforme-linux.md) ; 
Pour les conteneurs de servlet: [Politique de support des conteneurs servlette](./plateforme-servlet.md)

### Installation du plugin hors ligne

Les installations de plugins hors ligne nécessitent des efforts supplémentaires en raison des exigences de dépendance.

[L'outil Plugin Installation Manager](https://github.com/jenkinsci/plugin-installation-manager-tool) est l'outil recommandé pour l'installation hors ligne des plugins. Il télécharge les plugins spécifiés par l'utilisateur et toutes les dépendances de ces plugins.[] L'outil Plugin Installation Manager](https://github.com/jenkinsci/plugin-installation-manager-tool) signale également les mises à jour disponibles pour les plugins et les avertissements de sécurité les concernant. Il est inclus dans les [images Docker officielles de Jenkins](https://github.com/jenkinsci/docker/blob/master/README.md#preinstalling-plugins). Il est également utilisé pour installer des plugins dans le cadre des [instructions d'installation de Docker](https://www.jenkins.io/doc/book/installing/docker/). Pour toute question ou réponse, veuillez vous référer au [canal de discussion Gitter](https://app.gitter.im/#/room/#jenkinsci_plugin-installation-manager-cli-tool:gitter.im).

Si vous souhaitez transférer les plugins individuels, vous devrez également récupérer toutes les dépendances. Il existe plusieurs scripts et outils de récupération de dépendance sur GitHub. Par exemple : 

* [Outil de gestion du plugin](https://github.com/jenkinsci/plugin-installation-manager-tool/blob/master/README.md) - Utilitaire de ligne de commande Java pour installer les plugins Jenkins et leurs dépendances ;
* [samrocketman/jenkins-bootstrap-shared](https://github.com/samrocketman/jenkins-bootstrap-shared) - Java est requis ; regroupe Jenkins et les plugins dans un programme d'installation de paquets immuable. Formats pris en charge : RPM, DEB, Docker. Peut utiliser Jenkins et des plugins via Nexus ou Artifactory puisque Gradle est utilisé pour assembler les plugins.

## Assistant de configuration post-installation

Après avoir téléchargé, installé et exécuté Jenkins en utilisant l'une des procédures ci-dessus (à l'exception de l'installation avec l'opérateur Jenkins), l'assistant de configuration post-installation commence.

Cet assistant de configuration vous guide à travers quelques étapes "uniques" rapides pour déverrouiller Jenkins, la personnaliser avec des plugins et créer le premier utilisateur administrateur à travers lequel vous pouvez continuer à accéder à Jenkins.

### Déverrouiller Jenkins

Lorsque vous accédez à un nouveau contrôleur Jenkins pour la première fois, il vous est demandé de le déverrouiller à l'aide d'un mot de passe généré automatiquement. 

1. Parcourez `http: // localhost: 8080` (ou le port que vous avez configuré pour Jenkins lors de l'installation) et attendez que la page de déverrouillage de Jenkins apparaisse. 
![Déverrouiller la page Jenkins](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-01-unlock-jenkins-page.jpg)
2. À partir de la sortie du journal de la console Jenkins, copiez le mot de passe alphanumérique généré automatiquement (entre les 2 ensembles d'astérisques). 
![Copie de mot de passe d'administration initial](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-02-copying-initial-admin-password.png) 

    **Note :**

      * La commande: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` imprimera le mot de passe à la console ;
      * Si vous exécutez Jenkins dans Docker à l'aide de l'image officielle `jenkins/jenkins`, vous pouvez utiliser `sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword ` pour imprimer le mot de passe dans la console sans avoir à exécuter dans le conteneur. 

3. Sur la page **Déverrouiller Jenkins**, collez ce mot de passe dans le champ de **mot de passe de l'administrateur** et cliquez sur **Continuer**. 

    **Note :** 

      * Le journal de la console Jenkins indique l'emplacement (dans le répertoire domestique Jenkins) où ce mot de passe peut également être obtenu. Ce mot de passe doit être entré dans l'assistant d'installation sur les nouvelles installations de Jenkins avant de pouvoir accéder à l'interface utilisateur principale de Jenkins. Ce mot de passe sert également de mot de passe du compte administrateur par défaut (avec le nom d'utilisateur "admin") s'il vous arrive de sauter l'étape de création utilisateur ultérieure dans l'assistant de configuration.

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
