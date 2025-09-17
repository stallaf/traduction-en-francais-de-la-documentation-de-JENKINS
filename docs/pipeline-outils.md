# Outils de développement de Pipelines

Jenkins Pipeline comprend une [documentation intégrée](./pipeline-commencer.md#documentation-intégrée) et le [générateur d'extraits](./pipeline-commencer.md#générateur-dextraits) qui sont des ressources clés lors du développement de Pipelines. Ils fournissent une aide détaillée et des informations personnalisées à la version actuellement installée de Jenkins et des plugins connexes. Dans cette section, nous discuterons d'autres outils et ressources qui pourraient aider au développement de Pipelines Jenkins.

## Linter de Pipeline en ligne de commande

Jenkins peut valider, ou « _lint_ », un Pipeline déclaratif à partir de la ligne de commande avant de l'exécuter. Cela peut être fait à l'aide d'une commande CLI Jenkins ou en effectuant une requête HTTP POST avec les paramètres appropriés. Nous recommandons d'utiliser [l'interface SSH](./gestion-cli.md#ssh) pour exécuter le linter. Consultez la [documentation CLI Jenkins](./gestion-cli.md) pour plus de détails sur la configuration correcte de Jenkins pour un accès sécurisé à la ligne de commande.

``` bash title="<i>Linting via la CLI avec SSH</i>"
# ssh (Jenkins CLI)
# JENKINS_PORT=[sshd port on controller]
# JENKINS_HOST=[Jenkins controller hostname]
ssh -p $JENKINS_PORT $JENKINS_HOST declarative-linter < Jenkinsfile
```

``` bash title="<i>Linting via HTTP POST grâce à curl</i>"
# curl (REST API)
# Ces instructions partent du principe que le domaine de sécurité de Jenkins
# n'est pas « Aucun » et que vous disposez d'un compte.
# JENKINS_URL=[URL racine du contrôleur Jenkins]
# JENKINS_AUTH=[votre nom d'utilisateur Jenkins et un jeton API dans le
# format : votre_nom_d'utilisateur:jeton_API]
curl -X POST --user "$JENKINS_AUTH" -F "jenkinsfile=<Jenkinsfile" "$JENKINS_URL/pipeline-model-converter/validate"
```

### Exemples

Vous trouverez ci-dessous deux exemples du Pipeline linter en action. Le premier montre le résultat obtenu lorsque le linter reçoit un fichier `Jenkinsfile` non valide, dans lequel une partie de la déclaration de `agent` est manquante.

``` groovy title="<i>Jenkinsfile</i>"
pipeline {
  agent
  stages {
    stage ('Initialize') {
      steps {
        echo 'Placeholder.'
      }
    }
  }
}
```

``` bash title="<i>Sortie Linter pour Jenkinsfile non valide</i>"
# transmettre un fichier Jenkinsfile qui ne contient pas de section « agent »
ssh -p 8675 localhost declarative-linter < ./Jenkinsfile
Errors encountered validating Jenkinsfile:
WorkflowScript: 2: Définition de section non valide : « agent ». Une configuration supplémentaire est nécessaire. @ ligne 2, colonne 3.
     agent
     ^

WorkflowScript: 1: Section obligatoire « agent » manquante à la ligne 1, colonne 1.
   pipeline &#125;
   ^
```

Dans ce deuxième exemple, le `Jenkinsfile` a été mis à jour pour inclure l'agent manquant. Le Linter rapporte désormais que le Pipeline est valide.

``` groovy title="<i>Jenkinsfile</i>"
pipeline {
  agent any
  stages {
    stage ('Initialize') {
      steps {
        echo 'Placeholder.'
      }
    }
  }
}
```

``` bash title="<i>Sortie Linter pour Jenkinsfile valide</i>"
ssh -p 8675 localhost declarative-linter < ./Jenkinsfile
Jenkinsfile validé avec succès.
```

## Blue Ocean Editor

[L'éditeur Blue Ocean Pipeline](./blueocean-editeur-pipeline.md) fournit un moyen [WYSIWYG](https://en.wikipedia.org/wiki/WYSIWYG)  de créer des Pipelines déclaratifs. L'éditeur offre une vue structurelle de toutes les étapes, des branches parallèles et des étapes d'un Pipeline. L'éditeur valide les modifications au fur et à mesure, éliminant de nombreuses erreurs avant même qu'elles ne soient engagées. Dans les coulisses, il génère toujours du code de Pipeline déclaratif.

!!! note "<i>Statut de l'océan bleu</i>"
    Blue Ocean ne recevra pas d'autres mises à jour de fonctionnalité. Blue Ocean continuera de fournir une visualisation de Pipeline facile à utiliser, mais il ne sera pas amélioré davantage. Il ne recevra que des mises à jour sélectives pour des problèmes de sécurité importants ou des défauts fonctionnels.

    Le [générateur d'extrait de syntaxe Pipeline](./pipeline-commencer.md#générateur-dextraits) aide les utilisateurs lorsqu'ils définissent les étapes du Pipeline avec leurs arguments. Il s'agit de l'outil préféré pour la création de Pipelines Jenkins, car il fournit une aide en ligne pour les étapes Pipeline disponibles dans votre contrôleur Jenkins. Il utilise les plugins installés sur votre contrôleur pour générer la syntaxe du Pipeline. Reportez-vous à la page de [référence des étapes du Pipeline](https://www.jenkins.io/doc/pipeline/steps/) pour plus d'informations sur toutes les étapes Pipeline disponibles.

## Le Pipeline "Replay" fonctionne avec des modifications

En règle générale, un Pipeline sera défini à l'intérieur de l'interface utilisateur classique de Jenkins, ou en s'engageant dans un `Jenkinsfile` dans le contrôleur de source. Malheureusement, aucune approche n'est idéale pour une itération rapide ou un prototypage d'un Pipeline. La fonction "Replay" permet des modifications rapides et l'exécution d'un Pipeline existant sans en modifier la configuration ou créer un nouveau _commit_.

### Usage

Pour utiliser la fonction "Replay" : 

1. Sélectionnez une exécution précédemment terminée dans l'historique de construction. 
![Section historique de la construction d'un Pipeline Jenkins montrant toutes les versions précédemment exécutées. L'historique comprend: Build #2 et Build #3, qui a fonctionné avec succès ; construire #4 et construire #5, qui a échoué ; et construire #6, qui a fonctionné avec succès. Cliquez sur la dernière version réussie, qui est la construction n °6 dans ce cas.](https://www.jenkins.io/doc/book/resources/pipeline/replay-previous-run.png)
2. Cliquez sur "Replay" dans le menu de gauche. 
![Barre latérale de l'interface utilisateur Jenkins Classic pour la construction "Exemple-pipeline" #6 Affichage des options : statut, modifications, sortie de la console, modification des informations de construction, supprimer la construction #6, calendrier, aperçu du Pipeline, console de Pipeline, redémarrer à partir de la scène, de la relecture, des étapes Pipeline, des espaces de travail et une version précédente.](https://www.jenkins.io/doc/book/resources/pipeline/replay-left-bar.png)
3. Faire des modifications et cliquez sur "Exécuter". Dans cet exemple, nous avons changé "Ruby-2.3" en "Ruby-2.4". 
![La section de relecture de l'exemple-pipeline de Jenkins pour la construction #6 affiche le script de Pipeline, où la version Ruby a été changée de 2,3 à 2,4. Une option «Syntaxe de Pipeline» et un bouton bleu «Exécuter» est affiché sous le script.](https://www.jenkins.io/doc/book/resources/pipeline/replay-modified.png)
4. Vérifiez les résultats des modifications.

Une fois que vous êtes satisfait des modifications, vous pouvez utiliser _Replay_ pour les visualiser à nouveau, les recopier dans votre tâche Pipeline ou votre fichier `Jenkinsfile`, puis les valider à l'aide de vos processus d'ingénierie habituels.

### Caractéristiques 

* **_Can be called multiple times on the same run_** (Peut être appelé plusieurs fois sur la même exécution) - permet des tests parallèles faciles de différentes modifications. 
* **_Can also be called on Pipeline runs that are still in-progress_** (Peut également être appelé sur des cycles de Pipeline qui sont toujours en cours) - tant qu'un Pipeline contenait du Groovy syntaxiquement correct et a pu commencer, il peut être rejoué. 
* **_Referenced Shared Library code is also modifiable_** (Le code de bibliothèque partagé référencé est également modifiable) - si une exécution de Pipeline fait référence à une bibliothèque partagée, le code de la bibliothèque partagée sera également affiché et modifiable dans la page Replay.
* **_Access Control via dedicated "Run / Replay" permission_** (Contrôle d'accès via l'autorisation dédiée "Exécuter/Replay") - impliqué par "Job/Configure". Si le Pipeline n'est pas configurable (par exemple, le Pipeline de branche d'un multibranche) ou «travail/configurer» n'est pas accordé, les utilisateurs peuvent toujours expérimenter la définition du Pipeline via la relecture 
* **_Can be used for Re-run_** (Peut être utilisé pour rediffuser) - les utilisateurs manquant de "Run/Replay" mais qui obtiennent "Job/Build" peuvent toujours utiliser la relecture pour exécuter une construction à nouveau avec la même définition. **Remarque :** l'étiquette passe à "_Rebuild_" (Reconstruire) dans ce cas.

### Limites 

* **_Pipeline runs with syntax errors cannot be replayed_** (Les Pipelines comportant des erreurs de syntaxe ne peuvent pas être rejoués), ce qui signifie que leur code ne peut pas être consulté et que les modifications qui y ont été apportées ne peuvent pas être récupérées. Lorsque vous utilisez **Replay** pour des modifications plus importantes, enregistrez vos modifications dans un fichier ou un éditeur en dehors de Jenkins avant de les exécuter. Voir J[ENKINS-37589](https://issues.jenkins.io/browse/JENKINS-37589.)
* **_Replayed Pipeline behavior may differ from runs started by other methods_** (Le comportement des Pipelines rejoués peut différer de celui des exécutions lancées par d'autres méthodes). Pour les Pipelines qui ne font pas partie d'un Pipeline multibranches, les informations de validation peuvent différer entre l'exécution d'origine et l'exécution rejouée. Voir [JENKINS-36453](https://issues.jenkins.io/browse/JENKINS-36453).

## Integrations IDE

### Éditeur d'Eclipse Jenkins

Le plugin `Jenkins Editor` Eclipse est disponible sur [Eclipse Marketplace](https://marketplace.eclipse.org/content/jenkins-editor). Cet éditeur de texte spécial offre certaines fonctionnalités permettant de définir des Pipelines, par exemple :

* Validation des scripts Pipeline par [Jenkins Linter Validation](./pipeline-developpement.md#linter). Les échecs sont signalés par des marqueurs Eclipse ;
* Une structure avec des icônes dédiées (pour les Pipelines Jenkins déclaratifs) ;
* Mise en évidence de la syntaxe et des mots-clés ;
* Validation Groovy.

!!! note
    Le plugin Jenkins Editor est un outil tiers qui n'est pas pris en charge par le projet Jenkins. 

### Connecteur Linter Jenkins Pipeline pour Visual Studio Code

L'extension `Jenkins Pipeline Linter Connector` pour [VisualStudio Code](https://code.visualstudio.com/) prend le fichier que vous avez actuellement ouvert, le transfère vers votre serveur Jenkins et affiche le résultat de la validation dans VS Code.

Vous trouverez cette extension dans le navigateur d'extensions VS Code ou à l'adresse suivante : [marketplace.visualstudio.com/items?itemName=janjoerke.jenkins-pipeline-linter-connector](https://marketplace.visualstudio.com/items?itemName=janjoerke.jenkins-pipeline-linter-connector).

L'extension ajoute quatre entrées de paramètres à VS Code qui permettent de sélectionner le serveur Jenkins que vous souhaitez utiliser pour la validation.

### Plugin Neovim nvim-jenkinsfile-linter

Le plugin Neovim [nvim-jenkinsfile-linter](https://github.com/ckipp01/nvim-jenkinsfile-linter) vous permet de valider un fichier Jenkinsfile à l'aide de l'API Pipeline Linter de votre contrôleur Jenkins et de signaler tout diagnostic existant dans votre éditeur.

### Package Atom linter-jenkins

Le package Atom [linter-jenkins](https://atom.io/packages/linter-jenkins) vous permet de valider un fichier Jenkins à l'aide de l'API Pipeline Linter d'un Jenkins en cours d'exécution. Vous pouvez l'installer directement à partir du gestionnaire de packages Atom. Il nécessite également l'installation du [support du langage Jenkinsfile dans Atom](https://atom.io/packages/language-jenkinsfile).

### Package Jenkinsfile pour Sublime Text

Le package [Jenkinsfile](https://github.com/june07/sublime-Jenkinsfile) pour Sublime Text vous permet de valider un fichier Jenkinsfile à l'aide de l'API Pipeline Linter d'un contrôleur Jenkins en cours d'exécution via un canal sécurisé (SSH). Vous pouvez l'installer directement à partir du gestionnaire de packages Sublime Text.

Vous trouverez le package dans l'interface Sublime Text via le package Package Control, sur GitHub ou sur packagecontrol.io :

* [https://github.com/june07/sublime-Jenkinsfile](https://github.com/june07/sublime-Jenkinsfile) ;
* [https://packagecontrol.io/packages/Jenkinsfile](https://packagecontrol.io/packages/Jenkinsfile).

### Cadre de test unitaire Pipeline

Le [Pipeline Unit Testing Framework](https://github.com/jenkinsci/JenkinsPipelineUnit) vous permet de [tester](https://en.wikipedia.org/wiki/Unit_testing) les Pipelines et les [bibliothèques partagées](./pipeline-bibliotheques-partagees.md) avant de les exécuter dans leur intégralité. Il fournit un environnement d'exécution simulé dans lequel les étapes réelles du Pipeline sont remplacées par des objets simulés que vous pouvez utiliser pour vérifier le comportement attendu. Nouveau et encore perfectible, mais prometteur. Le fichier [README](https://github.com/jenkinsci/JenkinsPipelineUnit/blob/master/README.md) de ce projet contient des exemples et des instructions d'utilisation.
