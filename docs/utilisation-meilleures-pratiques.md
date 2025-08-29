# Meilleures Pratiques

L'intégration continue (CI) avec exécution automatisée des tests et analyse des tendances a révolutionné la façon dont les entreprises approchent la gestion de la construction, la gestion des versions, l'automatisation du déploiement et l'orchestration des tests. Dans cette section, nous explorerons les meilleures pratiques qui visent à éclairer les dirigeants, les chefs d'entreprise, les développeurs de logiciels et les architectes sur les contributions inestimables que Jenkins peut apporter tout au long du cycle de vie du projet.

## Automatiser la définition du travail

Jenkins a la possibilité de créer, de mettre à jour et de supprimer automatiquement les travaux en fonction des référentiels qu'il identifie dans votre système de gestion de configuration logicielle. Tirez parti de cette fonctionnalité et optimisez votre gestion des tâches en structurant les définitions de votre travail d'une manière qui maximise les avantages offerts par les capacités automatiques de gestion des tâches de Jenkins.

Il existe plusieurs alternatives pour la gestion automatique des tâches, notamment : 

* [Utiliser les dossiers d'organisation](#Utiliser des dossiers d'organisation) : créer, mettre à jour et supprimer automatiquement des dossiers Pipeline multibranches et des tâches Pipeline (méthode recommandée).
* [Utiliser les pipelines multibranches](#Utiliser des pipelines multibranches) : créer, mettre à jour et supprimer automatiquement des tâches Pipeline.
* [Utiliser Pipeline](#Utiliser un pipeline) : tâches Pipeline définies manuellement pour un meilleur contrôle du processus de gestion des tâches.

### Utiliser des dossiers d'organisation

Les dossiers d'organisation constituent un moyen pratique d'automatiser la création et la suppression de tâches dans Jenkins à mesure que des référentiels et des branches sont ajoutés ou supprimés. Si vous utilisez des organisations GitHub, des équipes Bitbucket, des groupes GitLab ou des organisations Gitea, Jenkins peut détecter automatiquement la création d'un nouveau référentiel et générer un projet de pipeline multibranche pour celui-ci.

Reportez-vous à la documentation des [dossiers d'organisation](./pipeline-comme-code.md#dossiers-organisation) pour plus de détails.

_Dossiers d'organisation GitHub_
<iframe width="800" height="420" src="https://www.youtube.com/embed/LbXKUKQ24T8" title="How to Create a GitHub Organization in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

_Dossiers de groupe Gitlab_
<iframe width="800" height="420" src="https://www.youtube.com/embed/it6TOeQ6EHg" title="How to Create a GitLab Group in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

_Dossiers de projet Bitbucket_
<iframe width="800" height="420" src="https://www.youtube.com/embed/85b6fiVolfk" title="How to Create a Bitbucket Project in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

_Dossiers d'organisation Gitea_
<iframe width="800" height="420" src="https://www.youtube.com/embed/NO3sZWRxgQM" title="How to Create a Gitea Organization in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Utiliser des pipelines multibranches

Si vous n'êtes pas en mesure d'utiliser des dossiers d'organisation, vous pouvez opter pour des pipelines multibranches comme alternative. Cependant, il est important de noter que les dossiers d'organisation sont préférés aux pipelines multibranches car ils fournissent l'automatisation de la création et de la suppression de projets multibranches lorsque des référentiels sont ajoutés ou supprimés.

_Pipelines multibranches GitHub_
<iframe width="800" height="420" src="https://www.youtube.com/embed/aDmeeVDrp0o" title="How to Create a GitHub Branch Source Multibranch Pipeline in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

_Dossiers de groupe Gitlab_
<iframe width="800" height="420" src="https://www.youtube.com/embed/y4XGFluzPHY" title="How to Create a GitLab Multibranch Pipeline in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

_Dossiers de projet Bitbucket_
<iframe width="800" height="420" src="https://www.youtube.com/embed/LNfthmZuRDI" title="How to Create a Bitbucket Cloud Branch Source Multibranch Pipeline in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Reportez-vous à la d[ocumentation des pipelines multibranches](./pipeline-comme-code.md#projet-pipeline-multibranches) pour plus de détails.

### Utiliser un pipeline

Si les dossiers d'organisation ne sont pas une option pour vous, envisagez d'utiliser des pipelines multibranches comme alternative. Cependant, il est important de souligner que les dossiers d'organisation sont préférés, en raison de leur capacité à créer et supprimer automatiquement des projets multibranches lorsque des référentiels sont ajoutés ou supprimés.

Reportez-vous à la documentation de [Pipeline](./pipeline-presentation.md) pour plus de détails.

_Différences entre le Freestyle et Pipeline dans Jenkins_
<iframe width="800" height="420" src="https://www.youtube.com/embed/IOUm1lw7F58" title="What Is The Difference Between Freestyle and Pipeline in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Gérez vos tâches

Les définitions de tâches Jenkins peuvent être gérées et optimisées pour améliorer les interactions et la productivité des utilisateurs.

### Rapport sur les résultats de la compilation

Les tableaux et graphiques fournissent des informations précieuses sur l'état d'avancement et la progression des projets, en mettant en évidence les tendances et les schémas. Les résultats des tests automatisés, notamment les tests unitaires, les tests d'intégration et les tests de bout en bout, peuvent révéler des faiblesses ou des instabilités. Les rapports de couverture permettent d'identifier les domaines dans lesquels les tests automatisés ne sont pas exécutés. Les messages d'avertissement du compilateur sont souvent les premiers à signaler un problème. Les outils d'analyse statique sont efficaces pour signaler les codes à risque ou présentant des risques potentiels pour la sécurité. Les résultats des tests de performance permettent d'identifier les retards ou les domaines préoccupants.

Le plugin [d'avertissements de prochaine génération](https://plugins.jenkins.io/warnings-ng) offre un accès pratique à de nombreux rapports, notamment : 

* Avertissements et erreurs du compilateur (comme GCC, Clang, Javac ou Golang) ;
* Avertissements et erreurs de l'analyse statique (ponts spot, vérification, PMD, peluche, CPD ou Simian) ;
* Rapports de couverture de code.

_Comment utiliser le plugin de prochaine génération des avertissements_
<iframe width="800" height="420" src="https://www.youtube.com/embed/tj3xYFA6Q2o" title="How to Use the Warnings Next Generation Plugin in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### S'appuyer sur les agents

Utilisez des agents pour effectuer les compilations au lieu de les exécuter sur le contrôleur. L'utilisation d'agents offre une sécurité et une évolutivité accrues.

Pour plus de détails, consultez la documentation relative à [l'isolation du contrôleur](./securite-isolation-du-controleur.md).

### Signaler les défaillances aux bonnes personnes

Configurez les notifications pour les travaux défaillants et instables, afin de garantir que les bonnes personnes les reçoivent sans provoquer de distractions inutiles pour les autres. De nombreux utilisateurs de Jenkins préfèrent être informés que lorsqu'un échec est probablement de leur responsabilité. Cette approche reconnaît que s'ils ne sont pas responsables de l'échec, ils ne sont peut-être pas la personne la plus appropriée à enquêter.

Affinez votre système de notification afin de donner la priorité aux derniers contributeurs lorsque de nouveaux échecs de test se produisent, car ils sont susceptibles d'être à l'origine du problème.

_Envoi de notifications Slack_
<iframe width="800" height="420" src="https://www.youtube.com/embed/EDVZli8GdUM" title="How to Send Slack Notifications From Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Utiliser des noms de projet simples

Jenkins utilise les noms de projet pour l'organisation de dossiers connexes. Cependant, il est important de noter que certains outils peuvent rencontrer des problèmes avec des espaces, le signe dollars ou des caractères similaires dans les chemins de fichier. Pour garantir la compatibilité, il est recommandé de limiter les noms de projet aux caractères alphanumériques (`a-z,A-Z,0-9,_,-,+`). Pour améliorer l'apparence des noms de projet, vous pouvez utiliser la fonction de nom d'affichage. Cela vous permet de personnaliser la présentation, tout en conservant les caractères restreints du nom du projet sous-jacent. Pour appliquer des conventions de dénomination cohérentes sur tous les projets, activez le paramètre "restreindre la dénomination du projet" dans la configuration du système. Cela garantit que les restrictions de dénomination sont appliquées uniformément.

### Identifiez vos dépendances

Lorsque l'on traite des projets interdépendants, il peut être difficile de savoir quelle version d'un projet est utilisée par un autre. Cependant, Jenkins propose une solution appelée « empreinte digitale des fichiers » pour simplifier ce processus.

Reportez-vous à la page [d'empreintes digitales](./utilisation-empreintes.md) pour plus d'informations.

### N'utilisez pas le type de tâche Maven

Jenkins fournit le [plugin d'intégration Maven](https://plugins.jenkins.io/maven-plugin) depuis de nombreuses années, permettant aux utilisateurs de créer des projets Maven en utilisant la sélection "Maven Project" à partir du menu Jenkins "nouvel élément". Bien que le type de travail Maven offre un niveau d'intégration plus élevé avec les constructions Maven, il peut parfois introduire des complexités inutiles en raison de cette intégration profonde.

Envisagez d'utiliser des dossiers d'organisation, des pipelines multibranches ou des travaux de pipeline au lieu du type de travail Maven. Ces alternatives offrent plus de flexibilité et de simplicité dans la gestion de vos tâches et flux de travail de Jenkins.

Le projet Jenkins utilise des dossiers d'organisation pour construire  [Jenkins Core](https://ci.jenkins.io/job/Core/) et [Jenkins Plugins](https://ci.jenkins.io/job/Plugins/) sur ci.jenkins.io. Un Pipeline Jenkins construit facilement des projets Maven et offre un bien meilleur contrôle aux utilisateurs de Maven.

Reportez-vous à la [documentation du plugin Maven](https://plugins.jenkins.io/maven-plugin/#plugin-content-risks) pour plus de détails.

## Gérez votre contrôleur

Le contrôleur Jenkins joue un rôle crucial en tant que ressource centrale, nécessitant une gestion efficace pour des performances optimales. En suivant ces pratiques, vous pouvez vous assurer que votre contrôleur offre la meilleure expérience possible aux utilisateurs.

### Sécuriser le contrôleur

Les installations de Jenkins sont livrées avec la sécurité activée par défaut, ce qui est un aspect crucial de la protection de votre système. Bien qu'il soit techniquement possible de désactiver la sécurité, il est fortement conseillé de ne pas le faire. La désactivation de la sécurité peut rendre votre contrôleur Jenkins vulnérable à l'accès non autorisé et aux violations de sécurité potentielles. Il est important de maintenir un environnement sécurisé en gardant la sécurité activée à tout moment.

Reportez-vous au chapitre [Sécuriser Jenkins](./securite-securiser.md) du manuel des utilisateurs pour plus de détails.

### Sauvegarder régulièrement

Même les systèmes les plus fiables peuvent subir des échecs. C’est pourquoi il est crucial de se préparer et de vérifier régulièrement la santé de vos sauvegardes. Les sauvegardes sont un élément essentiel pour assurer l'intégrité et la disponibilité de vos données. Tester régulièrement vos sauvegardes et la vérification de leur exhaustivité et de leur restaurabilité vous aidera à atténuer l'impact de toute défaillance potentielle et à vous assurer que vos données peuvent être récupérées efficacement en cas de besoin. Il est essentiel de hiérarchiser la santé de la sauvegarde et la réalisation de contrôles de routine pour maintenir un système robuste et résilient.

Plus de détails peuvent être trouvés dans la [documentation de sauvegarde](./administration-systeme-sauvegarde.md).

### Évitez la surcharge de travail

Planifiez vos tâches de manière stratégique afin d'équilibrer le nombre de tâches exécutées simultanément. Si vous utilisez des déclencheurs temporisés ou des interrogations périodiques, envisagez d'utiliser la syntaxe `H` dans l'expression cron pour introduire une variation dans la planification. Cela permet de répartir plus uniformément les heures de début des tâches et d'éviter qu'elles ne démarrent toutes simultanément. De plus, tirez parti des jetons prédéfinis tels que `@hourly` pour répartir davantage les heures de début de vos tâches. Ces jetons peuvent aider à créer un calendrier plus équilibré et à réduire le risque de contention des ressources.

En mettant en œuvre ces techniques de planification, vous pouvez optimiser l'utilisation de vos ressources et garantir une exécution plus fluide de votre travail.

## Évitez les collisions de ressources

Lorsque plusieurs tâches s'exécutent simultanément, il existe un risque de collision, en particulier si elles nécessitent un accès exclusif à certaines ressources ou à certains services de configuration. Pour éviter les interférences et garantir une exécution fluide, il est important de gérer efficacement l'accès aux ressources. Pour les builds impliquant des bases de données ou des services en réseau, il est essentiel de mettre en œuvre des mesures permettant d'éviter les conflits. Le [plugin Lockable Resources](https://plugins.jenkins.io/lockable-resources) offre des fonctionnalités de verrouillage des ressources très précises pour les tâches Jenkins. En utilisant ce plugin, vous pouvez vous assurer qu'un seul travail à la fois a accès à une ressource spécifique, ce qui évite les conflits et garantit une synchronisation correcte. Dans les cas où le verrouillage des ressources avec le plugin Lockable Resources n'est pas suffisant, vous pouvez contrôler davantage les builds simultanés à l'aide du [plugin Throttle Concurrent Builds](https://plugins.jenkins.io/throttle-concurrents). Ce plugin vous permet de limiter le nombre de builds pouvant s'exécuter simultanément, ce qui offre un contrôle supplémentaire et évite la surcharge des ressources partagées.

En tirant parti de ces plugins, vous pouvez gérer efficacement les conflits de ressources et la concurrence, assurant une exécution fluide et fiable de vos travaux Jenkins.

_Comment utiliser des ressources verrouillables_
<iframe width="800" height="420" src="https://www.youtube.com/embed/y_z8mqV8G68" title="How To Use Jenkins Lockable Resources" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

