# Meilleures pratiques en matière de Pipelines

<div class="couleur-introduction">
Ce guide fournit une petite sélection des meilleures pratiques pour les Pipelines et souligne les erreurs les plus courantes.
</div>

L'objectif est d'orienter les auteurs et les responsables de la maintenance des Pipelines vers des modèles qui permettent une meilleure exécution des Pipelines et d'éviter les pièges dont ils pourraient ne pas avoir conscience. Ce guide ne prétend pas être une liste exhaustive de toutes les meilleures pratiques possibles en matière de Pipelines, mais vise plutôt à fournir un certain nombre d'exemples spécifiques utiles pour repérer les pratiques courantes. Utilisez-le comme un guide général et non comme un mode d'emploi extrêmement détaillé.

Ce guide est organisé par domaine, ligne directrice, puis liste des exemples spécifiques.

## Généralités

## Veillez à utiliser le code Groovy comme lien dans les Pipelines

Utilisez le code Groovy pour connecter un ensemble d'actions plutôt que comme fonctionnalité principale de votre Pipeline. En d'autres termes, au lieu de vous fier à la fonctionnalité du Pipeline (étapes Groovy ou Pipeline) pour faire avancer le processus de construction, utilisez des étapes uniques (telles que `sh`) pour accomplir plusieurs parties de la construction. À mesure que leur complexité augmente (quantité de code Groovy, nombre d'étapes utilisées, etc.), les Pipelines nécessitent davantage de ressources (CPU, mémoire, stockage) sur le contrôleur. Considérez le Pipeline comme un outil permettant d'accomplir une construction plutôt que comme le cœur d'une construction.

Exemple : utilisation d'une seule étape de construction Maven pour piloter la construction tout au long de son processus de construction/test/déploiement.

### Exécution de scripts shell dans Jenkins Pipeline

L'utilisation d'un script shell dans Jenkins Pipeline peut aider à simplifier les compilations en combinant plusieurs échelons en une seule étape. Le script shell permet également aux utilisateurs d'ajouter ou de mettre à jour des commandes sans avoir à modifier chaque échelon ou étape séparément.

Cette vidéo passe en revue l'utilisation d'un script shell dans Jenkins Pipeline et les avantages qu'il offre.
<iframe width="800" height="420" src="https://www.youtube.com/embed/mbeQWBNaNKQ" title="How to Run a Shell Script in Jenkins Pipeline" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Éviter le code Groovy complexe dans les Pipelines

Pour un Pipeline, le code Groovy s'exécute toujours sur le contrôleur, ce qui signifie qu'il utilise les ressources du contrôleur (mémoire et CPU). Il est donc essentiel de réduire la quantité de code Groovy exécuté par les Pipelines (y compris toutes les méthodes appelées sur les classes importées dans les Pipelines). Voici les exemples les plus courants de méthodes Groovy à éviter :

1. **JsonSlurper** : Cette fonction (et d'autres fonctions similaires telles que XmlSlurper ou readFile) peut être utilisée pour lire un fichier sur le disque, analyser les données de ce fichier dans un objet JSON et injecter cet objet dans un Pipeline à l'aide d'une commande telle que JsonSlurper().parseText(readFile(« $LOCAL_FILE »)). Cette commande charge deux fois le fichier local dans la mémoire du contrôleur et, si le fichier est très volumineux ou si la commande est exécutée fréquemment, elle nécessitera beaucoup de mémoire.
    1. Solution : au lieu d'utiliser JsonSlurper, utilisez une étape shell et renvoyez la sortie standard. Ce shell ressemblerait à ceci : `def JsonReturn = sh label: “”, returnStdout: true, script: “echo « $LOCAL_FILE »| jq « $PARSING_QUERY »”`. Cela utilisera les ressources de l'agent pour lire le fichier et $PARSING_QUERY aidera à analyser le fichier pour le réduire.
1. **HttpRequest** : Cette commande est souvent utilisée pour récupérer des données provenant d'une source externe et les stocker dans une variable. Cette pratique n'est pas idéale, car non seulement cette requête provient directement du contrôleur (ce qui peut donner des résultats incorrects pour des éléments tels que les requêtes HTTPS si le contrôleur ne dispose pas de certificats chargés), mais la réponse à cette requête est également stockée deux fois.
    1. Solution : utilisez une étape shell pour effectuer la requête HTTP à partir de l'agent, par exemple à l'aide d'un outil tel que `curl` ou `wget`, selon le cas. Si le résultat doit être intégré ultérieurement dans le Pipeline, essayez de filtrer le résultat autant que possible côté agent afin que seules les informations minimales requises soient transmises au contrôleur Jenkins.

### Réduire la répétition d'étapes similaires dans le pipeline

Combinez autant que possible les étapes dU pipeline en une seule étape afin de réduire la charge induite par son moteur d'exécution. Par exemple, si vous exécutez trois étapes shell consécutives, chacune de ces étapes doit être démarrée et arrêtée, ce qui nécessite la création et le nettoyage de connexions et de ressources sur l'agent et le contrôleur. Cependant, si vous placez toutes les commandes dans une seule étape shell, une seule étape doit être démarrée et arrêtée.

Exemple : au lieu de créer une série d'étapes `echo` ou `sh`, combinez-les en une seule étape ou un seul script.
  
### Éviter les appels à `Jenkins.getInstance`

L'utilisation de Jenkins.instance ou de ses méthodes d'accès dans un Pipeline ou une bibliothèque partagée indique une mauvaise utilisation du code dans ce Pipeline/cette bibliothèque partagée. L'utilisation des API Jenkins à partir d'une bibliothèque partagée non sandboxée signifie que la bibliothèque est à la fois partagée et est une sorte de plugin Jenkins. Vous devez être très prudent lorsque vous interagissez avec les API Jenkins à partir d'un Pipeline afin d'éviter de graves problèmes de sécurité et de performances. Si vous devez utiliser les API Jenkins dans votre compilation, l'approche recommandée consiste à créer un plugin minimal en Java qui implémente un wrapper sécurisé autour de l'API Jenkins à laquelle vous souhaitez accéder à l'aide de l'API Step de Pipeline. L'utilisation des API Jenkins à partir d'un fichier Jenkinsfile sandboxé signifie directement que vous avez probablement dû mettre en liste blanche des méthodes qui permettent à toute personne capable de modifier un Pipeline de contourner les protections du sandbox, ce qui représente un risque de sécurité important. La méthode mise en liste blanche est exécutée en tant qu'utilisateur système, disposant de tous les droits d'administrateur, ce qui peut conduire les développeurs à disposer de droits plus élevés que prévu.

Solution : La meilleure solution serait de contourner les appels effectués, mais s'ils doivent l'être, il serait préférable de mettre en œuvre un plugin Jenkins capable de collecter les données nécessaires.

### Nettoyage des anciennes versions Jenkins

En tant qu'administrateur Jenkins, la suppression des versions anciennes ou indésirables permet au contrôleur Jenkins de fonctionner efficacement. Si vous ne supprimez pas les anciennes versions, vous disposez de moins de ressources pour les versions plus récentes et plus pertinentes. Cette vidéo passe en revue l'utilisation de la directive [buildDiscarder](./pipeline-syntaxe.md#options) dans les tâches Pipeline individuelles. La vidéo passe également en revue le processus permettant de conserver des versions historiques spécifiques.

_Comment nettoyer les anciennes versions de Jenkins_
<iframe width="800" height="420" src="https://www.youtube.com/embed/_Z7BlaTTGlo" title="How to Clean up Old Jenkins Builds" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Utilisation de bibliothèques partagées

###Ne remplacez pas les étapes intégrées du Pipeline

Dans la mesure du possible, évitez les étapes personnalisées/remplacées du Pipeline. Le remplacement des étapes intégrées du Pipeline consiste à utiliser des bibliothèques partagées pour remplacer les API standard du Pipeline telles que `sh` ou `timeout`. Ce processus est dangereux, car les API du Pipeline peuvent changer à tout moment, ce qui peut entraîner la rupture du code personnalisé ou donner des résultats différents de ceux attendus. Lorsque le code personnalisé ne fonctionne plus en raison de modifications apportées à l'API du Pipeline, le dépannage est difficile car, même si le code personnalisé n'a pas changé, il peut ne plus fonctionner de la même manière après une mise à jour de l'API. Cela ne signifie pas qu'il continuera à fonctionner de la même manière après une mise à jour de l'API. Enfin, en raison de l'utilisation omniprésente de ces étapes dans les Pipelines, si quelque chose est codé de manière incorrecte/inefficace, les résultats peuvent être catastrophiques pour Jenkins.

## Éviter les fichiers de déclaration de variables globales volumineux

Les fichiers de déclaration de variables volumineux peuvent nécessiter beaucoup de mémoire pour peu ou pas d'avantages, car le fichier est chargé pour chaque Pipeline, que les variables soient nécessaires ou non. Il est recommandé de créer des fichiers de variables de petite taille qui ne contiennent que les variables pertinentes pour l'exécution en cours.

### Éviter les bibliothèques partagées très volumineuses

L'utilisation de bibliothèques partagées volumineuses dans les Pipelines nécessite de vérifier un fichier très volumineux avant que le Pipeline puisse démarrer et de charger la même bibliothèque partagée pour chaque tâche en cours d'exécution, ce qui peut entraîner une augmentation de la surcharge mémoire et un ralentissement du temps d'exécution.

## Réponses à d'autres questions fréquentes

### Gestion de la concurrence dans les Pipelines

Évitez de partager des espaces de travail entre plusieurs exécutions de Pipeline ou plusieurs Pipelines distincts. Cette pratique peut entraîner des modifications inattendues des fichiers au sein de chaque Pipeline ou le renommage des espaces de travail.

Idéalement, les volumes/disques partagés sont montés dans un emplacement séparé et les fichiers sont copiés depuis cet emplacement vers l'espace de travail actuel. Une fois la compilation terminée, les fichiers peuvent être recopiés s'ils ont été mis à jour.

Créez des conteneurs distincts qui génèrent les ressources nécessaires à partir de zéro (les agents de type cloud sont parfaits pour cela). La création de ces conteneurs garantit que le processus de compilation démarre à chaque fois depuis le début et est facilement reproductible. Si la création de conteneurs ne fonctionne pas, désactivez la concurrence sur le Pipeline ou utilisez le plugin Lockable Resources pour verrouiller l'espace de travail lorsqu'il est en cours d'exécution, afin qu'aucune autre compilation ne puisse l'utiliser pendant qu'il est verrouillé. **AVERTISSEMENT :** la désactivation de la concurrence ou le verrouillage de l'espace de travail lorsqu'il est en cours d'exécution peut entraîner le blocage des Pipelines lors de l'attente de ressources si celles-ci sont verrouillées de manière arbitraire.

Sachez également que ces deux méthodes ralentissent l'obtention des résultats des compilations par rapport à l'utilisation de ressources uniques pour chaque tâche.

### Éviter `NotSerializableException`

Le code du Pipeline est transformé en CPS afin que les Pipelines puissent reprendre après un redémarrage de Jenkins. Autrement dit, pendant que le Pipeline exécute votre script, vous pouvez arrêter Jenkins ou perdre la connectivité avec un agent. À son retour, Jenkins se souvient de ce qu'il était en train de faire et votre script Pipeline reprend son exécution comme s'il n'avait jamais été interrompu. Une technique connue sous le nom d'exécution « [continuation-passing style (CPS)](https://en.wikipedia.org/wiki/Continuation-passing_style) » joue un rôle clé dans la reprise des Pipelines. Cependant, certaines expressions Groovy ne fonctionnent pas correctement à la suite de la transformation CPS.

En arrière-plan, CPS repose sur la capacité à sérialiser l'état actuel du Pipeline ainsi que le reste du Pipeline à exécuter. Cela signifie que l'utilisation d'objets non sérialisables dans le Pipeline déclenchera une exception `NotSerializableException` lorsque le Pipeline tentera de persister son état.

Consultez la section [Incompatibilités entre les méthodes CPS du Pipeline](pipeline-incompatibilites-cps.md) pour plus de détails et quelques exemples de situations pouvant poser problème.

Vous trouverez ci-dessous des techniques permettant de garantir le bon fonctionnement du Pipeline.

#### Assurez-vous que les variables persistantes sont sérialisables

Les variables locales sont capturées dans le cadre de l'état du Pipeline pendant la sérialisation. Cela signifie que le stockage d'objets non sérialisables dans des variables pendant l'exécution du Pipeline entraînera le déclenchement d'une exception `NotSerializableException`.

#### N'assignez pas d'objets non sérialisables à des variables.

Une stratégie consiste à utiliser des objets non sérialisables pour toujours déduire leur valeur « juste à temps » au lieu de calculer leur valeur et de la stocker dans une variable.

#### Utilisation de `@NonCPS`

Si nécessaire, vous pouvez utiliser l'annotation `@NonCPS` pour désactiver la transformation CPS pour une méthode spécifique dont le corps ne s'exécuterait pas correctement s'il était transformé en CPS. Sachez simplement que cela signifie également que la fonction Groovy devra redémarrer complètement, car elle n'est pas transformée.

!!! info
    Les étapes de Pipeline asynchrones (telles que `sh` et `sleep`) sont toujours transformées en CPS et ne peuvent pas être utilisées dans une méthode annotée avec `@NonCPS`. En général, vous devez éviter d'utiliser des étapes de Pipeline dans des méthodes annotées avec `@NonCPS`.

#### Durabilité du pipeline

Il convient de noter que la modification de la durabilité du Pipeline peut empêcher le déclenchement d'une exception `NotSerializableException` qui aurait normalement dû se produire. En effet, la réduction de la durabilité du Pipeline via PERFORMANCE_OPTIMIZED signifie que l'état actuel du Pipeline est conservé beaucoup moins fréquemment. Par conséquent, il ne tente jamais de sérialiser les valeurs non sérialisables et aucune exception n'est donc déclenchée.
	
!!! warning
    Cette remarque a pour but d'informer les utilisateurs de la cause profonde de ce comportement. Il n'est pas recommandé de définir le paramètre de durabilité du Pipeline sur Performance Optimized uniquement pour éviter les problèmes de sérialisabilité.