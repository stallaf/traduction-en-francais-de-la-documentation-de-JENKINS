# Pipeline

Ce chapitre couvre tous les aspects recommandés de la fonctionnalité du Pipeline Jenkins, y compris comment : 

* [Commencez avec Pipeline](./Pipeline-commencer.md), qui explique comment [définir un Pipeline Jenkins](./Pipeline-commencer.md#definir-un-Pipeline) (votre Pipeline) via [Blue Ocean](./Pipeline-commencer.md#via-blue-ocean), via [l'interface utilisateur](./Pipeline-commencer.md#via-interface-utilisateur) classique ou dans [SCM](./Pipeline-commencer.md#definir-un-Pipeline-dans-scm) ;
* [Créez et utilisez un fichier Jenkinsfile](./Pipeline-jenkinsfile.md), qui couvre des scénarios d'utilisation sur la manière de créer et de construire votre fichier `Jenkinsfile` ;
* Travaillez avec des [branches et des _pull requests_](./Pipeline-multibranches.md) ;
* [Utilisez Docker avec Pipeline](./Pipeline-docker.md), qui explique comment Jenkins peut invoquer des conteneurs Docker sur des agents/nœuds (à partir d'un fichier `Jenkinsfile`) pour construire vos projets Pipeline ;
* [Étendez Pipeline avec des bibliothèques partagées](./Pipeline-bibliotheques-partagees.md) ;
* Utilisez différents [outils de développement](./Pipeline-developpement.md) pour faciliter la création de votre Pipeline ;
* Travaillez avec la [syntaxe Pipeline](.:Pipeline-syntaxe.md), qui fournit une référence complète de toute la syntaxe declarative Pipeline.

Pour un aperçu du contenu dans le manuel de l'utilisateur Jenkins, reportez-vous à la [vue d'ensemble du manuel de l'utilisateur](./Pipeline-commencer.md).

## Qu'est-ce que Jenkins Pipeline ?

Jenkins Pipeline (ou simplement "Pipeline" avec un "P" en capital) est une suite de plugins qui prend en charge la mise en œuvre et l'intégration de _Pipelines de livraison continue_ dans Jenkins.

Un _Pipeline de livraison continu (CD)_ est une expression automatisée de votre processus permettant de faire passer un logiciel du contrôle de version à vos utilisateurs et clients. Chaque modification apportée à votre logiciel (validée dans le contrôle de source) passe par un processus complexe avant d'être publiée. Ce processus implique de créer le logiciel de manière fiable et reproductible, ainsi que de faire passer le logiciel créé (appelé « build ») par plusieurs étapes de test et de déploiement.

Pipeline fournit un ensemble extensible d'outils pour modéliser les Pipelines de livraison simples à complex "comme code" via la  [syntaxe du langage spécifique au domaine du Pipeline (DSL)](./Pipeline-syntaxe.md).(1)
{ .annotate }

1.  [Langage spécifique à un domaine](https://en.wikipedia.org/wiki/Domain-specific_language)

La définition d'un Pipeline Jenkins est écrite dans un fichier texte (appelé [JenkinsFile](./Pipeline-jenkinsfile)) qui à son tour peut être engagé dans le référentiel de contrôle source d'un projet.(1) Ceci est le fondement de "Pipeline en tant que code" ; traiter le Pipeline CD comme faisant partie de l'application à verser et examiné comme tout autre code.
{ .annotate }

1.  [Gestion du contrôle des sources](https://en.wikipedia.org/wiki/Version_control)

La création d'un fichier `Jenkinsfile` et son intégration dans le contrôle de source offrent plusieurs avantages immédiats :

* Crée automatiquement un processus de construction Pipeline pour toutes les branches et toutes les demandes d'extraction ;
* Révision/itération du code sur le Pipeline (avec le reste du code source) ;
* Piste d'audit pour le Pipeline ;
* Source unique de vérité (1) pour le Pipeline, qui peut être consultée et modifiée par plusieurs membres du projet. 
    { .annotate }

    1.  [Source unique de vérité](https://en.wikipedia.org/wiki/Single_source_of_truth)

Bien que la syntaxe de définir un Pipeline, soit dans l'interface utilisateur Web, soit avec un `Jenkinsfile`, est la même, il est généralement considéré comme la meilleure pratique de définir le Pipeline dans un `JenkinsFile` et de vérifier cela au contrôle de la source.

### Syntaxe de Pipeline déclaratif contre scénarisé

Un `JenkinsFile` peut être écrit à l'aide de deux types de syntaxe - Déclarative et Scriptée.

Les Pipelines déclaratifs et scriptés sont construits fondamentalement différemment. Le Pipeline déclaratif est conçu pour faciliter l'écriture et la lecture du code du Pipeline, et fournit des fonctionnalités syntaxiques plus riches par rapport à la syntaxe des Pipelines scriptés.
Cependant, beaucoup de composants syntaxiques individuels (ou "étapes") écrits dans un `JenkinsFile` sont communs au Pipeline déclaratif et scripté. En savoir plus sur la façon dont ces deux types de syntaxe diffèrent dans les [concepts de Pipeline](./Pipeline-concept.md) et la [vue d'ensemble de la syntaxe des Pipelines](./Pipeline-apercu-syntaxe.md) ci-dessous.

## Pourquoi Pipeline ?

Jenkins est, fondamentalement, un moteur d'automatisation qui prend en charge un certain nombre de modèles d'automatisation. Pipeline ajoute un ensemble puissant d'outils d'automatisation sur Jenkins, prenant en charge les cas d'utilisation qui s'étendent d'une simple intégration continue aux Pipelines CD complets. En modélisant une série de tâches connexes, les utilisateurs peuvent profiter des nombreuses fonctionnalités du Pipeline : 

* **Code** : les Pipelines sont implémentés dans le code et généralement enregistrés dans le contrôle de source, ce qui permet aux équipes de modifier, réviser et itérer leur Pipeline de livraison.
* **Durables** : les Pipelines peuvent survivre aux redémarrages planifiés et imprévus du contrôleur Jenkins.
* **Susceptibles d'être mis en pause** : les Pipelines peuvent être arrêtés et attendre une intervention humaine ou une approbation avant de poursuivre leur exécution.
* **Polyvalent** : les Pipelines prennent en charge les exigences complexes de la CD dans le monde réel, notamment la possibilité de bifurquer/rejoindre, de boucler et d'effectuer des tâches en parallèle.
* **Extensible** : le plugin Pipeline prend en charge des extensions personnalisées de son DSL (1) et de multiples options d'intégration avec d'autres plugins.
    { .annotate }

    1.  [Langage spécifique à un domaine](https://en.wikipedia.org/wiki/Domain-specific_language)

Alors que Jenkins a toujours permis des formes rudimentaires d'enchaînement de tâches Freestyle pour effectuer des tâches séquentielles, (1) Pipeline fait de ce concept un élément central dans Jenkins.
{ .annotate }

1.  Des plugins supplémentaires ont été utilisés pour implémenter des comportements complexes à l'aide de Freestyle Jobs, tels que les plugins Copy Artifact, Parameterized Trigger et Promoted Builds.

_Quelle est la différence entre le freestyle et le Pipeline à Jenkins_
<iframe width="800" height="420" src="https://www.youtube.com/embed/IOUm1lw7F58" title="What Is The Difference Between Freestyle and Pipeline in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

S'appuyant sur la valeur fondamentale d'extensibilité de Jenkins, Pipeline est également extensible à la fois par les utilisateurs grâce aux bibliothèques partagées Pipeline et par les développeurs de plugins. (1) 
{ .annotate }

1. [Plugin GitHub Branch Source](https://plugins.jenkins.io/github-branch-source)

L'organigramme ci-dessous est un exemple d'un scénario de CD facilement modélisé dans le Pipeline Jenkins :

![Flux Pipeline](https://www.jenkins.io/doc/book/resources/Pipeline/realworld-Pipeline-flow.png)

## Concepts de Pipeline

Les concepts suivants sont des aspects clés du Pipeline Jenkins, qui se lient étroitement à la syntaxe des Pipelines (reportez-vous à [l'aperçu](https://www.jenkins.io/doc/book/Pipeline/#Pipeline-syntax-overview) ci-dessous).

### Pipeline

Un Pipeline est un modèle défini par l'utilisateur d'un Pipeline de CD. Le code d'un Pipeline définit l'ensemble de votre processus de construction, qui comprend généralement les étapes de construction d'une application, de test, puis de livraison.

De plus, un bloc de `Pipeline` est un [élément clé de la syntaxe déclarative des Pipelines](https://www.jenkins.io/doc/book/Pipeline/#declarative-Pipeline-fundamentals).

### Nœud

Un nœud est une machine qui fait partie de l'environnement Jenkins et est capable d'exécuter un Pipeline.

De plus, un bloc `node` est un [élément clé de la syntaxe des Pipelines scriptés](https://www.jenkins.io/doc/book/Pipeline/#scripted-Pipeline-fundamentals).

### Scène

Un bloc `stage` définit un sous-ensemble conceptuellement distinct de tâches effectuées tout au long du Pipeline (par exemple, les phases « Build », « Test » et « Deploy »), qui est utilisé par de nombreux plugins pour visualiser ou présenter l'état/la progression du Pipeline Jenkins. (1)
{ .annotate }

1. [Blue Ocean](https://www.jenkins.io/doc/book/blueocean), [Pipeline : plugin Stage View](https://plugins.jenkins.io/Pipeline-stage-view)

### Étape

C'est une seule tâche. Fondamentalement, une étape indique à Jenkins ce qu'il doit faire à un moment donné (ou « étape » du processus). Par exemple, pour exécuter la commande shell `make`, utilisez l'étape `sh` : `sh 'make'`. Lorsqu'un plugin étend le Pipeline DSL (1) , cela signifie généralement que le plugin a implémenté une nouvelle étape.
{ .annotate }

1. [Langage spécifique à un domaine](https://en.wikipedia.org/wiki/Domain-specific_language)

## Présentation de la syntaxe des Pipelines

Les squelettes de code Pipeline suivants illustrent les différences fondamentales entre la [syntaxe de Pipeline déclaratif](#principes-fondamentaux-syntaxe-Pipeline-declaratif)  et la [scripté](#principes-fondamentaux-syntaxe-Pipeline-scripté).

Sachez que les [stages](#etapes) et les [steps](#mesures) (ci-dessus) sont des éléments communs à la syntaxe déclarative et à la syntaxe de Pipeline scriptée.

## Principes fondamentaux du Pipeline déclaratif

Dans la syntaxe déclarative du Pipeline, le bloc `Pipeline` définit tout le travail effectué tout au long de votre Pipeline.

``` groovy title="Jenkinsfile (Pipeline déclaratif)"
Pipeline {
    agent any // (1)
    stages {
        stage('Build') { // (2) 
            steps {
                // (3)
            }
        }
        stage('Test') { // (4)
            steps {
                // (5)
            }
        }
        stage('Deploy') { // (6)
            steps {
                // (7)
            }
        }
    }
}
```

1.  Exécutez ce Pipeline ou l'une de ses étapes sur n'importe quel agent disponible.
2. Définit l'étape « Build ».
3. Effectuez certaines étapes liées à l'étape « Build ».
4.  Définit l'étape « Test ».
5.  Effectuez certaines étapes liées à l'étape « Test ».
6.  Définit l'étape « Deploy ».
7. Effectuez certaines étapes liées à l'étape « Deploy ».

### Principes fondamentaux du Pipeline scripté

Dans la syntaxe du Pipeline scripté, un ou plusieurs blocs `node` effectuent le travail de base tout au long du Pipeline. Bien que ce ne soit pas une exigence obligatoire de la syntaxe des Pipelines scriptées, la limite de travail de votre Pipeline à l'intérieur d'un bloc `node` fait deux choses : 

1. Planifie l'exécution des étapes contenues dans le bloc en ajoutant un élément à la file d'attente Jenkins. Dès qu'un exécuteur est disponible sur un nœud, les étapes s'exécutent. 
2. Crée un espace de travail (un répertoire spécifique à ce Pipeline particulier) où le travail peut être effectué sur des fichiers vérifiés à partir du contrôle source. 
**Attention :** selon votre configuration Jenkins, certains espaces de travail peuvent ne pas être automatiquement nettoyés après une période d'inactivité. Pour plus d'informations, consultez les tickets et la discussion liés à [JENKINS-2111](https://issues.jenkins.io/browse/JENKINS-2111).

``` groovy title="Jenkinsfile (Pipeline scénarisé)"
node {  //(1)
    stage('Build') { //(2)
        //(3)
    }
    stage('Test') { //(4)
        //(5)
    }
    stage('Deploy') { //(6)
        //(7)
    }
}
```

1. Exécutez ce Pipeline ou l'une de ses étapes sur n'importe quel agent disponible.
2. Définit l'étape « Build ». Les blocs d'étape sont facultatifs dans la syntaxe Scripted Pipeline. Cependant, l'implémentation de blocs d'étape dans un Scripted Pipeline permet une visualisation plus claire du sous-ensemble de tâches/étapes de chaque `stage` dans l'interface utilisateur Jenkins.
2. Effectuez certaines étapes liées à l'étape « Build ».
3. Définit l'étape « Test ».
4. Effectuez certaines étapes liées à l'étape « Test ».
5. Définit l'étape « Deploy ».
6. Effectuez certaines étapes liées à l'étape « Deploy ».

## Exemple de Pipeline

Voici un exemple de fichier `Jenkinsfile` utilisant la syntaxe Declarative Pipeline. Son équivalent en syntaxe scripté est accessible en cliquant sur le lien **Toggle Scripted Pipeline** ci-dessous :

``` groovy title="Jenkinsfile (Pipeline déclaratif)"
Pipeline { //(1)
    agent any //(2)
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') { //(3)
            steps { //(4)
                sh 'make' //(5)
            }
        }
        stage('Test'){
            steps {
                sh 'make check'
                junit 'reports/**/*.xml' //(6)
            }
        }
        stage('Deploy') {
            steps {
                sh 'make publish' //
            }
        }
    }
}
```

1. [Pipeline](#syntaxe-de-Pipeline-déclaratif) est une syntaxe spécifique au Pipeline déclaratif qui définit un « bloc » contenant tout le contenu et toutes les instructions nécessaires à l'exécution de l'ensemble du Pipeline.
2. [agent](#syntaxe-agent) est une syntaxe spécifique au Pipeline déclaratif qui demande à Jenkins d'allouer un exécuteur (sur un nœud) et un espace de travail pour l'ensemble du Pipeline.
3. `stage` est un bloc syntaxique qui [décrit une étape de ce Pipeline](#etape). Pour en savoir plus sur les blocs `stage` dans la syntaxe du Pipeline déclaratif, consultez la page [Syntaxe du Pipeline](./pipeline-syntaxe.md#stage). Comme mentionné [ci-dessus](#principes-fondamentaux-du-pipeline-scripté), les blocs `stage` sont facultatifs dans la syntaxe du Pipeline scripté.
4. [steps](./pipeline-syntaxe#mesures.md) est une syntaxe spécifique au Pipeline déclaratif qui décrit les étapes à exécuter dans ce `stage`.
5. `sh` est un [step](./pipeline-syntaxe#mesures.md) du Pipeline (fournie par le [plugin Pipeline : Nodes and Processes](https://plugins.jenkins.io/workflow-durable-task-step)) qui exécute la commande shell donnée.
6. `junit` est un autre [step](./pipeline-syntaxe#mesures.md) du Pipeline (fournie par le [plugin JUnit](https://plugins.jenkins.io/junit)) permettant d'agréger les rapports de test.

En savoir plus sur la syntaxe Pipeline sur la [page dédiée](./pipeline-syntaxe.md).