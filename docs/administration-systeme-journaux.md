# Affichage des Journaux

## Journaux sur le système

Lorsque vous exécutez `jenkins.war` manuellement avec `java -jar jenkins.war`, toutes les informations de journalisation sont par défaut transmises à la sortie standard. De nombreux paquets natifs Jenkins modifient ce comportement afin de garantir que les informations de journalisation soient transmises à un emplacement plus conventionnel pour la plateforme.

### Linux (rpm et deb)

Par défaut, les journaux peuvent être consultés en exécutant `journalctl -u jenkins.service`.

Pour personnaliser l'emplacement des journaux, exécutez `systemctl edit jenkins` et ajoutez ce qui suit :

``` cfg
[Service]
Environment="JENKINS_LOG=%L/jenkins/jenkins.log"
```

### Windows (msi)

Par défaut, les journaux doivent se trouver dans `%JENKINS_HOME%/jenkins.out` et `%JENKINS_HOME%/jenkins.err`, sauf si vous les avez personnalisés dans `%JENKINS_HOME%/jenkins.xml`.

### macOS

Les fichiers journaux doivent se trouver dans `/var/log/jenkins/jenkins.log`, sauf si vous les avez personnalisés dans `org.jenkins-ci.plist`.

### Fichier War

Lorsque Jenkins est lancé à partir d'une ligne de commande avec `java -jar jenkins.war`, le fichier journal est écrit dans le répertoire `JENKINS_HOME`. Si aucune valeur n'est attribuée à la variable d'environnement `JENKINS_HOME`, le fichier journal est écrit dans le répertoire `.jenkins/log`.

### Docker

Si vous exécutez Jenkins dans Docker en tant que conteneur détaché, vous pouvez utiliser `docker logs <containerId>` pour afficher les journaux Jenkins.

## Journaux dans Jenkins

Jenkins utilise `java.util.logging` pour la journalisation. Le système `java.util.logging` envoie par défaut tous les journaux supérieurs à `INFO` vers stdout.

Jenkins est équipé d'une interface graphique permettant de configurer/collecter/rapporter les enregistrements de journaux de votre choix. Cette page vous montre comment procéder.

Tout d'abord, sélectionnez « System Log » (Journal système) dans la page « Manage Jenkins » (Gérer Jenkins) :

![Gérer Jenkins](https://www.jenkins.io/doc/book/resources/managing/SystemLog.png)

À partir de là, vous pouvez créer un enregistreur de journaux personnalisé, qui vous aide à regrouper les journaux pertinents tout en filtrant le bruit.

![Enregistreurs de journaux](https://www.jenkins.io/doc/book/resources/managing/Log-recorder.png)

Choisissez un nom qui vous semble logique.

![Entrer le nom de l'enregistreur de journaux](https://www.jenkins.io/doc/book/resources/managing/Example-logger.png)

Il vous sera ensuite demandé de configurer les enregistreurs et leurs niveaux dont vous souhaitez collecter les sorties. Selon la partie de Jenkins que vous surveillez, vous devrez spécifier différents enregistreurs. Indiquez-nous les symptômes de votre problème dans la liste des utilisateurs et nous devrions être en mesure de vous dire où vous devez chercher. De plus, il s'agit en réalité d'une simple enveloppe autour du package java.util.logging. Si vous programmez en Java, vous devriez donc être en mesure de deviner où chercher.

![Spécifiez les enregistreurs](https://www.jenkins.io/doc/book/resources/managing/Config-logger.png)

Une fois la configuration terminée, Jenkins commencera à collecter des données. Les journaux collectés sont disponibles depuis l'interface utilisateur Web.

## Rendre les journaux personnalisés disponibles en dehors de l'interface utilisateur Web

La solution la plus simple consiste à installer le [plugin Support Core](https://plugins.jenkins.io/support-core), qui permet d'enregistrer automatiquement les journaux personnalisés sur le disque.

## Journalisation de débogage dans Jenkins

1. Créez un fichier `logging.properties` ;
2. Définissez les niveaux de journalisation et un `ConsoleHandler` ;
3. Transmettez ce fichier à la JVM en ajoutant la propriété système `-Djava.util.logging.config.file=<pathTo>/logging.properties`.

Vous trouverez ci-dessous un exemple de fichier **logging.properties**.

!!! info " "
    Dans un environnement de production normal, le niveau par défaut est INFO. Il n'est pas recommandé d'utiliser la journalisation de débogage en production.

``` cfg
handlers = java.util.logging.ConsoleHandler

# voir https://docs.oracle.com/en/java/javase/17/docs/api/java.logging/java/util/logging/SimpleFormatter.html
java.util.logging.SimpleFormatter.format = [%1$tF %1$tT][%4$-6s][%2$s] %5$s %6$s %n

# Conservez ce niveau sur ALL ou FINEST, sinon il sera filtré avant l'application d'autres niveaux.
java.util.logging.ConsoleHandler.level = ALL

# Niveau par défaut.
.level= INFO

# Niveau de verbosité élevé pour un package dédié com.myplugin.*.
com.myplugin.level = ALL
```

