# API d'Accès à Distance

Jenkins fournit une API d'accès à distance  utilisable par les machines pour ses fonctionnalités. Elle existe actuellement en trois versions :

1. XML.
2. JSON avec support JSONP. 
3. Python.

L'API d'accès à distance est proposée dans un style similaire à REST. Autrement dit, il n'y a pas de point d'entrée unique pour toutes les fonctionnalités, celles-ci sont plutôt disponibles sous l'URL `« .../api/ »`, où la partie `« ... »` correspond aux données sur lesquelles elle agit.

Par exemple, si votre installation Jenkins se trouve sur [https://ci.jenkins.io](https://ci.jenkins.io/), visiter [https://ci.jenkins.io/api/](https://ci.jenkins.io/api/) affichera uniquement les fonctionnalités API de niveau supérieur disponibles - principalement une liste des travaux configurés pour ce contrôleur Jenkins.
Ou si vous souhaitez accéder aux informations sur une version particulière, par exemple [https://ci.jenkins.io/job/Websites/job/jenkins.io/job/master/lastSuccessfulBuild/](https://ci.jenkins.io/job/Websites/job/jenkins.io/job/master/lastSuccessfulBuild/), puis allez sur [https://ci.jenkins.io/job/Websites/job/jenkins.io/job/master/lastSuccessfulBuild/api/ ](https://ci.jenkins.io/job/Websites/job/jenkins.io/job/master/lastSuccessfulBuild/api/) et vous verrez la liste des fonctionnalités disponibles pour cette version.

## Que pouvez-vous en faire ?

L'API distante peut être utilisée pour faire des choses comme celles-ci : 

1. Récupérez les informations de Jenkins pour la consommation programmatique. 
2. déclencher une nouvelle compilation.
3. Créer/copier des travaux.

## Soumettre des tâches

**Tâches sans paramètres**

Il vous suffit d'effectuer un POST HTTP sur `JENKINS_URL/job/JOBNAME/build`.

Cela fonctionne également pour les Pipelines multibranches et les dossiers d'organisation. Il déclencherait un scan.

**Tâches avec paramètres**

Exemple simple - Envoi "Paramètres de Chaîne":

``` sh title="SH"
curl JENKINS_URL/job/JOB_NAME/buildWithParameters \
  --user USER:TOKEN \
  --data id=123 --data verbosity=high
```

Un autre exemple - Envoi d'un "Paramètre de Fichier":

``` sh title="SH"
curl JENKINS_URL/job/JOB_NAME/buildWithParameters \
  --user USER:PASSWORD \
  --form FILE_LOCATION_AS_SET_IN_JENKINS=@PATH_TO_FILE
```

Le symbole «@» est important dans cet exemple. De plus, le chemin d'accès au fichier est un chemin absolu. Afin de faire fonctionner cette commande, vous devez configurer votre tâche Jenkins pour prendre un paramètre de fichier et faire correspondre le champ **localisation du fichier** dans la configuration du travail Jenkins avec la clé de l'option `--form`.

## API et sécurité distantes

Lorsque votre Jenkins est sécurisé, vous pouvez utiliser l'authentification de base HTTP pour authentifier les demandes d'API distantes. Voir [l'authentification des clients scriptés](./administration-systeme-authentification.md) pour plus de détails.

## Protection du CSRF

**REMARQUE :**  les jetons API sont préférables aux miettes pour la protection CSRF.

## Sélection XPATH

L'API XML prend en charge une sélection par XPath en utilisant le paramètre de requête «xpath». Ceci est pratique pour extraire des informations dans des environnements où la manipulation XML est fastidieuse (comme le script shell.) Voir le [problème #626](https://issues.jenkins.io/browse/JENKINS-626) pour un exemple de la façon de l'utiliser.
Voir `.../api/` sur votre serveur Jenkins pour des détails plus à jour.

### Exclusion XPath

Tout comme le paramètre de requête « xpath » ci-dessus, vous pouvez utiliser (éventuellement plusieurs) modèles de requête « exclude » pour exclure des nœuds du XML résultant. Tous les nœuds correspondant au XPath spécifié seront supprimés du XML.
Consultez `.../api/` sur votre serveur Jenkins pour obtenir des informations plus récentes.

## Contrôle de la profondeur

Parfois, l'API distante ne vous donne pas suffisamment d'informations en un seul appel. Par exemple, si vous souhaitez trouver la dernière construction réussie d'une vue donnée, vous vous rendrez compte que l'invocation à l'API distante de la vue ne vous donnera pas cela, et vous devrez appeler récursivement l'API distante de chaque projet. Le contrôle de la profondeur résout ce problème. Le contrôle de la profondeur est fondamentalement connecté au modèle de données Jenkins.

Le modèle de données que Jenkins maintient en interne peut être considéré comme une grande structure d'arbre, et lorsque vous passez un appel API distant, vous en obtenez un petit sous-arbre. Le sous-arbre est enraciné à l'objet pour lequel vous avez passé un appel API distant, et le sous-arbre est coupé au-delà de certaines profondeurs pour éviter de retourner trop de données. Vous pouvez ajuster ce comportement de coupure en spécifiant le paramètre de requête en profondeur. Lorsque vous spécifiez une valeur de profondeur positive, la coupure du sous-arbre se produit beaucoup plus tard.

Le résultat net est donc que si vous spécifiez une valeur de profondeur plus grande, vous verrez que l'API distante renvoie désormais plus de données. En raison de l'algorithme, cela fonctionne de telle manière que les données renvoyées par une plus grande valeur de profondeur incluent toutes les données renvoyées par une valeur de profondeur plus petite.

Voir `.../api/` sur votre serveur Jenkins pour des détails plus à jour.

## Emballages API Python

[Jenkinsapi](https://pypi.python.org/pypi/jenkinsapi), [Python-Jenkins](https://pypi.python.org/pypi/python-jenkins/), [api4jenkins](https://pypi.org/project/api4jenkins/), [aiojenkins](https://pypi.org/project/aiojenkins/) sont des emballages Python orientés objet pour l'API Python REST qui visent à fournir une manière plus conventionnellement pythonique de contrôler un serveur Jenkins. Il fournit une API de haut niveau contenant un certain nombre de fonctions pratiques. Les services actuellement proposés comprennent :

* Interroger les résultats des tests d'une compilation terminée ;
* Obtenir les objets représentant les dernières compilations d'une tâche ;
* Rechercher des artefacts à l'aide de critères simples ;
* Bloquer jusqu'à ce que les tâches soient terminées ;
* Installer les artefacts dans des structures de répertoires personnalisées ;
* Prise en charge de l'authentification pour les contrôleurs Jenkins ;
* Possibilité de rechercher des compilations par révision Subversion ;
* Possibilité d'ajouter/supprimer/interroger des agents Jenkins.

## Emballages API Ruby

[Jenkins API Client](https://rubygems.org/gems/jenkins_api_client) est un projet de wrapper Ruby orienté objet qui consomme l'API JSON de Jenkins et vise à fournir un accès à toutes les API distantes que Jenkins fournit. Il est disponible en Rubygem et peut être utile pour interagir avec les fonctionnalités du travail, du nœud, de la vue, du buildQueue et du système. Les services actuellement offerts comprennent : 

* Création de tâches en envoyant un fichier xml ou en spécifiant des paramètres comme options avec davantage d'options de personnalisation, notamment le contrôle des sources, les notifications, etc. ;
* Création de tâches (avec paramètres), arrêt des compilations, consultation des détails des compilations récentes, obtention des paramètres de compilation, etc. ;
* Liste des tâches disponibles dans Jenkins avec filtre par nom de tâche et filtre par statut de tâche ;
* Ajout/suppression de projets en aval ;
* Enchaînement de tâches, c'est-à-dire qu'à partir d'une liste de projets, chaque projet est ajouté en aval du projet précédent ;
* Obtention d'une sortie console progressive ;
* Authentification par nom d'utilisateur/mot de passe ;
* Interface de ligne de commande avec de nombreuses options fournies dans les bibliothèques ;
* Création, affichage des vues ;
* Ajout et suppression de tâches dans les vues ;
* Ajout/suppression d'agents Jenkins, consultation des détails des agents ;
* Obtention des tâches dans la file d'attente de compilation, ainsi que leur ancienneté, leur cause, leur raison, leur heure d'arrivée prévue, leur ID, leurs paramètres et bien plus encore ;
* Mise en veille, annulation de la mise en veille, redémarrage sécurisé, redémarrage forcé et attente jusqu'à ce que Jenkins soit disponible après un redémarrage ;
* Possibilité de lister les plugins installés/disponibles, d'obtenir des informations sur les plugins, d'installer/désinstaller des plugins et bien plus encore avec les plugins.

Le code source du projet [est ici](https://github.com/arangamani/jenkins_api_client).

## Emballages de l'API Java

La bibliothèque [Jenkins-Rest](https://github.com/cdancy/jenkins-rest) est un projet Java axé sur l'objet qui donne accès à l'API Jenkins REST programmatiquement à certaines API distantes que Jenkins fournit. Elle est construite à l'aide de [jclouds toolkit](https://jclouds.apache.org/) et peut facilement être étendue pour prendre en charge davantage de points de terminaison REST. Ses fonctionnalités évoluent et les utilisateurs sont invités à contribuer à l'ajout de nouveaux points de terminaison via des _pull requests_. Dans son état actuel, cette bibliothèque permet de soumettre une tâche, de suivre sa progression dans la file d'attente, de surveiller son exécution jusqu'à son achèvement et d'obtenir le statut de la compilation. Les services actuellement proposés comprennent :

* Définition des points de terminaison (propriété ou variable d'environnement) ;
* Authentification (basique et jeton API via propriété ou variable d'environnement) ;
* Prise en charge de Crumbs Issuer (détection automatique des crumbs) ;
* Prise en charge des dossiers ;
* API Jobs (build, buildInfo, buildWithParameters, config, create, delete, description, disable, enable, jobInfo, lastBuildNumber, lastBuildTimestamp et progressiveText) ;
* API du gestionnaire de plugins (installNecessaryPlugins, liste des plugins actuels) ;
* API de file d'attente (annuler, liste des éléments de la file d'attente, interroger un élément de la file d'attente) ;
* API de statistiques (charge globale) ;
* API des systèmes (systemInfo).

Le projet peut évoluer rapidement, cette liste n'est exacte qu'à la date d'écriture.

## Détection de la version Jenkins

Pour vérifier la version de Jenkins, chargez la page d'accueil ou n'importe quelle page `.../api/*` et vérifiez l'en-tête de réponse `X-Jenkins`. Celui-ci contient le numéro de version de Jenkins, par exemple « 1.404 ». C'est également un bon moyen de vérifier si une URL est une URL Jenkins.
