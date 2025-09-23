# Pipeline à echelle variable

<div class="couleur-introduction">
L'un des principaux goulots d'étranglement de Pipeline est qu'il écrit <strong>FRÉQUEMMENT</strong> des données transitoires sur le disque afin que les Pipelines en cours d'exécution puissent gérer un redémarrage inattendu de Jenkins ou un crash du système. Cette durabilité est utile pour de nombreux utilisateurs, mais son coût en termes de performances peut poser problème.
</div>

Pipeline inclut désormais des fonctionnalités permettant aux utilisateurs d'améliorer les performances en réduisant la quantité de données écrites sur le disque et la fréquence à laquelle elles sont écrites, au détriment d'une légère perte de durabilité. Dans certains cas particuliers, les utilisateurs peuvent ne pas être en mesure de reprendre ou de visualiser les Pipelines en cours d'exécution si Jenkins s'arrête soudainement sans avoir eu la possibilité d'écrire les données.

Comme ces paramètres impliquent un compromis entre vitesse et durabilité, ils sont initialement facultatifs. Pour activer les modes optimisés pour les performances, les utilisateurs doivent définir explicitement un _paramètre de vitesse/durabilité_ pour les Pipelines. Si aucun choix explicite n'est fait, les Pipelines utilisent actuellement le paramètre « durabilité maximale » par défaut et écrivent sur le disque comme auparavant. Certaines optimisations d'E/S pour ce mode sont incluses dans les mêmes versions du plugin, mais les avantages sont beaucoup moins importants.

## Comment définir les paramètres de vitesse/durabilité ?

Il existe trois façons de configurer le paramètre de durabilité :

1. **Au niveau global**, vous pouvez choisir un paramètre de durabilité par défaut sous « Manage Jenkins » > « System », (« Gérer Jenkins » > « Système », )intitulé « Pipeline Speed/Durability Settings » (Paramètres de vitesse/durabilité duPpipeline). Vous pouvez les remplacer par les paramètres plus spécifiques ci-dessous.
2. **Par tâche de Pipeline :** en haut de la configuration de la tâche, sous « Custom Pipeline Speed/Durability Level » (Niveau de vitesse/durabilité du Pipeline personnalisé), cela remplace le paramètre global. Vous pouvez également utiliser une étape « propriétés » : le paramètre s'appliquera à la PROCHAINE exécution après l'exécution de l'étape (même résultat).
3. **Par branche pour un projet multibranches :** configurez une stratégie de propriété de branche personnalisée (sous le SCM) et ajoutez une propriété pour le niveau de vitesse/durabilité du Pipeline personnalisé. Cela remplace le paramètre global. Vous pouvez également utiliser une étape « propriétés » pour remplacer le paramètre, mais n'oubliez pas que vous devrez peut-être exécuter à nouveau l'étape pour annuler cette modification.

Les paramètres de durabilité prendront effet lors de la prochaine exécution applicable du Pipeline, et non immédiatement. Le paramètre sera affiché dans le journal.

## Des paramètres de durabilité plus performants peuvent-ils m'aider ?

