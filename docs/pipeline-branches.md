# Branches et _Pull Requests_ (Demande d'Extraction)

Dans la [section précédente](./pipeline-jenkinsfile.md), un fichier `Jenkinsfile` pouvant être enregistré dans le contrôleur de source a été implémenté. Cette section traite du concept des Pipelines multibranches qui s'appuient sur le fichier `Jenkinsfile` pour offrir des fonctionnalités plus dynamiques et automatiques dans Jenkins.

_Création d'un Pipeline multibranches dans Jenkins_
<iframe width="800" height="420" src="https://www.youtube.com/embed/B_2FXWI6CWg" title="Jenkins Multibranch Pipeline With Git Tutorial" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Création d'un Pipeline multibranches

Le type de projet Pipeline Multibranche vous permet de mettre en œuvre différents fichiers Jenkinsfiles pour différentes branches d'un même projet. Dans un projet Multibranch Pipeline, Jenkins détecte, gère et exécute automatiquement les Pipelines pour les branches qui contiennent un fichier `Jenkinsfile` dans le contrôle de source.

Cela élimine le besoin de créer et de gérer manuellement les Pipelines.

Pour créer un Pipeline multibranches :

* Cliquez sur **_New Item_** (Nouvel élément) sur la page d'accueil de Jenkins.

Colonne de gauche de l'interface utilisateur classique de Jenkins affichant les options : New Item (Nouvel élément), Build History (Historique de compilation), Project Relationship (Relation entre projets), Check File Fingerprint (Vérifier l'empreinte digitale du fichier), Manage Jenkins (Gérer Jenkins) et My Views (Mes vues).

* Entrez un nom pour votre Pipeline, sélectionnez **_Multibranch Pipeline_** (Pipeline Multibranches) et cliquez sur **OK**.

!!! Bug
    Jenkins utilise le nom du Pipeline pour créer des répertoires sur le disque. Les noms de Pipeline qui contiennent des espaces peuvent révéler des bogues dans les scripts qui ne prévoient pas que les chemins d'accès contiennent des espaces.

![Page Nouvel élément de Jenkins avec le nom d'élément « multibranch-example » et l'option Pipeline multibranche sélectionnée parmi les options de type d'élément suivantes : Projet Freestyle, Pipeline, Projet multi-configuration, Dossier, Pipeline multibranche et Dossier d'organisation.](https://www.jenkins.io/doc/book/resources/pipeline/new-item-multibranch-creation.png)

* Ajoutez une **_Branch Source_** (Source de Branche) (par exemple, Git).

![Configurez la page du Pipeline «exemple-multibranche». Dans la section «Sources de Branche», Git est sélectionné dans le menu déroulant «Ajouter Source». ](https://www.jenkins.io/doc/book/resources/pipeline/multibranch-branch-source.png)

* Entrez l'emplacement du référentiel Git.

![Configurez la page du pipeline «exemple-multibranche». La section de configuration de la source de branche pour l'option de source de branche «git» est ouverte avec des options pour ajouter un lien de référentiel de projet, des informations d'identification, des comportements et une stratégie de propriété.](https://www.jenkins.io/doc/book/resources/pipeline/multibranch-branch-source-configuration.png)

* **Enregistrez** le projet de Pipeline Multibranche.

Lorsque vous cliquez sur **_Save_** (Enregistrer), Jenkins analyse automatiquement le référentiel désigné et crée les éléments appropriés pour chaque branche du référentiel contenant un fichier `Jenkinsfile`.

Par défaut, Jenkins ne réindexera pas automatiquement le référentiel en cas d'ajouts ou de suppressions de branches (sauf si vous utilisez un [dossier d'organisation](./pipeline-branches.md#dossiers-dorganisation)), il est donc souvent utile de configurer un Pipeline Multibranches pour réindexer périodiquement dans la configuration :

![Configurez la page du pipeline «exemple-multibranches». Dans la section «Scan Multibranch Pipeline Triggers», l'option «périodiquement sinon autrement» est vérifiée et l'intervalle est défini sur «30 minutes».](https://www.jenkins.io/doc/book/resources/pipeline/multibranch-branch-indexing.png)

### Variables d'environnement supplémentaires

Les Pipelines Multibranches exposent des informations supplémentaires sur la construction de la branche via la variable globale `env`, telles que :

**_BRANCH_NAME_** (Nom de la branche)<br>
Nom de la branche pour laquelle ce Pipeline s'exécute, par exemple Master.

**_CHANGE_ID_** (ID de changement)<br> 
Un identifiant correspondant à une sorte de demande de changement, comme un numéro de _pull request_.

Des variables d'environnement supplémentaires sont répertoriées dans la [référence de variables globales](./pipeline-commencer.md#référence-de-variable-globale).

### Référence aux variables globales

Soutenir les demandes de traction

Les Pipelines Multibranches peuvent être utilisés pour valider les demandes de traction/modification avec le plugin approprié. Cette fonctionnalité est fournie par les plugins suivants : 

* [GitHub Branch Source](https://plugins.jenkins.io/github-branch-source) ;
* [Bitbucket Branch Source](https://plugins.jenkins.io/cloudbees-bitbucket-branch-source) ;
* [GitLab Branch Source](https://plugins.jenkins.io/gitlab-branch-source) ;
* [Gitea](https://plugins.jenkins.io/gitea) ;
* [Tuleap Git Branch Source](https://plugins.jenkins.io/tuleap-git-branch-source) ;
* [AWS CodeCommit Jobs](https://plugins.jenkins.io/aws-codecommit-jobs) ;
* [DAGsHub Branch Source](https://plugins.jenkins.io/dagshub-branch-source).

Veuillez consulter leur documentation pour plus d'informations sur la façon d'utiliser ces plugins.

## Utilisation de dossiers d'organisation

Les dossiers d'organisation permettent à Jenkins de surveiller l'ensemble d'une organisation GitHub, d'une équipe/d'un projet Bitbucket, d'une organisation GitLab ou d'une organisation Gitea, et de créer automatiquement de nouveaux Pipelines Multibranches pour les référentiels contenant des branches et des _pull requests_ contenant un fichier `Jenkinsfile`.

Les dossiers d'organisation sont mis en œuvre pour : 

* Github dans le plugin [GitHub Branch Source](https://plugins.jenkins.io/github-branch-source) ; 
* Bitbucket dans le plugin [Bitbucket Branch Source](https://plugins.jenkins.io/cloudbees-bitbucket-branch-source) ; 
* Gitlab dans le plugin [GitLab Branch Source](https://plugins.jenkins.io/gitlab-branch-source) ;
* Gitea dans le plugin [Gitea](https://plugins.jenkins.io/gitea).

