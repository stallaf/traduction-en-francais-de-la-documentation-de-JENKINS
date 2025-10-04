# Sécurisation des Informations d'Identification SCM pour les Dossiers d'Organisation et les Pipelines Multibranches

<div class="couleur-introduction">
Les dossiers d'organisation et les Pipelines multibranches sont des types de projets Jenkins qui permettent à un seul élément de configuration de gérer et de créer automatiquement un nombre potentiellement important de tâches de Pipeline pour différents référentiels ou différentes branches d'un même référentiel. Ces projets interagissent avec des fournisseurs SCM externes de différentes manières, ce qui nécessite généralement que les informations d'identification avec accès à l'API du fournisseur SCM soient mises à la disposition de Jenkins (« scan credentials »). Les sections suivantes traitent des problèmes potentiels liés à la visibilité et à l'accessibilité de ces informations d'identification.
</div>

## Identifiants

Pour de nombreuses configurations de source de branche, telles que celles fournies par le plugin GitHub Branch Source et le plugin Bitbucket Branch Source, l'utilisateur Jenkins qui configure le projet n'a besoin de fournir qu'un seul identifiant, qui sera utilisé de deux manières :

* Par le contrôleur, pour effectuer des opérations de plugin telles que les analyses de dossiers d'organisation, l'indexation des branches Multibranch Pipeline, la modification des webhooks, les mises à jour du statut des commits, etc. Ces opérations interagissent généralement avec les API des fournisseurs SCM qui peuvent nécessiter des autorisations importantes allant au-delà de l'accès en lecture/écriture au contenu d'un référentiel.
* Par les nœuds, pour vérifier les référentiels en cours de construction par les tâches Pipeline enfants.

Les informations d'identification définies à l'aide de la portée globale sont disponibles pour tous les enfants de l'objet où elles sont définies. Cela signifie que toutes les informations d'identification disponibles pour les dossiers d'organisation et les pipelines multibranches seront également disponibles pour leurs tâches de pipeline enfants.

