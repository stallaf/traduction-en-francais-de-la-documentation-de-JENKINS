# Linux

Les installateurs de Jenkins sont disponibles pour plusieurs distributions Linux. 

* [Debian/Ubuntu](#debian linux) ;
* [Fedora](#fedora) ;
* [Red Hat Enterprise Linux et dérivés](#red hat).

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

## Debian/Ubuntu

Sur Debian et les distributions basées sur Debian comme Ubuntu, vous pouvez installer Jenkins via `apt`.

_Comment installer Jenkins sur Ubuntu 24.04_

<iframe width="640" height="360" src="https://www.youtube.com/embed/8fVOdFdzlKc" title="How to Install Jenkins on Ubuntu 24.04" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Vous devez choisir la version de support à long terme de Jenkins ou la version hebdomadaire de Jenkins.

### Version de support à long terme

Une [version LTS (Long-Term Support)](https://www.jenkins.io/download/lts/) est choisie toutes les 12 semaines à partir du flux de versions régulières en tant que version stable pour cette période. Elle peut être installée à partir du [debian-stable apt repository](https://pkg.jenkins.io/debian-stable/).

``` bash title="BASH"
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

### Version hebdomadaire

Une nouvelle version est produite chaque semaine pour fournir des corrections de bogues et des fonctionnalités aux utilisateurs et aux développeurs de plugins. Il peut être installé à partir du [référentiel Debian apt](https://pkg.jenkins.io/debian/).

``` bash title="BASH"
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

En commençant par Jenkins 2.335 et Jenkins 2.332.1, le package est configuré avec `systemd` plutôt que dans l'ancien System V `init`. Plus d'informations sont disponibles dans ["Gestion des services SystemD"](administration-systeme-service-systemd).

L'installation du paquet permettra de :

* Configurer Jenkins en tant que démon lancé au début. Exécutez `systemctl cat jenkins ` pour plus de détails ; 
* Créer un utilisateur «Jenkins» pour exécuter ce service ; 
* Acheminer la sortie du journal de la console vers `systemd-journald`.
Exécuter `journalctl -u jenkins.service` si vous dépannez Jenkins ;
* Remplir le fichier `/lib/systemd/system/jenkins.service` avec les paramètres de configuration pour le lancement, par exemple `JENKINS_HOME` ;
* Définir Jenkins pour écouter sur le port 8080. Accéder à ce port avec un navigateur pour démarrer la configuration.

!!! info

    Si Jenkins ne démarre pas car un port est utilisé, exécutez `systemctl edit jenkins` et ajoutez ce qui suit :
    ``` console
    [Service]
    Environnement = "jenkins_port = 8081"
    ```
    Ici, "8081" a été choisi mais vous pouvez mettre un autre port disponible.

### Installation de Java

Jenkins exige que Java fonctionne, mais toutes les distributions Linux ne comprennent pas Java par défaut. De plus, [toutes les versions Java ne sont pas compatibles](./plateforme-java.md) avec Jenkins.

Il existe plusieurs implémentations Java que vous pouvez utiliser. [OpenJDK](https://openjdk.java.net/) est le plus populaire en ce moment, nous l'utiliserons dans ce guide.

Mettez à jour les référentiels Debian apt, installez OpenJDK 21 et vérifiez l'installation avec les commandes :

``` bash title="BASH"
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
openjdk version "21.0.3" 2024-04-16
OpenJDK Runtime Environment (build 21.0.3+11-Debian-2)
OpenJDK 64-Bit Server VM (build 21.0.3+11-Debian-2, mixed mode, sharing)
```

!!! info

    Pourquoi utiliser `apt` et non `apt-get` ou une autre commande ? La commande apt est disponible depuis 2014. Elle a une structure de commande similaire à `apt-get` mais a été créée pour être une expérience plus agréable pour les utilisateurs typiques. Les tâches de gestion des logiciels simples comme l'installation, la recherche et la suppression sont plus faciles avec `apt`.

## Fedora

Vous pouvez installer Jenkins via `dnf`. Vous devez d'abord ajouter le référentiel Jenkins du site Web Jenkins au gestionnaire de packages.

### Version de support à long terme

Une [version LTS (support à long terme)](https://www.jenkins.io/download/lts/) est choisie toutes les 12 semaines à partir du flux de versions régulières en tant que version stable pour cette période. Elle peut être installée à partir du référentiel yum ` redhat-stable `.

``` bash title="BASH"
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo dnf upgrade
# Add required dependencies for the jenkins package
sudo dnf install fontconfig java-21-openjdk
sudo dnf install jenkins
sudo systemctl daemon-reload
```

### Versions hebdomadaires

Une nouvelle version est produite chaque semaine pour fournir des corrections de bogues et des fonctionnalités aux utilisateurs et aux développeurs de plugins. Elle peut être installée à partir du référentiel yum [redhat](https://pkg.jenkins.io/redhat/.

``` bash title="BASH"
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io-2023.key
sudo dnf upgrade
# Add required dependencies for the jenkins package
sudo dnf install fontconfig java-21-openjdk
sudo dnf install jenkins
```

### Démarrer Jenkins

Vous pouvez régler le service Jenkins pour qu'il s'initialise au démarrage à l'aide de la commande :

``` bash title="BASH"
sudo systemctl enable jenkins
```

Vous pouvez démarrer le service Jenkins avec la commande :

``` bash title="BASH"
sudo systemctl start jenkins
```

Vous pouvez vérifier l'état du service Jenkins à l'aide de la commande :

``` bash title="BASH"
sudo systemctl status jenkins
```

Si tout a été configuré correctement, vous devriez voir une sortie comme celle-ci :

``` bash title="BASH"
Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
Active: active (running) since Tue 2018-11-13 16:19:01 +03; 4min 57s ago
```

!!! info

    Si vous avez un pare-feu installé, vous devez ajouter Jenkins comme exception. Vous devez modifier `YOURPORT` dans le script ci-dessous par le port que vous souhaitez utiliser. Le port `8080` est le plus courant.

    ``` bash title="BASH"
    Votre PORT = 8080
    YOURPORT=8080
    PERM="--permanent"
    SERV="$PERM --service=jenkins"

    firewall-cmd $PERM --new-service=jenkins
    firewall-cmd $SERV --set-short="Jenkins ports"
    firewall-cmd $SERV --set-description="Jenkins port exceptions"
    firewall-cmd $SERV --add-port=$YOURPORT/tcp
    firewall-cmd $PERM --add-service=jenkins
    firewall-cmd --zone=public --add-service=http --permanent
    firewall-cmd --reload
    ```

## Red Hat Enterprise Linux et dérivés

Vous pouvez installer Jenkins via `yum` sur Red Hat Enterprise Linux, Almalinux, Rocky Linux, Oracle Linux, Centos et d'autres distributions basées sur Red Hat.

_Comment installer Jenkins sur Rocky Linux 9_

<iframe width="640" height="360" src="https://www.youtube.com/embed/2-L0WohfsqY" title="How to Install Jenkins on Rocky Linux 9" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Vous devez choisir la version de support à long terme de Jenkins ou la version hebdomadaire de Jenkins.

### Version de support à long terme

Une [version LTS (support à long terme)](https://www.jenkins.io/download/lts/) est choisie toutes les 12 semaines à partir du flux de versions régulières en tant que version stable pour cette période. Elle peut être installée à partir du référentiel yum `redhat-stable`.

``` bash title="BASH"
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
# Add required dependencies for the jenkins package
sudo yum install fontconfig java-21-openjdk
sudo yum install jenkins
sudo systemctl daemon-reload
```

### Versions hebdomadaires

Une nouvelle version est produite chaque semaine pour fournir des corrections de bogues et des fonctionnalités aux utilisateurs et aux développeurs de plugins. Elle peut être installée à partir du référentiel yum [redhat](https://pkg.jenkins.io/redhat/).

``` bash title="BASH"
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io-2023.key
sudo yum upgrade
# Add required dependencies for the jenkins package
sudo yum install fontconfig java-21-openjdk
sudo yum install jenkins
```

### Démarrer Jenkins

Vous pouvez régler le service Jenkins pour qu'il s'initialise au démarrage à l'aide de la commande :

``` bash title="BASH"
sudo systemctl enable jenkins
```

Vous pouvez démarrer le service Jenkins avec la commande :

``` bash title="BASH"
sudo systemctl start jenkins
```

Vous pouvez vérifier l'état du service Jenkins à l'aide de la commande :

``` bash title="BASH"
sudo systemctl status jenkins
```

Si tout a été configuré correctement, vous devriez voir une sortie comme celle-ci :

``` bash title="BASH"
Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
Active: active (running) since Tue 2023-06-22 16:19:01 +03; 4min 57s ago
...
```

!!! info

    Si vous avez un pare-feu installé, vous devez ajouter Jenkins comme exception. Vous devez modifier `YOURPORT` dans le script ci-dessous par le port que vous souhaitez utiliser. Le port `8080` est le plus courant.

    ``` bash title="BASH"
    YOURPORT=8080
    PERM="--permanent"
    SERV="$PERM --service=jenkins"

    firewall-cmd $PERM --new-service=jenkins
    firewall-cmd $SERV --set-short="Jenkins ports"
    firewall-cmd $SERV --set-description="Jenkins port exceptions"
    firewall-cmd $SERV --add-port=$YOURPORT/tcp
    firewall-cmd $PERM --add-service=jenkins
    firewall-cmd --zone=public --add-service=http --permanent
    firewall-cmd --reload
    ```

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

