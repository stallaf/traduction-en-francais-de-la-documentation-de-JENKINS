# Pipeline en tant que code

<div class="couleur-introduction">
<i>Pipeline en tant que code</i> décrit un ensemble de fonctionnalités qui permettent aux utilisateurs de Jenkins de définir des processus de travail pipelinés avec du code, stockés et versés dans un référentiel source. Ces fonctionnalités permettent à Jenkins de découvrir, de gérer et d'exécuter des travaux pour plusieurs référentiels et succursales de source - éliminant le besoin de création et de gestion manuelles.
</div>

Pour utiliser <i>Pipeline comme code</i>, les projets doivent contenir un fichier nommé `Jenkinsfile` dans la racine du référentiel, qui contient un "script Pipeline".

De plus, l'une des tâches doit être configurée dans Jenkins :

* _Pipeline multibranches_ : crée automatiquement plusieurs branches d'un seul référentiel ;
* _Dossiers d'organisation_ : analyse une **organisation GitHub** ou une **équipe Bitbucket** pour découvrir les référentiels d'une organisation, créant automatiquement des tâches de _Pipeline multibranches_ gérées pour ceux-ci ;
* _Pipeline_ : les tâches de Pipeline régulières disposent d'une option permettant de spécifier le Pipeline « Utiliser SCM ».

Fondamentalement, les référentiels d'une organisation peuvent être considérés comme une hiérarchie, où chaque référentiel peut avoir des éléments enfants de branches et de _pull requests_.

``` cfg title="<i>Exemple de structure de référentiel</i>"
+--- GitHub Organization
    +--- Project 1
        +--- master
        +--- feature-branch-a
        +--- feature-branch-b
    +--- Project 2
        +--- master
        +--- pull-request-1
        +--- etc...
```

!!! info
    Avant les tâches _Pipeline Multibranche_ et les _Dossiers Organisation_,
    `plugin:cloudbees-folder[Folders]` pouvait être utilisé pour créer cette hiérarchie dans Jenkins en organisant les référentiels en dossiers contenant des tâches pour chaque branche individuelle.

 _Pipeline multibranche_ et les _dossiers d'organisation_ éliminent le processus manuel en détectant respectivement les branches et les référentiels, et la création de dossiers appropriés avec des travaux dans Jenkins automatiquement.

## Le fichier Jenkinsfile

La présence du fichier `Jenkinsfile` à la racine d'un référentiel permet à Jenkins de gérer et d'exécuter automatiquement des tâches en fonction des branches du référentiel.

Le fichier `Jenkinsfile` doit contenir un script Pipeline, spécifiant les étapes à suivre pour exécuter la tâche. Le script dispose de toutes les fonctionnalités de Pipeline, depuis quelque chose d'aussi simple que l'appel d'un générateur Maven jusqu'à une série d'étapes interdépendantes, qui ont une exécution parallèle coordonnée avec des phases de déploiement et de validation.

Une façon simple de se lancer avec Pipeline consiste à utiliser le _générateur d'extraits_ disponible dans l'écran de configuration d'une tâche Jenkins _Pipeline_. À l'aide du _générateur d'extraits_, vous pouvez créer un script Pipeline comme vous le feriez via les menus déroulants dans d'autres tâches Jenkins.

## Pipeline multibranches et dossier d'organisation

