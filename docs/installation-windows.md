# Windows

La façon la plus simple d'installer Jenkins sur Windows consiste à utiliser l'installateur Windows Jenkins. Ce programme installera Jenkins en tant que service en utilisant un JVM 64 bits choisi par l'utilisateur. Gardez à l'esprit que pour exécuter Jenkins en tant que service, le compte qui exécute Jenkins doit avoir la permission de se connecter en tant que service.

## Conditions préalables

Exigences matérielles minimales : 

* 256 Mo de RAM ;
* 1 Go d'espace disque (bien que 10 Go soit un minimum recommandé si l'exécution de Jenkins comme un conteneur Docker).

Configuration matérielle recommandée pour une petite équipe : 

* 4 Go + de RAM  ;
* 50 Go + d'espace disque.

Recommandations matérielles complètes : 

Matériel : voir la page des recommandations matérielles](./mise-a-echelle-recommandations-materielles.md).

Exigences logicielles : 

* Java: Voir la [page des exigences Java](./plateforme-java.md) ; 
* Navigateur Web : Voir la [page de compatibilité du navigateur Web](./administration-systeme-navigateurs.md) ; 
* Pour le système d'exploitation Windows:  [politique de support Windows](./plateforme-windows.md) ; 
* Pour le système d'exploitation Linux : [Politique de support Linux](./plateforme-linux.md) ;
* Pour les conteneurs de servlet : [Politique de support des conteneurs servlet](./plateforme-servlet.md).

## Étapes d'installation à l'aide du programme d'installation de Windows MSI

