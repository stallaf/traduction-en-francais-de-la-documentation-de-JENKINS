# Extension avec des bibliothèques partagées

Comme le Pipeline est adopté pour de plus en plus de projets dans une organisation, les modèles communs sont susceptibles d'émerger. Souvent, il est utile de partager des parties de Pipelines entre divers projets pour réduire les redondances et garder le code "SEC" [^1].

[^1]: [en.wikipedia.org/wiki/Don't_repeat_yourself](https://en.wikipedia.org/wiki/Don't_repeat_yourself)

Pipeline prend en charge la création de "bibliothèques partagées" qui peuvent être définies dans des référentiels de contrôle de source externe et chargés dans des Pipelines existants.

<iframe width="800" height="420" src="https://www.youtube.com/embed/Wj-weFEsTb0" title="Getting Started With Shared Libraries in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Définir des bibliothèques partagées

Une bibliothèque partagée est définie par un nom, une méthode de récupération du code source (par exemple via SCM) et, éventuellement, une version par défaut. Le nom doit être un identifiant court, car il sera utilisé dans des scripts.

La version peut être tout ce qui est compris par ce SCM ; par exemple, les branches, les balises et les hachages de _commit_ fonctionnent tous pour Git. Vous pouvez également déclarer si les scripts doivent demander explicitement cette bibliothèque (détaillé ci-dessous) ou si elle est présente par défaut. De plus, si vous spécifiez une version dans la configuration Jenkins, vous pouvez empêcher les scripts de sélectionner une version _différente_.

La meilleure façon de spécifier le SCM est d'utiliser un plugin SCM qui a été spécialement mis à jour pour prendre en charge une nouvelle API permettant de vérifier une version nommée arbitraire (option _Modern SCM_). À l'heure où nous écrivons ces lignes, les dernières versions des plugins Git et Subversion prennent en charge ce mode ; les autres devraient suivre.

Si votre plugin SCM n'a pas été intégré, vous pouvez sélectionner _Legacy SCM_ et choisir n'importe quelle option proposée. Dans ce cas, vous devez inclure `${library.yourLibName.version} ` quelque part dans la configuration du SCM, afin que lors du _checkout_, le plugin développe cette variable pour sélectionner la version souhaitée. Par exemple, pour Subversion, vous pouvez définir _l'URL du référentiel_ [sur svnserver/project/${library.yourLibName.version}](https://svnserver/project/$%7Blibrary.yourLibName.version%7D), puis utiliser des versions telles que `trunk` ou `branches/dev` ou `tags/1.0`.

### Structure de répertoire

La structure du répertoire d'un référentiel de bibliothèque partagée est la suivante :

``` console
(root)
+- src                     # Groovy source files
|   +- org
|       +- foo
|           +- Bar.groovy  # pour la classe org.foo.Bar
+- vars
|   +- foo.groovy          # pour la variable  globale 'foo'
|   +- foo.txt             # aide pour la variable 'foo'
+- resources       # fichiers de ressources (bibliothèques externes uniquement)
|   +- org
|       +- foo
|           +- bar.json    # Données d'aide statiques pour org.foo.Bar
```

Le répertoire `src` devrait ressembler à une structure de répertoire source Java standard. Ce répertoire est ajouté au _classpath_ lors de l'exécution de Pipelines.

Le répertoire `vars` héberge des fichiers de script exposés en tant que variable dans les Pipelines. Le nom du fichier est le nom de la variable dans le Pipeline. Donc, si vous aviez un fichier appelé `vars/log.groovy` avec une fonction comme `def info(message)…` dedans, vous pouvez accéder à cette fonction comme `log.info "hello world" ` dans le Pipeline. Vous pouvez mettre autant de fonctions que vous le souhaitez à l'intérieur de ce fichier. Lisez la suite ci-dessous pour plus d'exemples et d'options.

