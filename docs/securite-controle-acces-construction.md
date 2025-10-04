# Contrôle d'Accès pour la Construction

<div class="couleur-introduction">
Tout comme le contrôle d'accès pour les utilisateurs, les constructions dans Jenkins s'exécutent avec une autorisation utilisateur associée. Par défaut, les compilations s'exécutent en tant qu'utilisateur SYSTEM interne qui dispose de toutes les autorisations pour s'exécuter sur n'importe quel nœud, créer ou supprimer des tâches, démarrer et annuler d'autres builds, etc.
</div>

!!! info " "
    L'autorisation Agent/Build nécessite la mise en place d'un contrôle d'accès pour les builds, car c'est l'authentification du build qui est vérifiée, et non celle de l'utilisateur qui lance le build.

Dans une configuration Jenkins avec un contrôle d'accès fin, cela n'est pas souhaitable. Par exemple, le fait d'exécuter les builds en tant que SYSTEM pourrait permettre aux utilisateurs ayant accès à la configuration et au lancement d'un job de lancer les builds de n'importe quel autre job à l'aide du [plugin Pipeline Build Step](https://plugins.jenkins.io/pipeline-build-step).

!!! info " "
    Certains plugins implémentent leur propre contrôle d'accès au-delà de l'autorisation de build. Ne pas utiliser l'autorisation de build ou exécuter des builds en tant que SYSTEM est un indicateur que des problèmes tels que celui décrit ci-dessus peuvent se produire.

Une solution à ce problème consiste à configurer le contrôle d'accès pour les builds. Cela est implémenté par le [plugin Authorize Project](https://plugins.jenkins.io/authorize-project), qui permet une configuration flexible de l'autorisation de build globale et par projet. Ce plugin n'est toutefois pas activement maintenu, et cette approche présente des limites connues.
