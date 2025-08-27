# Fichier WAR

Le fichier ARchive (WAR) de l'application Web Jenkins regroupe [Winstone](https://github.com/jenkinsci/winstone), un conteneur de servlets [Jetty](https://www.eclipse.org/jetty/), et peut être lancé sur n'importe quel système d'exploitation ou plateforme disposant d'une version de Java prise en charge par Jenkins. Pour plus d'informations, consultez la page Configuration requise pour Java.

## Condition préalable

Exigences matérielles minimales : 

* 256 Mo de RAM ;
* 1 Go d'espace disque (bien que 10 Go soit un minimum recommandé si l'exécution de Jenkins comme un conteneur Docker).

Configuration matérielle recommandée pour une petite équipe : 

* 4 Go + de RAM ;
* 50 Go + d'espace disque.

Recommandations matérielles complètes : 

Matériel : voir la page des [recommandations matérielles](./mise-a-echelle-recommandations-materielles.md).

Exigences logicielles : 

* Java: Voir la page des [exigences Java](./plateforme-java.md) ; 
* Navigateur Web: Voir la page de [compatibilité du navigateur Web](./plateforme-navigateurs.md) ;
* Pour le système d'exploitation Windows : [politique de support Windows](./plateforme-windows.md) ; 
* Pour le système d'exploitation Linux: [Politique de support Linux](./plateforme-linux.md) ; 
Pour les conteneurs de servlet: [Politique de support des conteneurs servlette](./plateforme-servlet.md)

## Exécutez le fichier WAR

Le fichier d'archive d'application Web Jenkins (WAR) peut être démarré à partir de la ligne de commande comme ceci:  

1. Téléchargez le [dernier fichier WAR Jenkins](https://www.jenkins.io/download) dans un répertoire approprié sur votre machine ; 
2. Ouvrez une fenêtre de terminal/invite de commande dans le répertoire de téléchargement ;
3. Exécutez la commande `java -jar jenkins.war` ;
4. Accédez à `http://localhost:8080` et attendez que la page _**Unlock Jenkins**_ (Déverrouiller Jenkins) s'affiche ;
5. Continuez avec [l'assistant de configuration post-installation](#Assistant de configuration post-installation) ci-dessous.

**Notes :**

* Ce processus n'installe pas automatiquement de plugins spécifiques. Ils doivent être installés séparément via la page [Manage Jenkins](./gestion-jenkins.md)> [Plugins](./gestion-plugins.md) dans Jenkins ;
* Vous pouvez modifier le port en spécifiant l'option `--httpport` lorsque vous exécutez la commande `java -jar jenkins.war`. Par exemple, pour rendre Jenkins accessible via le port 9090, puis exécutez Jenkins à l'aide de la commande : `java -jar jenkins.war --httpPort=9090` ;
* Vous pouvez modifier le répertoire où Jenkins stocke sa configuration avec la variable d'environnement `JENKINS_HOME`. Par exemple, pour placer les fichiers de configuration Jenkins dans un sous-répertoire nommé ` my-jenkins-config`, définissez ` JENKINS_HOME=my-jenkins-config `avant d'exécuter la commande `java -jar jenkins.war`. Utilisez les commandes Windows : 

``` bat title="Windows"
C:\Temp > set JENKINS_HOME=my-jenkins-config
C:\Temp > java -jar jenkins.war
```

ou la commande Unix : 
``` unix title="Unix"
JENKINS_HOME=my-jenkins-config java -jar jenkins.war
```

Pour plus de détails sur les arguments de ligne de commande qui peuvent ajuster le démarrage de Jenkins, utilisez la commande :
`java -jar jenkins.war --help`

## Assistant de configuration post-installation

Après avoir téléchargé, installé et exécuté Jenkins en utilisant l'une des procédures ci-dessus (à l'exception de l'installation avec l'opérateur Jenkins), l'assistant de configuration post-installation commence.

Cet assistant de configuration vous guide à travers quelques étapes "uniques" rapides pour déverrouiller Jenkins, la personnaliser avec des plugins et créer le premier utilisateur d'administrateur à travers lequel vous pouvez continuer à accéder à Jenkins.

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
