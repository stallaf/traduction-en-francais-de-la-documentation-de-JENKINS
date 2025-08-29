# Politique d'assistance Java

Il existe des exigences distinctes d'exécution et de tâches pour les installations de Jenkins.

### Exécution du système Jenkins

Les versions Java suivantes sont nécessaires pour exécuter Jenkins :

| Versions Java supportées | Version à long terme (LTS) | Version hebdomadaire
| --------------------------------------- | ---------------------------------------- | ---------- |
| Java 17 ou Java 21    | 2.479.1 (octobre 2024)    | 2.463 (juin 2024)
| Java 11, Java 17 ou Java 21   | 2.426.1 (novembre 2023) | 2.419 (août 2023)
| Java 11 ou Java 17 | 2.361.1 (septembre 2022) | 2.357 (juin 2022)
| Java 8, Java 11 ou Java 17    |   2.346.1 (juin 2022) | 2.340 (mars 2022)
| Java 8 ou Java 11 | 2.164.1 (mars 2019) | 2.164 (février 2019)
| Java 8    | 2.60.1 (juin 2017) | 2,54 (avril 2017)
| Java 7 | 1.625.1 (octobre 2015) | 1.612 (mai 2015)

!!! danger "_Versions Java prises en charge_"

    Si vous installez une version Java non prise en charge, votre contrôleur Jenkins ne s'exécutera pas.

Ces exigences s'appliquent à tous les composants du système Jenkins, y compris le contrôleur Jenkins, tous les types d'agents, les clients CLI et d'autres composants. Vous n'avez pas besoin de créer votre application avec la même version de Java utilisée pour exécuter Jenkins lui-même ; Consultez la section "Exécution d'outils basés sur Java et constructions sur Jenkins" ci-dessous.

!!! info "_Mise à niveau de Java vers une version plus récente_"

    Vous souhaitez mettre à niveau une configuration Jenkins existante vers une version plus récente de Java ? Reportez-vous aux directives de [mise à niveau de Java 11 vers 17](./plateforme-maj-java-17.md) et aux directives de [mise à niveau de Java 17 vers 21](./plateforme-maj-java-21.md).

Les instructions d'installation de Docker sont incluses dans ["Télécharger et exécuter Jenkins dans Docker"](./installation-docker.md#télécharger-et-exécuter-jenkins-dans-docker).

Le projet Jenkins effectue un flux de test complet avec le JDK/JRES suivant :

* OpenJDK JDK / JRE 17 - 64 bits ;
* OpenJDK JDK / JRE 21 - 64 bits.

Les JRE/JDKs d'autres fournisseurs sont pris en charge et peuvent être utilisés. Reportez-vous à [notre outil de suivi des problèmes](https://issues.jenkins.io/issues/?jql=labels%3Djdk) pour connaître les problèmes de compatibilité Java connus. Les responsables de Jenkins testent activement les [machines virtuelles Java basées sur HotSpot](https://en.wikipedia.org/wiki/HotSpot_(virtual_machine)), telles que celles d'OpenJDK, Eclipse Temurin et Amazon Corretto. Les responsables de Jenkins ne testent pas les [machines virtuelles Java basées sur Eclipse OpenJ9](https://en.wikipedia.org/wiki/OpenJ9). Le [groupe d'intérêt spécial sur les plateformes](https://www.jenkins.io/sigs/platform/) ne travaille pas activement sur les machines virtuelles Java basées sur OpenJ9.

## Exécution d'outils et de construction Java sur Jenkins

Les versions JDK utilisées pour créer des projets Java ou exécuter des outils Java sont indépendantes de la version Java utilisée pour exécuter les processus du contrôleur et de l'agent Jenkins. Pendant les builds, toute version JRE ou JDK compatible avec le système hôte peut être lancée. Cela inclut :

* Exécution de `java` ou `javac` à partir des étapes de construction de shell et similaires ;
* Exécution des étapes de compilation Maven/Ant/… à l'aide d'un JDK géré par un programme d'installation d'outils JDK.

Certains plugins ont des exigences plus strictes et peuvent nécessiter une version pour exécuter la même version Java utilisée pour exécuter le contrôleur et les agents Jenkins. Un exemple de plugin notable est le [plugin d'intégration Maven](https://plugins.jenkins.io/maven-plugin). Il nécessite que la version JDK utilisée pour les constructions Maven soit au moins la même version Java utilisée dans le contrôleur Jenkins. Ces cas sont généralement documentés dans la documentation du plugin.

## Surveillance des versions Java

Les contrôleurs Jenkins modernes et les agents Jenkins vérifient les exigences de Java et informent les utilisateurs lorsqu'ils sont lancés avec une version non pris en charge.

Le [plugin Versions Node Monitors](https://plugins.jenkins.io/versioncolumn) permet une surveillance détaillée des versions Java.

## JDKs utilisés dans Jenkins

Le projet Jenkins utilise |Eclipse Temurin](https://projects.eclipse.org/projects/adoptium.temurin) comme son principal JDK pour construire et tester les applications basées sur Java. Cela comprend : 

* Images de conteneurs ;
* Construction de la version de base de Jenkins ;
* [Publications automatisées des plugins](https://www.jenkins.io/doc/developer/publishing/releasing-cd/) ;
* [Construction et test d'intégration continue](https://ci.jenkins.io/) ;
* Test d'infrastructure.

Certaines des raisons du choix de Temurin sont : 

* Disponibilité sur de nombreuses versions Java SE différentes et sur un large éventail de plates-formes, y compris différents systèmes d'exploitation et architectures ;
* Entretien régulier et soutien à long terme fourni par la Fondation Eclipse.
  