# Configuration du  Système

<div class="couleur-introduction">
Jenkins stocke sa configuration globale dans des fichiers sur le contrôleur Jenkins. Les administrateurs et les utilisateurs privilégiés modifient la configuration globale à partir des pages de configuration Jenkins.
</div>

## Répertoire d'accueil Jenkins

Le répertoire `JENKINS_HOME` est la racine de la structure de répertoires du disque que Jenkins utilise pour effectuer des compilations et stocker des archives. Le répertoire d'accueil Jenkins est répertorié dans **Manage Jenkins > System** 'Gérer Jenkins > Système)  sous l'en-tête **Home directory** (Répertoire personnel).

![Répertoire d'accueil](https://www.jenkins.io/images/system-administration/administering-jenkins/home-dir.png)

* Sous Windows, il est défini par défaut sur `C:\ProgramData\Jenkins\.jenkins` ;
* Sous Ubuntu, il est défini par défaut sur `/var/lib/jenkins`.

!!! info " "
    Les emplacements `JENKINS_HOME` par défaut varient selon la méthode d'installation :

    * **Programme d'installation Windows** : C:\ProgramData\Jenkins\.jenkins

    * **Exécution à partir du fichier** `.war` : ~/.jenkins

    * **Paquet Debian/Ubuntu** (`apt install jenkins`) : /var/lib/jenkins

    * **Paquet Red Hat** (`yum install jenkins`) : /var/lib/jenkins

    * **Paquet Fedora** (`dnf install jenkins`) : /var/lib/jenkins

    * **Paquet openSUSE** (`zypper install jenkins`) : /var/lib/jenkins

Mais vous pouvez modifier cela de l'une des manières suivantes :

* Définissez la variable d'environnement `JENKINS_HOME` ;
* Définissez la propriété système Java `JENKINS_HOME`.

Vous pouvez également modifier cet emplacement après avoir utilisé Jenkins pendant un certain temps. Pour ce faire :

1.  Arrêtez complètement Jenkins.
2.  Déplacez le contenu de l'ancien `JENKINS_HOME` vers le nouvel emplacement.
3.  Définissez la variable `JENKINS_HOME` sur le nouvel emplacement.
4. Redémarrez Jenkins.

La structure du répertoire de l'arborescence `JENKINS_HOME` est souvent organisée comme suit :

``` cfg
JENKINS_HOME
 +- builds            (build records)
    +- [BUILD_ID]     (subdirectory for each build)
         +- build.xml      (build result summary)
         +- changelog.xml  (change log)
 +- config.xml         (Jenkins root configuration file)
 +- *.xml              (other site-wide configuration files)
 +- fingerprints       (stores fingerprint records, if any)
 +- identity.key.enc   (RSA key pair that identifies an instance)
 +- jobs               (root directory for all Jenkins jobs)
     +- [JOBNAME]      (sub directory for each job)
         +- config.xml (job configuration file)
     +- [FOLDERNAME]   (sub directory for each folder)
         +- config.xml (folder configuration file)
         +- jobs       (subdirectory for all nested jobs)
 +- plugins            (root directory for all Jenkins plugins)
     +- [PLUGIN]       (sub directory for each plugin)
     +- [PLUGIN].jpi   (.jpi or .hpi file for the plugin)
 +- secret.key         (deprecated key used for some plugins' secure operations)
 +- secret.key.not-so-secret  (used for validating _$JENKINS_HOME_ creation date)
 +- secrets        (root directory for the secret+key for credential decryption)
     +- hudson.util.Secret   (used for encrypting some Jenkins data)
     +- master.key           (used for encrypting the hudson.util.Secret key)
     +- InstanceIdentity.KEY (used to identity this instance)
     +- jenkins.model.Jenkins.crumbSalt   (used for encrypting some Jenkins data)
     +- initialAdminPassword (used for initial login)
 +- userContent        (files served under your https://server/userContent/)
 +- workspace          (working directory for the version control system)
```

## Configuration de la page Système

La page Configurer le système (`/manage/configure`) permet aux administrateurs de configurer les paramètres globaux du contrôleur Jenkins. Voici les principales sections disponibles sur cette page :

### Message système

Le message système s'affiche en haut de chaque page dans Jenkins. Il peut être utilisé pour communiquer des informations importantes à tous les utilisateurs, telles que les calendriers de maintenance ou les annonces.

Exemple :

``` console
The system is undergoing maintenance on YYYY-MM-DD.
Expect downtime.
```

### Nombre d'exécuteurs

Le nombre d'exécuteurs détermine le nombre de compilations simultanées que Jenkins peut effectuer. Pour les contrôleurs Jenkins, il est recommandé de définir cette valeur sur `0` afin d'éviter les conflits de ressources. Les exécuteurs doivent plutôt être configurés sur les agents (nœuds).

!!! info " "
    **Pourquoi définir les exécuteurs sur 0 sur le contrôleur ?** Définir les exécuteurs sur `0` garantit que le contrôleur se consacre à la gestion des compilations et à la coordination des agents, plutôt qu'à l'exécution des compilations elle-même. Cela améliore la stabilité et les performances.

    Par exemple, les agents dotés de capacités spécifiques peuvent être configurés avec plusieurs exécuteurs afin de gérer efficacement les compilations simultanées.

### Étiquettes

Les étiquettes sont utilisées pour regrouper les agents (nœuds) en fonction de critères spécifiques. Elles facilitent l'attribution de tâches à des agents spécifiques. Par exemple, vous pouvez étiqueter les agents en fonction de leur système d'exploitation ou de leurs capacités matérielles.

Exemple :

``` console
linux, windows, docker, high-memory
```

Les étiquettes peuvent être attribuées dans les paramètres de configuration de l'agent.

### Utilisation

Cette section définit la manière dont les tâches sont attribuées aux agents. Les options disponibles sont les suivantes :

* **Use this node as much as possible** (Utiliser ce nœud autant que possible) : les tâches seront attribuées à ce nœud dans la mesure du possible ;
* **Only build jobs with label expressions matching this node** (Ne créer que les tâches dont les expressions d'étiquette correspondent à ce nœud) : les tâches ne seront attribuées que si leur étiquette correspond.

### Période de repos

La période de repos définit un délai avant le début d'une compilation après la détection d'une modification. Cela permet d'éviter de déclencher plusieurs compilations pour des modifications très rapprochées.

Exemple :

``` console
Set a quiet period of 5 seconds to batch changes.
(Définissez une période de repos de 5 secondes pour regrouper les modifications).
```

Cela est particulièrement utile pour les intégrations de contrôle de version, où les commits peuvent se succéder rapidement.

### Emplacement Jenkins

Cette section définit l'URL du contrôleur Jenkins. Elle est importante pour une communication correcte avec les agents et les outils externes. Si Jenkins se trouve derrière un proxy inverse, assurez-vous que l'URL correspond à la configuration du proxy.

!!! info " "
    Pour obtenir des instructions détaillées sur la configuration d'un proxy inverse, consultez le [guide de configuration du proxy inverse](./administration-systeme-configuration-du-proxy-inverse.md ).

### Statistiques d'utilisation

Jenkins collecte des statistiques d'utilisation anonymes afin d'améliorer le logiciel. Cette section vous permet d'activer ou de désactiver cette fonctionnalité.

### Configuration spécifique aux plugins

De nombreux plugins ajoutent leurs propres options de configuration globales à la page **Configure System ** (Configurer le système). Étant donné que les plugins étendent les fonctionnalités de Jenkins, leurs paramètres apparaissent souvent dans cette section et offrent des options de personnalisation et de réglage fin. Reportez-vous à l'aide en ligne de chaque plugin pour comprendre ces paramètres.
	
!!! info " "
    **Conseil** : utilisez l'icône en forme de point d'interrogation (`?`) à côté de chaque paramètre pour accéder à l'aide détaillée relative à cette configuration spécifique.

## Ressources supplémentaires

* [Administration du système](./administration-systeme-presentation.md) ;
* [Configuration de la sécurité](./securite-presentation.md) ;
* [Gestion des plugins](./gestion-plugins.md).

