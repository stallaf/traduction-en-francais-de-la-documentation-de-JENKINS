# Incompatibilités entre les méthodes CPS des Pipelines

<div class="couleur-introduction">
Jenkins Pipeline utilise une bibliothèque appelée Groovy CPS pour exécuter les scripts Pipeline. Bien que Pipeline utilise le parseur et le compilateur Groovy, contrairement à un environnement Groovy classique, il exécute la plupart du programme dans un interpréteur spécial. Celui-ci utilise une transformation de type CPS (continuation-passing style) pour convertir votre code en une version capable d'enregistrer son état actuel sur le disque (un fichier appelé <code>program.dat</code> dans votre répertoire de compilation) et de continuer à s'exécuter même après le redémarrage de Jenkins. (Vous trouverez plus d'informations techniques sur le Pipeline sur la page du <a href="https://plugins.jenkins.io/workflow-cps">plugin Groovy</a> et la <a href="https://github.com/cloudbees/groovy-cps/blob/master/README.md">page de la bibliothèque</a>).
</div>

Bien que la transformation CPS soit généralement transparente pour les utilisateurs, il existe des limitations quant aux constructions du langage Groovy qui peuvent être prises en charge, et dans certaines circonstances, cela peut entraîner un comportement contre-intuitif. [JENKINS-31314](https://issues.jenkins.io/browse/JENKINS-31314) permet au runtime de détecter l'erreur la plus courante : appeler du code transformé par CPS à partir de code non transformé par CPS. Les éléments suivants sont transformés par CPS :

* Presque tous les scripts Pipeline que vous écrivez (y compris dans les bibliothèques) ;
* La plupart des étapes Pipeline, y compris toutes celles qui prennent un bloc.

Les éléments suivants ne sont pas transformés en CPS :

* Le bytecode Java compilé, y compris
    * la plate-forme Java ;
    * le noyau Jenkins et les plugins ;
    * le runtime pour le langage Groovy ;
* Les corps de constructeurs dans votre script Pipeline ;
* Toute méthode de votre script Pipeline marquée avec l'annotation `@NonCPS` ;
* Quelques étapes Pipeline qui ne prennent pas de bloc et agissent instantanément, telles que `echo` ou `properties`.

Le code transformé en CPS peut appeler du code non transformé en CPS ou d'autres codes transformés en CPS, et le code non transformé en CPS peut appeler d'autres codes non transformés en CPS, mais le code non transformé en CPS **ne peut pas** appeler du code transformé en CPS. Si vous essayez d'appeler du code transformé en CPS à partir de code non transformé en CPS, l'interpréteur CPS ne peut pas fonctionner correctement, ce qui entraîne des résultats incorrects et souvent confus.

## Problèmes courants et solutions

### Utilisation des étapes de Pipeline à partir de `NonCPS`

Parfois, les utilisateurs appliquent l'annotation `@NonCPS`  à une définition de méthode afin de contourner la transformation CPS à l'intérieur de cette méthode. Cela peut être fait pour contourner les limitations de la couverture du langage Groovy (car le corps de la méthode s'exécutera en utilisant la sémantique native de Groovy), ou pour obtenir de meilleures performances (l'interpréteur impose une surcharge importante). Cependant, ces méthodes ne doivent pas appeler de code transformé par CPS, tel que les étapes de pipeline. Par exemple, ce qui suit ne fonctionnera pas :

``` groovy title="@NonCPS"
def compileOnPlatforms() {
  ['linux', 'windows'].each { arch ->
    node(arch) {
      sh 'make'
    }
  }
}
compileOnPlatforms()
```

L'utilisation des étapes `node`  ou `sh`  de cette méthode est illégale et entraînera un comportement anormal. L'avertissement dans les journaux résultant de l'exécution de ce script se présente comme suit :

!!! quote " "
    _appel prévu de WorkflowScript.compileOnPlatforms mais capture finale de node_

Pour corriger ce problème, il suffit de supprimer l'annotation, qui n'était pas nécessaire. (Les utilisateurs de longue date de Pipeline pouvaient penser qu'elle l'était, avant la correction de [JENKINS-26481](https://issues.jenkins.io/browse/JENKINS-26481)).

### Appel de méthodes non transformées CPS avec des arguments transformés CPS

Certaines méthodes Groovy et Java prennent des types complexes comme paramètres afin de prendre en charge un comportement dynamique. Un cas courant est celui des méthodes de tri qui permettent aux appelants de spécifier une méthode à utiliser pour comparer des objets ([JENKINS-44924](https://issues.jenkins.io/browse/JENKINS-44924)). De nombreuses méthodes similaires dans la bibliothèque standard Groovy fonctionnent correctement après la correction de [JENKINS-26481](https://issues.jenkins.io/browse/JENKINS-26481), mais certaines méthodes restent non corrigées. Par exemple, ce qui suit ne fonctionnera pas :

``` groovy
def sortByLength(List<String> list) {
  list.toSorted { a, b -> Integer.valueOf(a.length()).compareTo(b.length()) }
}
def sorted = sortByLength(['333', '1', '4444', '22'])
echo(sorted.toString())
```

Le retour transmis à `Iterable.toSorted` est transformée en CPS, mais `Iterable.toSorted` n'est pas transformé en CPS en interne, ce qui empêche le fonctionnement prévu. Actuellement, la valeur renvoyée par l'appel à `toSorted` correspond à la valeur renvoyée par le premier appel au retour. Dans l'exemple, cela entraîne la définition de `sorted` sur `-1`, et l'avertissement dans les journaux se présente comme suit :

!!! quote " "
    _appel prévu de java.util.ArrayList.toSorted mais finissant par intercepter org.jenkinsci.plugins.workflow.cps.CpsClosure2.call_

Pour corriger ce problème, aucun argument passé à ces méthodes ne doit être transformé en CPS. Pour ce faire, vous pouvez encapsuler la méthode problématique (`Iterable.toSorted` dans l'exemple) dans une autre méthode et annoter la méthode externe avec `@NonCPS`, ou créer une définition de classe explicite pour la fermeture et annoter toutes les méthodes de cette classe avec `@NonCPS`.

### Constructeurs

Il arrive parfois que les utilisateurs tentent d'utiliser du code transformé en CPS, tel que des étapes de Pipeline, à l'intérieur d'un constructeur dans un script Pipeline. Malheureusement, la construction d'objets via le `new` opérateur dans Groovy ne peut pas être transformée en CPS ([JENKINS-26313](https://issues.jenkins.io/browse/JENKINS-26313)) et ne fonctionnera donc pas. Voici un exemple qui appelle une méthode transformée en CPS dans un constructeur :

``` groovy
class Test {
  def x
  public Test() {
    setX()
  }
  private void setX() {
    this.x = 1;
  }
}
def x = new Test().x
echo "${x}"
```

La construction de `Test` échouera lorsque le constructeur appellera `Test.setX`, car `setX` est une méthode transformée par CPS. L'avertissement dans les journaux résultant de l'exécution de ce script se présente comme suit :

!!! quote " "
    _appel prévu de Test.<init>, mais capture de Test.setX_

Pour corriger ce problème, assurez-vous que toutes les méthodes définies dans un script Pipeline qui sont appelées à partir d'un constructeur sont annotées avec `@NonCPS` et que les constructeurs n'appellent aucune étape Pipeline. Si vous devez appeler du code transformé CPS, tel qu'une étape Pipeline, à partir du constructeur, vous devez déplacer la logique liée aux méthodes transformées CPS hors du constructeur, par exemple dans une méthode d'usine statique qui appelle le code transformé CPS, puis transmet les résultats au constructeur.

### Remplacement des méthodes non transformées par CPS

Les utilisateurs peuvent créer une classe dans un script Pipeline qui étend une classe préexistante définie en dehors du script Pipeline, par exemple à partir des bibliothèques standard Java ou Groovy. Dans ce cas, la sous-classe doit s'assurer que toutes les méthodes de remplacement sont annotées avec `@NonCPS` et n'utilisent aucun code transformé CPS en interne. Sinon, les méthodes de remplacement échoueront si elles sont appelées à partir d'un contexte non CPS. Par exemple, ce qui suit ne fonctionnera pas :

``` groovy
class Test {
  @Override
  public String toString() {
    return "Test"
  }
}
def builder = new StringBuilder()
builder.append(new Test())
echo(builder.toString())
```

L'appel de la redéfinition de `toString` transformée par CPS à partir d'un code non transformé par CPS, tel que `StringBuilder.append`, n'est pas autorisé et ne fonctionnera pas comme prévu dans la plupart des cas. L'avertissement dans les journaux résultant de l'exécution de ce script se présente comme suit :

!! quote " "
    _appel prévu de java.lang.StringBuilder.append mais capture finale de Test.toString_

Pour corriger ce problème, ajoutez l'annotation `@NonCPS` à la méthode de remplacement et supprimez toute utilisation de code transformé par CPS, tel que les étapes Pipeline, de la méthode.

### Fermetures à l'intérieur de `GString` 

Dans Groovy, il est possible d'utiliser un retour dans un `GString` afin que cellui-ci soit évalué chaque fois que le `GString` est utilisé comme une chaîne. Cependant, dans les scripts Pipeline, cela ne fonctionnera pas comme prévu, car la fermeture à l'intérieur du GString sera transformée en CPS. Voici un exemple :

``` groovy
def x = 1
def s = « x = ${-> x} »
x = 2
echo(s)
```

L'utilisation d'un retour à l'intérieur d'un `GString`  comme dans cet exemple ne fonctionnera pas. L'avertissement provenant des journaux lors de l'exécution de ce script ressemble à ceci :

!!! quote " "
    _expected to call WorkflowScript.echo but wound up catching org.jenkinsci.plugins.workflow.cps.CpsClosure2.call_

Pour corriger ce problème, remplacez le GString d'origine par un retour qui renvoie un GString utilisant une expression normale plutôt qu'un retour, puis appelez le retour à l'endroit où vous auriez utilisé le `GString` d'origine, comme suit :

``` groovy
def x = 1
def s = { -> « x = ${x} » }
x = 2
echo(s()
```

## Faux positifs

Malheureusement, certaines expressions peuvent déclencher cet avertissement à tort, même si elles s'exécutent correctement. Si vous rencontrez un tel cas, veuillez créer un [nouveau ticket](https://www.jenkins.io/participate/report-issue/redirect/#21713) (après avoir vérifié qu'il n'existe pas déjà) pour `workflow-cps-plugin`.