Les projets de **Pipeline multibranches** et les **dossiers d'organisation GitHub** étendent les fonctionnalités des dossiers Jenkins en introduisant des dossiers calculés, qui gèrent automatiquement leur contenu en créant et en mettant à jour dynamiquement des éléments enfants. Ces fonctionnalités nécessitent l'installation du [plugin GitHub Branch Source](https://plugins.jenkins.io/github-branch-source), car celui-ci fournit les fonctionnalités nécessaires à la gestion des branches et des référentiels.

* **Projets de Pipeline multibranches** : la fonctionnalité de dossier calculé comprend les options **Scan Multibranch Pipeline Log** (Analyser le Journal du Pipeline Multibranches) et **Scan Multibranch Pipeline Now** (Analyser le Pipeline Multibranches Maintenant), qui gèrent dynamiquement les éléments du Pipeline. Ces options permettent à Jenkins d'analyser les référentiels, de générer des éléments enfants pour les branches éligibles et de mettre à jour automatiquement la liste des Pipelines en fonction des modifications.
* **Projets d'Organisation GitHub** : la fonctionnalité de dossier calculé comprend les options **Scan Organization Log** (Analyser le Journal de l'Organisation) et **Scan Organization Now** (Analyser l'Organisation Maintenant). Cette fonctionnalité remplit les référentiels en tant que Pipelines multibranches individuels et fournit des informations sur les analyses des branches et des référentiels.

Les analyses de dossiers peuvent être déclenchées automatiquement via des rappels webhook chaque fois que des branches ou des référentiels sont créés ou supprimés. Les analyses peuvent également être configurées pour s'exécuter périodiquement via les paramètres **Build Triggers** (Déclencheurs de Compilation), qui sont réglés par défaut sur une nouvelle analyse après un jour d'inactivité.

![Page de configuration du projet « CloudBeersInc », de type Dossier de l'organisation GitHub. Dans la section « Déclencheurs d'analyse de l'organisation », l'option « Périodiquement si aucune autre analyse n'est effectuée » est cochée et l'intervalle est défini sur « 1 jour ».](https://www.jenkins.io/doc/book/resources/pipeline-as-code/github-organization-build-triggers-settings.png)

Le journal de la dernière tentative d'analyse de l'organisation est disponible dans le **Scan Organization Log** (Journal d'Analyse de l'Organisation). Si l'analyse ne produit pas l'ensemble de référentiels attendu, le journal peut contenir des informations utiles pour aider à diagnostiquer le problème.

![La page « Scan Organization Log » (Journal d'analyse de l'organisation) dans Jenkins pour le projet « CloudBeersInc », de type GitHub Organization Folder, affiche la progression et les résultats de l'analyse. Le panneau de gauche contient diverses options de navigation, la section « Scan Organization Log » (Journal d'analyse de l'organisation) étant sélectionnée. Le journal indique que l'analyse a été lancée par l'utilisateur admin, ainsi que l'heure et la date. Il fournit des détails sur la progression de l'analyse, notamment la consultation de GitHub pour le référentiel CloudBeersInc/community-docs et le traitement de la branche principale.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/scan-organization-log.png)

## Configuration

Les projets _Pipelines Multibranches_ et les _Dossiers d'Organisation_ disposent tous deux d'options de configuration permettant une sélection précise des référentiels. Ces fonctionnalités permettent également de sélectionner deux types d'informations d'identification à utiliser lors de la connexion aux systèmes distants :

* _scan_ des informations d'identification, qui sont utilisées pour accéder aux API GitHub ou Bitbucket ;
* _checkout_ des informations d'identification, qui sont utilisées lorsque le référentiel est cloné à partir du système distant ; il peut être utile de choisir une clé SSH ou « - anonymous - », qui utilise les informations d'identification par défaut configurées pour l'utilisateur du système d'exploitation

!!! warning
	Si vous utilisez une _organisation GitHub_, vous devez [créer un jeton d'accès GitHub](https://github.com/settings/tokens/new?scopes=repo,public_repo,admin:repo_hook,admin:org_hook&description=Jenkins+Access) à utiliser pour éviter de stocker votre mot de passe dans Jenkins et prévenir tout problème lors de l'utilisation de l'API GitHub. Lorsque vous utilisez un jeton d'accès GitHub, vous devez utiliser un _nom d'utilisateur_ standard avec des informations d'identification par _mot de passe_, où le nom d'utilisateur est le même que votre nom d'utilisateur GitHub et le mot de passe est votre jeton d'accès.

### Projets Pipeline multibranches

Les projets _Pipelines Multibranches_ sont l'une des fonctionnalités fondamentales du _Pipeline as Code_. Les modifications apportées à la procédure de compilation ou de déploiement peuvent évoluer en fonction des exigences du projet et la tâche reflète toujours l'état actuel du projet. Cela vous permet également de configurer différentes tâches pour différentes branches d'un même projet, ou de renoncer à une tâche si nécessaire. Le fichier `Jenkinsfile` situé dans le répertoire racine d'une branche ou d'une _pull request_ identifie un projet multibranches.

!!! info
	Les projets de _Pipelines Multibranches_ exposent le nom de la branche en cours de construction avec la variable d'environnement `BRANCH_NAME` et fournissent une commande spéciale `checkout scm` Pipeline, qui garantit la vérification du commit spécifique à l'origine du fichier Jenkinsfile. Si le fichier Jenkinsfile doit vérifier le référentiel pour une raison quelconque, veillez à utiliser `checkout scm`, car il prend également en compte les référentiels d'origine alternatifs pour gérer des éléments tels que les _pull requests_. 

Pour créer un _Pipeline multibranche_, accédez à : _New Item → Multibranch Pipeline_ (Nouvel élément → Pipeline multibranches). Configurez la source SCM comme il convient. Il existe des options pour de nombreux types de référentiels et de services différents, notamment Git, Mercurial, Bitbucket et GitHub. Si vous utilisez GitHub, par exemple, cliquez sur **Add source** (Ajouter une source), sélectionnez GitHub et configurez le propriétaire, les informations d'identification de scan et le référentiel appropriés.

Les autres options disponibles pour les projets de _Pipeline Multibranches_ sont les suivantes :

* **API endpoint** (Point de terminaison API) : un point de terminaison API alternatif pour utiliser un GitHub Enterprise auto-hébergé ;
* **Checkout credentials** (Informations d'identification de vérification) : informations d'identification alternatives à utiliser lors de la vérification du code (clonage) ;
* **Include branches** (Inclure les branches) : expression régulière permettant de spécifier les branches à inclure ;
* **Exclude branches** (Exclure les branches) : expression régulière permettant de spécifier les branches à exclure ; notez que cela aura priorité sur les inclusions ;
* **Property strategy** (Stratégie de propriété) : si nécessaire, définissez des propriétés personnalisées pour chaque branche.

Après avoir configuré ces éléments et enregistré la configuration, Jenkins analysera automatiquement le référentiel et importera les branches appropriées.

### Dossiers d'organisation

Les dossiers d'organisation constituent un moyen pratique de permettre à Jenkins de gérer automatiquement les référentiels qui sont automatiquement inclus dans Jenkins. En particulier, si votre organisation utilise _GitHub Organizations_ ou _Bitbucket Teams_, chaque fois qu'un développeur crée un nouveau référentiel avec un fichier `Jenkinsfile`, Jenkins le détecte automatiquement et crée un projet _Pipeline Multibranche_ pour celui-ci. Cela évite aux administrateurs ou aux développeurs d'avoir à créer manuellement des projets pour ces nouveaux référentiels.

Pour créer un _dossier d'organisation_ dans Jenkins, accédez à : **New Item → Organization Folder** (Nouvel élément → Dossier d'organisation) ou **New Item → Bitbucket Team** (Nouvel élément → Équipe Bitbucket) et suivez les étapes de configuration pour chaque élément, en veillant à spécifier les informations d'identification de scan appropriées et un **propriétaire** spécifique pour le nom de l'organisation GitHub ou de l'équipe Bitbucket, respectivement.

Les autres options disponibles sont les suivantes :

* **Repository name pattern** (Modèle de nom de référentiel) : expression régulière permettant de spécifier les référentiels inclus ;
* **API endpoint ** (Point de terminaison API) : point de terminaison API alternatif pour utiliser un GitHub Enterprise auto-hébergé ;
* **Checkout credentials** (Identifiants de vérification) : identifiants alternatifs à utiliser lors de la vérification du code (clonage).

Une fois ces éléments configurés et la configuration enregistrée, Jenkins analysera automatiquement l'organisation et importera les référentiels appropriés et les branches résultantes.

### Stratégie relative aux éléments orphelins

Les dossiers calculés peuvent supprimer immédiatement les éléments ou les conserver en fonction de la stratégie de conservation souhaitée. Par défaut, les éléments sont supprimés dès que le calcul du dossier détermine qu'ils ne sont plus présents. Si votre organisation a besoin que ces éléments restent disponibles pendant une période plus longue, il suffit de configurer la stratégie relative aux éléments orphelins de manière appropriée. Il peut être utile de conserver des éléments afin d'examiner les résultats de la construction d'une branche après sa suppression, par exemple :

![Section Stratégie relative aux éléments orphelins dans la page de configuration Jenkins pour le dossier de l'organisation, montrant l'option « Supprimer les anciens éléments » cochée, avec les champs « Nombre de jours de conservation des anciens éléments » et « Nombre maximal d'anciens éléments à conserver », tous deux avec des zones de saisie vides. La case « Abandonner les compilations » n'est pas cochée.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/orphaned-item-strategy.png)

### Icône et stratégie d'affichage

Vous pouvez également configurer une icône personnalisée pour l'affichage des dossiers en installant le plugin [Custom Folder Icon](https://plugins.jenkins.io/custom-folder-icon). Par exemple, il peut être utile d'afficher l'état global des builds enfants. Vous pouvez également faire référence à la même icône que celle que vous utilisez dans votre compte d'organisation GitHub.

![Section Apparence de la page de configuration Jenkins pour le dossier d'organisation, montrant le champ d'icône avec l'option « Icône de dossier personnalisée » sélectionnée. Cette section comprend une option permettant de choisir un fichier image pour l'icône, ainsi qu'un bouton Appliquer pour enregistrer vos modifications.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/folder-icon.png)

## Exemple

Pour illustrer l'utilisation d'un dossier d'organisation pour gérer les référentiels, nous utiliserons l'organisation fictive CloudBeers, Inc.

Allez dans **New Item** (Nouvel élément). Entrez « CloudBeersInc » comme nom d'élément. Sélectionnez **Organization Folde** (Dossier d'organisation) et cliquez sur **OK**.

![Page Nouvel élément Jenkins avec le nom d'élément « CloudBeersInc » et l'option « Dossier d'organisation » sélectionnée parmi les options de type d'élément suivantes : Projet Freestyle, Pipeline, Projet multi-configuration, Dossier, Pipeline multi-branche et Dossier d'organisation.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/organization-folder-creation.png)

Vous pouvez également saisir un nom plus descriptif dans le champ _Description_, tel que « CloudBeers GitHub ». Dans la section _Sources du référentiel_, remplissez la section « Organisation GitHub ». Assurez-vous que le **owner** (propriétaire) corresponde exactement au nom de l'organisation GitHub. Dans notre cas, il doit s'agir de : _CloudBeersInc_. La valeur par défaut est la même que celle saisie pour le nom de l'élément à la première étape. Ensuite, sélectionnez ou ajoutez de nouvelles **Credentials** (informations d'identification). Nous allons saisir notre nom d'utilisateur GitHub et notre jeton d'accès comme mot de passe.

![La section Projets de la page de configuration Jenkins pour une source de référentiel d'organisation GitHub comprend des champs permettant de spécifier le point de terminaison de l'API, les informations d'identification et le propriétaire du référentiel, qui est défini sur « CloudBeersInc ». De plus, il existe des options permettant d'activer l'affichage des avatars et de définir des comportements tels que la découverte de branches, les demandes d'extraction à partir de l'origine et les demandes d'extraction à partir de fourches. Chaque comportement dispose d'un menu déroulant de stratégie pour des configurations spécifiques, ainsi que d'options permettant de définir des niveaux de confiance pour les pull requests provenant de forks.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/github-configuration-for-organization-folder.png)

Après l'enregistrement, le « Folder Computation » (Calcul du Dossier) s'exécutera pour rechercher les référentiels éligibles, suivi des compilations multibranches.

![Vue Jenkins Build Queue (file d'attente de build) et Build Executor Status (état de l'exécuteur de build). La section Build Queue (file d'attente de build) affiche « No builds in the queue » (aucun build dans la file d'attente). La section Build Executor Status (état de l'exécuteur de build) affiche les builds en cours sous « Built-In Node » (nœud intégré) avec des indicateurs de progression. Trois tâches sont en cours d'exécution : « Référentiel PR-demo (branche principale) de l'organisation CloudBeersInc », « Référentiel community-docs (branche principale) de l'organisation CloudBeersInc » et « Référentiel multibranch-demo (branche principale) de l'organisation CloudBeersInc ». Chacune d'entre elles est accompagnée d'une barre de progression bleue. En dessous, « docker-ssh-jenkins-agent » est répertorié comme inactif avec un statut « 1 inactif » en bas.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/organization-repositories-scan.png
)

Actualisez la page après l'exécution de la tâche pour vous assurer que l'affichage des référentiels a été mis à jour.

![Vue du tableau de bord Jenkins pour l'organisation « CloudBeersInc ». La section Statut affiche le nombre de référentiels comme « Référentiels (3) » avec un tableau répertoriant trois référentiels : « community-docs », « multibranch-demo » et « PR-demo ». Chaque entrée de référentiel comprend des colonnes intitulées « S » pour le statut de la dernière compilation, « W » pour le statut météo des compilations agrégées récentes, « Nom » et « Description ». Le référentiel « multibranch-demo » contient une description indiquant « Démonstration simple de l'utilisation des pipelines multibranches ». Des options de taille d'icône (S, M, L) sont disponibles sous le tableau.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/organization-folder-repositories.png)

À ce stade, vous avez terminé la configuration de base du projet et pouvez désormais explorer vos référentiels importés. Vous pouvez également examiner les résultats des tâches exécutées dans le cadre du _Calcul initial du Dossier_.

![Vue du tableau de bord Jenkins pour une tâche nommée « PR-demo » dans le dossier de l'organisation « CloudBeersInc ». La page répertorie quatre branches portant les noms suivants : main, stephenc-patch-1, stephenc-patch-2 et stephenc-patch-3. Les branches main et stephenc-patch-1 indiquent des builds réussis avec une coche verte et une dernière heure de réussite de 2 min 39 s et 2 min 34 s respectivement. La branche stephenc-patch-2 a un statut instable indiqué par un point d'exclamation orange avec une dernière heure de réussite de 2 min 34 s, tandis que la branche stephenc-patch-3 a un statut d'échec indiqué par une croix rouge.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/PR-demo-branches.png)

## Livraison continue avec Pipeline

La livraison continue permet aux organisations de fournir des logiciels avec moins de risques. Le chemin vers la livraison continue commence par la modélisation du Pipeline de livraison de logiciels utilisé au sein de l'organisation, puis se concentre sur l'automatisation de l'ensemble du processus. Les retours d'information précoces et ciblés, rendus possibles par l'automatisation du Pipeline, permettent une livraison plus rapide des logiciels par rapport aux méthodes traditionnelles.

Jenkins est le couteau suisse de la chaîne d'outils de livraison de logiciels. Les développeurs et le personnel des opérations (DevOps) ont des mentalités différentes et utilisent des outils différents pour accomplir leurs tâches respectives. Comme Jenkins s'intègre à une grande variété d'outils, il sert de point de rencontre entre les équipes de développement et d'exploitation.

De nombreuses organisations orchestrent depuis plusieurs années des Pipelines à l'aide des plugins Jenkins existants. À mesure que leur automatisation se perfectionne et que leur expérience Jenkins s'enrichit, les organisations souhaitent inévitablement aller au-delà des Pipelines simples et créer des flux complexes spécifiques à leur processus de livraison.

Ces utilisateurs Jenkins ont besoin d'une fonctionnalité qui traite les Pipelines complexes comme des objets de premier ordre. C'est pourquoi le [plugin Pipeline](https://plugins.jenkins.io/workflow-aggregator) a été développé.

### Prérequis

La livraison continue est un processus, plutôt qu'un outil, et nécessite un état d'esprit et une culture qui doivent se répandre de haut en bas au sein d'une organisation. Une fois que l'organisation a adhéré à cette philosophie, la prochaine étape, qui est aussi la plus difficile, consiste à cartographier le flux du logiciel depuis son développement jusqu'à sa production.

La base d'un tel Pipeline sera toujours un outil d'orchestration tel que Jenkins, mais il existe certaines exigences clés auxquelles cet élément essentiel du Pipeline doit satisfaire avant de pouvoir être chargé de processus critiques pour l'entreprise :

* **Zero or low downtime disaster recovery :** (Récupération après sinistre sans interruption ou avec un temps d'arrêt minimal) : un _commit_, tout comme un héros mythique, rencontre des défis de plus en plus difficiles et longs à mesure qu'il progresse dans le Pipeline. Il n'est pas rare de voir des exécutions de Pipeline qui durent plusieurs jours. Une panne matérielle ou de Jenkins au sixième jour d'un Pipeline de sept jours a de graves conséquences sur la livraison ponctuelle d'un produit.
* **Audit runs and debug ability** (Exécution d'audits et capacité de débogage) : les responsables de la construction aiment voir le flux d'exécution exact à travers le Pipeline, afin de pouvoir facilement déboguer les problèmes.

Pour garantir qu'un outil puisse s'adapter à une organisation et automatiser de manière appropriée les Pipelines de livraison existants sans les modifier, l'outil doit également prendre en charge :

* **Complex pipelines :** (Pipelines complexes) : les Pipelines de livraison sont généralement plus complexes que les exemples canoniques (processus linéaire : développement → test → déploiement, avec quelques opérations à chaque étape). Les responsables de la construction veulent des constructions qui aident à paralléliser certaines parties du flux, à exécuter des boucles, à effectuer des réessais, etc. En d'autres termes, les responsables de la construction veulent des constructions de programmation pour définir les Pipelines.
* **Manual interventions :** (Interventions manuelles) : les Pipelines dépassent les limites intra-organisationnelles, ce qui nécessite des transferts et des interventions manuels. Les responsables de la construction recherchent des fonctionnalités telles que la possibilité de mettre un pipeline en pause pour permettre à un humain d'intervenir et de prendre des décisions manuelles.

Le plugin Pipeline permet aux utilisateurs de créer un tel Pipeline grâce à un nouveau type de tâche appelé Pipeline. La définition du flux est capturée dans un script Groovy, ajoutant ainsi des capacités de contrôle de flux telles que des boucles, des bifurcations et des réessais. Pipeline permet de définir des étapes avec la possibilité de configurer des concurrences, empêchant ainsi plusieurs compilations du même Pipeline d'essayer d'accéder à la même ressource en même temps.

### Concepts

_Type de tâche Pipeline_<br>
Il n'existe qu'un seul type de tâche permettant de capturer l'ensemble du Pipeline de livraison logicielle d'une organisation. Bien sûr, vous pouvez toujours connecter deux types de tâches Pipeline entre eux si vous le souhaitez. Un type de tâche Pipeline utilise un DSL basé sur Groovy pour les définitions de tâches. Le DSL offre l'avantage de définir les tâches par programmation :

``` groovy
node('linux'){
  git url: 'https://github.com/jglick/simple-maven-project-with-tests.git'
  def mvnHome = tool 'M3'
  env.PATH = "${mvnHome}/bin:${env.PATH}"
  sh 'mvn -B clean verify'
}
```

_Étapes_

Les limites intra-organisationnelles (ou conceptuelles) sont capturées à l'aide d'une primitive appelée « étapes ». Un Pipeline de déploiement se compose de différentes étapes, chacune s'appuyant sur la précédente. L'idée est d'utiliser le moins de ressources possible au début du Pipeline et d'identifier les problèmes évidents, plutôt que de consacrer beaucoup de ressources informatiques à quelque chose qui s'avérera finalement défectueux.

![Graphique représentant le temps sur l'axe des x et les commits sur l'axe des y. Trois points rouges représentent des commits individuels, chacun comportant trois étapes : Build, Selenium Test et Deploy. Le premier et le troisième commit comportent des étapes Deploy, tandis que le deuxième commit n'en comporte pas. Une ligne pointillée descendante relie l'étape Deploy du premier commit à l'étape Deploy du troisième commit, qui est légèrement en avant sur l'axe des x.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/stage-concurrency.png)

Considérons un Pipeline simple à trois étapes. Une implémentation naïve de ce Pipeline peut déclencher séquentiellement chaque étape à chaque _commit_. Ainsi, l'étape de déploiement est déclenchée immédiatement après la fin des étapes de test Selenium. Cependant, cela signifierait que le déploiement à partir du _commit_ deux remplace le dernier déploiement en cours à partir du _commit_ un. La bonne approche consiste à faire en sorte que les _commits_ deux et trois attendent la fin du déploiement à partir du _commit_ un, à consolider toutes les modifications qui ont eu lieu depuis le _commit_ un et à déclencher le déploiement. En cas de problème, les développeurs peuvent facilement déterminer si celui-ci a été introduit lors du _commit_ deux ou du _commit_ trois.

Le Pipeline offre cette fonctionnalité en améliorant la primitive d'étape. Par exemple, une étape peut avoir un niveau de concurrence défini sur un pour indiquer qu'à tout moment, un seul _thread_ doit s'exécuter dans l'étape. Cela permet d'obtenir l'état souhaité, à savoir exécuter un déploiement aussi rapidement qu'il le devrait.

``` groovy
stage name: 'Production', concurrency: 1
node {
    unarchive mapping: ['target/x.war' : 'x.war']
    deploy 'target/x.war', 'production'
    echo 'Déployé vers http://localhost:8888/production/'
}
```

_Portails et approbations_

La livraison continue signifie que les fichiers binaires sont prêts à être publiés, tandis que le déploiement continu signifie que les fichiers binaires sont transférés vers la production, ou déployés automatiquement. Bien que le déploiement continu soit un terme séduisant et un état souhaité, en réalité, les organisations veulent toujours qu'un humain donne son approbation finale avant que les bits ne soient transférés vers la production. Cela est pris en compte par la primitive « input » dans Pipeline. L'étape « input » peut attendre indéfiniment l'intervention d'un humain.

``` groovy
input message: "Est-ce que http://localhost:8888/staging/ vous dit quelque chose ?"
```

_Déploiement d'artefacts vers la préproduction/production_<br>
Le déploiement des binaires est la dernière étape d'un Pipeline. Les nombreux serveurs utilisés au sein de l'organisation et disponibles sur le marché rendent difficile la mise en place d'une étape de déploiement uniforme. Aujourd'hui, ces problèmes sont résolus par des produits de déploiement tiers dont la fonction est de se concentrer sur le déploiement d'une pile particulière vers un centre de données. Les équipes peuvent également écrire leurs propres extensions pour se connecter au type de tâche Pipeline et faciliter le déploiement.

Parallèlement, les créateurs de tâches peuvent écrire une simple fonction Groovy pour définir des étapes personnalisées permettant de déployer (ou de désinstaller) des artefacts à partir de la production.

``` groovy
def deploy(war, id) {
    sh "cp ${war} /tmp/webapps/${id}.war"
}
```

_Flux redémarrables_<br>
Tous les Ppelines sont reprenables. Ainsi, si Jenkins doit être redémarré pendant l'exécution d'un flux, celui-ci devrait reprendre au même point dans son exécution après le redémarrage de Jenkins. De même, si un flux exécute une étape `sh` ou `bat` longue lorsqu'un agent se déconnecte de manière inattendue, aucune progression ne devrait être perdue lorsque la connectivité est rétablie.

Dans certains cas, un flux de construction aura effectué un travail considérable et aura progressé jusqu'à un point où une erreur transitoire s'est produite : une erreur qui ne reflète pas les entrées de cette construction, telles que les modifications du code source. Par exemple, après avoir terminé une construction et un test longs d'un composant logiciel, le déploiement final sur un serveur peut échouer en raison de problèmes réseau.

_Affichage des étapes du Pipeline_<br>
Lorsque vous disposez de Pipelines de compilation complexes, il est utile de voir la progression de chaque étape et de repérer les échecs de compilation dans le Pipeline. Cela permet aux utilisateurs de déboguer les tests qui échouent à une étape donnée ou de détecter d'autres problèmes dans leur Pipeline. De nombreuses organisations souhaitent également rendre leurs Pipelines conviviaux pour les non-développeurs sans avoir à développer une interface utilisateur maison, ce qui peut s'avérer être un effort de développement long et continu.

La fonctionnalité Pipeline Stage View offre une visualisation étendue de l'historique de construction du Pipeline sur la page d'index d'un projet de flux. Cette visualisation comprend également des mesures utiles telles que le temps d'exécution moyen par étape et par construction, ainsi qu'une interface conviviale pour interagir avec les étapes de saisie.

![Jenkins Stage View affichant l'état de plusieurs étapes du Pipeline sur cinq compilations. Les étapes comprennent « Test », « Re-test », « Déploiement », « Déploiement à nouveau », « Continuer le déploiement », « Déploiement final » et « Nettoyage ». Chaque compilation est horodatée et étiquetée (n° 12 à n° 8). Le build n° 8 a réussi jusqu'à l'étape « Déploiement à nouveau », puis a échoué.  Les builds n° 9 et n° 10 ont entièrement réussi, les cellules vertes indiquant la durée de chaque étape en millisecondes. Le build n° 11 a échoué à toutes les étapes, les cellules rayées de rouge indiquant « échoué » et la durée correspondante. Le build n° 12 a en grande partie échoué, à l'exception des étapes initiales « Test » et « Retest », qui ont réussi. La ligne supérieure résume la durée moyenne des étapes, la durée totale du pipeline étant en moyenne de ~523 ms.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/pipeline-workflow-big-responsive.png)

La seule condition préalable à l'utilisation de ce plugin est un Pipeline avec des étapes définies dans le flux. Il peut y avoir autant d'étapes que vous le souhaitez, elles peuvent être disposées dans un ordre linéaire, et leurs noms s'afficheront sous forme de colonnes dans l'interface _Stage View_.

### Traçabilité des artefacts grâce aux empreintes digitales

La traçabilité est importante pour les équipes DevOps qui doivent être en mesure de suivre le code depuis la validation jusqu'au déploiement. Elle permet d'analyser l'impact en montrant les relations entre les artefacts et offre une visibilité sur le cycle de vie complet d'un artefact, depuis son référentiel de code jusqu'à son déploiement final en production.

Jenkins et la fonctionnalité Pipeline prennent en charge le suivi des versions des artefacts à l'aide d'empreintes digitales de fichiers, ce qui permet aux utilisateurs de retracer les compilations en aval qui utilisent un artefact donné. Pour créer une empreinte digitale avec Pipeline, il suffit d'ajouter l'argument « fingerprint: true » à n'importe quelle étape d'archivage d'artefact. Par exemple :

``` groovy
archiveArtifacts artifacts: '**', fingerprint: true
```

archivera tous les artefacts WAR créés dans le Pipeline et les identifiera à des fins de traçabilité. Le journal de traçabilité de cet artefact et la liste de tous les artefacts identifiés dans une compilation seront ensuite disponibles dans le menu de gauche de Jenkins :

Pour savoir où un artefact est utilisé et déployé, il suffit de suivre le lien « plus de détails » à côté du nom de l'artefact et de consulter les entrées correspondantes dans la liste « Utilisation ».

![Une page affichant les détails de l'empreinte digitale d'un fichier app.war. La section intitulée « Ce fichier a été utilisé aux emplacements suivants » l'identifie comme l'empreinte digitale n° 6, indiquant qu'il a été créé ou modifié dans la build n° 6 du Pipeline nommé « empreinte digitale ». La page affiche le hachage MD5 du fichier et la durée écoulée depuis sa création et son suivi. Ces informations se trouvent sous l'onglet « Voir l'empreinte digitale » de cette compilation de Pipeline.](https://www.jenkins.io/doc/book/resources/pipeline-as-code/fingerprinting.png)

Consultez la [documentation sur les empreintes digitales](./utilisation-empreintes-digitales.md) pour en savoir plus.
