# Information Système

<div class="couleur-introduction">
La page <strong>Manage Jenkins >> System Information</strong> (Gérer Jenkins >> Informations système) fournit des informations détaillées sur ce qui est disponible sur ce contrôleur Jenkins :
</div>

![Page System Information (Informations système)](https://www.jenkins.io/doc/book/resources/managing/system-info-page.png)

## Propriétés système

La section **System Properties** (Propriétés système) répertorie les propriétés système qui peuvent être utilisées comme arguments de la ligne de commande utilisée pour démarrer Jenkins.

![Propriétés système](https://www.jenkins.io/doc/book/resources/managing/system-properties.png)

## Variables d'environnement

La section **Environment Variables** (Variables d'Environnement) affiche les variables d'environnement reconnues sur votre système avec leurs valeurs actuelles. Cela inclut les variables d'environnement définies par Jenkins qui sont disponibles sur tous les systèmes, ainsi que les variables d'environnement associées aux plugins installés sur ce contrôleur.

![Variables d'environnement](https://www.jenkins.io/doc/book/resources/managing/environment-variables.png)

## Plugins

La section Plugins fournit une liste complète de tous les plugins installés, y compris leurs noms, leurs versions et d'autres détails pertinents.

![Plugins](https://www.jenkins.io/doc/book/resources/managing/system-plugins.png)

## Utilisation de la mémoire

La section **Memory Usage** (Utilisation de la Mémoire) fournit une représentation graphique de l'utilisation de la mémoire du contrôleur, classée en trois périodes pour une meilleure analyse :

* **Short** (Court) : la période courte couvre l'utilisation de la mémoire au cours des dernières minutes, ventilée par secondes, pour une surveillance en temps réel ;
* **Medium** (Moyen) : la période moyenne affiche les tendances de la mémoire au cours de la dernière heure, ventilées par minutes, ce qui est utile pour détecter les fuites de mémoire progressives ;
* **Long** (Long) : la période longue illustre les modèles d'utilisation de la mémoire sur une journée ou un mois afin d'identifier les problèmes récurrents. Cette ventilation permet aux administrateurs de surveiller les tendances de la mémoire, de détecter les pics inhabituels et d'optimiser l'allocation des ressources.

![Utilisation de la mémoire longue](https://www.jenkins.io/doc/book/resources/managing/memory-usage.png
)
## Vidage de thread

La section Vidage de thread fournit un lien vers une page qui capture un instantané en temps réel de tous les threads actifs s'exécutant dans la JVM du contrôleur Jenkins. Cela est essentiel pour diagnostiquer les problèmes de performances, les blocages ou l'utilisation excessive du processeur.

![Vidage de thread](https://www.jenkins.io/doc/book/resources/managing/thread-dump.png)

Chaque entrée de thread comprend :

1. **Name** (Nom) : l'identifiant du thread.
2. **ID** : l'ID unique du thread.
3. **State** (État) : l'état d'exécution actuel (RUNNABLE, TIMED_WAITING, BLOCKED).
4. **Stack Trace** (Trace de pile) : la séquence d'appels menant à l'état actuel.

