# Gestion des Noeuds

### Composants des compilations distribuées

Les compilations dans une [architecture de construction distribuées](https://www.jenkins.io/doc/book/scaling/architecting-for-scale/#distributed-builds-architecture) utilisent des nœuds, des agents et des exécuteurs, qui sont distincts du contrôleur Jenkins lui-même. Il est utile de comprendre ce que sont chacun de ces composants pour gérer les nœuds :

**Contrôleur Jenkins**<br>
Le contrôleur Jenkins est le service Jenkins lui-même et l'endroit où Jenkins est installé. Il s'agit également d'un serveur web qui agit comme un « cerveau » pour décider comment, quand et où exécuter les tâches. Les tâches de gestion telles que la configuration, l'autorisation et l'authentification sont exécutées sur le contrôleur, qui traite les requêtes HTTP. Les fichiers écrits lors de l'exécution d'un pipeline sont enregistrés dans le système de fichiers du contrôleur, à moins qu'ils ne soient transférés vers un référentiel d'artefacts tel que Nexus ou Artifactory.

**Nœuds**<br>
Les nœuds sont les « machines » sur lesquelles s'exécutent les agents de construction. Jenkins surveille chaque nœud connecté pour vérifier l'espace disque, l'espace temporaire libre, l'espace d'échange libre, l'heure/la synchronisation et le temps de réponse. Un nœud est mis hors ligne si l'une de ces valeurs dépasse le seuil configuré. Jenkins prend en charge deux types de nœuds :

* **agents** (décrits ci-dessous) ;
* **nœud intégré** 
    * Le nœud intégré est un nœud qui existe dans le processus du contrôleur. Il est possible d'utiliser des agents et le nœud intégré pour exécuter des tâches. Cependant, l'exécution de tâches sur le nœud intégré est déconseillée pour des raisons de sécurité, de performances et d'évolutivité. Le nombre d'exécuteurs configurés pour le nœud détermine la capacité du nœud à exécuter des tâches. Définissez le nombre d'exécuteurs sur 0 pour désactiver l'exécution de tâches sur le nœud intégré.

**Agents**<br>
Les agents gèrent l'exécution des tâches pour le compte du contrôleur Jenkins à l'aide d'exécuteurs. Un agent est un petit processus client Java (fichier jar unique de 170 Ko) qui se connecte à un contrôleur Jenkins et qui est considéré comme peu fiable. Un agent peut utiliser n'importe quel système d'exploitation prenant en charge Java. Tous les outils nécessaires à la compilation et aux tests sont installés sur le nœud où l'agent s'exécute. Comme ces outils font partie du nœud, ils peuvent être installés directement ou dans un conteneur, tel que Docker ou Kubernetes. Chaque agent est en fait un processus avec son propre identifiant de processus (PID) sur la machine hôte. En pratique, les nœuds et les agents sont essentiellement identiques, mais il est bon de se rappeler qu'ils sont conceptuellement distincts.

**Exécuteurs**<br>
Un exécuteur est un emplacement destiné à l'exécution de tâches. Il s'agit en fait d'un thread dans l'agent. Le nombre d'exécuteurs sur un nœud définit le nombre de tâches simultanées pouvant être exécutées. En d'autres termes, cela détermine le nombre de `stages` de Pipeline simultanées pouvant être exécutées en même temps. Le nombre correct d'exécuteurs par nœud de compilation doit être déterminé en fonction des ressources disponibles sur le nœud et des ressources requises pour la charge de travail. Pour déterminer le nombre d'exécuteurs à exécuter sur un nœud, tenez compte des besoins en CPU et en mémoire, ainsi que du volume d'activité d'E/S et réseau :

* Un exécuteur par nœud est la configuration la plus sûre ;
*   Un exécuteur par cœur de processeur peut fonctionner correctement si les tâches exécutées sont de petite taille ;
* Surveillez attentivement les performances d'E/S, la charge du processeur, l'utilisation de la mémoire et le débit d'E/S lorsque vous exécutez plusieurs exécuteurs sur un nœud.

## Création d'agents

Les agents Jenkins sont les « travailleurs » qui exécutent les opérations demandées par le contrôleur Jenkins. Le contrôleur Jenkins administre les agents et peut gérer les outils sur les agents. Les agents Jenkins peuvent être alloués de manière statique ou dynamique via des systèmes tels que Kubernetes, OpenShift, Amazon EC2, Azure, Google Cloud, IBM Cloud, Oracle Cloud et d'autres fournisseurs de cloud.

Ce tutoriel de 30 minutes de Darin Pope explique comment créer un agent Jenkins et le connecter à un contrôleur.

_Comment créer un nœud d'agent dans Jenkins_

<iframe width="640" height="360" src="https://www.youtube.com/embed/99DddJiH7lM" title="How to Create an Agent Node in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Lancer l'agent entrant via le planificateur Windows

Si vous rencontrez des difficultés pour installer l'agent entrant en tant que service Windows (c'est-à-dire que vous avez suivi [les instructions d'installation de l'agent en tant que service ici](https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins+as+a+Windows+service), mais que cela n'a pas fonctionné), une autre méthode pour démarrer automatiquement le service au démarrage de Windows consiste à utiliser le Planificateur Windows. 

Nous tirons parti de la capacité du planificateur Windows à exécuter des commandes au démarrage du système

1. Configurez votre nœud pour utiliser la méthode de lancement « Lancer les agents en les connectant au maître »
    * Cliquez sur Save (Enregistrer)
2. Notez la commande requise pour lancer l'agent
    * Sur la page Jenkins du nouveau nœud agent, notez la ligne de commande de l'agent affichée. 
        * Elle ressemblera à ceci :

``` java title="JAVA"
java \
    -jar agent.jar \
    -url <URL Jenkins> \
    -secret <clé secrète> \
```

1. Obtenez le fichier agent.jar et copiez-le sur votre nouveau nœud agent Windows.
    * Dans la ligne de commande indiquée à la dernière étape, « agent.jar » est un lien hypertexte. Cliquez dessus pour télécharger le fichier agent.jar ;
    * Copiez le fichier agent.jar dans un emplacement permanent sur votre machine agent.
2. Assurez-vous que vous disposez d'une version Java sur votre machine agent.
    * Si ce n'est pas le cas, procurez-vous et installez une [version prise en charge](./plateforme-java.md) de Java.
3. Exécutez la commande manuellement à partir d'une fenêtre CMD sur votre agent pour vérifier qu'elle fonctionne.
    * Ouvrez la fenêtre CMD ;
    * Exécutez la commande suivante :

``` java title="JAVA"
java \
-jar agent.jar \
-url <URL Jenkins> \
-secret <clé secrète> \
-name <nom de l'agent \
```

* Revenez à la page Web du nœud dans Jenkins.  Si tout fonctionne, la page devrait afficher « Agent connecté » ;
* Arrêtez la commande (Ctrl+C) ;
    * Enregistrez une nouvelle tâche planifiée pour exécuter la même commande ;
* Ouvrez le « Planificateur de tâches » sur votre ordinateur Windows ;
    * Démarrer → Exécuter : Planificateur de tâches ;
 * Créez une tâche de base (Menu : Action → Créer une tâche de base) ;
    * Première page de l'assistant :
        * Nom : Jenkins Agent ;
        * Description (facultatif) ;
        * Cliquez sur Suivant ;
    * Page suivante de l'assistant ;
        * Quand souhaitez-vous que la tâche démarre : sélectionnez « Au démarrage de l'ordinateur » ;
        * Cliquez sur Suivant ;
    * Page suivante de l'assistant ;
        * Quelle action souhaitez-vous que la tâche effectue : sélectionnez « Démarrer un programme » ;
        * Cliquez sur Suivant ;
    * Page suivante de l'assistant ;
        * Programme/Script : entrez « java.exe » (ou le chemin d'accès complet à votre fichier java.exe) ;
        * Ajouter des arguments : entrez le reste de la commande, par exemple :

``` java title="JAVA"
java
    -jar agent.jar \
    -url <URL Jenkins> \
    -secret <clé secrète> \
    -name <nom de l'agent>
```

Exemple :

``` java title="JAVA"
java \
    -jar D:\Scripts\jenkins\agent.jar \
    -url http://jenkinshost.example.com \
    -secret d6a84df1fc4f45ddc9c6ab34b08f13391983ffffffffffb3488b7d5ac77fbc7 \
    -name buildNode1
```

* Cliquez sur Suivant ;
    * Page suivante de l'assistant ;
* Cochez la case « Ouvrir la boîte de dialogue Propriétés pour cette tâche lorsque je clique sur Terminer » ;
* Cliquez sur Terminer ;
    * Mettez à jour les propriétés de la tâche ;
        * Dans l'onglet Général ;
* Sélectionnez l'utilisateur qui exécutera la tâche ;
* Sélectionnez « Exécuter que l'utilisateur soit connecté ou non » ;
    * Dans l'onglet Paramètres ;
* Décochez « Arrêter la tâche si elle s'exécute pendant plus de » ;
* Cochez « Exécuter la tâche dès que possible après un démarrage planifié manqué » ;
* Cochez « Si la tâche a échoué, redémarrer toutes les : 10 minutes » et « Tenter de redémarrer jusqu'à : 3 fois » ;
    * Cliquez sur OK ;
        * Démarrez la tâche planifiée et vérifiez à nouveau que l'agent est connecté ;
            * Revenez à la page Web du nœud dans Jenkins.  Si tout fonctionne, la page devrait indiquer « L'agent est connecté ».

## Installation d'un agent Jenkins sous Windows

Vous pouvez installer un agent Jenkins sous Windows à l'aide de la ligne de commande. Dans cette vidéo, Darin passe en revue la configuration et l'installation de l'agent Jenkins, y compris la création des fichiers nécessaires.

_Comment installer un agent Jenkins sous Windows._

<iframe width="800" height="420" src="https://www.youtube.com/embed/N8AQTlHoBKc" title="How to Install Jenkins Agent on Windows" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Création d'un agent macOS pour Jenkins

Cette vidéo passe en revue le processus de création d'un agent macOS pour Jenkins à l'aide de Java 11

<iframe width="800" height="420" src="https://www.youtube.com/embed/DteE1Zf8CIw" title="Setup a macOS Agent for Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