Le nom de base de chaque fichier `.groovy` doit être un identifiant Groovy (~ Java), conventionnellement `camelCased`. La correspondance `.txt`, si elle est présente, peut contenir de la documentation, traitée via le [formateur de balisage](https://wiki.jenkins.io/display/JENKINS/Markup+formatting) configuré du système (il peut donc vraiment s'agir de HTML, de marquage, etc., bien que l'extension `.txt` soit requise). Cette documentation ne sera visible que sur les pages de référence des variables globales accessibles à partir de la barre latérale de navigation des travaux de Pipeline qui importent la bibliothèque partagée. De plus, ces tâches doivent avoir été exécutées avec succès au moins une fois avant que la documentation de la bibliothèque partagée ne soit générée.

Les fichiers source Groovy dans ces répertoires obtiennent la même «transformation CPS» que dans le Pipeline scripté.

Un répertoire `resources` permet d'utiliser l'étape `libraryResource` à partir d'une bibliothèque externe pour charger des fichiers non Groovy associés. Actuellement, cette fonctionnalité n'est pas prise en charge pour les bibliothèques internes.

Les autres répertoires sous la racine sont réservés à des améliorations futures.

### Bibliothèques partagées globales

Il existe plusieurs endroits où les bibliothèques partagées peuvent être définies, selon le cas d'utilisation. Accédez à **_Manage Jenkins » System » Global Trusted Pipeline Libraries_** (Gérer Jenkins » Système » Bibliothèque Globale de Pipelines Fiables) pour configurer autant de bibliothèques que nécessaire.

![Page de configuration du système Jenkins, accessible en naviguant à partir du tableau de bord pour gérer Jenkins puis vers le système, affichant des sections pour les bibliothèques de Pipelines de confiance globales et les bibliothèques de Pipelines non fiables globales. Chaque section fournit des options pour ajouter des bibliothèques partageables.](https://www.jenkins.io/doc/book/resources/pipeline/add-global-pipeline-libraries.png)

Étant donné que ces bibliothèques seront utilisables à l'échelle globale, tout Pipeline du système peut utiliser les fonctionnalités implémentées dans ces bibliothèques.

Ces bibliothèques peuvent être marquées «de confiance», ce qui signifie qu'elles peuvent exécuter toutes les méthodes dans Java, Groovy, Jenkins Internal API, Jenkins Plugins ou bibliothèques tierces. Cela vous permet de définir des bibliothèques qui encapsulent les API individuellement dangereuses dans un wrapper de niveau supérieur qui est sûr pour une utilisation à partir de n'importe quel Pipeline. Attention : **toute personne capable d'envoyer des _commits_ vers ce référentiel SCM pourrait obtenir un accès illimité à Jenkins**. Vous devez disposer de l'autorisation _Overall/RunScripts_ pour configurer ces bibliothèques (celle-ci est généralement accordée aux administrateurs Jenkins).

Si vous ne disposez que des autorisations **_Overall/Manage_** (Globale/Gérer), vous pouvez toujours configurer des bibliothèques globales, mais uniquement celles qui ne sont « pas fiables » et qui s'exécutent dans le bac à sable Groovy, tout comme les scripts Pipeline classiques.

### Bibliothèques partagées au niveau du dossier

Tout dossier créé peut avoir des bibliothèques partagées associées. Ce mécanisme permet la portée de bibliothèques spécifiques à tous les Pipelines à l'intérieur du dossier ou du sous-dossier.

Les bibliothèques au niveau du dossier sont toujours « non fiables ».

### Bibliothèques partagées automatiques

D'autres plugins peuvent ajouter des moyens de définir des bibliothèques à la volée. Par exemple, le plugin [Pipeline: GitHub Groovy Libraries](https://plugins.jenkins.io/pipeline-github-lib) permet à un script d'utiliser une bibliothèque non fiable nommée comme `github.com/someorg/somerepo` sans aucune configuration supplémentaire. Dans ce cas, le dépôt GitHub spécifié serait chargé, à partir de la branche `master`, à l'aide d'un _checkout_ anonyme.

## Utilisation des bibliothèques

Les bibliothèques partagées marquées comme _chargées implicitement_ permettent aux Pipelines d'utiliser immédiatement les classes ou les variables globales définies par ces bibliothèques. Pour accéder à d'autres bibliothèques partagées, le fichier `Jenkinsfile` doit utiliser l'annotation `@Library`, en spécifiant le nom de la bibliothèque.

![La page de configuration du système Jenkins est accessible en naviguant depuis le tableau de bord vers Manage Jenkins, puis vers System, où s'affichent les options permettant de configurer une bibliothèque de pipelines globale de confiance. Elle comprend des champs pour le nom de la bibliothèque, la version par défaut, des cases à cocher pour « Autoriser le remplacement de la version par défaut » et « Inclure les modifications @Library dans les modifications récentes du travail », ainsi qu'un menu déroulant permettant de sélectionner la méthode de récupération (Modern SCM ou Legacy SCM).](https://www.jenkins.io/doc/book/resources/pipeline/configure-global-pipeline-library.png)

``` groovy
@Library('my-shared-library') _
/* Utilisation d'un spécificateur de version, tel que branch, tag, etc. */
@Library('my-shared-library@1.0') _
/* Accéder à plusieurs bibliothèques avec une seule instruction */
@Library(['my-shared-library', 'otherlib@abc1234']) _
```

L'annotation peut être n'importe où dans le script où une annotation est autorisée par Groovy. Lorsque vous faites référence aux bibliothèques de classe (avec `src/` répertoires), conventionnellement, l'annotation va sur une instruction `import`:

``` groovy
@Library('somelib')
import com.mycorp.pipeline.somelib.UsefulClass
```

!!! tip "Astuce"
    Pour les bibliothèques partagées qui définissent uniquement les variables globales (`vars/`), ou un `Jenkinsfile` qui n'a besoin que d'une variable globale, le modèle [d'annotation](http://groovy-lang.org/objectorientation.html#_annotation) `@Library('my-shared-library') _` peut être utile pour garder le code concis. Essentiellement, au lieu d'annoter une instruction `import` inutile, le symbole `_` est annoté.

    Il n'est pas recommandé de `import` une variable/fonction globale, car cela obligera le compilateur à interpréter les champs et les méthodes comme `static` même s'ils devaient être personnalisés. Le compilateur Groovy dans ce cas peut produire des messages d'erreur déroutants.

Les bibliothèques sont résolues et chargées pendant la _compilation_ du script, avant qu'il ne commence à s'exécuter. Cela permet au compilateur Groovy de comprendre la signification des symboles utilisés dans la vérification de type statique et autorise leur utilisation dans les déclarations de type du script, par exemple :

``` groovy
@Library('somelib')
import com.mycorp.pipeline.somelib.Helper

int useSomeLib(Helper helper) {
    helper.prepare()
    return helper.count()
}

echo useSomeLib(new Helper('some text'))
```

Cependant, les variables globales sont résolues au moment de l'exécution.

Cette vidéo passe en revue l'utilisation des fichiers de ressources d'une bibliothèque partagée. Un lien vers le référentiel d'exemples utilisé est également fourni dans la [description de la vidéo](https://youtu.be/eV7roTXrEqg).

<iframe width="800" height="420" src="https://www.youtube.com/embed/eV7roTXrEqg" title="Using Resource Files From a Jenkins Shared Library" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Chargement des bibliothèques dynamiquement

À partir de la version 2.7 du plugin _Pipeline: Shared Groovy Libraries_, il existe une nouvelle option pour charger des bibliothèques (non implicites) dans un script : une étape `library` qui charge une bibliothèque dynamiquement, à tout moment pendant la version.

Si vous êtes uniquement intéressé à utiliser des variables/fonctions globales (à partir du répertoire `vars/`), la syntaxe est assez simple :

``` groovy
library 'my-shared-library'
```

Par la suite, toutes les variables globales de cette bibliothèque seront accessibles au script.

L'utilisation de classes à partir de `src/` répertoire est également possible, mais plus délicate. Alors que l'annotation`@Library` prépare le «classpath» du script avant la compilation, au moment où une étape `library` est rencontrée, le script a déjà été compilé. Par conséquent, vous ne pouvez faire `import` ou faire référence de manière « statique » aux types de la bibliothèque.

``` groovy
library('my-shared-library').com.mycorp.pipeline.Utils.someStaticMethod()
```

Vous pouvez également accéder aux champs `static` et appeler les constructeurs comme s'il s'agissait de méthodes `static` nommées `new` :

``` groovy
def useSomeLib(helper) { // dynamic: cannot declare as Helper
    helper.prepare()
    return helper.count()
}

def lib = library('my-shared-library').com.mycorp.pipeline // preselect the package

echo useSomeLib(lib.Helper.new(lib.Constants.SOME_TEXT))
```

### Versions de bibliothèque

La « **_Default version_** » (version par défaut ) d'une bibliothèque partagée configurée est utilisée lorsque l'option « **_Load implicitly"_** » (Charger implicitement) est cochée ou si un Pipeline fait référence à la bibliothèque uniquement par son nom, par exemple `@Library(“my-shared-library”) _`. Si aucune « **_Default version_** » (version par défaut ) n'est **pas** définie, le Pipeline doit spécifier une version, par exemple `@Library(“my-shared-library@master”) _`.

Si l'option « **_Allow default version to be overridden_** » (Autoriser le remplacement de la version par défaut) est activée dans la configuration de la bibliothèque partagée, une annotation @Library peut également remplacer une version par défaut définie pour la bibliothèque. Cela permet également à une bibliothèque avec « **_Load implicitly"_** » (Charger implicitement) d'être chargée à partir d'une version différente si nécessaire.

Lorsque vous utilisez l'étape `library`, vous pouvez également spécifier une version :

``` groovy
library 'my-shared-library@master'
```

Puisqu'il s'agit d'une étape régulière, cette version pourrait être calculée plutôt qu'une constante comme avec l'annotation ; par exemple :

``` groovy
library "my-shared-library@$BRANCH_NAME"
```
cChargerait une bibliothèque en utilisant la même branche SCM que le `Jenkinsfile` multibranche. Comme autre exemple, vous pouvez choisir une bibliothèque par paramètre :

``` groovy
properties([parameters([string(name: 'LIB_VERSION', defaultValue: 'master')])])
library "my-shared-library@${params.LIB_VERSION}"
```

Notez que l'étape `library` ne peut pas être utilisée pour remplacer la version d'une bibliothèque chargée implicitement. Elle est déjà chargée au moment où le script démarre, et une bibliothèque d'un nom donné ne peut pas être chargée deux fois.

### Méthode de récupération

La meilleure façon de spécifier le SCM consiste à utiliser un plugin SCM qui a été spécialement mis à jour pour prendre en charge une nouvelle API permettant de vérifier une version nommée arbitraire (option **Modern SCM**). À l'heure où nous écrivons ces lignes, les dernières versions des plugins Git et Subversion prennent en charge ce mode.

![Page de configuration du système Jenkins accessible en naviguant à partir du tableau de bord pour gérer Jenkins, puis vers le système, affichant des paramètres pour ajouter une bibliothèque de Pipelines de confiance globales. Il comprend des options comme suit : Le nom de la bibliothèque est défini sur `` My-Shared-Library '', la version par défaut de «Main» et les cases à cocher pour «Autoriser la version par défaut pour être remplacée» et «inclure @Library Changes dans le travail Les modifications récentes» sont vérifiées. La méthode de récupération est définie sur SCM moderne, avec le champ de gestion de code source défini sur «GIT», y compris des options telles que le référentiel de projets, les informations d'identification et les comportements comme la découverte de branche.](https://www.jenkins.io/doc/book/resources/pipeline/global-pipeline-library-modern-scm.png)

#### Héritage SCM

Les plugins SCM qui n'ont pas encore été mis à jour pour prendre en charge les fonctionnalités les plus récentes requises par les bibliothèques partagées, peuvent toujours être utilisées via l'option **_Legacy SCM_** (SCM héritée). Dans ce cas, incluez `${library.yourlibrarynamehere.version}` partout où une branche/balise/reférence est configurée pour ce plugin SCM particulier. Cela garantit que lors de la vérification du code source de la bibliothèque, le plugin SCM élargira cette variable pour vérifier la version appropriée de la bibliothèque.

![Page de configuration du système Jenkins accessible en naviguant à partir du tableau de bord pour gérer Jenkins, puis vers le système, affichant des paramètres pour ajouter une bibliothèque de Pipelines de confiance globale. Il comprend des options comme suit : Le nom de la bibliothèque est défini sur " My-Shared-Library'', la version par défaut en "stable'' et cocher les cases pour "Autoriser la version par défaut pour être remplacée","inclure @Library Changes dans les modifications récentes du travail" et "Les versions de cache obtenues sur le contrôleur pour la récupération rapide '' sont vérifiées. Le «temps de rafraîchissement du cache en minutes» est défini sur 37. La méthode de récupération est définie sur Legacy SCM, avec le champ de gestion de code source défini sur «Subversion». L'URL du référentiel est paramétrée en tant que svn: //svn.example.com/pipeline-bibrary/branches/$ {biblirary.my-shared-library.version}. Aucune information d'identification n'est sélectionnée. Les options supplémentaires incluent le répertoire du module local. (Répertoire actuel), la profondeur du référentiel définie sur «Infinity» et les options pour «ignorer les externes» et «Annuler le processus sur l'échec externe» sont activées.](https://www.jenkins.io/doc/book/resources/pipeline/global-pipeline-library-legacy-scm.png)

#### Récupération dynamique

Si vous spécifiez uniquement un nom de bibliothèque (éventuellement avec la version après `@`) à l'étape `library`, Jenkins recherchera une bibliothèque préconfigurée de ce nom. (Ou dans le cas d'une bibliothèque automatique `github.com/owner/repo`, il le chargera.)

Mais vous pouvez également spécifier la méthode de récupération dynamiquement, auquel cas il n'est pas nécessaire que la bibliothèque ait été prédéfinie dans Jenkins. Voici un exemple :

``` groovy
library identifier: 'custom-lib@master', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'git@git.mycorp.com:my-jenkins-utils.git',
   credentialsId: 'my-private-key'])
   ```

Il est préférable de se référer à la **syntaxe des Pipelines** pour la syntaxe précise de votre SCM.

Notez que la version de la bibliothèque doit être spécifiée dans ces cas.

## Bibliothèques d'écriture

Au niveau de la base, tout [code Groovy](http://groovy-lang.org/syntax.html) valide est correct pour une utilisation. Différentes structures de données, méthodes d'utilité, etc., telles que :

``` groovy
// src/org/foo/Point.groovy
package org.foo

// point dans l'espace 3D
class Point {
  float x,y,z
}
```

### Accéder aux étapes

Les classes de bibliothèque ne peuvent pas appeler directement des étapes telles que `sh` ou `git`. Elles peuvent toutefois implémenter des méthodes, en dehors du champ d'application d'une classe englobante, qui à leur tour invoquent des étapes de Pipeline, par exemple :

``` groovy
// src/org/foo/Zot.groovy
package org.foo

def checkOutFrom(repo) {
  git url: "git@github.com:jenkinsci/${repo}"
}

return this
```

Qui peut ensuite être appelé à partir d'un Pipeline scripté :

``` groovy
def z = new org.foo.Zot()
z.checkOutFrom(repo)
```

Cette approche a des limites ; par exemple, elle empêche la déclaration d'une superclasse.

Alternativement, un ensemble de `steps` peut être passé de manière explicite à l'aide de `this` à une classe de bibliothèque, dans un constructeur, ou une seule méthode :

``` groovy
package org.foo
class Utilities implements Serializable {
  def steps
  Utilities(steps) {this.steps = steps}
  def mvn(args) {
    steps.sh "${steps.tool 'Maven'}/bin/mvn -o ${args}"
  }
}
```

Lors de l'enregistrement de l'état sur les classes, comme ci-dessus, la classe **doit** implémenter l'interface `Serializable`. Cela garantit qu'un Pipeline utilisant la classe, comme on le voit dans l'exemple ci-dessous, peut suspendre et reprendre correctement dans Jenkins.

``` groovy
@Library('utils') import org.foo.Utilities
def utils = new Utilities(this)
node {
  utils.mvn 'clean package'
}
```

Si la bibliothèque doit accéder aux variables globales, telles que `env`, celles-ci doivent être explicitement transmises dans les classes de bibliothèque ou les méthodes de manière similaire.

Au lieu de passer de nombreuses variables du Pipeline scripté dans une bibliothèque :

``` groovy
package org.foo
class Utilities {
  static def mvn(script, args) {
    script.sh "${script.tool 'Maven'}/bin/mvn -s ${script.env.HOME}/jenkins.xml -o ${args}"
  }
}
```

L'exemple ci-dessus montre le script transmis en une méthode `static`, invoqué à partir d'un Pipeline scripté comme suit : 

``` groovy
@Library('utils') import static org.foo.Utilities.*
node {
  mvn this, 'clean package'
}
```

### Définition des variables globales

En interne, les scripts dans le répertoire `vars` sont instanciés à la demande sous forme de singletons. Cela permet de définir plusieurs méthodes dans un seul fichier `.groovy` pour plus de commodité. Par exemple :

``` groovy title="<i>vars/log.groovy</i>"
def info(message) {
    echo "INFO: ${message}"
}

def warning(message) {
    echo "WARNING: ${message}"
}
```

``` groovy title="<i>Jenkinsfile</i>"
@Library('utils') _

log.info 'Starting'
log.warning 'Nothing to do!'
```

Le Pipeline déclaratif n'autorise pas les appels de méthode sur les objets en dehors des blocs "script". ([JENKINS-42360](https://issues.jenkins.io/browse/JENKINS-42360)). Les appels de méthode ci-dessus devraient être placés dans une directive de `script` :

``` groovy title="<i>Jenkinsfile</i>"
@Library('utils') _

pipeline {
    agent none
    stages {
        stage ('Example') {
            steps {
                // log.info 'Starting' //(1)
                script { //(2)
                    log.info 'Starting'
                    log.warning 'Nothing to do!'
                }
            }
        }
    }
}
```

1. Cet appel de méthode échouerait car il est en dehors d'une directive `script`. 
2. Directive `script` requise pour accéder aux variables globales dans le Pipeline déclaratif.

!!! note
    Une variable définie dans une bibliothèque partagée n'apparaîtra que dans la référence des variables globales (sous la syntaxe des Pipelines) après que Jenkins ait chargé et ait utilisé cette bibliothèque dans le cadre d'une exécution de Pipeline réussie.

!!! warning "_Évitez de préserver l'état dans les variables globales_"
    Toutes les variables globales définies dans une bibliothèque partagée doivent être apatrides, c'est-à-dire qu'elles devraient agir comme des collections de fonctions. Si votre Pipeline tentait de stocker un état dans des variables globales, cet état serait perdu en cas de redémarrage du contrôleur Jenkins. Utilisez une classe statique ou instanciez une variable locale d'une classe à la place.

Bien que l'utilisation de champs pour les variables globales soit découragée selon ce qui précède, il est possible de définir des champs et de les utiliser comme lecture seule. Pour définir un champ, vous devez utiliser une annotation :

``` groovy
@groovy.transform.Field
def yourField = "YourConstantValue"

def yourFunction....
```

### Définir des étapes personnalisées

Les bibliothèques partagées peuvent également définir des variables globales qui se comportent de manière similaire à des étapes intégrées, telles que `sh` ou `git`. Les variables globales définies dans les bibliothèques partagées doivent être nommées avec toutes les minuscules ou "_camelCased_" afin d'être chargées correctement par Pipeline. [^2]

[^2]: [gist.github.com/rtyler/e5e57f075af381fce4ed3ae57aa1f0c2](https://gist.github.com/rtyler/e5e57f075af381fce4ed3ae57aa1f0c2)

Par exemple, pour définir `sayHello`, le fichier `vars/sayHello.groovy ` doit être créé et doit implémenter une méthode `call`. La méthode `call` permet à la variable globale d'être invoquée d'une manière similaire à une étape :

``` groovy
// vars/sayHello.groovy
def call(String name = 'human') {
    // Toutes les étapes valides peuvent être appelées à partir de ce code, 
    // comme dans les autres pipelines scriptés.
    echo "Hello, ${name}."
}
```

Le Pipeline pourrait alors référencer et invoquer cette variable :

``` groovy
sayHello 'Joe'
sayHello() /* appeler avec les arguments par défaut */
```

S'il est appelé avec un bloc, la méthode `call` recevra un [retour](http://groovy-lang.org/closures.html). Le type doit être défini explicitement pour clarifier l'intention de l'étape, par exemple :

``` groovy
// vars/windows.groovy
def call(Closure body) {
    node('windows') {
        body()
    }
}
```

Le Pipeline peut ensuite utiliser cette variable comme n'importe quelle étape intégrée qui accepte un bloc :

``` groovy
windows {
    bat "cmd /?"
}
```

### Définir un DSL plus structuré

Si vous avez beaucoup de Pipelines qui sont pour la plupart similaires, le mécanisme variable global fournit un outil pratique pour construire un DSL de niveau supérieur qui capture la similitude. Par exemple, tous les plugins Jenkins sont construits et testés de la même manière, afin que nous puissions écrire une étape nommée `buildPlugin` :

``` groovy
// vars/buildPlugin.groovy
def call(Map config) {
    node {
        git url: "https://github.com/jenkinsci/${config.name}-plugin.git"
        sh 'mvn install'
        mail to: '...', subject: "${config.name} plugin build", body: '...'
    }
}
```

En supposant que le script it été chargé en tant que [bibliothèque partagée globale](./pipeline-bibliotheques-partagees.md#bibliothèques-partagées-globales) ou en tant que [bibliothèque partagée au niveau du dossier](./pipeline-bibliotheques-partagees.md#bibliothèques-partagées-au-niveau-du-dossier), le `Jenkinsfile` résultant sera considérablement plus simple :

``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
buildPlugin name: 'git'
```

Il existe également une astuce « builder pattern » utilisant `Closure.DELEGATE_FIRST` de Groovy, qui permet au fichier `Jenkinsfile` de ressembler davantage à un fichier de configuration qu'à un programme, mais cette méthode est plus complexe et source d'erreurs, et n'est donc pas recommandée.

### Utilisation de bibliothèques tierces

!!! info
    Bien que possible, l'accès aux bibliothèques tierces utilisant `@grab` à partir de bibliothèques de confiance a divers problèmes et n'est pas recommandée. Au lieu d'utiliser `@Grab`, l'approche recommandée consiste à créer un exécutable autonome dans le langage de programmation de votre choix (en utilisant les bibliothèques tierces que vous désirez), installez-la sur les agents Jenkins que vos Pipelines utilisent, puis invoquez cet exécutable dans vos Pipelines en utilisant l'étape `bat` ou `sh`.

Il est possible d'utiliser des bibliothèques Java tierces, généralement disponibles dans [Maven Central](https://search.maven.org/), à partir d'un code de bibliothèque fiable à l'aide de l'annotation `@Grab`. Pour plus de détails, consultez la [documentation Grape](https://docs.groovy-lang.org/latest/html/documentation/grape.html#_quick_start), mais en résumé :

``` groovy
@Grab('org.apache.commons:commons-math3:3.4.1')
import org.apache.commons.math3.primes.Primes
void parallelize(int count) {
  if (!Primes.isPrime(count)) {
    error "${count} was not prime"
  }
  // …
}
```

Les bibliothèques tierces sont mises en cache par défaut dans `~/.groovy/grapes/` sur le contrôleur Jenkins.

### Chargement des ressources

Les bibliothèques externes peuvent charger des fichiers complémentaires à partir d'un répertoire `resources/` à l'aide de l'étape `libraryResource`. L'argument est un chemin d'accès relatif, similaire au chargement des ressources Java :

``` groovy
def request = libraryResource 'com/mycorp/pipeline/somelib/request.json'
```

Le fichier est chargé sous forme de chaîne, ce qui permet de le transmettre à certaines API ou de l'enregistrer dans un espace de travail à l'aide de `writeFile`.

Il est conseillé d'utiliser une structure de package unique afin d'éviter tout conflit accidentel avec une autre bibliothèque.

## Pré-test des modifications apportées à la bibliothèque

Si vous remarquez une erreur dans une compilation utilisant une bibliothèque non fiable, cliquez simplement sur le lien _Replay_ pour essayer de modifier un ou plusieurs de ses fichiers source, puis vérifiez si la compilation obtenue se comporte comme prévu. Une fois que vous êtes satisfait du résultat, suivez le lien diff depuis la page d'état de la compilation, appliquez le diff au référentiel de la bibliothèque et _validez_.

(Même si la version demandée pour la bibliothèque était une branche, plutôt qu'une version fixe comme une balise, les builds rejoués utiliseront exactement la même révision que le build d'origine : les sources de la bibliothèque ne seront pas vérifiées à nouveau.)

Le _Replay_ n'est actuellement pas pris en charge pour les bibliothèques de confiance. La modification des fichiers de ressources n'est également pas prise en charge actuellement pendant la relecture.

### Définition des pipelines déclaratifs

À partir de Declarative 1.2, sorti fin septembre 2017, vous pouvez également définir des Pipelines déclaratifs dans vos bibliothèques partagées. Voici un exemple qui exécutera un Pipeline déclaratif différent selon que le numéro de build est pair ou impair :

``` groovy
// vars/evenOrOdd.groovy
def call(int buildNumber) {
  if (buildNumber % 2 == 0) {
    pipeline {
      agent any
      stages {
        stage('Even Stage') {
          steps {
            echo "Le numéro de build est pair."
          }
        }
      }
    }
  } else {
    pipeline {
      agent any
      stages {
        stage('Odd Stage') {
          steps {
            echo "Le numéro de build est impair."
          }
        }
      }
    }
  }
}
```

``` groovy
// Jenkinsfile
@Library('my-shared-library') _

evenOrOdd(currentBuild.getNumber())
```

À l'heure actuelle, seules des `pipelines` complets peuvent être définis dans des bibliothèques partagées. Cela ne peut être fait que dans `vars/*.groovy`, et uniquement dans une méthode `call`. Un seul Pipeline déclaratif peut être exécute dans une seule compilation. Si vous essayez d'en exécuter un deuxième, votre compilation échouera.

### Tester les modifications de la _pull request_ de la bibliothèque

En ajoutant `@Library('my-shared-library@pull/<your-pr-number>/head') _ ` en haut d'un fichier Jenkinsfile consommateur de bibliothèque, vous pouvez tester les modifications apportées à votre Pipeline library pull request _in situ_ si votre biblithèque Pipeline est hébergée sur GitHub et si la configuration SCM pour cette bibliothèque utilise GitHub.
Reportez-vous à la convention de nommage des branches pour les _pull requests_ ou _merge _requests_ pour d'autres fournisseurs tels que Assembla, Bitbucket, Gitea, GitLab et Tuleap.

Prenons, par exemple, une modification apportée au Pipeline partagé global ci.jenkins.io, dont le code source est stocké sur [github.com/jenkins-infra/pipeline-library/](https://github.com/jenkins-infra/pipeline-library/).

Supposons que vous écriviez une nouvelle fonctionnalité et que vous ayez ouvert une demande d'extraction sur la bibliothèque de Pipelines, numéro `123`.

En ouvrant une demande d'extraction sur le [référentiel de test dédié jenkins-infra-test-plugin](https://github.com/jenkinsci/jenkins-infra-test-plugin/) avec le contenu suivant, vous pourrez vérifier vos modifications sur ci.jenkins.io :

``` diff
--- jenkins-infra-test-plugin/Jenkinsfile
+++ jenkins-infra-test-plugin/Jenkinsfile
@@ -1,3 +1,4 @@
+ @Library('pipeline-library@pull/123/head') _
  buildPlugin(
    useContainerAgent: true,
    configurations: [
      [platform: 'linux', jdk: 21],
      [platform: 'windows', jdk: 17],
  ])
```


