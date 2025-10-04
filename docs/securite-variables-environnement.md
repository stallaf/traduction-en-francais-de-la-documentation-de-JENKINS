# Gestion des Variables d'Environnement

<div class="couleur-introduction">
Il est courant que les scripts de build exécutés par Jenkins reçoivent des paramètres sous forme de variables d'environnement. Par exemple, les projets freestyle paramétrés transmettent les valeurs des paramètres aux étapes de build Shell et Batch sous forme de variables d'environnement. Les déclencheurs ou les causes du build sont d'autres sources courantes de variables d'environnement. Si une pull request est construite, les variables d'environnement peuvent contenir l'ID de la pull request ou même le nom de la pull request.

Les utilisateurs et administrateurs Jenkins doivent être conscients des problèmes potentiels liés aux variables d'environnement et de leur impact sur le comportement des builds.
</div>

## Définition de variables d'environnement spéciales

Les variables d'environnement spéciales peuvent modifier le comportement des scripts de build : `PATH` (ou `Path` sous Windows) est généralement utilisé pour trouver les programmes lancés pendant un build. Le fait de remplacer sa valeur peut donc avoir un effet inattendu, car d'autres programmes que ceux habituellement attendus peuvent être lancés.

Des variables moins connues, telles que `LD_PRELOAD` (sous Linux) et `DYLD_LIBRARY_PATH` (sous macOS), peuvent avoir un effet similaire. De nombreuses autres variables d'environnement sont lues par différents programmes, et leur définition peut modifier leur comportement.

Tout utilisateur pouvant ajouter des variables d'environnement avec un nom de son choix peut être en mesure de modifier le comportement des compilations de cette manière. Bien qu'il s'agisse d'une fonctionnalité utile pour les utilisateurs disposant d'une autorisation Job/Configure, les administrateurs doivent vérifier la capacité des plugins à ajouter des variables d'environnement arbitraires dans l'environnement de compilation à partir de sources qui ne nécessitent pas d'autorisation pour configurer des tâches.

## Définition de valeurs de variables d'environnement non sécurisées

Tous les scripts et interpréteurs de scripts couramment utilisés lors des compilations ne sont pas conçus pour ce cas d'utilisation, et il peut être nécessaire de prendre des précautions particulières pour filtrer ou nettoyer les valeurs des variables d'environnement.

Windows Batch est un exemple de ce comportement problématique. Par défaut, `cmd.exe` évalue les valeurs des variables au fur et à mesure qu'elles sont lues. Une valeur de variable contenant des commandes Batch valides peut entraîner l'exécution de ces commandes par Jenkins. Consultez [ce rapport](https://threatpost.com/shellshock-like-weakness-may-affect-windows/108696/) pour obtenir un résumé du problème. Si Windows Batch permet d'écrire des scripts de manière à éviter ce problème (à l'aide de `EnableDelayedExpansion` et de la syntaxe `!variable!`), ce n'est pas le cas de tous les autres scripts ou interpréteurs de scripts. De plus, les administrateurs ne souhaitent pas nécessairement compter sur le fait que tous les utilisateurs Jenkins qui écrivent des scripts de build soient conscients de ce problème.

Depuis Jenkins 2.248 et LTS 2.249.1, les administrateurs peuvent définir globalement des filtres pour les variables d'environnement transmises aux étapes de build qui les prennent en charge. Les étapes de build Shell et Windows Batch intégrées prennent en charge ces filtres, tout comme les différentes étapes de pipeline fournies par [Pipeline : Nodes and Processes](https://plugins.jenkins.io/workflow-durable-task-step) 2.36 et versions ultérieures. Cette fonctionnalité peut être utilisée pour modifier ou supprimer des variables contenant des métacaractères potentiellement dangereux (tels que `^` ou `&` dans le cas de Windows Batch) de l'environnement de ces étapes de build, ou même pour faire échouer l'étape de build.

Les plugins suivants implémentent des fonctionnalités utiles pour filtrer les variables d'environnement potentiellement dangereuses :

* [Safe Batch Environment Filter](https://plugins.jenkins.io/safe-batch-environment-filter) permet aux administrateurs d'échouer automatiquement les étapes de build Batch (à la fois dans les projets classiques et dans les Pipelines) si ces étapes contiennent des variables d'environnement avec des métacaractères Batch ;
* [Generic Build Step Environment Filters](https://plugins.jenkins.io/generic-environment-filters) fournit des implémentations génériques pour les filtres d'environnement des étapes de build qui peuvent être utilisées pour filtrer les variables d'environnement pour d'autres interpréteurs de scripts ;
* [Pipeline: Keep Environment Step](https://plugins.jenkins.io/pipeline-keepenv-step) peut être utilisé pour filtrer les variables d'environnement inutilisées dans les pipelines afin d'éviter qu'une configuration globale ne fasse échouer les étapes de build.


