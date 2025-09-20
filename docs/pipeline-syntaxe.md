# Syntaxe Pipeline

Cette section s'appuie sur les informations introduites dans [commencez avec Pipeline](./pipeline-commencer.md) et doit être traitée uniquement comme une référence. Pour plus d'informations sur la façon d'utiliser la syntaxe  Pipelines dans des exemples pratiques, reportez-vous à : 

* [Utilisation d'un Jenkinsfile ](./pipeline-jenkinsfile.md) ;
* [Pipeline en tant que code](./pipeline-comme-code.md).

À partir de la version 2.5 du plugin, Pipeline prend en charge deux [syntaxes](#pipeline-declaratif) discrètes - déclaratives et scriptés. Pour les avantages et les inconvénients de chacun, reportez-vous à la [comparaison](#comparaison).

Comme discuté au [début de ce chapitre](#syntaxe-pipeline), la partie la plus fondamentale d'un Pipeline est [«l'étape»](#etapes). Fondamentalement, les étapes indiquent à Jenkins _quoi_ faire et servir de bloc de construction de base pour la syntaxe Pipeline déclarative et scripté.

Pour un aperçu des étapes disponibles, veuillez vous référer à la [référence des étapes Pipeline](https://www.jenkins.io/doc/pipeline/steps) qui contient une liste complète des étapes intégrées ainsi que celles fournies par les plugins.

## Pipeline déclaratif

Pipeline Déclaratif présente une syntaxe plus simplifiée et plus subjective au-dessus des sous-systèmes Pipeline. Pour les utiliser, installez le [plugin Pipeline: Declarative](https://plugins.jenkins.io/pipeline-model-definition).

Tous les Pipelines déclaratifs valides doivent être inclus dans un bloc `pipeline`, par exemple :

``` groovy
pipeline { 
    / * insérer le pipeline déclaratif ici * /
}
```

Les déclarations et expressions de base qui sont valides dans le Pipeline déclaratif suivent les mêmes règles que la [syntaxe de Groovy](http://groovy-lang.org/syntax.html) avec les exceptions suivantes : 

* Le niveau supérieur du Pipeline doit être un bloc, plus précisément : `pipeline { }` ;
* Pas de point-virgule comme séparateur d'instructions. Chaque instruction doit figurer sur une ligne distincte ;
* Les blocs doivent uniquement être composés de [sections](#sections-declaratives), de [directives](#directives-declaratices), [d'étapes](#etapes-declaratives) ou d'instructions d'affectation ;
* Une instruction de référence de propriété est traitée comme un appel de méthode sans argument. Ainsi, par exemple, `input` est traité comme `input()`.

Vous pouvez utiliser le [générateur de directif déclaratif](./pipeline-commencer.md#générateur-de-directif-déclaratif) pour vous aider à démarrer avec la configuration des directives et des sections de votre pipeline déclaratif.

### Limites

Il existe actuellement un [problème ouvert](https://issues.jenkins.io/browse/JENKINS-37984) qui limite la taille maximale du code dans le bloc `pipeline{}`. Cette limitation ne s'applique pas aux Pipelines scriptés.

### Sections

Les sections dans le Pipeline déclaratif contiennent généralement une ou plusieurs [directives](#directives- declaratives) ou [étapes](#etapes-declaratives).

#### agent

La section `agent` spécifie où l'ensemble du Pipeline, ou une étape spécifique, sera exécuté dans l'environnement Jenkins en fonction de l'emplacement de la section `agent`. La section doit être définie au niveau supérieur dans le bloc `pipeline`, mais son utilisation au niveau de l'étape est facultative.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Oui</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td><a href="#agent">Décrits ci-dessous</a></td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>Dans le bloc <code>pipeline</code>de niveau supérieur et chaque bloc <code>stage</code></td>
    </tr>
</table>
</div>

##### Différences entre les agents de niveau supérieur et les agents de niveau étape

Il existe certaines nuances lorsque vous ajoutez un agent au niveau supérieur ou au niveau étape lorsque la directive `options` est appliquée. Consultez la section [options](#options) pour plus d'informations.

###### Agents de niveau supérieur

Dans les `agents` déclarés au niveau supérieur d'un Pipeline, un agent est alloué, puis l'option `timeout` est appliquée. Le temps nécessaire à l'allocation de l'agent **n'est pas inclus** dans la limite définie par l'option `timeout`.

``` groovy
pipeline {
    agent any
    options {
        // Le compteur de délai d'attente démarre APRÈS l'affectation de 
        // l'agent.
        timeout(time: 1, unit: 'SECONDS')
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

###### Agents d'étape

Dans les `agents` déclarés au sein d'une étape, les options sont invoquées avant l'allocation de l'`agent` et avant la vérification des conditions `when` . Dans ce cas, lorsque vous utilisez `timeout`, celui-ci est appliqué avant l'allocation de l'`agent`. Le temps nécessaire à l'allocation de l'agent **est inclus** dans la limite définie par l'option `timeout`.

``` groovy
pipeline {
    agent none
    stages {
        stage('Example') {
            agent any
            options {
                // Le compteur de délai d'attente démarre AVANT l'affectation
                // de l'agent.
                timeout(time: 1, unit: 'SECONDS')
            }
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

Ce délai d'expiration inclut le temps nécessaire à l'approvisionnement de l'agent. Pour cette raison, le Pipeline peut échouer dans les cas où l'allocation de l'agent est retardée.

##### Paramètres

Afin de prendre en charge la grande variété de cas d'utilisation que peuvent rencontrer les auteurs de Pipelines, la section `agent` prend en charge plusieurs types de paramètres. Ces paramètres peuvent être appliqués au niveau supérieur du bloc `pipeline` ou dans chaque directive `stage`.

**any**<br>
Exécutez le Pipeline ou l'étape sur n'importe quel agent disponible. Par exemple : `agent any`.

**none**<br>
Lorsqu'il est appliqué au niveau supérieur du bloc `pipeline`, aucun agent global ne sera alloué pour l'ensemble de l'exécution du pipeline et chaque section `stage` devra contenir sa propre section `agent`. Par exemple : `agent none`.

**label**<br>
Exécute le Pipeline ou l'étape sur un agent disponible dans l'environnement Jenkins avec le label fourni. Par exemple : `agent { label “my-defined-label” }`.
Les conditions d'étiquette peuvent également être utilisées : Par exemple : `agent { label “my-label1 && my-label2” }` ou `agent { label “my-label1 || my-label2” }`.

**node**<br>
`agent { node { label “labelName” } }` se comporte de la même manière que `agent { label “labelName” }`, mais `node` permet des options supplémentaires (telles que `customWorkspace`).

**docker**<br>
Exécutez le Pipeline, ou l'étape, avec le conteneur donné qui sera provisionné dynamiquement sur un [nœud](./glossaire.md#node) préconfiguré pour accepter les Pipelines basés sur Docker, ou sur un nœud correspondant au paramètre `label` défini de manière facultative. `docker` accepte également en option un paramètre `args` qui peut contenir des arguments à passer directement à une invocation `docker run`, et une option `alwaysPull`, qui forcera un `docker pull` même si le nom de l'image est déjà présent. Par exemple : `agent { docker “maven:3.9.3-eclipse-temurin-17” }` ou

``` groovy
agent {
    docker {
        image 'maven:3.9.3-eclipse-temurin-17'
        label 'my-defined-label'
        args  '-v /tmp:/tmp'
    }
}
```

`docker` accepte également en option les paramètres `registryUrl` et `registryCredentialsId` qui permettent de spécifier le registre Docker à utiliser et ses informations d'identification. Le paramètre `registryCredentialsId` peut être utilisé seul pour les référentiels privés au sein du hub Docker. Par exemple :

``` groovy
agent {
    docker {
        image “myregistry.com/node”
        label “my-defined-label”
        registryUrl “https://myregistry.com/”
        registryCredentialsId “myPredefinedCredentialsInJenkins”
    }
}
```

**dockerfile**<br>
Exécutez le Pipeline, ou étape, avec un conteneur créé à partir d'un fichier `Dockerfile` contenu dans le référentiel source. Pour utiliser cette option, le fichier `Jenkinsfile` doit être chargé à partir d'un **Pipeline multibranche** ou d'un **Pipeline à partir de SCM**. En règle générale, il s'agit du fichier `Dockerfile` situé à la racine du référentiel source : `agent { dockerfile true }`. Si vous créez un fichier `Dockerfile` dans un autre répertoire, utilisez l'option `dir : agent { dockerfile { dir “someSubDir” } }`. Si votre fichier `Dockerfile` porte un autre nom, vous pouvez spécifier le nom du fichier avec l'option `filename`. Vous pouvez passer des arguments supplémentaires à la commande `docker build …`​ avec l'option `additionalBuildArgs`, comme `agent { dockerfile { additionalBuildArgs “--build-arg foo=bar” } }`. Par exemple, un référentiel avec le fichier `build/Dockerfile.build`, qui attend un argument de construction `version` :

``` groovy
agent {
    // Équivalent à « docker build -f Dockerfile.build --build-arg version=1.0.2 ./build/
    dockerfile {
        filename “Dockerfile.build”
        dir “build”
        label 'my-defined-label'
        additionalBuildArgs  “--build-arg version=1.0.2”
        args “-v /tmp:/tmp”
    }
}
```

`dockerfile` accepte également en option les paramètres `registryUrl` et `registryCredentialsId` qui permettent de spécifier le registre Docker à utiliser et ses informations d'identification. Par exemple :

``` groovy
agent {
    dockerfile {
        filename “Dockerfile.build”
        dir “build”
        label “my-defined-label”
        registryUrl “https://myregistry.com/”
        registryCredentialsId “myPredefinedCredentialsInJenkins”
    }
}
```

**kubernetes**<br>
Exécutez le Pipeline, ou l'étape, dans un pod déployé sur un cluster Kubernetes. Pour utiliser cette option, le fichier `Jenkinsfile` doit être chargé à partir d'un **Pipeline multibranche** ou d'un **Pipeline à partir de SCM**. Le modèle de pod est défini dans le bloc kubernetes { }. Par exemple, si vous voulez un pod contenant un conteneur Kaniko, vous le définiriez comme suit :

``` groovy
agent {
    kubernetes {
        defaultContainer 'kaniko'
        yaml '''
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - 99d
    volumeMounts:
      - name: aws-secret
        mountPath: /root/.aws/
      - name: docker-registry-config
        mountPath: /kaniko/.docker
  volumes:
    - name: aws-secret
      secret:
        secretName: aws-secret
    - name: docker-registry-config
      configMap:
        name: docker-registry-config
'''
   }
```

Vous devrez créer un secret ``aws-secret`` pour que Kaniko puisse s'authentifier avec ECR. Ce secret doit contenir le contenu de `~/.aws/credentials`. L'autre volume est un ConfigMap qui doit contenir le point de terminaison de votre registre ECR. Par exemple :

``` json
{
      « credHelpers »: {
        « <your-aws-account-id>.dkr.ecr.eu-central-1.amazonaws.com »: « ecr-login »
      }
}
```

Reportez-vous à l'exemple suivant pour référence : [https://github.com/jenkinsci/kubernetes-plugin/blob/master/examples/kaniko.groovy](https://github.com/jenkinsci/kubernetes-plugin/blob/master/examples/kaniko.groovy).

##### Options courantes

Voici quelques options qui peuvent être appliquées à deux implémentations d'`agent` ou plus. Elles ne sont pas obligatoires, sauf indication contraire.

**label**<br>
Une chaîne. Le label ou la condition de label sur lequel/laquelle exécuter le Pipeline ou une `stage` individuelle.

Cette option est valide pour `node`, `docker` et `dockerfile`, et est obligatoire pour `node`.

**customWorkspace**<br>
Une chaîne. Exécutez le Pipeline ou la `stage` individuelle à laquelle cet `agent` est appliqué dans cet espace de travail personnalisé, plutôt que dans l'espace de travail par défaut. Il peut s'agir d'un chemin relatif, auquel cas l'espace de travail personnalisé se trouvera sous la racine de l'espace de travail sur le nœud, ou d'un chemin absolu. Par exemple :

``` groovy
agent {
    node {
        label 'my-defined-label'
        customWorkspace '/some/other/path'
    }
}
```

Cette option est valide pour `node`, `docker` et `dockerfile`.

**reuseNode**<br>
Une valeur booléenne, _false_ par défaut. Si _true_, exécutez le conteneur sur le nœud spécifié au niveau supérieur du Pipeline, dans le même espace de travail, plutôt que sur un nouveau nœud.

Cette option est valide pour `docker` et `dockerfile`, et n'a d'effet que lorsqu'elle est utilisée sur un `agent` pour une `stage` individuelle.

**args**<br>
Une chaîne. Arguments d'exécution à passer à `docker run`.

Cette option est valide pour `docker` et `dockerfile`.

``` groovy title="<i>Exemple 1. Agent Docker, Pipeline Déclaratif</i>"
pipeline {
    agent { docker 'maven:3.9.3-eclipse-temurin-17' } //(1)
    stages {
        stage('Example Build') {
            steps {
                sh 'mvn -B clean verify'
            }
        }
    }
}
```

1. Exécute toutes les étapes définies dans ce Pipeline dans un conteneur nouvellement créé portant le nom et la balise indiqués (`maven:3.9.3-eclipse-temurin-17`).

``` groovy title="<i>Exemple 2. Section Agent Niveau-étape</i>"
pipeline {
    agent none //(1)
    stages {
        stage('Example Build') {
            agent { docker 'maven:3.9.9-eclipse-temurin-21' } //(2)
            steps {
                echo 'Hello, Maven'
                sh 'mvn --version'
            }
        }
        stage('Example Test') {
            agent { docker 'openjdk:21-jre' } //(3)
            steps {
                echo 'Hello, JDK'
                sh 'java -version'
            }
        }
    }
}
```

1. La définition de `agent none` au niveau supérieur du Pipeline garantit qu'aucun [exécuteur](./glossaire.md#executeur) ne sera attribué inutilement. L'utilisation de `agent none` oblige également chaque section `stage` à contenir sa propre section `agent`.
2. Exécutez les niveaux de cette étape dans un conteneur nouvellement créé à l'aide de cette image.
3. Exécutez les niveaux de cette étape dans un conteneur nouvellement créé à l'aide d'une image différente de celle de l'étape précédente.

**post**<br>
La section `post` définit une ou plusieurs [étapes](./pipeline-syntaxe.md#limites) supplémentaires qui sont exécutées en fin de Pipeline ou d'étape (selon l'emplacement de la section `post` dans le Pipeline). `post` peut prendre en charge n'importe lequel des blocs de [post-condition](./pipeline-syntaxe.md#post-conditions) suivants : `always`, `changed`, `fixed`, `regression`, `aborted`, `failure`, `success`, `unstable`, `unsuccessful` et `cleanup`. Ces blocs de condition permettent l'exécution d'étapes à l'intérieur de chaque condition en fonction du statut d'achèvement du Pipeline ou de l'étape. Les blocs de condition sont exécutés dans l'ordre indiqué ci-dessous.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Non</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td><i>Aucun</i></td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>Dans le bloc <code>pipeline</code>de niveau supérieur et chaque bloc <code>stage</code></td>
    </tr>
</table>
</div>

##### Conditions

`always`<br>
Exécutez les étapes de la section `post`, quel que soit le statut d'achèvement de l'exécution du Pipeline ou de l'étape.

`changed`<br>
N'exécutez les étapes dans `post` que si l'exécution du Pipeline actuel a un statut d'achèvement différent de celui de son exécution précédente.

`fixed`<br>
N'exécutez les étapes dans `post` que si l'exécution du Pipeline actuel est réussie et que l'exécution précédente a échoué ou était instable.

`regression`<br>
N'exécutez les étapes dans `post` que si le Pipeline actuel ou son statut est en échec, instable ou abandonné et que l'exécution précédente a réussi.

`aborted`<br>
N'exécutez les étapes dans  `post` que si l'exécution du Pipeline actuel a le statut « avorté », généralement en raison d'un abandon manuel du Pipeline. Ce statut est généralement indiqué en gris dans l'interface utilisateur Web.

`failure`<br>
N'exécutez les étapes dans  `post` que si l'exécution du Pipeline ou de l'étape actuel a le statut « échoué », généralement indiqué en rouge dans l'interface utilisateur Web.

`success`<br>
N'exécutez les étapes dans `post` que si l'exécution du Pipeline ou de l'étape actuel a le statut « succès », généralement indiqué en bleu ou en vert dans l'interface utilisateur Web.

`unstable`<br>
N'exécutez les étapes dans `post` que si l'exécution du Pipeline actuel a le statut « instable », généralement causé par des échecs de test, des violations de code, etc. Ce statut est généralement indiqué en jaune dans l'interface utilisateur Web.

`unsuccessful`<br>
N'exécutez les étapes dans `post` que si l'exécution du Pipeline ou de l'étape actuel n'a pas le statut « succès ». Ceci est généralement indiqué dans l'interface utilisateur Web en fonction du statut mentionné précédemment (pour les étapes, cela peut se produire si la compilation elle-même est instable).

`cleanup`<br>
Exécutez les étapes de cette condition `post` après que toutes les autres conditions `post` ont été évaluées, quel que soit le statut du Pipeline ou de l'étape.

``` groovy title="<i>Exemple 3. Section Post, Pipeline Déclarativf</i>"
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
    post { //(1)
        always {//(2) 
            echo 'I will always say Hello again!'
        }
    }
}
```

1. En règle générale, la section `post` doit être placée à la fin du Pipeline.
2. Les blocs [post-condition](./pipeline-syntaxe.md#conditions) contiennent les mêmes [étapes](./pipeline-syntaxe.md#pipeline-déclaratif) que la section [étapes](./pipeline-syntaxe.md#limites).

#### étapes

Contenant une séquence d'une ou plusieurs directives d'[étape](./pipeline-syntaxe.md#limites), la section `stages` est l'endroit où se trouve la majeure partie du « travail » décrit par un Pipeline. Il est recommandé que les `stages` contiennent au moins une directive d'étape pour chaque partie distincte du processus de livraison continue, telle que la compilation, le test et le déploiement.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Oui</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td><i>Aucun</i></td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>Dans le bloc <code>pipeline</code>ou à l'intérieur de <code>stage</code>.</td>
    </tr>
</table>
</div>

``` groovy title="<i>Exemple 4. Etapes, Pipeline Déclaratif</i>"
pipeline {
    agent any
    stages { //(1)
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

1. La section `stages`  suivra généralement les directives telles que `agent`,  `options`, etc.

#### échelons

La section `steps` définit une série d'une ou plusieurs [échelons](./pipeline-syntaxe.md#échelons) à exécuter dans une directive d'étape donnée.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Oui</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td><i>Aucun</i></td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>Dans chaque bloc<code>stage</code>.</td>
    </tr>
</table>
</div>

``` groovy title="<i>Exemple 5. Etape Unique, Pipeline Déclaratif</i>"
pipeline {
    agent any
    stages {
        stage('Example') {
            steps { //(1)
                echo 'Hello World'
            }
        }
    }
}
```

1. La section `steps` doit contenir un ou plusieurs échelons.

### Directives

#### environnement

La directive `environnement` spécifie une séquence de paires clé-valeur qui seront définies comme variables d'environnement pour tous les échelons, ou pour des échelons spécifiques, en fonction de l'emplacement de la directive `environnement` dans le Pipeline.

Cette directive prend en charge une méthode d'aide spéciale `credentials()` qui peut être utilisée pour accéder à des informations d'identification prédéfinies à l'aide de leur identifiant dans l'environnement Jenkins.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Non</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td><i>Aucun</i></td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>À l'intérieur du bloc<code>pipeline</code>ou dans les directives<code>stage</code>.</td>
    </tr>
</table>
</div>

##### Types d'informations d'identification pris en charge

**Texte secret**<br>
La variable d'environnement spécifiée sera définie sur le contenu du texte secret.

**Fichier secret**<br>
La variable d'environnement spécifiée sera définie sur l'emplacement du fichier temporairement créé.

**Nom d'utilisateur et mot de passe**<br>
La variable d'environnement spécifiée sera définie sur nom `username:password` et deux variables d'environnement supplémentaires seront automatiquement définies : `MYVARNAME_USR` et `MYVARNAME_PSW` respectivement.

**SSH avec clé privée**
La variable d'environnement spécifiée sera définie sur l'emplacement du fichier de clé SSH créé temporairement et deux variables d'environnement supplémentaires seront automatiquement définies : `MYVARNAME_USR` et `MYVARNAME_PSW` (contenant la phrase de passe).

!!! note
    Un type d'informations d'identification non pris en charge provoque l'échec du Pipeline avec le message suivant :<br>
    `org.jenkinsci.plugins.credentialsbinding.impl.CredentialNotFoundException : Aucun gestionnaire de liaison approprié n'a pu être trouvé pour le type <unsupportedType>`.

``` groovy title="<i>Exemple 6. Identifiants de Texte Secret, Pipeline Déclaratif</i>"
pipeline {
    agent any
    environment { //(1)
        CC = “clang”
    }
    stages {
        stage(“Example”) {
            environment { //(2)
                AN_ACCESS_KEY = credentials(“my-predefined-secret-text”) //(3)
            }
            steps {
                sh “printenv”
            }
        }
    }
}
```

1. Une directive `environnement` utilisée dans le bloc `pipeline` de niveau supérieur s'appliquera à toutes les étapes du pipeline.
2. Une directive `environnement` définie dans une `stage` n'appliquera les variables d'environnement données qu'aux échelons de cette `stage`.
3. Le bloc `environnement` dispose d'une méthode d'aide `credentials()` qui peut être utilisée pour accéder aux informations d'identification prédéfinies par leur identifiant dans l'environnement Jenkins.

``` groovy title="<i>Exemple 7. Identifiant et Mot de Passe</i>"
pipeline {
    agent any
    stages {
        stage('Example Username/Password') {
            environment {
                SERVICE_CREDS = credentials('my-predefined-username-password')
            }
            steps {
                sh 'echo "Service user is $SERVICE_CREDS_USR"'
                sh 'echo "Service password is $SERVICE_CREDS_PSW"'
                sh 'curl -u $SERVICE_CREDS https://myservice.example.com'
            }
        }
        stage('Example SSH Username with private key') {
            environment {
                SSH_CREDS = credentials('my-predefined-ssh-creds')
            }
            steps {
                sh 'echo "SSH private key is located at $SSH_CREDS"'
                sh 'echo "SSH user is $SSH_CREDS_USR"'
                sh 'echo "SSH passphrase is $SSH_CREDS_PSW"'
            }
        }
    }
}
```

## options

La directive `options` permet de configurer des options spécifiques au Pipeline à partir du pipeline lui-même. Le pipeline fournit un certain nombre de ces options, telles que `buildDiscarder`, mais elles peuvent également être fournies par des plugins, tels que `timestamps`.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Non</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td><i>Aucun</i></td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>À l'intérieur du bloc<code>pipeline</code>ou (avec certaines limitations) dans les directives<code>stage</code>.</td>
    </tr>
</table>
</div>

##### Options disponibles

**buildDiscarder**<br>
Conservez les artefacts et les sorties de console pour un nombre spécifique d'exécutions récentes du Pipeline. Par exemple : `options { buildDiscarder(logRotator(numToKeepStr: “1”)) }`.

**checkoutToSubdirectory**<br>
Effectuez le contrôle automatique des sources dans un sous-répertoire de l'espace de travail. Par exemple : `options { checkoutToSubdirectory(“foo”) }`.

**disableConcurrentBuilds**<br>
Interdit les exécutions simultanées du Pipeline. Peut être utile pour empêcher les accès simultanés aux ressources partagées, etc. Par exemple : `options { disableConcurrentBuilds() }` pour mettre une compilation en file d'attente lorsqu'une construction du Pipeline est déjà en cours d'exécution, ou `options { disableConcurrentBuilds(abortPrevious: true) }` pour interrompre celle en cours et démarrer la nouvelle compilation.

**disableResume**<br>
Ne permet pas au Pipeline de reprendre si le contrôleur redémarre. Par exemple `: options { disableResume() }`.

**newContainerPerStage**<br>
Utilisé avec l'agent de niveau supérieur `docker` ou `dockerfile`. Lorsqu'il est spécifié, chaque étape s'exécute dans un nouveau conteneur déployé sur le même nœud, plutôt que toutes les étapes s'exécutant dans le même déploiement de conteneur.

**overrideIndexTriggers**<br>
Permet de remplacer le traitement par défaut des déclencheurs d'indexation des branches. Si les déclencheurs d'indexation des branches sont désactivés au niveau du label multibranche ou de l'organisation, `options { overrideIndexTriggers(true) }` les activera uniquement pour cette tâche. Sinon, `options { overrideIndexTriggers(false) }` désactivera les déclencheurs d'indexation des branches uniquement pour cette tâche.

**preserveStashes**<br>
Conserve les _stashes_ des constructions terminées, pour une utilisation avec le redémarrage de l'étape. Par exemple : `options { preserveStashes() }` pour conserver les _stashes_ de la dernière construction terminée, ou `options { preserveStashes(buildCount: 5) }` pour conserver les _stashes_ des cinq dernieres compilations terminées.

**quietPeriod**<br>
Définit la période de silence, en secondes, pour le Pipeline, en remplaçant la valeur par défaut globale. Par exemple : `options { quietPeriod(30) }`.

**retry**<br>
En cas d'échec, réessaie l'ensemble du Pipeline le nombre de fois spécifié. Par exemple : `options { retry(3) }`.

**skipDefaultCheckout**<br>
Ignorer la vérification du code à partir du contrôle de source par défaut dans la directive `agent`. Par exemple : `options { skipDefaultCheckout() }`.

**skipStagesAfterUnstable**<br>
Ignore les étapes une fois que le statut de la compilation est passé à UNSTABLE. Par exemple : `options { skipStagesAfterUnstable() }`.

**timeout**<br>
Définit un délai d'expiration pour l'exécution du Pipeline, après lequel Jenkins doit l'abandonner. Par exemple : `options { timeout(time: 1, unit: “HOURS”) }`.

``` groovy title="<i>Exemple 8. Délai d'attente Global, Pipeline Déclaratif</i>"
pipeline {
    agent any
    options {
        timeout(time: 1, unit: 'HOURS') //(1)
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

1. Spécifie un délai d'exécution global d'une heure, après quoi Jenkins interrompra l'exécution du Pipeline.

**timestamps**<br>
Ajoute en tête de toutes les sorties de console générées par l'exécution du Pipeline l'heure à laquelle la ligne a été émise. Par exemple : `options { timestamps() }`.

**parallelsAlwaysFailFast**<br>
Définit échec rapide sur vrai pour toutes les étapes parallèles suivantes du Pipeline. Par exemple : `options { parallelsAlwaysFailFast() }`.

**disableRestartFromStage**<br>
Désactive complètement l'option « Restart From Stage » visible dans l'interface utilisateur classique de Jenkins et dans Blue Ocean. Par exemple : `options { disableRestartFromStage() }`. Cette option ne peut pas être utilisée à l'intérieur de l'étape.

!!! info
    Une liste complète des options disponibles est en attente de la finalisation du[] ticket d'assistance 820](https://github.com/jenkins-infra/helpdesk/issues/820).

##### Options d'étapes

La directive `options` pour une `stage` est similaire à la directive `options` à la racine du Pipeline. Cependant, les `options` au niveau de la `stage` ne peuvent contenir que des échelons telles que `retry`, `timeout` ou `timestamps`, ou des options déclaratives pertinentes pour une `stage`, telles que `skipDefaultCheckout`.

Au sein d'une `stage`, les échelons de la directive `options` sont invoquées avant d'entrer dans l'`agent` ou de vérifier les conditions `when`.

###### Options d'étape disponibles

**skipDefaultCheckout**<br>
Ignore la vérification du code à partir du contrôle de source par défaut dans la directive `agent`. Par exemple : `options { skipDefaultCheckout() }`.

**timeout**<br>
Définit une période d'expiration pour cette étape, après laquelle Jenkins doit l'abandonner. Par exemple : `options { timeout(time: 1, unit: “HOURS”) }`.

``` groovy title="<i>Exemple 9. Délai d'attente d'étape, Pipeline Déclaratif</i>"
pipeline {
    agent any
    stages {
        stage('Example') {
            options {
                timeout(time: 1, unit: 'HOURS') //(1)
            }
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

1. Spécifie un délai d'exécution d'une heure pour l'étape Example, après quoi Jenkins interrompra l'exécution du Pipeline.

**retry**<br>
En cas d'échec, réessayez cette étape le nombre de fois spécifié. Par exemple `: options { retry(3) }`.

**timestamps**<br>
Ajoutez en tête de toutes les sorties de console générées pendant cette étape l'heure à laquelle la ligne a été émise. Par exemple : `options { timestamps() }`.

**parameters**<br>
La directive `parameters` fournit une liste de paramètres que l'utilisateur doit fournir lors du déclenchement du Pipeline. Les valeurs de ces paramètres spécifiés par l'utilisateur sont mises à la disposition des étapes du Pipeline via l'objet `params`. Reportez-vous à la section [Paramètres, Pipeline déclaratif](#exemple-de-parametres) pour connaître son utilisation spécifique.

Chaque paramètre possède un _nom_ et une _valeur_, en fonction du type de paramètre. Ces informations sont exportées sous forme de variables d'environnement au démarrage de la compilation, ce qui permet aux parties suivantes de la configuration de compilation d'accéder à ces valeurs. Par exemple, utilisez la syntaxe `${PARAMETER_NAME}` avec les shells POSIX tels que `bash` et `ksh`, la syntaxe`${Env:PARAMETER_NAME}` avec PowerShell ou la syntaxe `%PARAMETER_NAME%` avec Windows `cmd.exe`.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Non</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td><i>Aucun</i></td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>Une seule fois, à l'intérieur du bloc<code>pipeline</code>.</td>
    </tr>
</table>
</div>

##### Paramètres disponibles

**string**<br>
Paramètre de type chaîne, par exemple : `parameters { string(name: “DEPLOY_ENV”, defaultValue: “staging”, description: “”) }`.

**text**<br>
Paramètre de type texte pouvant contenir plusieurs lignes, par exemple : `parameters { text(name: “DEPLOY_TEXT”, defaultValue: “One\nTwo\nThree\n”, description: “”) }`.

**booleanParam**<br>
Paramètre booléen, par exemple : `parameters { booleanParam(name: “DEBUG_BUILD”, defaultValue: true, description: “”) }`.

**choice**<br>
Un paramètre de choix, par exemple : `parameters { choice(name: “CHOICES”, choices: [“one”, “two”, “three”], description: “”) }`. La première valeur est la valeur par défaut.

**password**<br>
Un paramètre de mot de passe, par exemple : `parameters { password(name: “PASSWORD”, defaultValue: “SECRET”, description: “A secret password”) }`.

``` groovy title="<i>Exemple 10. Paramètres, Pipeline Déclaratif</i>"
pipeline {
    agent any
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'À qui dois-je dire bonjour ?')

        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Entrez quelques informations sur la personne.')

        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Basculer cette valeur')

        choice(name: 'CHOICE', choices: ['Un', 'Deux', 'Trois'], description: 'Choisis quelque chose')

        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Entre un mot de passe')
    }
    stages {
        stage('Example') {
            steps {
                echo "Hello ${params.PERSON}"

                echo "Biography: ${params.BIOGRAPHY}"

                echo "Toggle: ${params.TOGGLE}"

                echo "Choice: ${params.CHOICE}"

                echo "Password: ${params.PASSWORD}"
            }
        }
    }
}
```

!!! info
    Une liste complète des paramètres disponibles est en attente de la finalisation du [ticket d'assistance 820](https://github.com/jenkins-infra/helpdesk/issues/820).

#### déclencheurs

La directive `triggers` définit les méthodes automatisées selon lesquelles le Pipeline doit être redéclenché. Pour les Pipelines intégrés à une source telle que GitHub ou BitBucket, les déclencheurs peuvent ne pas être nécessaires, car une intégration basée sur des webhooks est probablement déjà présente. Les déclencheurs actuellement disponibles sont `cron`, `pollSCM` et `upstream`.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Non</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td><i>Aucun</i></td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>Une seule fois, à l'intérieur du bloc<code>pipeline</code>.</td>
    </tr>
</table>
</div>

**cron**<br>
Accepte une chaîne de type cron pour définir un intervalle régulier auquel le Pipeline doit être redéclenché, par exemple : `triggers { cron(“H */4 * * 1-5”) }`.

**pollSCM**<br>
Accepte une chaîne de caractères de type cron pour définir un intervalle régulier auquel Jenkins doit vérifier les nouvelles modifications de la source. Si de nouvelles modifications existent, le Pipeline sera redéclenché. Par exemple : `triggers { pollSCM(“H */4 * * 1-5”) }`.

**upstream**<br>
Accepte une chaîne de tâches séparées par des virgules et un seuil. Lorsque l'une des tâches de la chaîne atteint le seuil minimum, le Pipeline est relancé. Par exemple : `triggers { upstream(upstreamProjects: “job1,job2”, threshold: hudson.model.Result.SUCCESS) }`.

!!! info
    Le déclencheur `pollSCM` n'est disponible que dans Jenkins 2.22 ou versions ultérieures.

``` groovy title="<i>Exemple 11. Déclencheur, Pipeline Déclaratif</i>""
pipeline {
    agent any
    triggers {
        cron('H */4 * * 1-5')
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

#### Syntaxe cron Jenkins

La syntaxe cron Jenkins suit la syntaxe de [l'utilitaire cron](https://en.wikipedia.org/wiki/Cron) (avec quelques différences mineures). Plus précisément, chaque ligne se compose de 5 champs séparés par des tabulations ou des espaces :

| MINUTE | HEURE | JDM | MOIS | JDS |
| --------- | --------- | --------- | -------- | ---------- |
| Minutes dans l'heure (0-59) | L'heure de la journée (0-23) | Le jour du mois (1-31) | Le mois (1-12) | Le jour de la semaine (0-7), où 0 et 7 correspondent au dimanche. |

Pour spécifier plusieurs valeurs pour un champ, les opérateurs suivants sont disponibles. Par ordre de priorité,

* `*` spécifie toutes les valeurs valides ;
* `M-N` spécifie une plage de valeurs ;
* `M-N/X` ou `*/X` parcourt par intervalles de `X` la plage spécifiée ou toute la plage valide ;
* `A,B,…​,Z`  énumère plusieurs valeurs.

Pour permettre aux tâches planifiées périodiquement de produire une charge uniforme sur le système, le symbole `H` (pour « hash ») doit être utilisé dans la mesure du possible. Par exemple, l'utilisation de `0 0 * * *` pour une douzaine de tâches quotidiennes entraînera un pic important à minuit. En revanche, l'utilisation de `H H * * *` permettra toujours d'exécuter chaque tâche une fois par jour, mais pas toutes en même temps, ce qui permettra de mieux utiliser les ressources limitées.


Le symbole `H` peut être utilisé avec une plage. Par exemple, `H H(0-7) * * *` signifie une heure comprise entre minuit et 7 h 59. Vous pouvez également utiliser des intervalles avec `H`, avec ou sans plages.

Le symbole `H` peut être considéré comme une valeur aléatoire sur une plage, mais il s'agit en fait d'un hachage du nom de la tâche, et non d'une fonction aléatoire, de sorte que la valeur reste stable pour un projet donné.

Sachez que pour le champ du jour du mois, les cycles courts tels que `*/3` ou `H/3` ne fonctionneront pas de manière cohérente vers la fin de la plupart des mois, en raison de la durée variable des mois. Par exemple, `*/3` s'exécutera les 1er, 4e, ... 31e jours d'un mois long, puis à nouveau le jour suivant du mois suivant. Les hachages sont toujours choisis dans la plage 1-28, donc `H/3` produira un écart entre les exécutions de 3 à 6 jours à la fin d'un mois. Les cycles plus longs auront également des durées incohérentes, mais l'effet peut être relativement moins perceptible.

Les lignes vides et les lignes commençant par`#` seront ignorées comme des commentaires.

De plus, `@yearly`, `@annually`, `@monthly`, `@weekly`, `@daily`, `@midnight` et `@hourly` sont pris en charge en tant qu'alias pratiques. Ceux-ci utilisent le système de hachage pour l'équilibrage automatique. Par exemple, `@hourly` est identique à `H * * * *` et peut signifier à tout moment pendant l'heure. `@midnight` signifie en fait à un moment donné entre minuit et 2 h 59.

<div class="petite-police">
<table>
    <tr><td>toutes les quinze minutes (peut-être à :07, :22, :37, :52)</td></tr>
    <tr><td><code>triggers{ cron(“H/15 * * * *”) }</code></td></tr>
    <tr><td>toutes les dix minutes dans la première moitié de chaque heure (trois fois, peut-être à :04, :14, :24)</td></tr>
    <tr><td><code>triggers{ cron(“H(0-29)/10 * * * *”) }</code></td></tr>
    <tr><td>une fois toutes les deux heures à 45 minutes après l'heure, de 9 h 45 à 15 h 45 tous les jours de la semaine.</td></tr>
    <tr><td><code>triggers{ cron(“45 9-16/2 * * 1-5”) }</code></td></tr>
    <tr><td>une fois toutes les deux heures entre 9 h et 17 h tous les jours de la semaine (peut-être à 10 h 38, 12 h 38, 14 h 38, 16 h 38).</td></tr>
    <tr><td><code>triggers{ cron(“H H(9-16)/2 * * 1-5”) }</code></td></tr>
    <tr><td>une fois par jour les 1er et 15 de chaque mois, sauf en décembre.</td></tr>
    <tr><td><code>triggers{ cron(“H H 1,15 1-11 *”) }</code></td></tr>
</table>
</div>

#### étape

La directive `stage` se trouve dans la section `stages` et doit contenir une section [steps](#échelons), une section `agent` facultative ou d'autres directives spécifiques à l'étape. Concrètement, tout le travail réel effectué par un Pipeline sera regroupé dans une ou plusieurs directives `stage`.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Au moins un.</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td>Un paramètre obligatoire, une chaîne de caractères pour le nom de l'étape.</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>À l'intérieur de la section<code>stages</code>.</td>
    </tr>
</table>
</div>

``` groovy title="<i>Exemple 12. Etapes, Pipeline Déclaratif</i>"
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

**tools**<br>
Section définissant les outils à installer automatiquement et à ajouter au `PATH`. Elle est ignorée si `agent none` est spécifié.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Non.</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td><i>Aucun</i>.</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>À l'intérieur du bloc<code>pipeline</code>ou d'un bloc<code>stage</code>.</td>
    </tr>
</table>
</div>


##### Outils supportés

**maven**<br>
**jdk**<br>
**gradle**

``` groovy title="<i>Exemple 13. Outils, Pipeline Déclaratif</i>"
pipeline {
    agent any
    tools {
        maven 'apache-maven-3.0.1' //(1)
    }
    stages {
        stage('Example') {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
```

1. Le nom de l'outil doit être préconfiguré dans Jenkins sous **Manage Jenkins → Tools**.

#### input

La directive `input` sur une `stage` vous permet de demander une entrée à l'aide de [l'échelon' input](https://www.jenkins.io/doc/pipeline/steps/pipeline-input-step/#input-wait-for-interactive-input). La `stage` s'interrompt après l'application des `options` et avant d'entrer dans le bloc `agent` pour cette `stage` ou d'évaluer la condition `when` de la `stage`. Si `input` est approuvée, `stage` se poursuit. Tous les paramètres fournis dans le cadre de la soumission de `input` seront disponibles dans l'environnement pour le reste de `stage`.

##### Options de configuration

**message**<br>
Obligatoire. Ce message s'affichera à l'utilisateur lorsqu'il soumettra `input`.

**id**<br>
Identifiant facultatif pour cet `input`. La valeur par défaut est basée sur le nom de la `stage`.

**ok**<br>
Texte facultatif pour le bouton « ok » du formulaire `input`.

**submitter**<br>
Liste facultative, séparée par des virgules, des noms d'utilisateurs ou de groupes externes autorisés à soumettre cet `input`. Par défaut, tous les utilisateurs sont autorisés.

**submitterParameter**<br>
Nom facultatif d'une variable d'environnement à définir avec le nom du `submitter`, le cas échéant.

``` groovy title="<i>Exemple 14. Echelon Input, Pipeline Déclaratif</i>"
pipeline {
    agent any
    stages {
        stage('Exemple') {
            input {
                message "Voulez-vous continuer ?"
                ok "Oui, nous devrions"
                submitter "alice,bob"
                parameters {
                    string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'À qui dois-je dire bonjour ?')
                }
            }
            steps {
                echo "Bonjour, ${PERSON}, enchantée."
            }
        }
    }
}
```

#### when

La directive `when` permet au Pipeline de déterminer si l'étape doit être exécutée en fonction de la condition donnée. La directive `when` doit contenir au moins une condition. Si la directive `when` contient plusieurs conditions, toutes les conditions enfants doivent renvoyer la valeur _true_ (vrai) pour que l'étape soit exécutée. Cela revient à imbriquer les conditions enfants dans une condition `allOf` (voir les [exemples](#exemple-when) ci-dessous). Si une condition `anyOf` est utilisée, notez que la condition ignore les tests restants dès que la première condition « true » est trouvée.

Des structures conditionnelles plus complexes peuvent être construites à l'aide des conditions d'imbrication : `not`, `allOf` ou `anyOf`. Les conditions d'imbrication peuvent l'être à n'importe quelle profondeur arbitraire.

<div class="petite-police">
<table>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Requis</strong></td><td>Non.</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Paramètres</strong></td><td><i>Aucun</i>.</td>
    </tr>
    <tr>
        <td style="background-color:#3b3b3b;"><strong>Autorisé</strong></td><td>À l'intérieur d'une directive<code>stage</code>.</td>
    </tr>
</table>
</div>

##### Conditions intégrées

**branch**<br>
Exécutez l'étape lorsque la branche en cours de création correspond au modèle de branche (chemin d'accès ANT) fourni, par exemple : `when { branch “master” }`. Notez que cela ne fonctionne que sur un Pipeline multibranches.

Le paramètre facultatif `comparator` peut être ajouté après un attribut pour spécifier comment les modèles sont évalués pour une correspondance :

* `EQUALS` pour une simple comparaison de chaînes ;
* `GLOB` (valeur par défaut) pour un chemin glob de style ANT (identique à celui utilisé pour `changeset`, par exemple) ;
* `REGEXP` pour une correspondance d'expression régulière.

Par exemple : `when { branch pattern: « release-\\d+ », comparator: « REGEXP »}`.

**buildingTag**<br>
Exécute l'étape lorsque la compilation crée une balise. Par exemple : `when { buildingTag() }`.

**changelog**<br>
Exécute l'étape si le journal des modifications SCM de la construction contient un modèle d'expression régulière donné, par exemple : `when { changelog “.*^\\[DEPENDENCY\\] .+$” }`.

**changeset**<br>
Exécute l'étape si le changeset SCM de la construction contient un ou plusieurs fichiers correspondant au modèle donné. Exemple : `when { changeset « **/*.js » }`.

Le paramètre facultatif `comparator` peut être ajouté après un attribut pour spécifier comment les modèles sont évalués pour une correspondance :

* `EQUALS` pour une simple comparaison de chaînes ;
* `GLOB` (par défaut) pour un chemin ANT style glob insensible à la casse (cela peut être désactivé avec le paramètre caseSensitive) ;
* `REGEXP` pour une correspondance d'expression régulière.

Par exemple : `when { changeset pattern: « .TEST\\.java », comparator: “REGEXP” }` ou `when { changeset pattern: « */*TEST.java », caseSensitive: true }`.

**changeRequest**<br>
Exécute l'étape si la construction actuelle correspond à une « demande de modification » (également appelée _Pull Request_ sur GitHub et Bitbucket, _Merge Request_ sur GitLab, _Change_ dans Gerrit, etc.). Lorsqu'aucun paramètre n'est transmis, l'étape s'exécute à chaque demande de modification, par exemple : `when { changeRequest() }`.

En ajoutant un attribut de filtre avec un paramètre à la demande de modification, l'étape peut être exécutée uniquement sur les demandes de modification correspondantes. Les attributs possibles sont `id`, `target`, `branch`, `fork`, `url`, `title`, `author`, `authorDisplayName` et `authorEmail`. Chacun d'entre eux correspond à une variable d'environnement `CHANGE_*`, par exemple : `when { changeRequest target: “master” }`.

Le paramètre facultatif `comparator` peut être ajouté après un attribut pour spécifier comment les modèles sont évalués pour une correspondance :

* `EQUALS` pour une simple comparaison de chaînes (par défaut) ;
* `GLOB` pour un chemin glob de style ANT (identique à celui utilisé pour les changements, par exemple) ;
* `REGEXP` pour la correspondance d'expressions régulières.

Exemple : `when { changeRequest authorEmail: « [\\w_-.]+@example.com », comparator: “REGEXP” }`.

**environment**<br>
Exécute l'étape lorsque la variable d'environnement spécifiée est définie sur la valeur donnée, par exemple : `when { environment name: “DEPLOY_TO”, value: “production” }`.

**equals**<br>
Exécute l'étape lorsque la valeur attendue est égale à la valeur réelle, par exemple : `when { equals expected: 2, actual: currentBuild.number }`.

**expression**<br>
Exécute l'étape lorsque l'expression Groovy spécifiée est évaluée à vrai, par exemple : `when { expression { return params.DEBUG_BUILD } }`.

!!! info
    Lorsque vous renvoyez des chaînes à partir de vos expressions, celles-ci doivent être converties en booléens ou renvoyer `null` pour être évaluées comme fausses. Le simple fait de renvoyer « 0 » ou « false » (faux) sera toujours évalué comme « true » (vrai).

**tag**<br>
Exécute l'étape si la variable `TAG_NAME` correspond au modèle donné. Par exemple : `when { tag « release-* » }`. Si un modèle vide est fourni, l'étape s'exécutera si la variable `TAG_NAME` existe (identique à `buildingTag()`).

Le paramètre facultatif `comparator` peut être ajouté après un attribut pour spécifier comment les modèles sont évalués pour une correspondance :

* `EQUALS` pour une simple comparaison de chaînes ;
* `GLOB` (par défaut) pour un chemin glob de style ANT (identique à l'exemple `changeset`), ou
* `REGEXP` pour une correspondance d'expression régulière.

Par exemple : `when { tag pattern: « release-\\d+ », comparator: « REGEXP »}`.

**not**<br>
Exécute l'étape lorsque la condition imbriquée est fausse. Doit contenir une condition. Par exemple : `when { not { branch “master” } }`.

**allOf**<br>
Exécute l'étape lorsque toutes les conditions imbriquées sont vraies. Doit contenir au moins une condition. Par exemple : `when { allOf { branch “master”; environment name: “DEPLOY_TO”, value: “production” } }`.

**anyOf**<br>
Exécute l'étape lorsqu'au moins une des conditions imbriquées est vraie. Doit contenir au moins une condition. Par exemple : `when { anyOf { branch “master”; branch “staging” } }`.

**triggeredBy**<br>
Exécute l'étape lorsque la compilation actuelle a été déclenchée par le paramètre donné. Par exemple :

* `when { triggeredBy “SCMTrigger” }` ;
* `when { triggeredBy “TimerTrigger” }` ;
* `when { triggeredBy “BuildUpstreamCause” }` ;
* `when { triggeredBy cause: « UserIdCause », detail: « vlinde » }`.

##### Évaluation de `when` avant d'entrer `agent` dans une `stage`

Par défaut, la condition `when` pour une `stage` sera évaluée après être entré dans `agent` pour cette `stage`, si elle est définie. Cependant, cela peut être modifié en spécifiant l'option `beforeAgent` dans le bloc `when`. Si `beforeAgent` est défini sur `true`, la condition `when` sera évaluée en premier, et `agent` ne sera entré que si la condition `when` est évaluée à true.

##### Évaluer `when` avant la directive `input`

Par défaut, la condition _when_ d'une _stage_ n'est pas évaluée avant _input_, si celle-ci est définie. Cependant, cela peut être modifié en spécifiant l'option `beforeInput` dans le bloc when. Si `beforeInput` est défini sur _true_, la condition _when_ sera évaluée en premier, et _input_ ne sera saisie que si la condition _when_ est évaluée à _true_.

`beforeInput true` a priorité sur `beforeAgent` true.

##### Évaluation de la condition `when` avant la directive `options`

Par défaut, la condition `when` d'une `stage` sera évaluée après la saisie des `options` pour cette `stage`, si elles sont définies. Cependant, cela peut être modifié en spécifiant l'option `beforeOptions` dans le bloc `when`. Si `beforeOptions` est défini sur `true`, la condition `when` sera évaluée en premier, et les `options` ne seront saisies que si la condition `when` est évaluée à _true_.

`beforeOptions true` a priorité sur `beforeInput true` et `beforeAgent true`.

``` groovy title="<i>Exemple 15. Condition Unique, Pipeline Déclaratif</i>"
pipeline {
    agent any
    stages {
        stage('Exemple de Compilation') {
            steps {
                echo 'Bonjour le Monde'
            }
        }
        stage('Exemple de Déploiement') {
            when {
                branch 'production'
            }
            steps {
                echo 'Déploiement'
            }
        }
    }
}
```

``` groovy title="<i>Exemple 16. Condition Multiple, Pipeline Déclaratif</i>"
pipeline {
    agent any
    stages {
        stage('Exemple de Compilation') {
            steps {
                echo 'Bonjour le Monde'
            }
        }
        stage('Exemple de Déploiement') {
            when {
                branch 'production'
                environment name: 'DEPLOIEMENT_VERS', value: 'production'
            }
            steps {
                echo 'Déploiement'
            }
        }
    }
}
```

``` groovy title="<i>Exemple 17. Condition imbriquée (même comportement que l'exemple précédent)</i>"
pipeline {
    agent any
    stages {
        stage('ExEmple de Compilation') {
            steps {
                echo 'Bonjour le Monde'
            }
        }
        stage('Exemple de Déploiement') {
            when {
                allOf {
                    branch 'production'
                    environment name: 'DEPLOIEMENT_VERS', value: 'production'
                }
            }
            steps {
                echo 'Deploiement'
            }
        }
    }
}
```

``` groovy title="<i>Exemple 18. Condition multiple et condition imbriquée</i>"
pipeline {
    agent any
    stages {
        stage('Exemple de Compilation') {
            steps {
                echo 'Bonjour le Monde'
            }
        }
        stage('Exemple de Déploiement' ) {
            when {
                branch 'production'
                anyOf {
                    environment name: 'DEPLOIEMENT_VERS', value: 'production'
                    environment name: 'DEPLOIEMENT_VERS', value: 'staging'
                }
            }
            steps {
                echo 'Déploiement'
            }
        }
    }
}
```

``` groovy title="<i>Exemple 19. Condition d'expression et condition imbriquée</i>"
pipeline {
    agent any
    stages {
        stage('Exemple de Compilation') {
            steps {
                echo 'Bonjour le Monde'
            }
        }
        stage('Exemple de Déploiement') {
            when {
                expression { NOM_BRANCHE ==~ /(production|staging)/ }
                anyOf {
                    environment name: 'DEPLOIEMENT_VERS', value: 'production'
                    environment name: 'DEPLOIEMENT_VERS', value: 'staging'
                }
            }
            steps {
                echo 'Déploiement'
            }
        }
    }
}
```

``` groovy title="<i>Exemple 20. <code>beforeAgent</code></i>"
pipeline {
    agent none
    stages {
        stage('Exemple de Compilation') {
            steps {
                echo 'Bonjour le Monde'
            }
        }
        stage('Exemple de Déploiement') {
            agent {
                label "étiquette quelconque"
            }
            when {
                beforeAgent true
                branch 'production'
            }
            steps {
                echo 'Déploiement'
            }
        }
    }
}
```

``` groovy title="<i>Exemple 21. <code>beforeInput</code></i>"
pipeline {
    agent none
    stages {
        stage('Exemple de Compilation ') {
            steps {
                echo 'Bonjour le Monde'
            }
        }
        stage('Exemple de Déploiement') {
            when {
                beforeInput true
                branch 'production'
            }
            input {
                message "Déployer en production ?"
                id "simple entrée"
            }
            steps {
                echo 'Déploiement'
            }
        }
    }
}
```

``` groovy title="<i>Exemple 22. <code>beforeOptions</code></i>"
pipeline {
    agent none
    stages {
        stage('Exemple de build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Exemple de déploiement') {
            when {
                beforeOptions true
                branch 'testing'
            }
            options {
                lock label: 'testing-deploy-envs', quantity: 1, variable: 'deployEnv'
            }
            steps {
                echo « Deploying to ${deployEnv} »
            }
        }
    }
}
```

``` groovy title="<i>Exemple 23. <code>triggeredBy</code></i>"
pipeline {
    agent none
    stages {
        stage('Exemple de Compilation') {
            steps {
                echo 'Bonjour le Monde'
            }
        }
        stage('Exemple de Déploiement') {
            when {
                triggeredBy "TimerTrigger"
            }
            steps {
                echo 'Déploiement'
            }
        }
    }
}
```

### Étapes séquentielles

Les étapes dans les Pipelines Déclaratifs peuvent comporter une section `stages` contenant une liste d'étapes imbriquées à exécuter dans un ordre séquentiel.

!!! info
    Une étape doit comporter un seul et unique `steps`, `stages`, `parrallel` ou `matrix`. Il n'est pas possible d'imbriquer un bloc `parrallel` ou `matrix` dans une directive `stages` si celle-ci est elle-même imbriquée dans un bloc `parrallel` ou `matrix`. Cependant, une directive `stage` dans un bloc `parrallel` ou `matrix` peut utiliser toutes les autres fonctionnalités d'une `stage`, y compris `agent`, `tools`, `when`, etc. 

``` groovy title="<i>Exemple 24. Étapes séquentielles, Pipeline Déclaratif</i>"
pipeline {
    agent none
    stages {
        stage('Non-Sequential Stage') {
            agent {
                label 'for-non-sequential'
            }
            steps {
                echo "On Non-Sequential Stage"
            }
        }
        stage('Sequential') {
            agent {
                label 'for-sequential'
            }
            environment {
                FOR_SEQUENTIAL = "some-value"
            }
            stages {
                stage('In Sequential 1') {
                    steps {
                        echo "In Sequential 1"
                    }
                }
                stage('In Sequential 2') {
                    steps {
                        echo "In Sequential 2"
                    }
                }
                stage('Parallel In Sequential') {
                    parallel {
                        stage('In Parallel 1') {
                            steps {
                                echo "In Parallel 1"
                            }
                        }
                        stage('In Parallel 2') {
                            steps {
                                echo "In Parallel 2"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

### Parallèle

Les étapes dans le Pipeline déclaratif peuvent comporter une section `parallel` contenant une liste d'étapes imbriquées à exécuter en parallèle.

!!! info
    Une étape doit comporter une et une seule `steps`, `stage`, `parallel` ou `matrix`. Il n'est pas possible d'imbriquer un bloc `parallel` ou `matrix` dans une directive `stage` si celle-ci est elle-même imbriquée dans un bloc `parallel` ou `matrix`. Cependant, une directive `stage` dans un bloc `parallel` ou `matrix` peut utiliser toutes les autres fonctionnalités d'une `stage`, y compris `agent`, `outils`, `when`, etc.

De plus, vous pouvez forcer l'abandon de toutes vos étapes `parallel` en cas d'échec de l'une d'entre elles, en ajoutant `failFast true` à la `stage` contenant le `parallel`. Une autre possibilité pour ajouter `failfast` consiste à ajouter une option à la définition du Pipeline : `parallelsAlwaysFailFast()`.

``` groovy title="<i>Exemple 25. Etapes Parallèles, Pipeline Déclaratif</i>"
pipeline {
    agent any
    stages {
        stage('Non-Parallel Stage') {
            steps {
                echo 'This stage will be executed first.'
            }
        }
        stage('Parallel Stage') {
            when {
                branch 'master'
            }
            failFast true
            parallel {
                stage('Branch A') {
                    agent {
                        label "for-branch-a"
                    }
                    steps {
                        echo "On Branch A"
                    }
                }
                stage('Branch B') {
                    agent {
                        label "for-branch-b"
                    }
                    steps {
                        echo "On Branch B"
                    }
                }
                stage('Branch C') {
                    agent {
                        label "for-branch-c"
                    }
                    stages {
                        stage('Nested 1') {
                            steps {
                                echo "In stage Nested 1 within Branch C"
                            }
                        }
                        stage('Nested 2') {
                            steps {
                                echo "In stage Nested 2 within Branch C"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

``` groovy title="<i>Exemple 26. <code>parallelsAlwaysFailFast</code></i>"
pipeline {
    agent any
    options {
        parallelsAlwaysFailFast()
    }
    stages {
        stage('Non-Parallel Stage') {
            steps {
                echo 'This stage will be executed first.'
            }
        }
        stage('Parallel Stage') {
            when {
                branch 'master'
            }
            parallel {
                stage('Branch A') {
                    agent {
                        label "for-branch-a"
                    }
                    steps {
                        echo "On Branch A"
                    }
                }
                stage('Branch B') {
                    agent {
                        label "for-branch-b"
                    }
                    steps {
                        echo "On Branch B"
                    }
                }
                stage('Branch C') {
                    agent {
                        label "for-branch-c"
                    }
                    stages {
                        stage('Nested 1') {
                            steps {
                                echo "In stage Nested 1 within Branch C"
                            }
                        }
                        stage('Nested 2') {
                            steps {
                                echo "In stage Nested 2 within Branch C"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

### Matrice

Les étapes dans le Pipeline déclaratif peuvent comporter une section `matrix` définissant une matrice multidimensionnelle de combinaisons nom-valeur à exécuter en parallèle. Nous appellerons ces combinaisons « cellules » dans une matrice. Chaque cellule d'une matrice peut inclure une ou plusieurs étapes à exécuter séquentiellement à l'aide de la configuration de cette cellule.

!!! info
    Une étape doit comporter une et une seule `steps`, `stage`, `parallel` ou `matrix`. Il n'est pas possible d'imbriquer un bloc `parallel` ou `matrix` dans une directive `stage` si celle-ci est elle-même imbriquée dans un bloc `parallel` ou `matrix`. Cependant, une directive `stage` dans un bloc `parallel` ou `matrix` peut utiliser toutes les autres fonctionnalités d'une `stage`, y compris `agent`,  `tools`, `when`, etc. 

De plus, vous pouvez forcer l'abandon de toutes les cellules de votre `matrix` dès que l'une d'entre elles échoue, en ajoutant `failFast true` à la `stage` contenant la `matrix`. Une autre option pour ajouter `failfast` consiste à ajouter une option à la définition du pipeline : `parallelsAlwaysFailFast()`.

La section `matrix` doit inclure une section `axes` et une section `stages`. La section `axes` définit les valeurs de chaque `axis` de la matrice. La section `stages` définit une liste de `stages` à exécuter séquentiellement dans chaque cellule. Une `matrix` peut comporter une section `excludes` permettant de supprimer les cellules non valides de la matrice. La plupart des directives disponibles sur `stage`, notamment `agent`, `tools`, `when`, etc., peuvent également être ajoutées à `matrix` afin de contrôler le comportement de chaque cellule.

#### axes

La section `axes` spécifie une ou plusieurs directives`axis`. Chaque `axis` se compose d'un `name` et d'une liste de `values`. Toutes les valeurs de chaque axe sont combinées avec les autres pour produire les cellules.

``` groovy title="<i>Exemple 27. Un axe avec 3 cellules</i>"
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
    }
    // ...
}
```

``` groovy title="<i>Exemple 28. Deux axes avec 12 cellules (trois par quatre)</i>"
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'BROWSER'
            values 'chrome', 'edge', 'firefox', 'safari'
        }
    }
    // ...
}
```

``` groovy title="<i>Exemple 29. Matrice à trois axes avec 24 cellules (trois par quatre par deux)</i>."
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'BROWSER'
            values 'chrome', 'edge', 'firefox', 'safari'
        }
        axis {
            name 'ARCHITECTURE'
            values '32-bit', '64-bit'
        }
    }
    // ...
}
```

#### étapes

La section `stages` spécifie une ou plusieurs `stages` à exécuter séquentiellement dans chaque cellule. Cette section est identique à toute autre [section étapes](#étapes-séquentielles).

``` groovy title="<i>Exemple 30. Un axe avec 3 cellules, chaque cellule exécute trois étapes : « build », « test » et « deploy »</i>."
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
    }
    stages {
        stage('build') {
            // ...
        }
        stage('test') {
            // ...
        }
        stage('deploy') {
            // ...
        }
    }
}
```

``` groovy title="Exemple 31. Deux axes avec 12 cellules (trois par quatre)</i>"
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'BROWSER'
            values 'chrome', 'edge', 'firefox', 'safari'
        }
    }
    stages {
        stage('build-and-test') {
            // ...
        }
    }
}
```

#### excludes (facultatif)

La section facultative `excludes` permet aux auteurs de spécifier une ou plusieurs expressions de filtre `exclude` qui sélectionnent les cellules à exclure de l'ensemble étendu de cellules de matrice (alias, sparsening). Les filtres sont construits à l'aide d'une structure de directive de base composée d'une ou plusieurs directives `axis` d'exclusion, chacune avec un `name` et une liste de `values`.

Les directives `axis` à l'intérieur d'une `exclude` génèrent un ensemble de combinaisons (similaire à la génération des cellules de matrice). Les cellules de la matrice qui correspondent à toutes les valeurs d'une combinaison `exclude`` sont supprimées de la matrice. Si plusieurs directives `exclude`` sont fournies, chacune est évaluée séparément pour supprimer les cellules.

Lorsqu'il s'agit d'une longue liste de valeurs à exclure, les directives exclude `axis` peuvent utiliser `notValues` au lieu de `values`. Celles-ci excluront les cellules qui ne correspondent à aucune des valeurs transmises à `notValues`.

``` groovy title="<i>Exemple 32. Matrice à trois axes avec 24 cellules, exclure « 32 bits, Mac » (4 cellules exclues)</i>."
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'BROWSER'
            values 'chrome', 'edge', 'firefox', 'safari'
        }
        axis {
            name 'ARCHITECTURE'
            values '32-bit', '64-bit'
        }
    }
    excludes {
        exclude {
            axis {
                name 'PLATFORM'
                values 'mac'
            }
            axis {
                name 'ARCHITECTURE'
                values '32-bit'
            }
        }
    }
    // ...
}
```

Exclure la combinaison `linux`, `safari` et exclure toute plateforme autre que `windows` avec le navigateur `edge`.

``` groovy title="<i>Exemple 33. Matrice à trois axes avec 24 cellules, exclure les combinaisons « 32 bits, Mac » et les combinaisons de navigateurs non valides (9 cellules exclues)</i>."
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'BROWSER'
            values 'chrome', 'edge', 'firefox', 'safari'
        }
        axis {
            name 'ARCHITECTURE'
            values '32-bit', '64-bit'
        }
    }
    excludes {
        exclude {
            // 4 cells
            axis {
                name 'PLATFORM'
                values 'mac'
            }
            axis {
                name 'ARCHITECTURE'
                values '32-bit'
            }
        }
        exclude {
            // 2 cells
            axis {
                name 'PLATFORM'
                values 'linux'
            }
            axis {
                name 'BROWSER'
                values 'safari'
            }
        }
        exclude {
            // 3 more cells and '32-bit, mac' (already excluded)
            axis {
                name 'PLATFORM'
                notValues 'windows'
            }
            axis {
                name 'BROWSER'
                values 'edge'
            }
        }
    }
    // ...
}
```

#### Directives au niveau des cellules de la matrice (facultatif)

Matrice permet aux utilisateurs de configurer efficacement l'environnement global de chaque cellule, en ajoutant des directives au niveau de l'étape sous `matrix` elle-même. Ces directives se comportent de la même manière qu'avec une étape, mais elles peuvent également accepter des valeurs fournies par la matrice pour chaque cellule.

Les directives `axis` et `exclude` définissent l'ensemble statique de cellules qui composent la matrice. Cet ensemble de combinaisons est généré avant le début de l'exécution du Pipeline. Les directives « par cellule », en revanche, sont évaluées au moment de l'exécution.

Ces directives comprennent :

* [agent](#agent) ;
* [environment](#environnement) ;
* [input](#input) ;
* [options](#options) ;
* [post](#post) ;
 * [tools](#outils-supportés) ;
 * [when](#when)

``` groovy title="<i>Exemple 34. Exemple de matrice complète, Pipeline Déclaratif</i>"
pipeline {
    parameters {
        choice(name: 'PLATFORM_FILTER', choices: ['all', 'linux', 'windows', 'mac'], description: 'Run on specific platform')
    }
    agent none
    stages {
        stage('BuildAndTest') {
            matrix {
                agent {
                    label "${PLATFORM}-agent"
                }
                when { anyOf {
                    expression { params.PLATFORM_FILTER == 'all' }
                    expression { params.PLATFORM_FILTER == env.PLATFORM }
                } }
                axes {
                    axis {
                        name 'PLATFORM'
                        values 'linux', 'windows', 'mac'
                    }
                    axis {
                        name 'BROWSER'
                        values 'firefox', 'chrome', 'safari', 'edge'
                    }
                }
                excludes {
                    exclude {
                        axis {
                            name 'PLATFORM'
                            values 'linux'
                        }
                        axis {
                            name 'BROWSER'
                            values 'safari'
                        }
                    }
                    exclude {
                        axis {
                            name 'PLATFORM'
                            notValues 'windows'
                        }
                        axis {
                            name 'BROWSER'
                            values 'edge'
                        }
                    }
                }
                stages {
                    stage('Build') {
                        steps {
                            echo "Do Build for ${PLATFORM} - ${BROWSER}"
                        }
                    }
                    stage('Test') {
                        steps {
                            echo "Do Test for ${PLATFORM} - ${BROWSER}"
                        }
                    }
                }
            }
        }
    }
}
```

### Echelons

Les Pipelines déclaratifs peuvent utiliser toutes les étapes disponibles documentées dans la [Référence Échelons du Pipeline](https://www.jenkins.io/doc/pipeline/steps), qui contient une liste complète des échelons, auxquelles s'ajoutent ceux répertoriés ci-dessous, qui ne sont prises en charge que dans le Pipeline déclaratif.

#### script

L'étape `script` prend un bloc de [Pipeline scripté](#pipeline-scripté) et l'exécute dans le Pipeline déclaratif. Dans la plupart des cas, l'échelon `script` n'est pas nécessaire dans les Pipelines déclaratifs, mais elle peut constituer une « porte de sortie » utile. Les blocs de script de taille et/ou de complexité non négligeables doivent plutôt être déplacés vers des [bibliothèques partagées](./pipeline-bibliotheques-partagees.md).

``` groovy title="<i>Exemple 35. Bloc Script dans un Pipeline Déclaratif</i>"
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'

                script {
                    def browsers = ['chrome', 'firefox']
                    for (int i = 0; i < browsers.size(); ++i) {
                        echo "Testing the ${browsers[i]} browser"
                    }
                }
            }
        }
    }
}
```

## Pipeline scripté

Tout comme le [Pipeline déclaratif](#pipeline-déclaratif), le Pipeline scripté repose sur le sous-système Pipeline sous-jacent. Contrairement au Pipeline déclaratif, le pipeline scripté est en réalité un DSL[^1] polyvalent développé avec [Groovy](http://groovy-lang.org/syntax.html). La plupart des fonctionnalités offertes par le langage Groovy sont mises à la disposition des utilisateurs du Pipeline scripté, ce qui en fait un outil très expressif et flexible permettant de créer des Pipelines de livraison continue.

#### Contrôle de flux

Le Pipeline scripté est exécuté en série, du haut vers le bas d'un fichier `Jenkinsfile`, comme la plupart des scripts traditionnels en Groovy ou dans d'autres langages. Le contrôle de flux repose donc sur des expressions Groovy, telles que les conditions `if/else`, par exemple :

``` groovy title="<i>Exemple 36. Instruction conditionnelle if, Pipeline Scripté</i>."
node {
    stage('Exemple') {
        if (env.BRANCH_NAME == 'master') {
            echo 'Exécution sur la branche principale.'
        } else {
            echo 'Exécution ailleurs'
        }
    }
}
```

Une autre façon de gérer le contrôle du flux du Pipeline scripté consiste à utiliser la prise en charge de la gestion des exceptions de Groovy. Lorsque des [échelons](#échelons-scriptés) échouent pour une raison quelconque, elles génèrent une exception. La gestion des comportements en cas d'erreur doit utiliser les blocs` try/catch/finally` de Groovy, par exemple :

``` groovy title="<i>Exemple 37. Bloc Try/Catch, Pipeline Scripté</i>."
node {
    stage('Exemple') {
        try {
            sh 'exit 1'
        }
        catch (exc) {
            echo 'Quelque chose a échoué, je devrais sonner une alarme !'
            throw
        }
    }
}
```

### Échelons

Comme indiqué [au début de ce chapitre](./pipeline-presentation.md), la partie la plus fondamentale d'un Pipeline est l'« échelon ». Fondamentalement, les échelons indiquent à Jenkins ce qu'il doit faire et constituent les éléments de base de la syntaxe des Pipelines déclaratifs et scriptés.

Les Pipelines scriptés n'introduisent **pas** d'échelon spécifique à leur syntaxe ; la [référence des étapes de Pipeline](https://www.jenkins.io/doc/pipeline/steps) contient une liste complète des échelons fournis par les Pipelines et les plugins.

### Différences par rapport au Groovy standard

Afin d'assurer la durabilité, c'est-à-dire que les Pipelines en cours d'exécution puissent survivre à un redémarrage du [contrôleur](./glossaire.md#controller) Jenkins, le Pipeline scripté doit sérialiser les données vers le contrôleur. En raison de cette exigence de conception, certaines expressions idiomatiques Groovy telles que `collection.each { item → /* perform operation */ }` ne sont pas entièrement prises en charge. Pour plus d'informations, consultez [JENKINS-27421](https://issues.jenkins.io/browse/JENKINS-27421) et [JENKINS-26481](https://issues.jenkins.io/browse/JENKINS-26481).

## Comparaison syntaxique
<iframe width="800" height="420" src="https://www.youtube.com/embed/GJBlskiaRrI" title="What Is the Difference Between Scripted and Declarative Pipeline" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Cette vidéo présente certaines différences entre la syntaxe Pipeline scripté et Pipeline déclaratif.

Lorsque Jenkins Pipeline a été créé, Groovy a été choisi comme base. Jenkins est depuis longtemps livré avec un moteur Groovy intégré afin d'offrir des capacités de script avancées aux administrateurs et aux utilisateurs. De plus, les développeurs de Jenkins Pipeline ont trouvé que Groovy constituait une base solide sur laquelle construire ce qui est aujourd'hui appelé le DSL « Scripted Pipeline ». [^1].

En tant qu'environnement de programmation complet, le Pipeline scripté offre une flexibilité et une extensibilité considérables aux utilisateurs de Jenkins. La courbe d'apprentissage de Groovy n'est généralement pas souhaitable pour tous les membres d'une équipe donnée. Le Pipeline déclaratif a donc été créé afin d'offrir une syntaxe plus simple et plus subjective pour la création de Pipelines Jenkins.

Les deux sont fondamentalement identiques en termes de sous-système Pipeline. Il s'agit dans les deux cas d'implémentations durables du concept « Pipeline as code ». Elles peuvent toutes deux utiliser des étapes intégrées à Pipeline ou fournies par des plugins. Elles peuvent également utiliser des [bibliothèques partagées](./pipeline-bibliotheques-partagees.md).

Elles diffèrent toutefois en termes de syntaxe et de flexibilité. Le langage déclaratif limite les possibilités offertes à l'utilisateur avec une structure plus stricte et prédéfinie, ce qui en fait un choix idéal pour les pipelines de livraison continue plus simples. Scripted impose très peu de limites, dans la mesure où les seules limites en matière de structure et de syntaxe sont généralement définies par Groovy lui-même, plutôt que par des systèmes spécifiques à Pipeline, ce qui en fait un choix idéal pour les utilisateurs expérimentés et ceux qui ont des besoins plus complexes. Comme son nom l'indique, Pipeline déclaratif encourage un modèle de programmation déclaratif. [^2] Pipeline scripté suit quant à lui un modèle de programmation plus impératif. [^3]

[^1]:[ Langage spécifique à un domaine](https://en.wikipedia.org/wiki/Domain-specific_language).
[^2]:[Programmation déclarative](https://en.wikipedia.org/wiki/Declarative_programming).
[^3]:[Programmation impérative](https://en.wikipedia.org/wiki/Imperative_programming.)