Tout utilisateur disposant d'une autorisation Job/Create dans un emplacement où des informations d'identification sont disponibles doit être considéré comme fiable pour utiliser ces informations de manière arbitraire (et dans de nombreux cas, Job/Configure suffit pour utiliser les informations d'identification de manière arbitraire). Lorsque vous utilisez Jenkins Pipeline, cette confiance est étendue à tout utilisateur capable de modifier `Jenkinsfile` et dont les modifications sont utilisées par Jenkins. Les utilisateurs autorisés à modifier `Jenkinsfile` dépendent de la configuration de la source de branche pour le pipeline, car ils sont capables de modifier `Jenkinsfile`. Par exemple, tout utilisateur capable d'écrire dans une branche du référentiel est considéré comme fiable par le plugin GitHub Branch Source.

## Implications

Les informations d'identification SCM pour les dossiers d'organisation et les pipelines multibranches peuvent être utilisées de manière arbitraire par toute personne capable de modifier le fichier `Jenkinsfile` d'un référentiel créé par ce dossier d'organisation ou ce pipeline multibranches.

!!! info " "
    Une mauvaise utilisation de `withCredentials` autour des étapes du pipeline qui exécutent du code contrôlé par l'utilisateur peut permettre à des utilisateurs non fiables de capturer les informations d'identification même s'ils n'ont pas la possibilité de modifier le fichier `Jenkinsfile`, mais cela n'a rien à voir avec les problèmes abordés ici. Par exemple, si un pipeline multibranche construit des PR à partir de fourches et que le fichier Jenkinsfile lie un identifiant à une étape `sh` qui exécute des tests dans le référentiel, un utilisateur peut déposer un PR à partir d'une fourche qui modifie les tests pour accéder à l'identifiant sans avoir à modifier le fichier `Jenkinsfile`.

    De nombreux plugins SCM offrent une option de configuration permettant d'effectuer le checkout du code source à l'aide de SSH. Ces options n'empêchent pas les informations d'identification de scan d'être utilisées dans le Pipeline d'autres manières, par exemple par une étape `withCredentials`.

Cela peut être indésirable de deux manières :

* L'accès en écriture à un référentiel peut être utilisé pour obtenir des informations d'identification qui donnent accès à d'autres référentiels ;
* L'accès en écriture à un référentiel peut être utilisé pour obtenir des informations d'identification avec des autorisations allant au-delà de l'accès en lecture/écriture au référentiel.

## Scénarios affectés

Les préoccupations ci-dessus peuvent ne pas avoir d'importance pour les utilisateurs disposant de modèles de sécurité simples.

Vous devez examiner attentivement les implications ci-dessus si :

* Vous utilisez un seul dossier d'organisation dans Jenkins pour créer plusieurs référentiels SCM pour lesquels les utilisateurs ayant accès à un référentiel SCM ne doivent pas avoir accès aux autres référentiels en cours de création ;
* Vos utilisateurs SCM doivent uniquement disposer d'un accès en lecture/écriture aux référentiels et ne doivent pas disposer d'autorisations supplémentaires.

Notez que si vous autorisez des utilisateurs non fiables à créer ou à configurer des tâches dans des emplacements où des informations d'identification sont disponibles, ce problème n'a pas d'importance, car ces utilisateurs peuvent utiliser directement les informations d'identification.

De plus, ces préoccupations ne s'appliquent qu'aux utilisateurs qui ne disposent pas de l'autorisation Job/Configure (ou d'autorisations plus puissantes) dans Jenkins, mais qui ont la possibilité de modifier les pipelines, par exemple parce qu'ils ont un accès en écriture aux référentiels qui contiennent des fichiers Jenkinsfiles. Si les utilisateurs non fiables n'ont pas la possibilité de modifier les fichiers Jenkins, par exemple parce que les pipelines multibranches sont configurés pour utiliser un pipeline centralisé via un plugin tel que [Multibranch Pipeline Inline Definition](https://plugins.jenkins.io/inline-pipeline) ou [Pipeline: Multibranch with defaults](https://plugins.jenkins.io/pipeline-multibranch-defaults), ces problèmes n'ont pas d'importance.

## Solutions et contournements

### Contournements génériques

Si votre principale préoccupation est d'empêcher les utilisateurs disposant d'un accès en lecture/écriture SCM à un référentiel d'utiliser les informations d'identification de scan pour accéder à d'autres référentiels créés par un dossier d'organisation, plusieurs options s'offrent à vous :

* En supposant que votre fournisseur SCM vous permette de créer des identifiants qui ne peuvent accéder qu'à des référentiels spécifiques (tels que les jetons d'accès personnels granulaires de GitHub), vous pouvez créer plusieurs dossiers d'organisation configurés pour créer des sous-ensembles distincts des référentiels en question, en définissant les identifiants restreints directement sur chaque dossier d'organisation dans Jenkins ;
* En fonction des fonctionnalités liées aux identifiants de votre fournisseur SCM, vous ne pourrez peut-être pas utiliser les dossiers d'organisation de manière sécurisée et devrez plutôt créer un pipeline multibranche pour chaque référentiel avec un identifiant défini qui autorise uniquement l'accès à ce référentiel spécifique.

Malheureusement, si votre principale préoccupation est d'empêcher les utilisateurs disposant d'un accès en lecture/écriture SCM à un référentiel d'utiliser les informations d'identification de scan pour obtenir des autorisations élevées, il n'existe pas de solution générique. Dans certains cas, vous pouvez supprimer des autorisations des informations d'identification afin de les rendre moins puissantes. Par exemple, vous pouvez supprimer les autorisations liées aux webhooks des informations d'identification, ce qui empêchera Jenkins de gérer automatiquement les webhooks, au profit d'une configuration manuelle des webhooks par un administrateur.

### Identifiants améliorés de l'application GitHub

Si vous utilisez le plugin GitHub Branch Source avec les identifiants de l'application GitHub, le plugin offre diverses fonctionnalités à partir de la version 1844.v4a_9883d49126 qui peuvent être utilisées pour améliorer la sécurité. Reportez-vous à la [documentation du plugin](https://github.com/jenkinsci/github-branch-source-plugin/blob/master/docs/github-app.adoc#enhancing-security-using-repository-access-strategies-and-default-permissions-strategies) pour plus de détails.


