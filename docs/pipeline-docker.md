# Utilisation de Docker avec Pipeline

De nombreuses organisations utilisent [Docker](https://www.docker.com/) pour unifier à travers les machines leurs environnements de construction et de test et fournir un mécanisme efficace pour le déploiement d'applications. En commençant par les versions de Pipeline 2.5 et plus, Pipeline a un support intégré pour interagir avec Docker à partir d'un `Jenkinsfile`.

Bien que cette page couvre les bases de l'utilisation de Docker à partir d'un `Jenkinsfile`, il ne couvrira pas les principes fondamentaux de Docker, auxquels vous pouvez vous référer dans le [Docker Getting Started Guide](https://docs.docker.com/get-started/).

## Personnalisation de l'environnement d'exécution

Pipeline est conçu pour utiliser facilement les images [Docker](https://docs.docker.com/) comme environnement d'exécution pour une seule [étape](./glossaire.md#stage) ou l'ensemble du Pipeline. Cela signifie qu'un utilisateur peut définir les outils requis pour son Pipeline, sans avoir à configurer manuellement les agents. Tout outil qui peut être [emballé dans un conteneur Docker](https://hub.docker.com/) peut être utilisé avec facilité, en effectuant uniquement des modifications mineures à un `Jenkinsfile`.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent {
        docker { image 'node:22.19.0-alpine3.22' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --eval "console.log(process.platform,process.env.CI)"'
            }
        }
    }
}
```

??? "Activer/désactiver le Pipeline Scripté (_Avancé_)" 

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    node {
        /* Nécessite l'installation du plugin Docker Pipeline.  */
        docker.image('node:22.19.0-alpine3.22').inside {
            stage('Test') {
                sh 'node --eval "console.log(process.platform,process.env.CI)"'
            }
        }
    }
    ```

Lorsque le Pipeline s'exécute, Jenkins démarrera automatiquement le conteneur spécifié et exécutera les étapes définies à l'intérieur :

``` console
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] sh
[guided-tour] Running shell script
+ node --eval 'console.log(process.platform,process.env.CI)'
linux true
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
```

### Arguments supplémentaires

Des arguments supplémentaires, tels que le `registryUrl`, sont décrits dans la documentation de syntaxe des paramètres de l'agent.

### Synchronisation de l'espace de travail

S'il est important de maintenir l'espace de travail synchronisé avec d'autres étapes, utilisez `reuseNode true`. Sinon, une étape dockérisée peut être exécutée sur le même agent ou tout autre agent, mais dans un espace de travail temporaire.

Par défaut, pour une étape conteneurisée, Jenkins : 

* Choisit un agent ;
* Crée un nouvel espace de travail vide ;
* Clone du code du Pipeline à l'intérieur ;
* Monte ce nouvel espace de travail dans le conteneur.

Si vous avez plusieurs agents Jenkins, votre étape conteneurisée peut être démarrée sur l'une d'entre elles.

Lorsque `reuseNode` est défini sur `true`, aucun nouvel espace de travail ne sera créé et l'espace de travail actuel de l'agent en cours sera monté dans le conteneur. Après cela, le conteneur sera démarré sur le même nœud, de sorte que toutes les données seront synchronisées.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'gradle:8.14.0-jdk21-alpine'
                    // Exécutez le conteneur sur le nœud spécifié au
                    // niveau supérieur du Pipeline, dans le même espace 
                    // de travail plutôt que sur un tout nouveau nœud :
                    reuseNode true
                }
            }
            steps {
                sh 'gradle -g gradle-user-home --version'
            }
        }
    }
}
```

??? "Activer/désactiver le Pipeline Scripté (_Avancé_)" 

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    // L'option "reuseNode true" n'est actuellement pas prise en charge dans 
    // le pipeline scripté.
    ```

### Les données de mise en cache pour les conteneurs

