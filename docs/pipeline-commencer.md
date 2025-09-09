# Commencer avec Pipeline

Comme mentionné [précédemment](./Pipeline-presentation.md), Jenkins Pipeline est une suite de plugins qui prend en charge la mise en œuvre et l'intégration de Pipeline de livraison continue dans Jenkins. Le Pipeline fournit un ensemble extensible d'outils pour modéliser les Pipelines de livraison simples à complexe "comme code" via le Pipeline DSL. (1)
{ .annotate }

1. [Langage spécifique à un domaine](https://en.wikipedia.org/wiki/domain-specific_language)

Cette section décrit comment démarrer avec la création de votre projet Pipeline dans Jenkins et vous présente les différentes façons dont un `Jenkinsfile` peut être créé et stocké.

## Conditions préalables

Pour utiliser le Pipeline Jenkins, vous aurez besoin de : 

* Jenkins 2.x ou version ultérieure (les versions plus anciennes antérieures à 1.642.3 peuvent fonctionner mais ne sont pas recommandées) ;
* Plugin Pipeline, (1) qui est installé dans le cadre des « plugins suggérés » (spécifiés lors de l'exécution de [l'assistant de configuration post-installation](./installation-linux.md#assistant-de-configuration-post-installation) après [l'installation de Jenkin](./installation-presentation.md)).

1. [Plugin Pipeline](https://plugins.jenkins.io/workflow-aggregator)

En savoir plus sur la façon d'installer et de gérer les plugins dans la [gestion des plugins](./gestion-plugins.md).

## Définir un Pipeline

Le [Pipeline déclaratif et scripté](./pipeline-presentation.md#syntaxe-de-pipeline-déclaratif-contre-scénarisé) est DSLs (1) pour décrire des parties de votre Pipeline de livraison de logiciel. Le Pipeline scripté est écrit sous une forme limitée de [syntaxe Groovy](http://groovy-lang.org/semantics.html).

1. [Langage spécifique à un domaine](https://en.wikipedia.org/wiki/Domain-specific_language)

Les composantes pertinentes de la syntaxe Groovy seront introduites selon les besoins tout au long de cette documentation, donc si une compréhension de Groovy est utile, il n'est pas nécessaire de travailler avec Pipeline.

Un Pipeline peut être créé de l'une des manières suivantes : 

* [Grâce à Blue Ocean](#via-bleu-ocean) - après avoir configuré un projet Pipeline dans Blue Ocean, l'interface utilisateur Blue Ocean vous aide à rédiger le fichier `Jenkinsfile` de votre Pipeline et à le valider dans le contrôle de source.
* [Grâce à l'interface utilisateur classique](#via-linterface-utilisateur) - vous pouvez saisir un Pipeline de base directement dans Jenkins via l'interface utilisateur classique.
* [Dans SCM](#definir-un-pipeline-dans scm) -  vous pouvez rédiger manuellement un fichier `Jenkinsfile`, que vous pouvez ensuite enregistrer dans le référentiel de contrôle de source de votre projet. (1)

1. [Gestion du contrôle des sources](https://en.wikipedia.org/wiki/Version_control)

La syntaxe pour définir un Pipeline avec l'une ou l'autre approche est la même, mais bien que Jenkins prenne en charge la saisie directe du Pipeline dans l'interface utilisateur classique, il est généralement considéré comme une bonne pratique de définir le Pipeline dans un fichier `Jenkinsfile` que Jenkins chargera ensuite directement à partir du contrôle de source.

Cette vidéo fournit des instructions de base sur la façon d'écrire des Pipelines déclaratifs et scriptés.

_Écrire un script de Pipeline à Jenkins_
<iframe width="800" height="420" src="https://www.youtube.com/embed/TiTrcFEsj7A" title="How to Write a Pipeline Script in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Via Blue Ocean

Si vous débutez avec Jenkins Pipeline, l'interface utilisateur Blue Ocean vous aide à [configurer votre projet Pipeline](./blueocean-creer-des-pipelines.md), crée et écrit automatiquement votre Pipeline (c'est-à-dire le fichier `Jenkinsfile`) pour vous grâce à l'éditeur graphique Pipeline.

Dans le cadre de la configuration de votre projet Pipeline dans Blue Ocean, Jenkins configure une connexion sécurisée et correctement authentifiée au référentiel de contrôle source de votre projet. Par conséquent, toutes les modifications que vous apportez au `Jenkinsfile` via Blue Ocean’s Pipeline Editor sont automatiquement enregistrées et envoyées au contrôle de sources.

En savoir plus sur Blue Ocean dans le chapitre [Blue Ocean](./bleuocean-presentation.md) et la page [commencer avec Blue Ocean](./blueocean-commencer.md).

!!! info "_Statut de Blue Ocean_"
    Blue Ocean ne recevra pas d'autres mises à jour de fonctionnalité. Blue Ocean continuera de fournir une visualisation de Pipeline facile à utiliser, mais il ne sera pas amélioré davantage. Il ne recevra que des mises à jour sélectives pour des problèmes de sécurité importants ou des défauts fonctionnels.

    Le [générateur d'extrait de syntaxe Pipeline](#generateur-dextrait) aide les utilisateurs lorsqu'ils définissent les étapes du Pipeline avec leurs arguments. Il s'agit de l'outil préféré pour la création de Pipeline Jenkins car il fournit une aide en ligne pour les étapes du Pipeline disponibles dans votre contrôleur Jenkins. Il utilise les plugins installés sur votre contrôleur pour générer la syntaxe du Pipeline. Reportez-vous à la page de [référence des étapes du Pipeline](./pipeline-mesures.md) pour plus d'informations.

### À travers l'interface utilisateur classique

Un `Jenkinsfile` créé à l'aide de l'interface utilisateur classique est stocké par Jenkins lui-même (dans le répertoire de la maison Jenkins).

Pour créer un Pipeline de base via l'interface utilisateur Jenkins Classic : 

1. Si nécessaire, assurez-vous que vous êtes connecté à Jenkins. 
2. Dans le tableau de bord Jenkins, sélectionnez un nouvel élément. 
![Jenkins Classic UI Colonne de gauche](https://www.jenkins.io/doc/book/resources/pipeline/classic-ui-new-item.png)
3. Dans le champ **_Enter an item name_**(Entrez un nom d'article), spécifiez le nom de votre nouveau projet Pipeline. 
**Attention :** Jenkins utilise ce nom d'élément pour créer des répertoires sur le disque. Il est recommandé d'éviter d'utiliser des espaces dans les noms des éléments, car cela peut découvrir des bogues dans des scripts qui ne gèrent pas correctement les espaces dans les chemins de répertoire. 
4. Faites défiler vers le bas et cliquez sur **Pipeline**, puis cliquez sur **OK** à la fin de la page pour ouvrir la page de configuration du Pipeline (dont l'onglet **Général** est sélectionné).
![nouvel article](https://www.jenkins.io/doc/book/resources/pipeline/new-item-creation.png)
5. Cliquez sur l'onglet **Pipeline** dans le panneau latéral de la page pour faire défiler jusqu'à la section **Pipeline**. 
**Remarque :** Si à la place, vous définissez votre `Jenkinsfile` dans le contrôleur de source, suivez les instructions dans SCM ci-dessous. 
6. Dans la section **Pipeline**, assurez-vous que le champ de **Definition** indique l'option **script Pipeline**. 
7. Entrez votre code Pipeline dans la zone de texte **Script**.
Par exemple, copiez l'exemple de code Pipeline Déclaratif suivant (sous l'en-tête _Jenkinsfile ( ... )_) ou sa version scriptée et collez-le dans la zone de texte **Script**. (L'exemple Déclaratif ci-dessous est utilisé tout au long de la suite de cette procédure.)

``` groovy title="Jenkinsfile (Pipeline Déclaratif)"
pipeline {
    agent any // (1)
    stages {
        stage('Stage 1') {
            steps {
                echo 'Hello world!' // (2)
            }
        }
    }
} 
```

1. `agent` demande à Jenkins d'attribuer un exécuteur (sur n'importe quel agent/nœud disponible dans l'environnement Jenkins) et un espace de travail pour l'ensemble du Pipeline.
2.  `echo` écrit une chaîne simple dans la sortie de la console.

??? note "Activer/désactiver le Pipeline scripté _(Avancé)_"

    ``` groovy title="_Jenkinsfile (Pipeline Scripté)_"
    node { //(1)
        stage('Stage 1') {
            echo 'Hello World' 
        }
    }
    ```

    1.  `node` fait effectivement la même chose que `agent` (ci-dessus).
    ![script](https://www.jenkins.io/doc/book/resources/pipeline/example-pipeline-in-classic-ui.png)
    **Note :** Vous pouvez également sélectionner des exemples prédéfinis de Pipeline _scripté_ à partir de l'option **_try sample Pipeline_** (Essayer un exemple de pipeline) située en haut à droite de la zone de texte Script. Veuillez noter qu'aucun exemple prédéfini de pipeline déclaratif n'est disponible dans ce champ.

8 . Cliquez sur **_Save_** (Enregistrer) pour ouvrir la page d'affichage du projet/élément Pipeline.

9 . Sur cette page, cliquez sur **_Build Now_** (Créer Maintenant) à gauche pour exécuter le Pipeline.

![Configurez la page avec l'onglet Pipeline sélectionné dans l'interface utilisateur Jenkins Classic](https://www.jenkins.io/doc/book/resources/pipeline/classic-ui-left-column-on-item-build-now.png). 

10 . Sous **_Build History_** (l'historique de construction] à gauche, cliquez sur **#1** pour accéder aux détails de cette exécution particulière. 

11 . Cliquez sur **_Console Output_** (sortie de la console) pour voir la sortie complète de l'exécution du Pipeline. La sortie suivante montre une exécution réussie de votre Pipeline. 

![Jenkins Classic UI pour 'Example-Pipeline', montrant l'onglet de sortie de la console pour Build # 1, exécuté par l'utilisateur administrateur, avec un statut de finition réussi.](https://www.jenkins.io/doc/book/resources/pipeline/hello-world-console-output.png)

**Notes :**

* Vous pouvez également accéder à la sortie de la console directement depuis le tableau de bord en cliquant sur le globe coloré à gauche du numéro de construction (par exemple **#1**) ;
* La définition d'un Pipeline via l'interface utilisateur classique est pratique pour tester les extraits de code Pipeline, ou pour gérer des Pipelines simples ou qui ne nécessitent pas de code source pour être vérifié/cloné à partir d'un référentiel. Comme mentionné ci-dessus, contrairement à `Jenkinsfiles` que vous définissez via Blue Ocean ([ci-dessus](#via-blue-ocean)) ou dans le contrôle de la source ([ci-dessous](#définir-un-pipeline-dans-scm)), `Jenkinsfiles` entré dans la zone de texte du script des projets de Pipelines est stocké par Jenkins lui-même, dans le répertoire domestique de Jenkins. Par conséquent, pour un contrôle et une flexibilité plus élevés sur votre pipeline, en particulier pour les projets de contrôle des sources susceptibles de gagner de la complexité, il est recommandé d'utiliser [Blue Ocean](#via-blue-ocean) ou le [contrôleur de source](#définir-un-pipeline-dans-scm) pour définir votre `Jenkinsfiles`.

### Dans SCM

Les pipelines complexes sont difficiles à écrire et à maintenir dans la zone de texte de script de [l'interface utilisateur](#à-travers-linterface-utilisateur-classique) de la page de configuration du Pipeline.

Pour faciliter cette opération, le fichier `Jenkinsfile` de votre Pipeline peut être rédigé dans un éditeur de texte ou un environnement de développement intégré (IDE), puis validé dans le contrôleur de source[^1] (éventuellement avec le code d'application que Jenkins va compiler). Jenkins peut ensuite extraire votre fichier `Jenkinsfile` du contrôleur de source dans le cadre du processus de compilation de votre projet Pipeline, puis procéder à l'exécution de votre Pipeline.

[^1] : [Gestion du contrôle des sources](https://en.wikipedia.org/wiki/Version_control)

Pour configurer votre projet Pipeline pour utiliser un `Jenkinsfile` à partir du contrôleur de source : 

1. Suivez la procédure ci-dessus pour définir votre Pipeline via [l'interface utilisateur classique](#à-travers-linterface-utilisateur-classique) jusqu'à ce que vous atteigniez l'étape 5 (accéder à la section **Pipeline** sur la page de configuration du Pipeline). 
2. Dans le champ **Definition**, choisissez **_Pipeline script from SCM_** (script Pipeline à partir de l'option SCM). 
3. Dans le champ **SCM**, choisissez le type de système de contrôle source du référentiel contenant votre `Jenkinsfile`. 
4. Complétez les champs spécifiques au système de contrôle source de votre référentiel. 
**Conseil :** si vous n'êtes pas sûr de la valeur à spécifier pour un champ donné, cliquez sur l'icône **?** à droite pour plus d'informations. 
5. Dans le champ **_Script Path_** (chemin de script), spécifiez l'emplacement (et le nom) de votre `Jenkinsfile`. Cet emplacement est celui où Jenkins vérifie/clone le référentiel contenant votre fichier `Jenkinsfile`, qui doit correspondre à la structure de fichiers du référentiel. La valeur par défaut de ce champ suppose que votre fichier `Jenkinsfile` soit nommé « Jenkinsfile » et se trouve à la racine du référentiel.

Lorsque vous mettez à jour le référentiel désigné, une nouvelle version est déclenchée, à condition que le Pipeline soit configuré avec un déclencheur d'interrogation SCM.

!!! tip
    Étant donné que le code du Pipeline (c'est-à-dire le Pipeline scripté en particulier) est écrit dans une syntaxe similaire à Groovy, si votre IDE ne met pas correctement en évidence la syntaxe de votre fichier `Jenkinsfile`, essayez d'insérer la ligne `#!/usr/bin/env groovy` en haut du fichier `Jenkinsfile`,[^4], ce qui devrait résoudre le problème.

[^4] [Shebang (définition générale)](https://en.wikipedia.org/wiki/Shebang_(Unix))

## Documentation intégrée

Pipeline est livré avec des fonctionnalités de documentation intégrées pour faciliter la création de Pipelines de complexités variables. Cette documentation intégrée est automatiquement générée et mise à jour en fonction des plugins installés dans le contrôleur Jenkins.

La documentation intégrée peut être trouvée mondiale sur `${YOUR_JENKINS_URL}/pipeline-syntax`. La même documentation est également liée en tant que **_Pipeline Syntax_** (Syntaxe Pipeline) dans la barre latérale pour tout projet de Pipeline configuré.

![Barre latérale de l'interface utilisateur Jenkins Classic pour "Exemple-Pipeline", montrant les options:  statut, modifications, construire maintenant, configurer, supprimer le Pipeline, vue complète, favorits, étapes, renommer et syntaxe du Pipeline.](https://www.jenkins.io/doc/book/resources/pipeline/classic-ui-left-column-on-item-pipeline-syntax.png)

### Générateur d'extraits

L'utilitaire "Snippet Generator" intégré est utile pour créer des morceaux de code pour des étapes individuelles, découvrir de nouvelles étapes fournies par des plugins ou tester différents paramètres pour une étape particulière.

Le générateur d'extraits est rempli dynamiquement avec une liste des étapes disponibles pour le contrôleur Jenkins. Le nombre d'étapes disponibles dépend des plugins installés qui exposent explicitement les étapes à utiliser dans Pipeline.

Pour générer un extrait de code avec le générateur d'extraits :

1. Accédez au lien **_Pipeline Syntax_** (Syntaxe Pipeline) (mentionné ci-dessus) à partir d'un Pipeline configuré, ou à l'adresse `${YOUR_JENKINS_URL}/pipeline-syntax`.
2. Sélectionnez l'étape souhaitée dans le menu déroulant **_Sample Step_** (Exemple d'étape).
3. Utilisez la zone remplie dynamiquement sous le menu déroulant **_Sample Step_** (Exemple d'étape) pour configurer l'étape sélectionnée.
4. Cliquez sur **_Generate Pipeline Script_** (Générer un script de Pipeline) pour créer un extrait Pipeline qui peut être copié et collé dans un Pipeline.

![Jenkins Classic UI pour "Example-Pipeline", montrant la page de syntaxe du Pipeline avec l'onglet Générateur d'extraits sélectionné. L'extrait généré pour la scène nommée «Deploy» s'affiche dans le bloc de script.](https://www.jenkins.io/doc/book/resources/pipeline/snippet-generator.png)

Pour accéder à des informations supplémentaires et/ou à la documentation sur l'étape sélectionnée, cliquez sur l'icône d'aide (indiquée par la flèche rouge dans l'image ci-dessus).

### Référence de variable globale

En plus du générateur d'extraits, qui ne fait apparaître que les étapes, Pipeline fournit également une « référence de variables globales » intégrée. Tout comme le générateur d'extraits, elle est également remplie dynamiquement par des plugins. Contrairement au générateur d'extraits, la référence de variables globales ne contient toutefois que la documentation relative aux variables fournies par Pipeline ou les plugins, qui sont disponibles pour les Pipelines.

Les variables fournies par défaut dans le Pipeline sont :

**env**<br>
Expose les variables d'environnement, par exemple : `env.PATH` ou `env.BUILD_ID`. Consultez la référence des variables globales intégrées à l'adresse `${YOUR_JENKINS_URL}/pipeline-syntax/globals#env` pour obtenir une liste complète et à jour des variables d'environnement disponibles dans Pipeline.

**params**<br>
Expose tous les paramètres définis pour le Pipeline comme une [**_Map_**](http://groovy-lang.org/syntax.html#_maps) (Carte) en lecture seule, par exemple : `params.MY_PARAM_NAME`.

**currentBuild**<br>
Peut être utilisé pour découvrir des informations sur le Pipeline en cours d'exécution, avec des propriétés telles que `currentBuild.result`, `currentBuild.displayName`, etc. Consultez la référence des variables globales intégrées à l'adresse `${YOUR_JENKINS_URL}/pipeline-syntax/globals` pour obtenir une liste complète et à jour des propriétés disponibles sur `currentBuild`.

Cette vidéo passe en revue l'utilisation de la variable `currentBuild` dans Jenkins Pipeline.
<iframe width="800" height="420" src="https://www.youtube.com/embed/gcUORgHuna4" title="What Is currentBuild in Jenkins?" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Générateur de directif déclaratif

Bien que le générateur de snippets aide à générer des étapes pour un Pipeline scripté ou pour le bloc de `steps` d'une `stage` dans un Pipeline déclaratif, il ne couvre pas les [sections(./pipeline-syntaxe-pipeline.md#sections-declaratives)] et les [directives(./pipeline-syntaxe-pipeline.md#sections-directives)] utilisées pour définir un Pipeline déclaratif. L'utilitaire « Générateur de directives déclaratives » vous aide dans cette tâche. Tout comme le [générateur d'extraits](./pipeline-commencer.md#générateur-dextraits), le générateur de directives vous permet de choisir une directive déclarative, de la configurer dans un formulaire et de générer la configuration de cette directive, que vous pouvez ensuite utiliser dans votre Pipeline déclaratif.

Pour générer une directive déclarative à l'aide du générateur de directive déclarative : 

1. Accédez au lien **_Pipeline Syntax_** (Syntaxe du Pipeline) (référencé ci-dessus) à partir d'un Pipeline configuré, puis cliquez sur le lien **_Declarative Directive Generator_** (Générateur de Directives Déclaratives) dans le sidepanel, ou accédez directement à `${YOUR_JENKINS_URL}/directive-generator`.
2. Sélectionnez la directive souhaitée dans le menu déroulant. 
3. Utilisez la zone dynamiquement peuplée sous la liste déroulante pour configurer la directive sélectionnée. 
4. Cliquez sur **_Generate Directive_** (Générer la Directive) pour créer la configuration de la directive à copier dans votre Pipeline.

Le générateur de directives peut générer la configuration de directives imbriquées, telles que les conditions à l'intérieur d'une directive `when`, mais il ne peut pas générer d'étapes de Pipeline. Pour le contenu des directives qui contiennent des étapes, telles que les `steps` à l'intérieur d'une `stage` ou les conditions telles que `always` ou `failure` à l'intérieur de `post`, le générateur de directives ajoute à la place un commentaire de remplacement. Vous devrez toujours ajouter les étapes à votre Pipeline manuellement.

``` groovy title="Jenkinsfile (Declarative Pipeline)"
stage('Stage 1') {
    steps {
        // Une ou plusieurs étapes doivent être incluses dans le bloc d'étapes.
    }
}
```

## Lectures complémentaires

Cette section n'aborde que superficiellement les possibilités offertes par Jenkins Pipeline, mais elle devrait vous fournir suffisamment de bases pour commencer à expérimenter avec un contrôleur Jenkins de test.

Dans la section suivante, le [JenkinsFile](./pipeline-utiliser-un-jenkinsfile.md), nous aborderons d'autres étapes du Pipeline ainsi que des modèles permettant de mettre en œuvre avec succès des Pipelines Jenkins dans le monde réel.

### Ressources supplémentaires 

* [Référence des étapes du Pipeline](./ressources-pipeline-etapes.md) qui englobe toutes les étapes fournies par les plugins distribués dans le Centre de mise à jour Jenkins ;
* [Exemples de Pipelines](./ressources-pipeline-exemples.md), une collection d'exemples de pipelines copiables, sélectionnés par la communauté.