* Oui, si votre contrôleur Jenkins utilise NFS, un stockage magnétique, exécute plusieurs pipelines à la fois ou affiche unE attente E/S élevé ;
* Oui, si vous exécutez des Pipelines comportant de nombreuses étapes (plusieurs centaines) ;
* Oui, si votre Pipeline stocke des fichiers volumineux ou des données complexes dans des variables du script, conserve cette variable dans son champ d'application pour une utilisation future, puis exécute des étapes. Cela peut sembler étrangement spécifique, mais cela arrive plus souvent que vous ne le pensez ;
    * Par exemple : étape `readFile` avec un fichier XML/JSON volumineux, ou utilisation des informations de configuration issues de l'analyse d'un tel fichier avec [l'une des étapes utilitaires](https://www.jenkins.io/doc/pipeline/steps/pipeline-utility-steps/#code-readjson-code-read-json-from-files-in-the-workspace) ;
    * Un autre modèle courant est un objet « résumé » contenant des données provenant de nombreuses branches (journaux, résultats ou statistiques). Cela est souvent visible, car vous y ajoutez fréquemment des éléments via des opérations d'ajout/append ou `Map.put()` ;
    * Les grands tableaux de données ou les `Maps` d'informations de configuration sont un autre exemple courant de cette situation ;
* Non, si vos Pipelines passent presque tout leur temps à attendre que quelques scripts shell/batch se terminent. Ce n'est PAS un bouton magique qui permet d'accélérer tout !
* Non, si les Pipelines écrivent d'énormes quantités de données dans les journaux (la journalisation reste inchangée) ;
* Non, si vous n'utilisez pas de Pipelines ou si votre système est surchargé par d'autres facteurs.
* Non, si vous n'activez pas les modes haute performance pour les Pipelines.

## À quoi dois-je renoncer avec ce « compromis » sur le paramètre de durabilité ?

**La stabilité de Jenkins LUI-MÊME n'est pas modifiée par ce paramètre**, qui s'applique uniquement aux Pipelines. Dans le pire des cas, le comportement des Pipelines revient à quelque chose qui s'apparente aux compilations Freestyle : l'exécution de Pipelines qui ne peuvent pas conserver les données transitoires peut ne pas pouvoir reprendre ou s'afficher dans Blue Ocean/Stage View/etc., mais les journaux s'afficheront. Cela n'affecte que les Pipelines en cours d'exécution et uniquement lorsque Jenkins est arrêté brusquement et non de manière normale avant qu'ils ne soient terminés.

**Un arrêt « en douceur »** consiste pour Jenkins à suivre un processus d'arrêt complet, par exemple en visitant http://[jenkins-server]/exit ou en utilisant des scripts d'arrêt de service normaux (si Jenkins fonctionne correctement). L'envoi d'un SIGTERM/SIGINT à Jenkins déclenchera un arrêt en douceur. Notez que les Pipelines en cours d'exécution n'ont pas besoin d'être terminés (vous n'avez pas besoin d'utiliser /safeExit pour arrêter).

**Un arrêt « brutal »** se produit lorsque Jenkins ne parvient pas à effectuer les processus d'arrêt normaux. Cela peut se produire si le processus est interrompu de force. Les causes les plus courantes sont l'utilisation de SIGKILL pour interrompre le processus Jenkins ou la suppression du conteneur/de la machine virtuelle exécutant Jenkins. Le simple fait d'arrêter ou de mettre en pause le conteneur/la VM ne provoquera pas cela, tant que le processus Jenkins est en mesure de redémarrer. Un arrêt brutal peut également se produire en raison de pannes catastrophiques du système d'exploitation, notamment lorsque le OOMKiller Linux attaque le processus Java Jenkins pour libérer de la mémoire.

**Écritures atomiques** : tous les paramètres, à **l'exception** de « durabilité maximale », évitent actuellement les écritures atomiques. Cela signifie que si le système d'exploitation exécutant Jenkins tombe en panne, les données mises en mémoire tampon pour être écrites sur le disque ne seront pas vidées et seront perdues. Ce cas de figure est assez rare, mais peut se produire à la suite d'opérations de conteneurisation ou de virtualisation qui arrêtent le système d'exploitation ou déconnectent le stockage. Généralement, ces données sont vidées assez rapidement sur le disque, de sorte que la fenêtre de perte de données est brève. Sous Linux, cette écriture sur le disque peut être forcée en exécutant la commande « sync ». Dans certains cas rares, cela peut également entraîner l'impossibilité de charger une compilation.

## Configuration requise pour utiliser les paramètres de durabilité

* Jenkins LTS 2.73+ ou supérieur (ou une version hebdomadaire 2.62+) ;
* Pour **tous** les plugins Pipeline ci-dessous, au moins la version minimale spécifiée doit être installée ;
    * Pipeline : API (workflow-api) v2.25 ;
    * Pipeline : Groovy (workflow-cps) v2.43 ;
    * Pipeline : Job (workflow-job) v2.17 ;
    * Pipeline : API de prise en charge (workflow-support) v2.17 ;
    * Pipeline : Multibranch (workflow-multibranch) v2.17 - facultatif, uniquement nécessaire pour activer ce paramètre pour les pipelines multibranches.
* Redémarrez le contrôleur pour utiliser les plugins mis à jour - remarque : vous devez tous les installer pour en tirer pleinement parti.

## Quels sont les paramètres de durabilité ?

* Mode optimisé pour les performances (« PERFORMANCE_OPTIMIZED ») : réduit **considérablement** les E/S disque. Si les Pipelines ne se terminent pas ET que Jenkins n'est pas arrêté correctement, ils peuvent perdre des données et se comporter comme des projets Freestyle — voir les détails ci-dessus ;
* Survivabilité/durabilité maximale (« MAX_SURVIVABILITY ») : se comporte exactement comme le Pipeline auparavant, option la plus lente. Utilisez-la pour exécuter vos Pipelines les plus critiques ;
* Moins durable, un peu plus rapide (« SURVIVABLE_NONATOMIC ») : écrit les données à chaque étape, mais évite les écritures atomiques. Ce mode est plus rapide que le mode de durabilité maximale, en particulier sur les systèmes de fichiers en réseau. Il comporte un léger risque supplémentaire (détails ci-dessus sous « À quoi dois-je renoncer : les écritures atomiques »).

## Meilleures pratiques et conseils suggérés pour les paramètres de durabilité

* Utilisez le mode « optimisé pour les performances » pour la plupart des Pipelines, en particulier les Pipelines de test de compilation de base ou tout ce qui peut simplement être réexécuté si nécessaire ;
* Utilisez le mode « durabilité maximale » ou « moins durable » pour les Pipelines lorsque vous avez besoin d'un enregistrement garanti de leur exécution (audit). Ces deux modes enregistrent chaque étape exécutée. Par exemple, utilisez l'un de ces deux modes lorsque :
    * vous disposez d'un Pipeline qui modifie l'état d'une infrastructure critique ;
    * vous effectuez un déploiement en production ;
* Définissez une valeur par défaut globale (voir ci-dessus) de « optimisation des performances » pour le paramètre de durabilité, puis, si nécessaire, définissez « durabilité maximale » pour des tâches de Pipeline spécifiques ou des branches de Pipeline multibranches (« master » ou branches de publication) ;
* Vous pouvez forcer un Pipeline à conserver les données en le mettant en pause.

## Autres suggestions de mise à l'échelle

* Utilisez les fonctions annotées @NonCPS pour les tâches plus complexes. Cela implique un traitement, une logique et des transformations plus élaborés. Vous pouvez ainsi tirer parti des fonctionnalités supplémentaires de Groovy et des fonctionnalités fonctionnelles pour obtenir un code plus puissant, plus concis et plus performant ;
    * Cela fonctionne toujours sur le contrôleur, donc soyez conscient de la complexité du travail, mais c'est beaucoup plus rapide que le code Pipeline natif car il n'offre pas de durabilité et utilise un modèle d'exécution plus rapide. Néanmoins, soyez attentif au coût CPU et déchargez les exécuteurs lorsque le coût devient trop élevé ;
    * Les fonctions @NonCPS peuvent utiliser un sous-ensemble beaucoup plus large du langage Groovy, tel que les itérateurs et les fonctionnalités fonctionnelles, ce qui les rend plus concises et rapides à écrire ;
    * Les fonctions @NonCPS **ne doivent pas utiliser** de étapes Pipeline en interne, mais vous pouvez stocker le résultat d'une étape Pipeline dans une variable et l'utiliser comme entrée pour une fonction @NonCPS ;
        * **Compris :** l'utilisation d'une étape ne garantit pas la génération d'une erreur (une demande de fonctionnalité est en cours pour implémenter cela), mais vous ne devez pas vous fier à ce comportement. Vous pourriez constater un traitement incorrect des exceptions ;
    *   Alors que le Pipeline normal est limité aux variables locales sérialisables, les fonctions @NonCPS peuvent utiliser en interne des types plus complexes et non sérialisables (par exemple, des comparateurs d'expressions régulières, etc.). Les paramètres et les types de retour doivent toutefois rester sérialisables ;
        * **Compris** : les utilisations incorrectes ne sont pas garanties de générer une erreur avec le Pipeline normal (les optimisations peuvent masquer le problème), mais il n'est pas sûr de se fier à ce comportement ;
    * **Compris** : lors de l'utilisation de fonctions @NonCPS en cours d'exécution, l'erreur réelle peut parfois être masquée par le Pipeline, ce qui crée un message d'erreur confus. Pour remédier à cela, utilisez un bloc `try/catch` et éventuellement une commande `echo` pour afficher le message d'erreur en texte brut dans le bloc `catch` ;
    * **Dans la mesure du possible, exécutez Jenkins avec un stockage SSD rapide plutôt qu'avec des disques durs. Cela peut faire une _énorme_ différence** ;
* En général, essayez d'adapter l'outil à la tâche à accomplir. Envisagez d'écrire de courts scripts Shell/Batch/Groovy/Python lorsque vous exécutez un processus complexe à l'aide d'un agent de build. Parmi les bons exemples, citons le traitement de données, la communication interactive avec les API REST et l'analyse/la création de modèles de fichiers XML ou JSON volumineux. Les étapes `sh` et `bat` sont utiles pour les invoquer, en particulier avec `returnStdout: true` pour renvoyer la sortie de ce script et l'enregistrer en tant que variable (Scripted Pipeline) ;
* Le Pipeline DSL n'est pas conçu pour des tâches de mise en réseau et de calcul arbitraires, mais pour la création de scripts CI/CD ;
* Utilisez les dernières versions des plugins Pipeline et Script Security, le cas échéant. Ceux-ci incluent des améliorations régulières des performances ;
* Essayez de simplifier le code Pipeline en réduisant le nombre d'étapes exécutées et en utilisant un code Groovy plus simple pour les pipelines scriptés ;
* Consolidez les étapes séquentielles du même type si possible, par exemple en utilisant une étape Shell pour invoquer un script d'aide plutôt que d'exécuter plusieurs étapes ;
* Essayez de limiter la quantité de données écrites dans les journaux par les Pipelines. Si vous écrivez plusieurs Mo de données de journal, par exemple à partir d'un outil de compilation, envisagez plutôt de les écrire dans un fichier externe, de les compresser et de les archiver en tant qu'artefact de compilation ;
* Lorsque vous utilisez Jenkins avec plus de 6 Go de mémoire, [utilisez les options de réglage de la collecte des déchets suggérées](https://www.jenkins.io/blog/2016/11/21/gc-tuning/) afin de minimiser les temps de pause et la surcharge liés à la collecte des déchets ;

