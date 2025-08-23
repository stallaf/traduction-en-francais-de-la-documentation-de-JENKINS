# Docker

[Docker](https://docs.docker.com/get-started/overview/) est une plate-forme pour exécuter des applications dans un environnement isolé appelé "conteneur" (ou conteneur Docker). Des applications comme Jenkins peuvent être téléchargées sous forme "d'images" en lecture seule (ou images Docker), chacune exécutée dans Docker en tant que conteneur. Un conteneur Docker est une "instance en cours d'exécution" d'une image Docker. Une image Docker est stockée en permanence, en fonction du moment où les mises à jour de l'image sont publiées, tandis que les conteneurs sont stockés temporairement. En savoir plus sur ces concepts [pour démarrer, partie 1: orientation et configuration](https://docs.docker.com/get-started/) dans la documentation Docker.

En raison de la plate-forme fondamentale de Docker et de la conception de conteneurs, une image Docker pour une application donnée, telle que Jenkins, peut être exécutée sur n'importe quel système d'exploitation pris en charge ou service cloud également exécutant Docker. Les systèmes d'exploitation pris en charge incluent macOS, Linux et Windows, et les services cloud pris en charge incluent AWS et Azure.

### Installation de Docker

Pour installer Docker sur votre système d'exploitation, suivez les instructions dans les [prérequis de la visite guidée](./tutoriel-visite-guidee.md).

Alternativement, visitez [Docker Hub](https://hub.docker.com/search?type=edition&offering=community) et sélectionnez **l'édition Docker Community** adaptée à votre système d'exploitation ou service cloud. Suivez les instructions d'installation sur leur site Web.

!!! warning

    Si vous installez Docker sur un système d'exploitation basé sur Linux, assurez-vous de configurer Docker afin qu'il puisse être géré en tant qu'utilisateur non root. En savoir plus à ce sujet dans les étapes [post-installation de Docker pour la page Linux](https://docs.docker.com/engine/installation/linux/linux-postinstall/) de leur documentation. Cette page contient également des informations sur la façon de configurer Docker pour démarrer sur le démarrage.

## Condition préalable

Exigences matérielles minimales : 

* 256 Mo de RAM ;
* 1 Go d'espace disque (bien que 10 Go soit le minimum recommandé si vous exécutez Jenkins en tant que conteneur Docker).

Configuration matérielle recommandée pour une petite équipe : 

* 4 Go + de RAM ;
* 50 Go + d'espace disque.

Recommandations matérielles complètes : 

Matériel : voir la page des [recommandations matérielles](./dimensionnement-recommandations-materielles.md).

Exigences logicielles : 

* Java: Voir la page des [exigences Java](./plateforme-java.md) ;
* Navigateur Web : Voir la page de [compatibilité du navigateur Web](./plateforme-navigateurs.md) ; 
* Pour le système d'exploitation Windows : [politique de support Windows](./plateforme-windows.md) ;
* Pour le système d'exploitation Linux : [Politique de support Linux](./plateforme-linux.md) ; 
* Pour les conteneurs de servlet : [Politique de support des conteneurs servlet](./plateforme-servlet.md).

### Télécharger et exécuter Jenkins dans Docker

Il existe plusieurs images Docker de Jenkins disponibles.

Utilisez l'image officielle[ jenkins/jenkins image](https://hub.docker.com/r/jenkins/jenkins/) du [référentiel Docker Hub](https://hub.docker.com/). Cette image contient [la version actuelle de support à long terme (LTS) de Jenkins](https://www.jenkins.io/download), qui est prête pour la production. Cependant, cette image ne contient pas Docker CLI et n'est pas regroupée avec les plugins Blue Ocean fréquemment utilisés et ses fonctionnalités. Pour utiliser la pleine puissance de Jenkins et Docker, vous voudrez peut-être passer par le processus d'installation décrit ci-dessous.

!!! info 

    Une nouvelle image `jenkins/jenkins` est publiée à chaque nouvelle version de Jenkins Docker. Vous pouvez consulter la liste des versions précédemment publiées de l'image `jenkins/jenkins` sur la page des [balises](https://hub.docker.com/r/jenkins/jenkins/tags/).

## Sur macOS et Linux

1 . Ouvrez une fenêtre de terminal ;

2 . Créer un [réseau de ponts](https://docs.docker.com/network/bridge/) dans Docker à l'aide de la commande [docker network create](https://docs.docker.com/engine/reference/commandline/network_create/) :

```bash title="BASH"
docker network create jenkins
```

3 . Afin d'exécuter des commandes Docker à l'intérieur des nœuds Jenkins, téléchargez et exécutez l'image Docker `docker:dind ` à l'aide de la commande [docker d'exécution](https://docs.docker.com/engine/reference/run/) suivante :

``` bash title="BASH"
docker run \ 
  --name jenkins-docker \ 1
  --rm \ 2
  --detach \ 3
  --privileged \ 4
  --network jenkins \ 5
  --network-alias docker \ 6
  --env DOCKER_TLS_CERTDIR=/certs \ 7
  --volume jenkins-docker-certs:/certs/client \ 8
  --volume jenkins-data:/var/jenkins_home \ 9
  --publish 2376:2376 \ 10
  docker:dind \ 11
  --storage-driver overlay2 12
```

<i style ="color:red;">1</i> (_Facultatif_) Spécifie le nom du conteneur Docker à utiliser pour exécuter l'image. Par défaut, Docker génère un nom unique pour le conteneur. 

<i style ="color:red;">2</i> (_Facultatif_) Supprime automatiquement le conteneur Docker (la réplique de l'image Docker) lorsqu'il est arrêté. 

<i style ="color:red;">3</i> (_Facultatif_) Exécute le conteneur Docker en arrière-plan. Vous pouvez arrêter ce processus en exécutant `docker stop jenkins-docker`.

<i style ="color:red;">4</i> L'exécution de Docker dans Docker nécessite actuellement un accès privilégié à la fonction correctement. Cette exigence peut être détendue avec des versions du noyau Linux plus récentes. 

<i style ="color:red;">5</i> Cela correspond au réseau créé à l'étape précédente. 

<i style ="color:red;">6</i> Rend le Docker dans Docker Container disponible en tant que nom d'hôte `docker` dans le réseau `jenkins`. 

<i style ="color:red;">7</i> Permet l'utilisation de TLS dans le serveur Docker. En raison de l'utilisation d'un conteneur privilégié, cela est recommandé, bien qu'il nécessite l'utilisation du volume partagé décrit ci-dessous. Cette variable d'environnement contrôle le répertoire racine où les certificats Docker TLS sont gérés. 

<i style ="color:red;">8</i> Mappe le répertoire `/certs/client` à l'intérieur du conteneur vers un volume Docker nommé `jenkins-docker-certs`, tel que créé ci-dessus.

<i style ="color:red;">9</i> Mappe le répertoire `/var/jenkins_home` à l'intérieur du conteneur au volume docker nommé `jenkins-data`. Cela permet à d'autres conteneurs Docker contrôlés par le démon Docker de ce conteneur de monter des données à partir de Jenkins.

<i style ="color:red;">10</i> (_Facultatif_) expose le port de démon Docker sur la machine hôte. Ceci est utile pour exécuter des commandes `docker` sur la machine hôte afin de  contrôler ce démon intérieur. 

<i style ="color:red;">11</i> L'image `docker: dind` elle-même. Téléchargez cette image avant de l'exécuter, à l'aide de la commande : `docker image pull docker:dind`.

<i style ="color:red;">12</i> Le pilote de stockage du volume Docker. Reportez-vous à la documentation [Docker Storage Drivers](https://docs.docker.com/storage/storagedriver/select-storage-driver) pour les options prises en charge. 

!!! Info

    Si vous avez des problèmes de copie et de collage de l'extrait de commande ci-dessus, utilisez la version sans annotation ci-dessous :

```bash title="BASH"
docker run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind --storage-driver overlay2
```

4 . Personnalisez l'image officielle de Jenkins Docker, en exécutant les deux étapes suivantes : 

a . Créez un dockerfile avec le contenu suivant : 

``` console
FROM jenkins/jenkins:2.516.2-jdk21
USER root
RUN apt-get update && apt-get install -y lsb-release ca-certificates curl && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
    https://download.docker.com/linux/debian $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" \
    | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt-get update && apt-get install -y docker-ce-cli && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow json-path-api"
```

b . Créez une nouvelle image Docker à partir de ce dockerfile et attribuez à l'image un nom significatif, tel que "myjenkins-blueocean:2.516.2-1" :

```bash title="BASH"
docker build -t myjenkins-blueocean:2.516.2-1 .
```

Si vous n'avez pas encore téléchargé l'image officielle de Jenkins Docker, le processus ci-dessus le télécharge automatiquement pour vous.

5 . Exécutez votre propre image `myjenkins-blueocean:2.516.2-1 ` comme conteneur dans Docker à l'aide de la commande [docker run](https://docs.docker.com/engine/reference/run/) suivante :

``` bash title="BASH"
docker run \
  --name jenkins-blueocean \ 1
  --restart=on-failure \ 2
  --detach \ 3
  --network jenkins \ 4
  --env DOCKER_HOST=tcp://docker:2376 \ 5
  --env DOCKER_CERT_PATH=/certs/client \ 
  --env DOCKER_TLS_VERIFY=1 \ 
  --publish 8080:8080 \ 6
  --publish 50000:50000 \ 7
  --volume jenkins-data:/var/jenkins_home \ 8
  --volume jenkins-docker-certs:/certs/client:ro 9
  myjenkins-blueocean:2.516.2-1 10
```

<i style ="color:red;">1</i> (Facultatif) Spécifie le nom du conteneur Docker pour cette instance de l'image Docker.

<i style ="color:red;">2</i> Redémarrez toujours le conteneur s'il s'arrête. S'il est arrêté manuellement, il n'est redémarré que lorsque Docker daemon restarts ou que le conteneur lui-même est redémarré manuellement. 

<i style ="color:red;">3</i> (Facultatif) Exécute le conteneur actuel en arrière-plan, appelé mode "détaché", et publie l'ID de conteneur. Si vous ne spécifiez pas cette option, le journal Docker en cours d'exécution pour ce conteneur s'affiche dans la fenêtre du terminal. 

<i style ="color:red;">4</i> Connecte ce conteneur au réseau `jenkins` précédemment défini. Le démon Docker est désormais disponible pour ce conteneur Jenkins via le nom d'hôte `docker`. 

<i style ="color:red;">5</i> Spécifie les variables d'environnement utilisées par `docker`, `docker-compose` et d'autres outils Docker pour se connecter au démon Docker à partir de l'étape précédente.

<i style ="color:red;">6</i> Mappe ou publie, port 8080 du conteneur actuel au port 8080 sur la machine hôte. Le premier numéro représente le port de l'hôte, tandis que le dernier représente le port du conteneur. Par exemple, pour accéder à Jenkins sur votre machine hôte via le port 49000, entrez  `-p 49000:8080` pour cette option. 

<i style ="color:red;">7</i> (Facultatif) Mappe port 50000 du conteneur actuel au port 50000 sur la machine hôte. Ce n'est nécessaire que si vous avez configuré un ou plusieurs agents Jenkins entrants sur d'autres machines, qui à leur tour interagissent avec votre conteneur `jenkins-blueocean `, connu sous le nom de "contrôleur" Jenkins. Les agents Jenkins entrants communiquent avec le contrôleur Jenkins via le port TCP 50000 par défaut. Vous pouvez modifier ce numéro de port sur votre contrôleur Jenkins via la page de [sécurité](./securite-gestion-de-la-securite.md). Par exemple, si vous mettez à jour le **port TCP pour les agents Jenkins entrants** de votre contrôleur Jenkins à 51000, vous devez relancer Jenkins via la commande `docker run …​`. Spécifiez l'option « publish » comme suit : la première valeur correspond au numéro de port sur la machine hébergeant le contrôleur Jenkins, et la dernière valeur correspond à la valeur modifiée sur le contrôleur Jenkins, par exemple `--publish 52000:51000`. Les agents Jenkins entrants communiquent avec le contrôleur Jenkins sur ce port (52000 dans cet exemple). Notez que les [agents WebSocket](https://www.jenkins.io/blog/2020/02/02/web-socket/) n'ont pas besoin de cette configuration.

<i style ="color:red;">8</i>Mappe le répertoire `/var/jenkins_home` dans le conteneur du [volume](https://docs.docker.com/engine/admin/volumes/volumes/) docker avec le nom `jenkins-data`. Au lieu de cartographier le répertoire / `var/jenkins_home` dans un volume Docker,  vous pouvez également mapper ce répertoire vers un répertoire du système de fichiers local de votre machine. Par exemple, spécifiez l'option `--volume $HOME/jenkins:/var/jenkins_home` pour mapper le répertoire `/var/jenkins_home` du conteneur au sous-répertoire `jenkins` dans le répertoire `$HOME` de votre machine locale, généralement `/Users/<your-username>/jenkins `. REMARQUE : si vous modifiez le volume ou le répertoire source à cet effet, le volume du conteneur `docker:dind` ci-dessus doit être mis à jour en conséquence.

<i style ="color:red;">9</i> Mappe le répertoire `/certs/client ` du volume `jenkins-docker-certs` créé précédemment. Les certificats Client TLS requis pour se connecter au démon Docker sont désormais disponibles dans le chemin spécifié par la variable d'environnement `DOCKER_CERT_PATH`. 

<i style ="color:red;">10</i> Le nom de l'image Docker, que vous avez construite à l'étape précédente. 

!!! Info

    Si vous avez des problèmes de copie et de collage de l'extrait de commande, utilisez la version sans annotation ci-dessous :

```bash title="BASH"
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.516.2-1
```

6 . Passez à [l'assistant d'installation post-installation](#assistant-de-configuration-post-installation).

## Sur Windows

Le projet Jenkins fournit une image de conteneur Linux, pas une image de conteneur Windows. Assurez-vous que votre installation Docker pour Windows est configurée pour exécuter des ` Linux Containers ` plutôt que des `Windows Containers`. Reportez-vous à la documentation Docker pour les instructions pour [passer aux conteneurs Linux](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers). Une fois configurée pour exécuter des conteneurs Linux, les étapes sont : 

1 . Ouvrez une fenêtre d'invite de commande et similaire aux instructions [macOS et Linux](#sur-macos-et-linux) ci-dessus, faites ce qui suit : 

2 . Créer un réseau de ponts dans Docker 

```bash title="BASH"
docker network create jenkins
```

3 . Exécutez une image Docker docker: dind

```bash title="BASH"
Docker Run --name Jenkins-Docker --rm --Detach ^ 
- priviled --network Jenkins --network-alias docker ^ 
--env docker_tls_certdir = / certs ^ 
- Volume Jenkins-Docker-Certs: / Certs / Client ^ 
--volume jenkins-data: / var / jenkins_home ^ 
- Édition 2376: 2376 ^ 
Docker: Dind
```

4 . Personnalisez l'image officielle de Jenkins Docker, en exécutant les deux étapes suivantes : 

a . Créez un dockerfile avec le contenu suivant:  

``` cfg title="DOKERFILE"
FROM jenkins/jenkins:2.516.2-jdk21
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow json-path-api"
```

b . Créez une nouvelle image Docker à partir de ce dockerfile et attribuez à l'image un nom significatif, par ex. "myjenkins-blueocean:2.516.2-1" :

``` bash titler="BASH"
docker build -t myjenkins-blueocean:2.516.2-1 .
```

Si vous n'avez pas encore téléchargé l'image officielle de Jenkins Docker, le processus ci-dessus le télécharge automatiquement pour vous.

5 . Exécutez votre propre image `myjenkins-blueocean:2.516.2-1 ` comme conteneur dans Docker à l'aide de la commande `docker run` suivante :

```bash title="BASH"
docker run --name jenkins-blueocean --restart=on-failure --detach ^
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 ^
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 ^
  --volume jenkins-data:/var/jenkins_home ^
  --volume jenkins-docker-certs:/certs/client:ro ^
  --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:2.516.2-1
```

6 . Passez à [l'assistant d'installation](#assistant-installation).

## Accéder au conteneur Docker

Si vous souhaitez accéder à votre conteneur Docker via une invite de commande de terminal grâce à [docker exec](https://docs.docker.com/engine/reference/commandline/exec/), ajoutez une option comme `--name jenkins-tutorial` à la commande `docker exec`. Cela permettra d'accéder au conteneur Docker Jenkins nommé « jenkins-tutorial ».

Vous pouvez accéder à votre conteneur Docker (via une fenêtre d'invite de commande terminal séparée) avec une commande `docker exec` telle que :
`docker exec -it jenkins-blueocean bash`.

## Accéder aux journaux Docker

Vous souhaiterez peut-être accéder au journal de la console Jenkins, par exemple, lors du [déverrouillage de Jenkins](€deverouillage-de-jenkins) dans le cadre de [l'assistant de configuration post-installation](#assistant-de-configuration-post-installation).

Accédez au journal de la console Jenkins via la fenêtre d'invite de commande d'un terminal à partir de laquelle vous avez exécuté la commande `docker run…`. Alternativement, vous pouvez également accéder au journal de la console Jenkins via les [journaux Docker](https://docs.docker.com/engine/reference/commandline/logs/) de votre conteneur à l'aide de la commande suivante :
`docker logs <docker-container-name>`.

Votre `<docker-container-name>` peut être obtenu à l'aide de la commande `docker ps`.

## Accéder au répertoire racine de Jenkins

Vous pouvez accéder au répertoire racine de Jenkins, pour vérifier les détails d'une construction dans le sous-répertoire de `l'espace de travail`, par exemple.

Si vous avez mappé le répertoire d'accueil Jenkins (`/var/jenkins_home`) à un répertoire du système de fichiers local de votre machine, par exemple dans la commande `docker run …`​ [ci-dessus](#télécharger-et-exécuter-jenkins-dans-docker), accédez au contenu du répertoire via l'invite de commande habituelle d'un terminal de votre machine.

Si vous avez spécifié l'option `--volume jenkins-data:/var/ jenkins_home` dans la commande `docker run…`, accédez au contenu du répertoire racine de Jenkins via l'invite de commande d'un terminal de votre conteneur à l'aide de la commande [docker container exec](https://docs.docker.com/engine/reference/commandline/container_exec/) :

`docker container exec -it <docker-container-name> bash`.

Selon la [section précédente](#accéder-aux-journaux-docker), procurez-vous votre `<docker-container-name>` à l'aide de la commande `docker conteneur ls`. Si vous avez spécifié l'option `--name jenkins-blueocea` dans la commande `docker container run …​` ci-dessus (reportez-vous à [l'accès au conteneur Jenkins/Blue Ocean Docker](#accéder-au-conteneur-docker) si nécessaire), utilisez la commande `docker container exec` :

`docker container exec -it jenkins-blueocean bash`.

## Assistant de configuration post-installation

Après avoir téléchargé, installé et exécuté Jenkins en utilisant l'une des procédures ci-dessus (à l'exception de l'installation avec l'opérateur Jenkins), l'assistant de configuration post-installation commence.

Cet assistant de configuration vous guide à travers quelques étapes rapides et ponctuelles pour déverrouiller Jenkins, le personnaliser à l'aide de plugins et créer le premier utilisateur administrateur grâce auquel vous pourrez continuer à accéder à Jenkins.

### Déverrouiller Jenkins

Lorsque vous accédez à un nouveau contrôleur Jenkins pour la première fois, il vous est demandé de le déverrouiller à l'aide d'un mot de passe généré automatiquement. 

1. Parcourez `http: // localhost: 8080` (ou le port que vous avez configuré pour Jenkins lors de l'installation) et attendez que la page de **déverrouillage de Jenkins** apparaisse. 

![Déverrouiller la page Jenkins ](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-01-unlock-jenkins-page.jpg)

2. À partir de la sortie du journal de la console Jenkins, copiez le mot de passe alphanumérique généré automatiquement (entre les 2 ensembles d'astérisques). 

![Copie de mot de passe d'administration initial ](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-02-copying-initial-admin-password.png)

**Note :** 

* La commande : `sudo cat/var/lib/jenkins/secrets/initialAdminPassword` imprimera le mot de passe à la console ; 
* Si vous exécutez Jenkins dans Docker à l'aide de l'image officielle Jenkins/Jenkins, vous pouvez utiliser `sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword` pour imprimer le mot de passe dans la console sans avoir à exécuter dans le conteneur. 

3. Sur la page **Déverrouiller Jenkins**, collez ce mot de passe dans le champ de **mot de passe de l'administrateur** et cliquez sur **Continuer**. 

**Note :** 

* Le journal de la console Jenkins indique l'emplacement (dans le répertoire domestique Jenkins) où ce mot de passe peut également être obtenu. Ce mot de passe doit être entré dans l'assistant d'installation sur les nouvelles installations de Jenkins avant de pouvoir accéder à l'interface utilisateur principale de Jenkins. Ce mot de passe sert également de mot de passe au compte administrateur par défaut (avec le nom d'utilisateur "admin") s'il vous arrive de sauter l'étape de création utilisateur ultérieure dans l'assistant de configuration.

### Personnalisation de jenkins avec des plugins

Après avoir [déverrouillé Jenkins](#déverrouiller-jenkins), la page **Personnaliser Jenkins** apparaît. Ici, vous pouvez installer n'importe quel nombre de plugins utiles dans le cadre de votre configuration initiale.

Cliquez sur l'une des deux options indiquées : 

* **Installez les plugins suggérés** - pour installer l'ensemble recommandé de plugins, qui sont basés sur les cas d'utilisation les plus courants. 
* **Sélectionnez des plugins à installer** - pour choisir le jeu de plugins à installer initialement. Lorsque vous accédez d'abord à la page de sélection des plugins, les plugins suggérés sont sélectionnés par défaut. 

!!! Info

    Si vous ne savez pas de quel plugins vous avez besoin, choisissez Installer les plugins suggérés. Vous pouvez installer (ou supprimer) des plugins Jenkins supplémentaires à tout moment via la page [Manage Jenkins](./manuel-utilisateur-gestion-jenkins.md)> [Plugins](./manuel-utilisateur-gestion-jenkins.md#plugins) dans Jenkins.

L'assistant de configuration montre la progression de Jenkins en cours de configuration et votre ensemble choisi de plugins Jenkins en cours d'installation. Ce processus peut prendre quelques minutes.

### Création du premier utilisateur : l'administrateur

Enfin, après avoir [personnalisé Jenkins avec des plugins](#personnalisation-de-jenkins-avec-des-plugins), il vous demande de créer votre premier utilisateur : l'administrateur. 

1.  Lorsque la page **_Create First Admin User_** apparaît, spécifiez les détails de votre utilisateur administrateur dans les champs respectifs et cliquez sur **_Save and Finish_** ;
2. Lorsque la page **_Jenkins is ready_** apparaît, cliquez sur **_Start using Jenkins_**.

**Notes :**

* Cette page peut indiquer que **_Jenkins is almost ready !_**. Au lieu de cela et si oui, cliquez sur **_Restart_**. 
* Si la page ne se rafraîchit pas automatiquement après une minute, utilisez votre navigateur Web pour actualiser la page manuellement. 

3. Si nécessaire, connectez-vous à Jenkins avec les informations d'identification de l'utilisateur que vous venez de créer et vous êtes prêt à commencer à utiliser Jenkins !