De nombreux outils de construction téléchargeront les dépendances externes et les mettront en cache localement pour une réutilisation future. Étant donné que les conteneurs sont initialement créés avec des systèmes de fichiers "propres", cela peut entraîner des Pipelines plus lents, car ils peuvent ne pas profiter des caches sur les disques entre les exécutions de Pipeline ultérieures.

Pipeline prend en charge l'ajout d'arguments personnalisés qui sont passés à Docker, permettant aux utilisateurs de spécifier des [volumes Docker](https://docs.docker.com/engine/tutorials/dockervolumes/) personnalisés à monter, qui peuvent être utilisés pour la mise en cache de données sur [l'agent](./glossaire.md#agent) entre les exécutions du Pipeline. L'exemple suivant mettra en cache `~/.m2` entre les exécutions du Pipeline en utilisant le [conteneur Maven](https://hub.docker.com/_/maven/), en évitant la nécessité de redémarrer les dépendances pour les exécutions de Pipeline ultérieures.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent {
        docker {
            image 'maven:3.9.9-eclipse-temurin-21'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B'
            }
        }
    }
}
```

??? "Activer/désactiver le Pipeline Scripté (_Avancé_)" 

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    node {
        /* Nécessite l'installation du plugin Docker Pipeline. */
        docker.image('maven:3.9.11-eclipse-temurin-21-alpine').inside('-v $HOME/.m2:/root/.m2') {
            stage('Build') {
                sh 'mvn -B'
            }
        }
    }
    ```

### En utilisant plusieurs conteneurs

Il est devenu de plus en plus courant que les bases de code comptent sur plusieurs technologies différentes. Par exemple, un référentiel pourrait avoir à la fois une implémentation API back-end basée sur Java et une implémentation frontale basée sur JavaScript. La combinaison de Docker et du pipeline permet à un `JenkinsFile` d'utiliser plusieurs types de technologies, en combinant la directive `agent {}` avec différentes étapes.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent none
    stages {
        stage('Back-end') {
            agent {
                docker { image 'maven:3.9.11-eclipse-temurin-21-alpine' }
            }
            steps {
                sh 'mvn --version'
            }
        }
        stage('Front-end') {
            agent {
                docker { image 'node:22.19.0-alpine3.22' }
            }
            steps {
                sh 'node --version'
            }
        }
    }
}
```

??? "Activer/désactiver le Pipeline Scripté (_Avancé_)" 

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    node {
        /* Nécessite l'installation du plugin Docker Pipeline.  */

        stage('Back-end') {
            docker.image('maven:3.9.11-eclipse-temurin-21-alpine').inside {
                sh 'mvn --version'
            }
        }

        stage('Front-end') {
            docker.image('node:22.19.0-alpine3.22').inside {
                sh 'node --version'
            }
        }
    }
    ```

### Utilisation d'un Dockerfile