_Comment installer Jenkins sur Windows_
<iframe width="682" height="384" src="https://www.youtube.com/embed/XuMrEDA8cAI" title="How to Install Jenkins on Windows" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Reportez-vous à la section Windows de la page [Jenkins à télécharger](https://www.jenkins.io/download/#downloading-jenkins) pour télécharger une version LTS ou une version hebdomadaire du programme d'installation de Windows. Une fois le téléchargement terminé, ouvrez le programme d'installation de Windows et suivez les étapes ci-dessous pour installer Jenkins.

**Étape 1 : Configuration de l'assistant**

Lors de l'ouverture du programme d'installation de Windows, un **assistant de configuration** d'installation apparaît, cliquez sur **Suivant** sur l'assistant de configuration pour démarrer votre installation.

![Assistant de configuration de l'installation Windows](https://www.jenkins.io/doc/book/resources/tutorials/windows-installation-setup-wizard.png)

**Étape 2 : Sélectionnez le dossier de destination** 

Sélectionnez le dossier de destination pour stocker votre installation Jenkins et cliquez sur **Suivant** pour continuer.

![Destination d'installation de Jenkins](https://www.jenkins.io/doc/book/resources/tutorials/windows-installation-setup-wizard.png)

**Étape 3 : Informations sur la connexion du service** 

Lors de l'installation de Jenkins, il est recommandé d'installer et d'exécuter Jenkins en tant que service Windows indépendant à l'aide d'un utilisateur local ou de domaine car il est beaucoup plus sûr que d'exécuter Jenkins à l'aide de **LocalSystem (équivalent Windows de Root)** qui accordera à Jenkins un accès complet à votre machine et services.

Pour exécuter le service Jenkins à l'aide d'un **utilisateur local ou de domaine**, spécifiez le nom d'utilisateur et le mot de passe du domaine avec lesquels vous souhaitez exécuter Jenkins, cliquez sur **Test Identials** pour tester vos informations d'identification de domaine et cliquez sur **Suivant**.

![Informations sur la connexion Jenkins Service](https://www.jenkins.io/doc/book/resources/tutorials/windows-jenkins-logon-credentials.png
) 

!!! info

    Le compte d'utilisateur exécutant Jenkins nécessite l'autorisation de connexion `LogonAsService` pour exécuter Jenkins en tant que service. Si vous obtenez une fenêtre contextuelle d'erreur de connexion non valide tout en essayant de tester vos informations d'identification, suivez les étapes [expliquées ici](#Identifiants de connexion au service non valides) pour la résoudre.

**Étape 4 : sélection de port** 

Spécifiez le port sur lequel Jenkins sera exécuté, puis cliquez sur le bouton **_Test Port_** pour vérifier si le port spécifié est libre sur votre machine. Si le port est libre, une coche verte s'affichera comme indiqué ci-dessous. Cliquez ensuite sur **Suivant**.

![Port Jenkins Sélectionnez](https://www.jenkins.io/doc/book/resources/tutorials/windows-select-port.png)

**Étape 5 : Sélectionnez Java Home Directory**

Le processus d'installation vérifie la présence de Java sur votre ordinateur et préremplit la boîte de dialogue avec le répertoire d'installation de Java. Si la version Java requise n'est pas installée sur votre ordinateur, vous serez invité à l'installer.

Une fois que votre répertoire domestique Java a été sélectionné, cliquez sur **Suivant** pour continuer.

![Sélectionnez Java Home Directory](https://www.jenkins.io/doc/book/resources/tutorials/windows-select-jdk-installation.png)

**Étape 6 : Configuration personnalisée** 

Sélectionnez d'autres services qui doivent être installés avec Jenkins et cliquez sur **Suivant**.

![Configuration personnalisée de Jenkins](https://www.jenkins.io/doc/book/resources/tutorials/windows-select-features-to-install.png)

**Étape 7 : Installer Jenkins**

Cliquez sur le bouton **Installer** pour démarrer l'installation de Jenkins.

![Windows installe Jenkins](https://www.jenkins.io/doc/book/resources/tutorials/windows-install.png)

De plus, cliquer sur le bouton **Installer** affichera la barre de progression de l'installation, comme indiqué ci-dessous :

![Progrès de l'installation de Windows](https://www.jenkins.io/doc/book/resources/tutorials/windows-installation-progress.png)

**Étape 8 : terminer l'installation de Jenkins** 

Une fois l'installation terminée, cliquez sur **Terminer** pour terminer l'installation.

![Installation de finition](https://www.jenkins.io/doc/book/resources/tutorials/windows-finish-install.png)

Jenkins sera installé en tant que **service Windows**. Vous pouvez valider cela en parcourant la section **services**, comme indiqué ci-dessous :

![Service Windows Jenkins](https://www.jenkins.io/doc/book/resources/tutorials/windows-jenkins-service.png)

!!! info

    Voir les [étapes de mise à niveau](https://www.jenkins.io/blog/2020/08/12/windows-installers-upgrade/#next-steps) lorsque vous passez à une nouvelle version.

## Installation silencieuse avec les installateurs MSI

Les programmes d'installation MSI peuvent être mis en place via une méthode silencieuse, qui peut afficher l'interface utilisateur de base (/qb) ou aucune interface utilisateur (/qn). La méthode silencieuse ne demande aucune saisie de la part de l'utilisateur. Vous pouvez donc passer des propriétés au programme d'installation afin de définir des valeurs spécifiques. Vous trouverez ci-dessous une ligne de commande très basique pour une installation silencieuse.

``` bat title="BAT"
msiexec.exe /i "path\to\jenkins.msi" /qn /norestart
```

_**Cela utilisera toutes les valeurs par défaut pour les choses qui seraient normalement une invite telle que :**_

* Répertoire d'installation ;
* Nom d'utilisateur/mot de passe du compte de service ;
* Répertoire d'installation de Java ;
* Le port pour Jenkins pour écouter.

Chacune de ces choses peut être remplacée en passant une paire de propriétés ` NAME=VALUE ` pour ce que vous voulez remplacer :

| Nom de la Propriété   | Description                     |
| --------------------------------- | --------------------                  |
| INSTALLDIR                    | Chemin d'accès au répertoire dans lequel installer Jenkins. (Par défaut : C:\Program Files\Jenkins).
| PORT                               | Le port sur lequel Jenkins sera à l'écoute. (Defaut : 8080).
| JAVA_HOME                 | Répertoire dans lequel se trouve java.exe. (Par défaut : dernière version recommandée du runtime Java trouvée dans le registre).
| SERVICE_USERNAME | Nom d'utilisateur sous lequel le service doit s'exécuter. Le compte doit disposer des autorisations LogonAsService. (Par défaut : en mode silencieux, le compte LOCALSYSTEM).
| SERVICE_PASSWORD | Mot de passe du compte SERVICE_USERNAME. Il ne doit être fourni que si SERVICE_USERNAME est fourni. (Par défaut : en mode silencieux, aucun pour LOCALSYSTEM).

Un exemple plus complexe, y compris la création d'un fichier journal pour le processus d'installation, est illustré ci-dessous :

``` bat title="BAT"
msiexec.exe /i "path\to\jenkins.msi" /qn /norestart INSTALLDIR="D:\Jenkins" JAVA_HOME="C:\Program Files\SomeJava" PORT=80 /L*v "path\to\logfile.txt"
```

Cela installerait Jenkins dans D:\Jenkins, utiliserait le runtime Java de C:\Program Files\SomeJava et Jenkins écouterait sur le port 80.

## Assistant de configuration post-installation

Après avoir téléchargé, installé et exécuté Jenkins, l'assistant de configuration post-installation commence.

Cet assistant de configuration vous guide à travers quelques étapes "uniques" rapides pour déverrouiller Jenkins, la personnaliser avec des plugins et créer le premier utilisateur administrateur à travers lequel vous pouvez continuer à accéder à Jenkins.

### Déverrouiller Jenkins

Lorsque vous accédez à un nouveau contrôleur Jenkins pour la première fois, il vous est demandé de le déverrouiller à l'aide d'un mot de passe généré automatiquement.

**Étape 1 :**

Parcourez `http: // localhost: 8080` (ou le port que vous avez configuré pour Jenkins lors de l'installation) et attendez que la page de déverrouillage de Jenkins apparaisse.

![Déverrouiller la page Jenkins](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-02-unlock-jenkins-page.png)

**Étape 2 :**

Le mot de passe d'administrateur initial doit être trouvé sous le chemin d'installation de Jenkins (défini à l'étape 2 dans l'installation de Jenkins).

Pour l'emplacement d'installation par défaut vers C:\Program Files\Jenkins, un fichier appelé **initialAdminPassword** peut être trouvé sous C:\Program Files\Jenkins\Secrets.

Cependant, si un chemin personnalisé pour l'installation de Jenkins a été sélectionné, vous devez vérifier cet emplacement pour le fichier **initialAdMinPassword**.

![Emplacement de mot de passe initial de Jenkins](https://www.jenkins.io/doc/book/resources/tutorials/windows-initial-password-location.png)

**Étape 3 :**

Ouvrez le fichier en surbrillance et copiez le contenu du fichier **initialAdMinPassword**.

![Fichier de mot de passe initial de Jenkins](https://www.jenkins.io/doc/book/resources/tutorials/windows-initial-password-file.png)

**Étape 4 :**

Sur la page **Déverrouiller Jenkins**, collez ce mot de passe dans le champ de **mot de passe de l'administrateur** et cliquez sur **Continuer**. 

**Notes :**

* Vous pouvez également accéder aux journaux Jenkins dans le fichier **jenkins.err.log** dans votre répertoire Jenkins spécifié lors de l'installation. 
* Le fichier journal Jenkins est un autre emplacement (dans le répertoire domestique Jenkins) où le mot de passe initial peut également être obtenu. 

![Fichier journal de Windows Jenkins](https://www.jenkins.io/doc/book/resources/tutorials/windows-jenkins-log.png)

Ce mot de passe doit être entré dans l'assistant d'installation sur les nouvelles installations de Jenkins avant de pouvoir accéder à l'interface utilisateur principale de Jenkins. Ce mot de passe sert également de mot de passe du compte d'administrateur par défaut (avec le nom d'utilisateur "admin") s'il vous arrive de sauter l'étape de création utilisateur ultérieure dans l'assistant de configuration.

### Personnalisation des jenkins avec des plugins

Après avoir [déverrouillé Jenkins](#déverrouiller-jenkins), la page **Personnaliser Jenkins** apparaît. Ici, vous pouvez installer n'importe quel nombre de plugins utiles dans le cadre de votre configuration initiale.

Cliquez sur l'une des deux options indiquées: 

* **Installez les plugins suggérés** - pour installer l'ensemble recommandé de plugins, qui sont basés sur les cas d'utilisation les plus courants. 
* **Sélectionnez des plugins à installer** - pour choisir le jeu de plugins à installer initialement. Lorsque vous accédez d'abord à la page de sélection des plugins, les plugins suggérés sont sélectionnés par défaut. 

!!! info

    Si vous ne savez pas de quel plugins vous avez besoin, choisissez **Installer les plugins suggérés**. Vous pouvez installer (ou supprimer) des plugins Jenkins supplémentaires à un moment ultérieur via la page Manage Jenkins> Plugins dans Jenkins.

L'assistant de configuration montre la progression de Jenkins en cours de configuration et votre ensemble choisi de plugins Jenkins en cours d'installation. Ce processus peut prendre quelques minutes.

### Création du premier utilisateur administrateur

Enfin, après avoir [personnalisé Jenkins avec des plugins](#personnalisation-de-jenkins-avec-des-plugins), Jenkins vous demande de créer votre premier utilisateur administrateur. 

1. Lorsque la page **Créer le Premier Utilisateur Administrateur** s'affiche, indiquez les informations relatives à votre utilisateur administrateur dans les champs correspondants, puis cliquez sur **Enregistrer et Terminer**.
2. Lorsque la page **Jenkins est prête** apparaît, cliquez sur **Démarrer à l'aide de Jenkins**. 

    **Notes :** 

    * Cette page peut indiquer que **Jenkins est presque prêt !** Au lieu de cela et si oui, cliquez sur **Redémarrer**. 
    * Si la page ne se rafraîchit pas automatiquement après une minute, utilisez votre navigateur Web pour actualiser la page manuellement. 

3. Si nécessaire, connectez-vous à Jenkins avec les informations d'identification de l'utilisateur que vous venez de créer et vous êtes prêt à commencer à utiliser Jenkins !

## Dépannage de l'installation de Windows

### Identifiants de connexion au service non valides

![Indematives de connexion de service non valide](https://www.jenkins.io/doc/book/resources/tutorials/windows-invalid-service-logon-credentials.png)


Lorsque vous installez un service qui doit s'exécuter sous un compte utilisateur de domaine, ce compte doit disposer du droit de se connecter en tant que service. Cette autorisation de connexion s'applique strictement à l'ordinateur local et doit être accordée dans la stratégie de sécurité locale.

Procédez comme suit pour modifier la stratégie de sécurité locale de l'ordinateur sur lequel vous souhaitez définir l'autorisation « Se connecter en tant que service » :

1. Connectez-vous à l'ordinateur avec des privilèges administratifs ;
2. Ouvrez les **Outils d'administration**, puis la **Stratégie de sécurité locale**, ou tapez `secpol.msc` dans la boîte de dialogue Exécuter (Win + R) et appuyez sur Entrée ;
3. Si la **politique de sécurité locale** est manquante dans votre système, consultez la réponse à la question [« Où télécharger GPEdit.msc pour Windows 10 Home ? »](https://answers.microsoft.com/en-us/windows/forum/all/where-to-download-gpeditmsc-for-windows-10-home/c39bd656-8d4a-4374-be39-394c09deec4e) sur Microsoft Community pour résoudre le problème.
4. Dans la fenêtre de la **politique de sécurité locale**, développez la stratégie locale et cliquez sur l'affectation des droits des utilisateurs ; 
5. Dans le volet droit, cliquez avec le bouton droit sur **Connectez-vous en tant que service** et sélectionnez Propriétés ; 
6. Cliquez sur le bouton **Ajouter l'utilisateur ou le groupe**… pour ajouter le nouvel utilisateur ; 
7. Dans la boîte de dialogue **Sélectionner des utilisateurs ou des groupes**, recherchez l'utilisateur que vous souhaitez ajouter et cliquez sur OK ;
8. Cliquez sur **OK** dans la boîte de dialogue **Propriétés de Connexion en tant que service** pour enregistrer les modifications.

Après avoir terminé les étapes ci-dessus, essayez à nouveau de vous connecter avec l'utilisateur ajouté.