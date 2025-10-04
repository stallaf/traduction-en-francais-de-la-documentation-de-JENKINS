# Sécurisation des Compilations

<div class="couleur-introduction">
La création de logiciels est le principal cas d'utilisation de Jenkins. Pour ce faire, Jenkins invoque des scripts de compilation contenant du code spécifié par l'utilisateur, et garantir que cela se fasse de manière sécurisée doit être une préoccupation majeure pour les administrateurs Jenkins. Les sections suivantes traitent des problèmes potentiels et des stratégies pour y remédier.
</div>

## Isolation des compilations distribuées

Comme indiqué dans la section [Isolation du contrôleur](./securite-isolation-du-controleur.md), Jenkins doit être configuré pour les _compilations distribuées_ afin de garantir que les scripts de compilation ne s'exécutent pas sur le contrôleur, où ils pourraient accéder au répertoire d'accueil de Jenkins ou interférer avec le fonctionnement de Jenkins.

Une autre préoccupation est que les builds doivent être isolés les uns des autres : alors que les instances Jenkins simples qui construisent un seul projet peuvent fonctionner en toute sécurité avec un seul agent statique, les administrateurs d'instances plus complexes doivent déterminer si les builds doivent être isolés les uns des autres. Voici quelques exemples :

* Jenkins héberge plusieurs équipes qui ne sont pas autorisées à accéder aux projets ou au code source des autres ;
* Jenkins compile les demandes d'extraction/fusion soumises à des référentiels publics, par exemple sur GitHub.

Dans ces scénarios, les scripts de build s'exécutant sur les mêmes agents peuvent être en mesure d'accéder à des données liées à d'autres projets sans rapport, voire de manipuler leurs résultats de build. Même sans intention malveillante, des scripts de build mal configurés peuvent accidentellement supprimer des données sans rapport ou provoquer l'épuisement des ressources sur l'agent statique, retardant ainsi les builds.

Les solutions à ce problème sont les suivantes :

* Utilisez des [fournisseurs de cloud](https://plugins.jenkins.io/ui/search/?labels=cloud) qui créent un nouvel agent pour chaque build ;
* Des plugins tels que [Job Restrictions Plugin](https://plugins.jenkins.io/job-restrictions) limitent les tâches pouvant être exécutées sur certains nœuds.

## Abus de _pull requests_

Lorsque vous utilisez des dossiers d'organisation ou des [Pipelines multibranches](https://plugins.jenkins.io/workflow-multibranch/), Jenkins crée automatiquement de nouvelles _pull requests_ par défaut. Cela peut ouvrir la porte à des abus, en particulier lorsqu'une instance Jenkins construit des projets à partir de référentiels publics, par exemple :

* Des _pull requests_ indésirables qui déclenchent de nombreuses nouvelles constructions ;
* _Pull requests_ malveillantes qui modifient les scripts de build, par exemple pour miner des cryptomonnaies.

Les stratégies pour faire face à cela comprennent :

* Utilisez des fournisseurs de services cloud et limitez le nombre d'agents pouvant être lancés en parallèle. Cette mesure est facile à mettre en œuvre, mais peut retarder les builds légitimes en raison de contraintes de ressources ;
* Définissez des [délais d'expiration](https://plugins.jenkins.io/build-timeout) dans la définition du Pipeline (généralement `Jenkinsfile`), car ceux-ci ne peuvent pas être modifiés par les utilisateurs sans accès commit, sauf si les paramètres de confiance du Pipeline le permettent.
* Ne construisez pas automatiquement de nouvelles _pull requests_ et demandez plutôt aux _committers_ de les accepter, par exemple en utilisant des plugins tels que [GitHub Label Filter](https://plugins.jenkins.io/github-label-filter/).

## Gestion sécurisée des variables d'environnement

Les scripts et outils invoqués pendant la construction peuvent modifier leur comportement en fonction des variables d'environnement qui leur sont définies. Ces variables peuvent provenir de diverses sources, telles que les paramètres de construction, les variables d'environnement prédéfinies de l'agent, les fichiers lus à partir de l'espace de travail et injectés dans l'environnement, etc.

La section [Gestion des variables d'environnement](./securite-variables-environnement.md) traite ce sujet en détail, y compris les solutions.