Pour les projets nécessitant un environnement d'exécution plus personnalisé, Pipeline prend également en charge la construction et l'exécution d'un conteneur à partir d'un `Dockerfile` dans le référentiel source. Contrairement à [l'approche précédente](./pipeline-docker.md#personnalisation-de-lenvironnement-dexécution) de l'utilisation d'un conteneur "Off-the-Shelf", l'utilisation de la syntaxe ` agent { dockerfile true }` construit une nouvelle image à partir d'un `Dockerfile`, plutôt que d'en tirer un de [Docker Hub](https://hub.docker.com/).

Réutilisons un exemple ci-dessus, avec un fichier `Dockerfile` plus personnalisé :

``` console title="<i>Dockerfile</i>"
FROM node:22.19.0-alpine3.22

RUN apk add -U subversion
```

En engageant cela à la racine du référentiel source, le `Jenkinsfile` peut être modifié pour construire un conteneur en fonction de ce `Dockerfile`, puis exécuter les étapes définies à l'aide de ce conteneur :

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent { dockerfile true }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
                sh 'svn --version'
            }
        }
    }
}
```

La syntaxe `agent { dockerfile true }` prend en charge un certain nombre d'autres options, qui sont décrites plus en détail dans la section [Syntaxe des Pipelines](./pipeline-syntaxe.md#agent).

_Utilisation d'un Dockerfile avec Pipeline Jenkins_
<iframe width="852" height="480" src="https://www.youtube.com/embed/Pi2kJ2RJS50" title="Jenkins Minute - Using a Dockerfile with Jenkins Pipeline/Utilisation d'un fichier Dockerfile avec Jenkins Pipeline" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Spécification d'une étiquette Docker

Par défaut, Pipeline suppose que tout [agent](./glossaire.md#agent) configuré est capable d'exécuter des Pipelines basés sur Docker. Pour les environnements Jenkins qui disposent d'agents macOS, Windows ou autres incapables d'exécuter le démon Docker, ce paramètre par défaut peut poser problème. Pipeline fournit une option globale sur la page **_Manage Jenkins_** (Gérer Jenkins) et au niveau du [dossier](./glossaire.md#dossier), permettant de spécifier quels agents (par [étiquette](./glossaire.md#label)) utiliser pour exécuter des Pipelines basés sur Docker. Pour activer cette option pour les étiquettes Docker, le plugin **Docker Pipeline** doit être installé.

![Accédez au tableau de bord, puis à « Manage Jenkins » (Gérer Jenkins) et enfin à « System » (Système). Dans la section « Declarative Pipeline (Docker) » (Pipeline Déclaratif (Docker)), définissez l'étiquette Docker, l'URL du registre Docker et les informations d'identification du registre.](https://www.jenkins.io/doc/book/resources/pipeline/configure-docker-label.png)

### Configuration du chemin d'accès pour les utilisateurs de macOS

Le répertoire `/usr/local/bin` n'est pas inclus par défaut dans le `PATH` macOS pour les images Docker. Si des exécutables de `/usr/local/bin` doivent être appelés depuis Jenkins, le `PATH` doit être étendu pour inclure `/usr/local/bin`. Ajoutez un nœud de chemin d'accès dans le fichier « /usr/local/Cellar/jenkins-lts/XXX/homebrew.mxcl.jenkins-lts.plist » comme suit :

``` xml title="<i>Contenu de Homebrew.mxcl.jenkins-lts.plist</i>"
<key>EnvironmentVariables</key>
<dict>
<key>PATH</key>
<string><!-- insérer ici le chemin d'accès révisé --></string>
</dict>
```

La `string PATH` révisée doit être une liste de répertoires séparés par des deux-points, dans le même format que la variable d'environnement `PATH`, et doit inclure :

* `/usr/local/bin`
* `/usr/bin`
* `/bin`
* `/usr/sbin`
* `/sbin`
* `/Applications/Docker.app/Contents/Resources/bin/`
* `/Users/XXX/Library/Group\ Containers/group.com.docker/Applications/Docker.app/Contents/Resources/bin` (où `XXX` est remplacé par votre nom d'utilisateur)

Maintenant, redémarrez Jenkins à l'aide de la commande `brew services restart jenkins-lts`.

## Utilisation avancée avec Pipeline Scripté

### Exécution de conteneurs « sidecar »

L'utilisation de Docker dans Pipeline est un moyen efficace d'exécuter un service sur lequel la construction, ou un ensemble de tests, peut compter. Semblable au [modèle sidecar](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar), Docker Pipeline peut exécuter un conteneur "en arrière-plan", tout en effectuant des travaux dans un autre. En utilisant cette approche side-car, un Pipeline peut avoir un conteneur "propre" fourni pour chaque exécution de Pipeline.

Considérez une suite de tests d'intégration hypothétique qui s'appuie sur une base de données MySQL locale en cours d'exécution. En utilisant la méthode `withRun`, implémentée dans la prise en charge du plugin [Docker Pipeline](https://plugins.jenkins.io/docker-workflow) pour le Pipeline Scripté, un `Jenkinsfile` peut exécuter MySQL en tant que side-car :

``` groovy
node {
    checkout scm
    /*
     * Afin de communiquer avec le serveur MySQL, ce Pipeline mappe
     * explicitement le port (`3306`) à un port connu sur la machine hôte.
     */
    docker.image('mysql:8-oracle').withRun('-e "MYSQL_ROOT_PASSWORD=my-secret-pw"' +
                                           ' -p 3306:3306') { c ->
        /* Attend que le service mysql soit opérationnel. */
        sh 'while ! mysqladmin ping -h0.0.0.0 --silent; do sleep 1; done'
        /* Exécute quelques tests qui nécessitent MySQL. */
        sh 'make check'
    }
}
```

Cet exemple peut être poussé plus loin en utilisant deux conteneurs simultanément. Un « sidecar » exécutant MySQL et un autre fournissant [l'environnement d'exécution](./pipeline-docker.md#personnalisation-de-lenvironnement-dexécution) à l'aide des [liens de conteneurs](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/) Docker.

``` groovy
node {
    checkout scm
    docker.image('mysql:8-oracle').withRun('-e "MYSQL_ROOT_PASSWORD=my-secret-pw"') { c ->
        docker.image('mysql:8-oracle').inside("--link ${c.id}:db") {
            /* Attend que le service mysql soit opérationnel.*/
            sh 'while ! mysqladmin ping -hdb --silent; do sleep 1; done'
        }
        docker.image('oraclelinux:9').inside("--link ${c.id}:db") {
            /*
             * Exécute quelques tests qui nécessitent MySQL en supposant
             * qu'il est disponible sur le nom d'hôte `db`.
             */
            sh 'make check'
        }
    }
}
```

L'exemple ci-dessus utilise l'objet exposé par `withRun`, qui a l'ID du conteneur en cours d'exécution disponible via la propriété `id`. En utilisant l'ID du conteneur, le Pipeline peut créer un lien en passant des arguments Docker personnalisés à la méthode `inside ()`.

La propriété `id` peut également être utile pour inspecter les journaux à partir d'un conteneur Docker en cours d'exécution avant la sortie du Pipeline:

``` groovy
sh "docker logs ${c.id}"
```

### Conteneurs de construction

Afin de créer une image Docker, le plugin [Docker Pipeline](https://plugins.jenkins.io/docker-workflow) fournit également une méthode `build()` permettant de créer une nouvelle image à partir d'un fichier `Dockerfile` dans le référentiel pendant l'exécution d'un Pipeline.

Un avantage majeur de l'utilisation de la syntaxe `docker.build("my-image-name")` est qu'un Pipeline Scripté peut utiliser la valeur de retour pour les appels de Pipeline Docker ultérieurs, par exemple :

``` groovy
node {
    checkout scm
    def customImage = docker.build("my-image:${env.BUILD_ID}")
    customImage.inside {
        sh 'make test'
    }
}
```

La valeur de retour peut également être utilisée pour publier l'image docker sur [Docker Hub](https://hub.docker.com/) ou un [registre personnalisé](#registre-personnalise), via la méthode `push()`, par exemple :

``` groovy
node {
    checkout scm
    def customImage = docker.build("my-image:${env.BUILD_ID}")
    customImage.push()
}
```

Une utilisation courante des "balises" d'image consiste à spécifier une dernière balise pour la dernière version validée d'une image Docker. La méthode `push()` accepte un paramètre `tag` facultatif, permettant au Pipeline de pousser `customImage` avec différentes balises, par exemple :

``` groovy
node {
    checkout scm
    def customImage = docker.build("my-image:${env.BUILD_ID}")
    customImage.push()

    customImage.push('latest')
}
```

La méthode `build()` construit le fichier `Dockerfile` dans le répertoire actuel par défaut. Cela peut être remplacé en fournissant un chemin d'accès au répertoire contenant un fichier `Dockerfile` comme deuxième argument de la méthode `build()`, par exemple :

``` groovy
node {
    checkout scm
    def testImage = docker.build("test-image", "./dockerfiles/test") //(1)

    testImage.inside {
        sh 'make test'
    }
}
```

1. Crée `test-image` à partir du fichier Dockerfile situé dans `./dockerfiles/test/Dockerfile`.

Il est possible de transmettre d'autres arguments à [docker build](https://docs.docker.com/engine/reference/commandline/build/) en les ajoutant au deuxième argument de la méthode `build()`. Lorsque vous transmettez des arguments de cette manière, la dernière valeur de la chaîne doit être le chemin d'accès au fichier Docker et doit se terminer par le dossier à utiliser comme contexte de compilation.

Cet exemple remplace le fichier `Dockerfile` par défaut en passant le drapeau `-f` :

``` groovy
node {
    checkout scm
    def dockerfile = 'Dockerfile.test'
    def customImage = docker.build("my-image:${env.BUILD_ID}",
                                   "-f ${dockerfile} ./dockerfiles") //(1)
}
```

1. Compile `my-image:${env.BUILD_ID}` à partir du fichier Dockerfile situé dans `./dockerfiles/Dockerfile.test`.

### Utilisation d'un serveur Docker distant

Par défaut, le plugin [Docker Pipeline](https://plugins.jenkins.io/docker-workflow) communiquera avec un démon Docker local, généralement accessible via `/var/run/docker.sock`.

Pour sélectionner un serveur Docker qui ne soit pas par défaut, tel que [Docker Swarm](https://docs.docker.com/swarm/), utilisez la méthode `withServer()`.

Vous pouvez passer un URI, et éventuellement l'ID d'un **Certificat Docker Server Authentication** préconfiguré dans Jenkins, à la méthode avec :

``` groovy
node {
    checkout scm

    docker.withServer('tcp://swarm.example.com:2376', 'swarm-certs') {
        docker.image('mysql:8-oracle').withRun('-p 3306:3306') {
            /* do things */
        }
    }
}
```

!!! danger
    `inside()` et `build()` ne fonctionnera pas correctement avec un serveur Swarm Docker prêt à l'emploi.

    Pour que `inside()` fonctionne, le serveur Docker et l'agent Jenkins doivent utiliser le même système de fichiers, afin que l'espace de travail puisse être monté.

    Actuellement, ni le plugin Jenkins ni l'interface CLI Docker ne détectent automatiquement le cas où le serveur fonctionne à distance. Un symptôme typique de ce problème serait des erreurs provenant de commandes `sh` imbriquées, telles que :

    ``` console
    cannot create /…@tmp/durable-…/pid: Directory nonexistent
    ```

    Lorsque Jenkins détecte que l'agent s'exécute lui-même dans un conteneur Docker, il transmet automatiquement l'argument `--volumes-from` au conteneur `inside`, garantissant ainsi qu'il peut partager un espace de travail avec l'agent.

    De plus, certaines versions de Docker Swarm ne prennent pas en charge les registres personnalisés.

### Utilisation d'un registre personnalisé

Par défaut, le plugin [Docker Pipeline](https://plugins.jenkins.io/docker-workflow) utilise le registre [Docker Hub](https://hub.docker.com/) comme registre Docker par défaut.

Pour utiliser un registre Docker personnalisé, les utilisateurs de Pipeline Scripté peuvent encapsuler les étapes à l'aide de la méthode `withRegistry()`, en transmettant l'URL du registre personnalisé, par exemple : 

``` groovy
node {
    checkout scm

    docker.withRegistry('https://registry.example.com') {

        docker.image('my-custom-image').inside {
            sh 'make test'
        }
    }
}
```

Pour un registre Docker nécessitant une authentification, ajoutez un élément d'identification "nom d'utilisateur/mot de passe" de la page d'accueil de Jenkins et utilisez l'ID d'identification comme deuxième argument à `withRegistry()` :

``` groovy
node {
    checkout scm

    docker.withRegistry('https://registry.example.com', 'credentials-id') {

        def customImage = docker.build("my-image:${env.BUILD_ID}")

        /* Push the container to the custom Registry */
        customImage.push()
    }
}
```
