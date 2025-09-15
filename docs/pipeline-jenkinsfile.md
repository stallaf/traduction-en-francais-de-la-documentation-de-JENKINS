# Utilisation d'un fichier Jenkinsfile

Cette section s'appuie sur les informations fournies dans la section « Commencer avec Pipeline » et présente d'autres étapes utiles, des modèles courants et des exemples concrets de fichiers `Jenkinsfile`.

La création d'un `Jenkinsfile`, qui est vérifiée dans le contrôle de la source ^[1], offre un certain nombre d'avantages immédiats : 

* Examen/itération de code sur le Pipeline  ;
* Piste d'audit pour le Pipeline ;
* Source unique de vérité ^[2] pour le pipeline, qui peut être consultée et modifiée par plusieurs membres du projet.

[^1] : [en.wikipedia.org/wiki/Source_control_management](https://en.wikipedia.org/wiki/Source_control_management) <br>
[^2] : [en.wikipedia.org/wiki/Single_Source_of_Truth](https://en.wikipedia.org/wiki/Single_Source_of_Truth)

Pipeline prend en charge [deux syntaxes](./pipeline-syntaxe.md), déclarative (introduite dans Pipeline 2.5) et Pipeline scripté. Qui soutiennent tous deux la construction de Pipelines de livraison continue. Les deux peuvent être utilisés pour définir un Pipeline dans l'interface utilisateur Web ou avec un `Jenkinsfile`, bien qu'il soit généralement considéré comme une meilleure pratique pour créer un `Jenkinsfile` et vérifier le fichier dans le référentiel de contrôle de source.

## Créer un Jenkinsfile

Comme indiqué dans la section [Définition d'un Pipeline dans SCM](./pipeline-commencer.md#dans-scm), un fichier `Jenkinsfile` est un fichier texte qui contient la définition d'un Pipeline Jenkins et qui est enregistré dans le contrôleur de source. Prenons l'exemple du Pipeline suivant, qui implémente une livraison continue en trois étapes :

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```

??? "Activer/désactiver le Pipeline scripté (_Avancé_)"

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    node {
        stage('Build') {
            echo 'Building....'
        }
        stage('Test') {
            echo 'Testing....'
        }
        stage('Deploy') {
            echo 'Deploying....'
        }
    }
    ```

Tous les Pipelines n'auront pas ces mêmes trois étapes, mais c'est un bon point de départ pour les définir sur la plupart des projets. Les sections ci-dessous démontreront la création et l'exécution d'un Pipeline simple dans une installation de test de Jenkins.

!!! note
    On suppose qu'il existe déjà un référentiel de contrôle de source configuré pour le projet et qu'un Pipeline a été défini dans Jenkins après [ces instructions](./pipeline-commencer.md#dans-scm).

En utilisant un éditeur de texte, idéalement celui qui prend en charge la mise en évidence de la syntaxe [Groovy](http://groovy-lang.org/), créez un nouveau `Jenkinsfile` dans le répertoire racine du projet.

L'exemple de Pipeline déclaratif ci-dessus contient la structure minimale nécessaire pour mettre en œuvre un Pipeline de livraison continue. La [directive agent](./pipeline-syntaxe.md#agent), qui est obligatoire, demande à Jenkins d'allouer un exécuteur et un espace de travail au Pipeline. Sans directive `agent`, non seulement le Pipeline déclaratif n'est pas valide, mais il ne serait pas en mesure d'effectuer le moindre travail ! Par défaut, la directive `agent` garantit que le référentiel source est extrait et mis à disposition pour les étapes des phases suivantes.

Les [directives d'étapes](./pipeline-syntaxe.md#etapes) et [de phases](./pipeline-syntaxe.md#phases) sont également nécessaires pour un Pipeline déclaratif valide car elles indiquent à Jenkins ce qu'il doit exécuter et à quelle étape cela doit être exécuté.

Pour une utilisation plus avancée de Pipeline scripté, l'exemple `node` ci-dessus est une première étape cruciale, car il alloue un exécuteur et un espace de travail pour le Pipeline. En substance, sans `node`, un Pipeline ne peut effectuer aucune tâche ! À partir de `node`, la première chose à faire sera de vérifier le code source de ce projet. Étant donné que le fichier `Jenkinsfile` est directement extrait du contrôle de source, Pipeline offre un moyen rapide et facile d'accéder à la bonne révision du code source.

``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
node {
    checkout scm // (1) 
    /* .. snip .. */
}
```

1. L'étape `checkout` vérifie le code à partir du contrôleur de source ; `scm` est une variable spéciale qui indique à l'étape `checkout` de cloner la révision spécifique qui a déclenché l'exécution de ce Pipeline.

### Construction

Pour de nombreux projets, le début du "travail" dans le Pipeline serait la phase "_build_" (construction). En général, cette phase du Pipeline consiste à assembler, compiler ou packager le code source. Le fichier `Jenkinsfile` ne remplace pas les outils de construction existants tels que GNU/Make, Maven, Gradle ou autres, mais peut plutôt être considéré comme une couche de liaison qui relie les différentes phases du cycle de vie du développement d'un projet (construction, test, déploiement).

Jenkins dispose d'un certain nombre de plugins permettant d'invoquer pratiquement tous les outils de compilation couramment utilisés, mais cet exemple se contentera d'invoquer `make` à partir d'une étape shell (`sh`). L'étape `sh` suppose que le système est basé sur Unix/Linux. Pour les systèmes basés sur Windows, il est possible d'utiliser `bat` à la place.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make' // (1)
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true // (2)
            }
        }
    }
}
```

1. L'étape `sh` invoque la commande `make` et ne continuera que si un code de sortie zéro est renvoyé par la commande. Tout code de sortie non nul échouera au pipeline.
2. `archiveArtifacts` capture les fichiers créés correspondant au modèle d'inclusion (`**/target/*.jar`) et les enregistre dans le contrôleur Jenkins pour une récupération ultérieure.

??? "Activer/désactiver le Pipeline scripté (_Avancé_)"

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    node {
        stage('Build') {
            sh 'make' // (1)
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true // (2)
        }
    }
    ```

(1) L'étape `sh` invoque la commande `make` et ne continuera que si un code de sortie zéro est renvoyé par la commande. Tout code de sortie non nul échouera au pipeline.

(2)  `archiveArtifacts` capture les fichiers créés correspondant au modèle d'inclusion (`**/target/*.jar`) et les enregistre dans le contrôleur Jenkins pour une récupération ultérieure.

!!! tip "Astuce"
    L'archivage des artefacts ne remplace pas l'utilisation de référentiels d'artefacts externes tels qu'Artifactory ou Nexus et ne doit être envisagé que pour la création de rapports de base et l'archivage de fichiers.

### Test

L'exécution de tests automatisés est un élément crucial de tout processus de livraison continu réussi. En tant que tel, Jenkins possède un certain nombre d'installations d'enregistrement, de rapport et de visualisation des tests fournies par un certain [nombre de plugins](https://plugins.jenkins.io/?labels=report). À un niveau fondamental, lorsqu'il y a des échecs de test, il est utile que Jenkins enregistre les échecs de reporting et de visualisation dans l'interface utilisateur Web. L'exemple ci-dessous utilise l'étape `junit`, fournie par le [plugin JUnit](https://plugins.jenkins.io/junit).

Dans l'exemple ci-dessous, si les tests échouent, le Pipeline est marqué "instable", comme indiqué par une balle jaune dans l'interface utilisateur Web. Sur la base des rapports de test enregistrés, Jenkins peut également fournir une analyse et une visualisation des tendances historiques.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                /* `make check` renvoie une valeur différente de zéro en cas
                * d'échec des tests, en utilisant `true` pour permettre au 
                * pipeline de continuer malgré tout.
                */
                sh 'make check || true' //(1)
                junit '**/target/*.xml' //(2)
            }
        }
    }
}
```

1. L'utilisation d'un shell en ligne conditionnel (`sh 'make chèque || true'`) garantit que l'étape `sh` voit toujours un code de sortie zéro, donnant à la étape `junit` la possibilité de capturer et de traiter les rapports de test. D'autres approches alternatives sont décrites plus en détail dans la section [Gestion des échecs](#gestion-des-echecs) ci-dessous.
2.  `junit` capture et associe les fichiers XML JUnit correspondant au modèle d'inclusion (`** / cible / *. xml`).

??? "Activer/désactiver le pipeline scripté (_Avancé_)"

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    node {
        /* .. snip .. */
        stage('Test') {
            /* `make check` renvoie une valeur différente de zéro en cas 
            *  d'échec des tests, l'utilisation de `true` permet néanmoins
            *  au pipeline de continuer
            */
            sh 'make check || true' // (1)
            junit '**/target/*.xml' // (2)
        }
        /* .. snip .. */
    }
    ```

(1) L'utilisation d'un shell en ligne conditionnel (`sh 'make chèque || true'`) garantit que l'étape `sh` voit toujours un code de sortie zéro, donnant à la étape `junit` la possibilité de capturer et de traiter les rapports de test. D'autres approches alternatives sont décrites plus en détail dans la section [Gestion des échecs](#gestion-des-echecs) ci-dessous.

(2) `junit` capture et associe les fichiers XML JUnit correspondant au modèle d'inclusion (`** / cible / *. xml`).

### Déploiement

Le déploiement peut impliquer diverses étapes, selon les exigences du projet ou de l'organisation, et peut aller de la publication d'artefacts construits sur un serveur Artifactory à la transmission de code vers un système de production.

À ce stade de l'exemple de pipeline, les étapes « Build » (Construire) et « Test » (Tester) ont été exécutées avec succès. En substance, l'étape « Deploy » (Déployer) ne sera exécutée que si les étapes précédentes ont été menées à bien, sinon le Pipeline aurait pris fin prématurément.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any

    stages {
        stage('Deploy') {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS' // (1)
              }
            }
            steps {
                sh 'make publish'
            }
        }
    }
}
```

1. L'accès à la variable `currentBuild.result` permet au Pipeline de déterminer s'il y a eu des échecs de test. Dans ce cas, la valeur serait `UNSTABLE`.

??? "Activer/désactiver le Pipeline scripté (_Avancé_)"

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    node {
        /* .. snip .. */
        stage('Deploy') {
            if (currentBuild.result == null || currentBuild.result == 'SUCCESS') { //(1)
                sh 'make publish'
            }
        }
        /* .. snip .. */
    }
    ```

En supposant que tout s'est déroulé correctement dans l'exemple de Pipeline Jenkins, chaque exécution réussie du Pipeline sera associée à l'archivage des artefacts de build, à la génération de rapports sur les résultats des tests et à l'affichage de la sortie complète de la console dans Jenkins.

Un Pipeline scripté peut inclure des tests conditionnels (illustrés ci-dessus), des boucles, des blocs _try/catch/finally_ et même des fonctions. La section suivante traitera plus en détail de cette syntaxe avancée du Pipeline scripté.

## Travailler avec votre Jenkinsfile

Les sections suivantes fournissent des détails sur la gestion :

* de la syntaxe spécifique du Pipeline dans votre fichier `Jenkinsfile` et
* des caractéristiques et fonctionnalités de la syntaxe du Pipeline qui sont essentielles à la création de votre application ou de votre projet Ppipeline.

### Utilisation de variables d'environnement

Jenkins Pipeline expose les variables d'environnement via la variable globale `env`, qui est disponible depuis n'importe quel emplacement dans un fichier `Jenkinsfile`. La liste complète des variables d'environnement accessibles depuis Jenkins Pipeline est documentée à l'adresse ${YOUR_JENKINS_URL}/pipeline-syntax/globals#env et comprend :

**BUILD_ID**<br>
L'ID de construction actuel, identique à BUILD_NUMBER pour les builds créés dans les versions Jenkins 1.597+.

**BUILD_NUMBER** <br>
Le numéro de construction actuel, tel que "153".

**BUILD_TAG**<br>
Chaîne de Jenkins-${JOB_NAME}-${BUILD_NUMBER}. Pratique à insérer dans un fichier de ressources, un fichier jar, etc. pour faciliter l'identification.

**BUILD_URL**<br>
L'URL où les résultats de cette construction peuvent être trouvés (par exemple, http: // buildServer / jenkins / job / myjobname / 17 /).

**EXECUOTR_NUMBER**<br>
Le numéro unique qui identifie l'exécuteur actuel (parmi les exécuteurs de la même machine) effectuant cette compilation. Il s'agit du numéro que vous voyez dans « _build executor status_ » (état de l'exécuteur de compilation), sauf que le numéro commence à 0 et non à 1.

**JAVA_HOME**<br>
Si votre travail est configuré pour utiliser un JDK spécifique, cette variable est définie sur le JAVA_HOME du JDK spécifié. Lorsque cette variable est définie, le chemin est également mis à jour pour inclure le sous-répertoire bin de JAVA_HOME.

**JENKINS_URL**<br>
URL complète de Jenkins, telle que https://example.com:port/jenkins/ (REMARQUE : disponible uniquement si l'URL Jenkins est définie dans « Configuration du système »).

**JOB_NAME**<br>
Nom du projet de cette construction, tel que "foo" ou "foo / bar".

**NODE_NAME**<br> 

Le nom du nœud sur lequel la construction actuelle fonctionne. Réglé sur «Master» pour le contrôleur Jenkins.

**WORKSPACE**<br>
Le chemin absolu de l'espace de travail.

La référence ou l'utilisation de ces variables d'environnement peut s'effectuer comme pour accéder à n'importe quelle clé dans une [carte](http://groovy-lang.org/syntax.html#_maps) Groovy, par exemple :

``` groovy title="<i>Jenkinsfile (Pipeline déclaratif)</i>"
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
            }
        }
    }
}
```

??? "Activer/désactiver le Pipeline Scripté (_Avancé_)"
    
    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    node {
        echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
    }
    ```

### Définir les variables d'environnement

La définition d'une variable d'environnement dans un Pipeline Jenkins est réalisée différemment selon que le Pipeline déclaratif ou Scripté est utilisé.

Le Pipeline Déclaratif prend en charge une directive [d'environnement](./pipeline-syntaxe.md#environnement), tandis que les utilisateurs du Pipeline Scripté doivent utiliser l'étape `withEnv`.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any
    environment { //(1)
        CC = 'clang'
    }
    stages {
        stage('Example') {
            environment { //(2)
                DEBUG_FLAGS = '-g'
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```

1. Une directive `environment` utilisée dans le bloc `pipeline` de niveau supérieur s'appliquera à toutes les étapes du Pipeline.
2. Une directive `environment` définie dans une `stage` n'appliquera les variables d'environnement données qu'aux paluiers de cette `stage`.

??? "Activer/désactiver le Pipeline Scripté (_Avancé_)"
    
    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    node {
        /* .. snip .. */
        withEnv(["PATH+MAVEN=${tool 'M3'}/bin"]) {
            sh 'mvn -B verify'
        }
    }
    ```

#### Définition dynamique des variables d'environnement

Les variables d'environnement peuvent être définies au moment de l'exécution et peuvent être utilisées par des scripts shell (`sh`), des scripts batch Windows (`bat`) et PowerShell (`powershell`). Chaque script peut renvoyer soit `returnStatus`, soit `returnStdout`. [Plus d'informations sur les scripts](./pipeline-etapes-etapes-du-flux-de-travail.md).

Vous trouverez ci-dessous un exemple dans un Pipeline déclaratif utilisant `sh` (shell) avec `returnStatus` et `returnStdout`.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any //(1)
    environment {
        //  Utilisation de returnStdout
        CC = """${sh(
                returnStdout: true,
                script: 'echo "clang"'
            )}""" //(2)
        //  Utilisation de returnStatus
        EXIT_STATUS = """${sh(
                returnStatus: true,
                script: 'exit 1'
            )}"""
    }
    stages {
        stage('Example') {
            environment {
                DEBUG_FLAGS = '-g'
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```

1. Un `agent` doit être défini au niveau supérieur du Pipeline. Cela échouera si l'agent est défini comme `agent none`.
2. Lors de l'utilisation de `retourStdout`, , un espace blanc sera ajouté à la fin de la chaîne renvoyée. Utilisez `.trim()` pour le supprimer.

### Gestion des informations d'identification

Les informations [d'identification configurées dans Jenkins](./utilisation-identifiants.md#garantie-didentification) peuvent être gérées dans des Pipelines pour une utilisation immédiate. En savoir plus sur l'utilisation des informations d'identification dans Jenkins sur la page [Utilisation des informations d'identification](./utilisation-identifiants.md).

_La bonne façon de gérer les informations d'identification dans Jenkins._
<iframe width="800" height="420" src="https://www.youtube.com/embed/yfjtMIDgmfs" title="The Correct Way to Handle Credentials in a Jenkins Pipeline" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

#### Pour le texte secret, les noms d'utilisateur, les mots de passe et les fichiers secrets

La syntaxe de Pipeline déclaratif de Jenkins dispose de la méthode d'aide `credentials()` (utilisée dans la directive [environment](./pipeline-syntaxe.md#environnement)) qui prend en charge les informations d'identification sous forme de [texte secret](#texte-secret), de [nom d'utilisateur et de mot de passe](./pipeline-jenkinsfile.md#noms-dutilisateur-les-mots-de-passe), ainsi que sous forme de [fichier secret](./pipeline-jenkinsfile.md#fichiers-secrets). Si vous souhaitez gérer d'autres types d'informations d'identification, reportez-vous à la section [Pour les autres types d'informations d'identification](./pipeline-jenkinsfile.md#autres-types-didentification).

##### Texte secret

Le code de Pipeline suivant montre un exemple de la façon de créer un Pipeline en utilisant des variables d'environnement pour les informations d'identification de texte secret.

Dans cet exemple, deux informations d'identification de texte secret sont affectées à des variables d'environnement séparées pour accéder à Amazon Web Services (AWS). Ces informations d'identification auraient été configurées dans Jenkins avec leurs ID d'identification respectifs `jenkins-aws-secret-key-id ` et J`jenkins-aws-secret-access-key`.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent {
        // Définissez ici les détails de l'agent.
    }
    environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
    }
    stages {
        stage('Example stage 1') {
            steps {
                // //(1)
            }
        }
        stage('Example stage 2') {
            steps {
                // //(2)
            }
        }
    }
}
```

1. Vous pouvez référencer les deux variables d'environnement d'identification (définies dans la directive [environnement](./pipeline-syntaxe.md#environnement) de ce Pipeline), dans les paliers de cette étape en utilisant la syntaxe `$AWS_ACCESS_KEY_ID` et `$ AWS_SECRET_ACCESS_KEY`. Par exemple, ici, vous pouvez vous authentifier à AWS en utilisant les informations d'identification de texte secret attribuées à ces variables d'identification. Pour maintenir la sécurité et l'anonymat de ces informations d'identification, si le travail affiche la valeur de ces variables d'identification à l'intérieur du Pipeline (telles que `echo $ AWS_SECRET_ACCESS_KEY`), Jenkins ne renvoie que la valeur «\*\*\*\*» afin de réduire le risque de divulgation d'informations confidentielles dans la sortie de la console et dans les journaux. Toutes les informations sensibles contenues dans les identifiants d'authentification (telles que les noms d'utilisateur) sont également renvoyées sous la forme « **** » dans la sortie de l'exécution du Pipeline. Cela ne fait que réduire le risque d'exposition accidentelle. Il n'empêche pas un utilisateur malveillant de capturer la valeur des informations d'identification par d'autres moyens. Un Ples ipeline qui utilise des informations d'identification peut également divulguer. Ne permettez pas aux travaux du Pipeline non fiables d'utiliser des informations d'identification de confiance. 

2. Dans cet exemple dePpipeline, les informations d'identification attribuées aux deux variables d'environnement `AWS_…`​ ont une portée globale pour l'ensemble du Pipeline, de sorte que ces variables d'identification peuvent également être utilisées dans les étapes de cette phase. Toutefois, si la directive `environnement` de ce Pipeline était déplacée vers une phase spécifique (comme c'est le cas dans l'exemple de Pipeline [Noms d'utilisateur et mots de passe](#nom-dutilisateur-et-mot-de-passe) ci-dessous), ces variables d'environnement `AWS_…`​ n'auraient alors qu'une portée limitée aux étapes de cette phase.

!!! Tip  "Astuce"
    Le stockage de clés AWS statiques dans les informations d'identification Jenkins n'est pas très sécurisé. Si vous pouvez exécuter Jenkins lui-même dans AWS (au moins l'agent), il est préférable d'utiliser des rôles IAM pour un [ordinateur](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) ou un [compte de service EKS](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html). Il est également possible d'utiliser la [fédération d'identités Web](https://github.com/jenkinsci/oidc-provider-plugin#accessing-aws). 

##### Noms d'utilisateur et mots de passe

Les extraits de code suivants montrent un exemple de la façon de créer un Pipeline à l'aide de variables d'environnement pour le nom d'utilisateur et les informations d'identification de mot de passe.

Dans cet exemple, les identifiants de nom d'utilisateur et de mot de passe sont attribués à des variables d'environnement pour accéder à un référentiel Bitbucket dans un compte ou une équipe communs à votre organisation ; ces identifiants auraient été configurés dans Jenkins avec l'ID d'identification `jenkins-bitbucket-common-creds`.

Lorsque vous définissez la variable [environnement](./pipeline-syntaxe.md#environnement) d'identification dans la directive d'environnement :

``` groovy 
environment {
    BITBUCKET_COMMON_CREDS = credentials('jenkins-bitbucket-common-creds')
}
```

Cela définit en fait les trois variables d'environnement suivantes : 

* `BITBUCKET_COMMON_CREDS ` - contient un nom d'utilisateur et un mot de passe séparé par une virgule dans le nom d'utilisateur du format : `username:password` ;
* `BITBUCKET_COMMON_CREDS_USR` - une variable supplémentaire contenant uniquement le composant du nom d'utilisateur ;
* `BITBUCKET_COMMON_CREDS_PSW` - une variable supplémentaire contenant le composant de mot de passe uniquement.

!!! tip "Astuce"
    Par convention, les noms de variables d'environnement sont généralement composées de lettres capitales, avec des mots individuels séparés par des traits de soulignements, vous pouvez cependant spécifier tout nom de variable légitime à l'aide de caractères en bas de casse. Gardez à l'esprit que les variables d'environnement supplémentaires créées par la méthode `credentials()` (ci-dessus) seront toujours annexées avec `_USR` et `_PSW` (c'est-à-dire dans le format d'un soulignement suivi de trois lettres majuscules).

L'extrait de code suivant montre l'exemple du Pipeline dans son intégralité :

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent {
        // Définissez ici les détails de l'agent.
    }
    stages {
        stage('Example stage 1') {
            environment {
                BITBUCKET_COMMON_CREDS = credentials('jenkins-bitbucket-common-creds')
            }
            steps {
                // 
            }
        }
        stage('Example stage 2') {
            steps {
                // 
            }
        }
    }
}
```

#### Autres types d'identification

Si vous devez définir des informations d'identification dans un Pipeline pour autre chose que du texte secret, des noms d'utilisateur et des mots de passe, ou des [fichiers secrets](#pour-le-texte-secret-les-noms-dutilisateur-les-mots-de-passe-et-les-fichiers-secrets) comme les clés SSH ou les certificats, utilisez la fonctionnalité de **générateur d'extraits** de Jenkins, auquel vous pouvez accéder via l'interface utilisateur classique de Jenkins.

Pour accéder au **générateur d'extraits** pour votre projet/article de Pipeline : 

1. Dans le tableau de bord Jenkins, sélectionnez le nom de votre projet/élément de pipeline. 
2. Dans le volet de navigation gauche, sélectionnez **_Pipeline Syntax_** (Syntaxe du Pipeline) et assurez-vous que l'option **_Snippet Generator_** (Générateur d'extraits) est disponible en haut du volet de navigation. 
3. Dans le **_Sample Step_** champ (Exemple d'étape), choisissez **_withCredentials: Bind credentials to variables._**  WithCredentials : (lie les informations d'identification aux variables.)
4. Sous **_Bindings_** (Liaisons), cliquez sur **_Add_** (Ajouter) et choisissez dans la liste déroulante : 
    * **_SSH User Private Key_** (Clé privée de l'utilisateur SSH) - pour gérer les [informations d'identification de la paire de clés publiques/privées](http://www.snailbook.com/protocols.html), à partir de laquelle vous pouvez spécifier : 
        * **_Key File Variable_** (Variable de fichier clé) - le nom de la variable d'environnement qui sera liée à ces informations d'identification. Jenkins attribue en fait cette variable temporaire à l'emplacement sécurisé du fichier de clé privée requis dans le processus d'authentification par paire de clés publique/privée SSH.
        * **_Passphrase Variable_** (_Optional_) (Variable de phrase de passe) (Facultative) - Le nom de la variable d'environnement qui sera lié à la phrase de passe associée à la paire de clés publiques/privées SSH. 
        * **_Username Variable_** (_Optional_) (Variable de nom d'utilisateur) (_Facultative_) - Le nom de la variable d'environnement qui sera lié au nom d'utilisateur associé à la paire de clés publiques/privées SSH. 
        * **_Credentials_** (Informations d'identification) - Choisissez les informations d'identification de clès publiques/privées SSH stockées dans Jenkins. La valeur de ce champ est l'ID d'identification, que Jenkins écrit à l'extrait généré. 
    * **_Certificate_** (Certificat) - Pour gérer les [certificats PKCS#12](https://tools.ietf.org/html/rfc7292), à partir desquels vous pouvez spécifier : 
        * **_Keystore Variable_** (_Optionnal_)  (Variable Keystore) (_Facultative_) - Le nom de la variable d'environnement qui sera lié à ces informations d'identification. Jenkins attribue en fait cette variable temporaire à l'emplacement sécurisé de la clé du certificat requise dans le processus d'authentification du certificat. 
        * **_Password Variable_** (_Optional_) (Variable de mot de passe) (_Facultatve_) - Le nom de la variable d'environnement qui sera lié au mot de passe associé au certificat. 
        * **_Alias Variable_** (_Optional_) (Variable d'alias) (_Facultative_) - Le nom de la variable d'environnement qui sera lié à l'alias unique associé au certificat. 
        * **_Credentials_** {Informations d'identification) - Choisissez les informations d'identification du certificat stockées dans Jenkins. La valeur de ce champ est l'ID d'identification, que Jenkins écrit à l'extrait généré. 
    * **_Docker client certificate_** (Certificat client Docker) - Pour gérer l'authentification du certificat Host Docker. 
5. Cliquez sur **_Generate Pipeline Script_** (Générer le Script Pipeline) et Jenkins génère un extrait de code de l'étape de Pipeline `withCredentials(…​) { …​ }` pour les informations d'identification que vous avez spécifiées, que vous pouvez ensuite copier et coller dans votre code Pipeline Déclaratif ou Scripté.
**Remarques :**
    * Les champs **_Credentials_** (Informations d'identification) (ci-dessus) affichent les noms des informations d'identification configurées dans Jenkins. Cependant, ces valeurs sont converties en identifiants d'informations après avoir cliqué sur **_Generate Pipeline Script_** (Générer le Script Pipeline).
    * Pour combiner plusieurs informations d'identification dans une seule étape de Pipeline `withCredentials(…​) { …​ }`, consultez la section [Combiner les informations d'identification en une seule étape](#combiner-les-informations-didentification-en-une-seule-étape) (ci-dessous) pour plus de détails.

**Exemple de clé privée de l'utilisateur SSH**

``` groovy 
withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'jenkins-ssh-key-for-abc', \
                                             keyFileVariable: 'SSH_KEY_FOR_ABC', \
                                             passphraseVariable: '', \
                                             usernameVariable: '')]) {
  // un block
}
```

Les définitions facultatives `passphraseVariable` et `usernameVariable` peuvent être supprimées dans votre code Pipeline final.

**Exemple de certificat**

``` groovy
withCredentials(bindings: [certificate(aliasVariable: '', \
                                       credentialsId: 'jenkins-certificate-for-xyz', \
                                       keystoreVariable: 'CERTIFICATE_FOR_XYZ', \
                                       passwordVariable: 'XYZ-CERTIFICATE-PASSWORD')]) {
  // un block
}
```

Les définitions de variables facultatives `aliasVariable` et `passwordVariable` peuvent être supprimées dans votre code Pipeline final.

L'extrait de code suivant montre un exemple de Pipeline dans son intégralité, qui implémente la clé privée de l'utilisateur SSH et les extraits de certificat ci-dessus :

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent {
        // définir les détails de l'agent
    }
    stages {
        stage('Example stage 1') {
            steps {
                withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'jenkins-ssh-key-for-abc', \
                                                             keyFileVariable: 'SSH_KEY_FOR_ABC')]) {
                  // //(1)
                }
                withCredentials(bindings: [certificate(credentialsId: 'jenkins-certificate-for-xyz', \
                                                       keystoreVariable: 'CERTIFICATE_FOR_XYZ', \
                                                       passwordVariable: 'XYZ-CERTIFICATE-PASSWORD')]) {
                  // //(2)
                }
            }
        }
        stage('Example stage 2') {
            steps {
                // //(3)
                )
            }
        }
    }
}
```

1. Au cours de cette étape, vous pouvez référencer la variable environnement d'informations d'identification à l'aide de la syntaxe `$SSH_KEY_FOR_ABC`. Par exemple, vous pouvez ici vous authentifier auprès de l'application ABC à l'aide des informations d'identification de la paire de clés publique/privée SSH configurée, dont le fichier de **clé privée utilisateur SSH** est attribué à `$SSH_KEY_FOR_ABC`.
2. Au cours de cette étape, vous pouvez référencer la variable environnement d'informations d'identification à l'aide de la syntaxe`$CERTIFICATE_FOR_XYZ` et `$XYZ-CERTIFICATE-PASSWORD`. Par exemple, vous pouvez ici vous authentifier auprès de l'application XYZ à l'aide des informations d'identification de certificat configurées, dont le fichier de stockage de clés et le mot de passe du **certificat** sont attribués respectivement aux variables `$CERTIFICATE_FOR_XYZ` et `$XYZ-CERTIFICATE-PASSWORD`.
3. Dans cet exemple Pipeline, les informations d'identification attribuées aux variables d'environnement `$SSH_KEY_FOR_ABC`, `$CERTIFICATE_FOR_XYZ` et `$XYZ-CERTIFICATE-PASSWORD` ne sont valables que dans leurs étapes `withCredentials( …​ ) { …​ }` respectives. Ces variables d'identification ne sont donc pas disponibles pour être utilisées dans les étapes `Example stage 2`.

Pour préserver la sécurité et l'anonymat de ces informations d'identification, si vous tentez de récupérer la valeur de ces variables d'identification à partir des étapes `withCredentials( …​ ) { …​ }`, le même comportement que celui décrit dans l'exemple de [texte secret](#texte-secret) (ci-dessus) s'applique également à ces types de variables d'identification et de certificat de paire de clés SSH publique/privée. Cela ne fait que réduire le risque **d'exposition accidentelle**. Cela n'empêche pas un utilisateur malveillant de capturer la valeur des informations d'identification par d'autres moyens. Un Pipeline qui utilise des informations d'identification peut également divulguer ces informations. Ne permettez pas à des tâches de Pipeline non fiables d'utiliser des informations d'identification fiables.

!!! note

    * Lorsque vous utilisez l'option **withCredentials: Bind credentials to variables** (Lier les informations d'identification aux variables) du champ **_Sample Step_** (Étape d'Exemple) dans le **_Snippet Generator_** (Générateur d'Extraits), seules les informations d'identification auxquelles votre projet/élément Pipeline actuel a accès peuvent être sélectionnées dans la liste de n'importe quel champ **_Credentials_** (Informations d'identification). Bien que vous puissiez écrire manuellement une étape `withCredentials( …​ ) { …​ }` pour votre Pipeline (comme dans les exemples ci-dessus), il est recommandé d'utiliser le **générateur d'extraits** afin d'éviter de spécifier des informations d'identification qui ne relèvent pas du champ d'application de ce projet/élément Pipeline, ce qui entraînerait l'échec de l'étape lors de son exécution.
    * Vous pouvez également utiliser le **_Snippet Generator_** (Générateur d'Extraits) pour générer des étapes `withCredentials( …​ ) { …​ }` afin de gérer les textes secrets, les noms d'utilisateur et les mots de passe, ainsi que les fichiers secrets. Cependant, si vous n'avez besoin de gérer que ces types d'informations d'identification, il est recommandé d'utiliser la procédure appropriée décrite dans la section [ci-dessus](#pour-le-texte-secret-les-noms-dutilisateur-les-mots-de-passe-et-les-fichiers-secrets) afin d'améliorer la lisibilité du code du Pipeline.
    * Utilisation **d'apostrophes simples** au lieu de **guillemets doubles** pour définir le `script` (le paramètre implicite de `sh`) dans Groovy ci-dessus. Les apostrophes simples entraîneront l'expansion du secret par le shell en tant que variable d'environnement. Les guillemets doubles sont potentiellement moins sûrs, car le secret est interpolé par Groovy, et les listes de processus typiques du système d'exploitation le divulgueront donc accidentellement :

        ``` groovy
        node {
        withCredentials([string(credentialsId: 'mytoken', variable: 'TOKEN')]) {
            sh /* FAUX ! */ """
                set +x
                curl -H 'Token: $TOKEN' https://some.api/
                """
            sh /* CORRECT */ '''
                set +x
                curl -H 'Token: $TOKEN' https://some.api/
                '''
            }
        }
        ```

##### Combiner les références en une seule étape

En utilisant le **générateur d'extraits**, vous pouvez rendre plusieurs informations d'identification disponibles dans une seule étape `withCredentials (…) {…}` en faisant ce qui suit : 

1. Dans le tableau de bord Jenkins, sélectionnez le nom de votre projet/élément de Pipeline. 
2. Dans le volet de navigation gauche, sélectionnez **_Pipeline Syntax_** (Syntaxe Pipeline) et assurez-vous que l'option **_Snippet Generator_** (Générateur d'Extraits) est disponible en haut du volet de navigation. 
3. Dans le champ **_Sample Step_** (Exemple d'Etape), choisissez **_ withCredentials: Bind credentials to variables._** (liez des informations d'identification aux variables). 
4. Cliquez sur **_Add_** (Ajouter) sous **_Bindings_** (Liaisons). 
5. Choisissez le type d'identification à ajouter à la liste de création `withCredentials( …​ ) { …​ } ` de la liste déroulante. 
6. Spécifiez les détails des **_Bindings_** (Liaisons) d'identification. Lisez la suite ci-dessus dans la procédure sous [d'autres types d'identification](#autres-types-didentification) (ci-dessus). 
7. Répétez à partir de Cliquez sur **_Add_** (Ajouter) (ci-dessus) pour chaque (ensemble de) information(s) d'identification(s) pour ajouter à l'étape `withCredentials (…) {…}`. 
8. Sélectionnez **_Generate Pipeline Script_** (Générer le Script Pipeline) pour générer l'extrait final `withCredentials( …​ ) { …​ }`.

### Interpolation de chaînes

Jenkins Pipeline utilise des règles identiques à celles de [Groovy](http://groovy-lang.org/) pour l'interpolation de chaînes. La prise en charge de l'interpolation de chaînes par Groovy peut être source de confusion pour de nombreux nouveaux utilisateurs du langage. Groovy prend en charge la déclaration d'une chaîne avec des guillemets simples ou doubles, par exemple :

``` groovy
def singlyQuoted = 'Hello'
def doublyQuoted = "World"
```

Seule la dernière chaîne prendra en charge l'interpolation basée sur le signe dollar (`$`), par exemple :

``` groovy
def username = 'Jenkins'
echo 'Hello Mr. ${username}'
echo "I said, Hello Mr. ${username}"
```

Entraînerait :

``` console
Hello Mr. ${username}
I said, Hello Mr. Jenkins
```

Comprendre comment utiliser l'interpolation des chaînes est essentiel pour utiliser certaines des fonctionnalités les plus avancées de Pipeline.

#### Interpolation des variables d'environnement sensibles

!!! warning
    L'interpolation de la chaîne Groovy ne doit jamais être utilisée avec des informations d'identification.

L'interpolation de chaînes Groovy peut entraîner la fuite de variables d'environnement sensibles (c'est-à-dire des informations d'identification, voir : [Gestion des informations d'identification](#gestion-des-informations-didentification)). En effet, la variable d'environnement sensible sera interpolée pendant l'évaluation Groovy, et la valeur de la variable d'environnement pourrait être disponible plus tôt que prévu, entraînant la fuite de données sensibles dans divers contextes.

Par exemple, considérez une variable d'environnement sensible passée à l'étape `sh`.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any
    environment {
        EXAMPLE_CREDS = credentials('example-credentials-id')
    }
    stages {
        stage('Example') {
            steps {
                /* FAUX ! */
                sh("curl -u ${EXAMPLE_CREDS_USR}:${EXAMPLE_CREDS_PSW} https://example.com/")
            }
        }
    }
}
```

Si Groovy effectue l'interpolation, la valeur sensible sera directement injectée dans les arguments de l'étape `sh`, ce qui signifie, entre autres, que la valeur littérale sera visible en tant qu'argument du processus `sh` sur l'agent dans les listes de processus du système d'exploitation. L'utilisation d'apostrophes simples au lieu de guillemets doubles pour référencer ces variables d'environnement sensibles permet d'éviter ce type de fuite.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any
    environment {
        EXAMPLE_CREDS = credentials('example-credentials-id')
    }
    stages {
        stage('Example') {
            steps {
                /* CORRECT */
                sh('curl -u $EXAMPLE_CREDS_USR:$EXAMPLE_CREDS_PSW https://example.com/')
            }
        }
    }
}
```

#### Injection par interpolation

!!! warning
    L'interpolation de la chaîne Groovy peut injecter des commandes Rogue dans des interprètes de commande via des caractères spéciaux.

Autre mise en garde : L'utilisation de l'interpolation de chaînes Groovy pour les variables contrôlées par l'utilisateur avec des étapes qui transmettent leurs arguments à des interpréteurs de commandes tels que les étapes `sh`, `bat`, `powershell` ou `pwsh` peut entraîner des problèmes analogues à l'injection SQL. Cela se produit lorsqu'une variable contrôlée par l'utilisateur (généralement une variable d'environnement, souvent un paramètre transmis à la compilation) contenant des caractères spéciaux (par exemple `/ \ $ & % ^ > < | ;`) est transmise aux étapes `sh`, `bat`, `powershell` ou `pwsh` à l'aide de l'interpolation Groovy. Voici un exemple simple :

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
  agent any
  parameters {
    string(name: 'STATEMENT', defaultValue: 'hello; ls /', description: 'What should I say?')
  }
  stages {
    stage('Example') {
      steps {
        /* FAUX ! */
        sh("echo ${STATEMENT}")
      }
    }
  }
}
```

Dans cet exemple, l'argument de l'étape `sh` est évalué par Groovy, et `STATEMENT` est interpolée directement dans l'argument comme si `sh('echo hello; ls /')` a été écrit dans le Pipeline. Lorsque cela est traité sur l'agent, au lieu d'afficher la valeur `hello; ls /`, il affichera `hello` puis procédera à la liste complète du répertoire racine de l'agent. Tout utilisateur capable de contrôler une variable interpolée par une telle étape serait en mesure de faire exécuter un code arbitraire sur l'agent par l'étape `sh`. Pour éviter ce problème, assurez-vous que les arguments des étapes telles que `sh` ou `bat` qui font référence à des paramètres ou à d'autres variables d'environnement contrôlées par l'utilisateur utilisent des guillemets simples afin d'éviter l'interpolation Groovy.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
  agent any
  parameters {
    string(name: 'STATEMENT', defaultValue: 'hello; ls /', description: 'What should I say?')
  }
  stages {
    stage('Example') {
      steps {
        /* CORRECT */
        sh('echo ${STATEMENT}')
      }
    }
  }
}
```

La déformation des informations d'identification est un autre problème qui peut se produire lorsque des informations d'identification contenant des caractères spéciaux sont transmises à une étape à l'aide de l'interpolation Groovy. Lorsque la valeur des informations d'identification est déformée, elle n'est plus valide et n'est plus masquée dans le journal de la console.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
  agent any
  environment {
    EXAMPLE_KEY = credentials('example-credentials-id') // La valeur secrète est 'sec%ret'
  }
  stages {
    stage('Example') {
      steps {
          /* FAUX ! */
          bat "echo ${EXAMPLE_KEY}"
      }
    }
  }
}
```

Ici, la commande `bat` reçoit `echo sec%ret` et le shell batch Windows supprime simplement le `%` et affiche la valeur `secret`. Comme il n'y a qu'un seul caractère de différence, la valeur `secret` n'est pas masquée. Même si cette valeur ne correspond pas à l'identifiant réel, il s'agit tout de même d'une divulgation importante d'informations sensibles. Là encore, les guillemets simples permettent d'éviter ce problème

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
  agent any
  environment {
    EXAMPLE_KEY = credentials('example-credentials-id') // La valeur secrète est 'sec%ret'
  }
  stages {
    stage('Example') {
      steps {
          /* CORRECT */
          bat 'echo %EXAMPLE_KEY%'
      }
    }
  }
}
```

### Gestion des paramètres

Le Pipeline déclaratif prend en charge les paramètres prêts à l'emploi, ce qui lui permet d'accepter les paramètres spécifiés par l'utilisateur lors de l'exécution via la [directive parameters](./pipeline-syntaxe.md#parametres). La configuration des paramètres avec le Pipeline scripté s'effectue à l'aide de l'étape `properties`, qui se trouve dans le générateur d'extraits de code.

Si vous avez configuré votre Pipeline pour accepter des paramètres à l'aide de l'option **_Build with Parameters_** (Construire avec des Paramètres), ces paramètres sont accessibles en tant que membres de la variable `params`.

En supposant qu'un paramètre de chaîne nommé "Greeting" ait été configuré dans le `Jenkinsfile`, il peut accéder à ce paramètre via `${params.Greeting}` :

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any
    parameters {
        string(name: 'Greeting', defaultValue: 'Hello', description: 'How should I greet the world?')
    }
    stages {
        stage('Example') {
            steps {
                echo "${params.Greeting} World!"
            }
        }
    }
}
```

??? "Activer/désactiver le Pipeline Scripté (_Avancé_)"

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    properties([parameters([string(defaultValue: 'Hello', description: 'How should I greet the world?', name: 'Greeting')])])

    node {
        echo "${params.Greeting} World!"
    }
    ```

### Échec de la gestion

Le Pipeline déclaratif prend en charge par défaut une gestion robuste des échecs via sa [section post](./pipeline-syntaxe.md#post) qui permet de déclarer un certain nombre de « conditions post » différentes telles que : `always`, `unstable`, `success`, `failure` et `changed`. La section [Syntaxe Pipeline] (./pipeline-syntaxe.md) fournit plus de détails sur l'utilisation des différentes conditions post.

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'make check'
            }
        }
    }
    post {
        always {
            junit '**/target/*.xml'
        }
        failure {
            mail to: team@example.com, subject: 'The Pipeline failed :('
        }
    }
}}
```

??? "Activer/désactiver le Pipeline Scripté (_Avancé_)"

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    node {
        /* .. snip .. */
        stage('Test') {
             try {
                sh 'make check'
             }
             finally {
                junit '**/target/*.xml'
            }
        }
        /* .. snip .. */
    }
    ```

Le Pipeline Scripté repose cependant sur la sémantique `try`/`catch`/`finally`  de Groovy pour gérer les échecs lors de l'exécution du Pipeline.

Dans l'exemple de test ci-dessus, l'étape `sh` a été modifiée pour ne jamais renvoyer un code de sortie différent de zéro (`sh “make check || true”`). Cette approche, bien que valide, implique que les étapes suivantes doivent vérifier `currentBuild.result` pour savoir s'il y a eu un échec du test ou non.

Une autre façon de gérer cela, qui préserve le comportement précoce des échecs dans le Pipeline, tout en donnant à `junit` la possibilité de capturer des rapports de test, est d'utiliser une série de blocs `try`/`finally` :

#### Étapes de manipulation des erreurs

Les Pipelines Jenkins fournissent des étapes dédiées pour la gestion des erreurs flexibles, vous permettant de contrôler la manière dont votre Pipeline réagit aux erreurs et aux avertissements. Ces étapes vous aident à faire apparaître clairement les erreurs et les avertissements dans Jenkins, vous permettant ainsi de contrôler si le Pipeline échoue, se poursuit ou se contente de signaler un avertissement. Pour plus d'informations, consultez :

* [catchError](https://www.jenkins.io/doc/pipeline/steps/workflow-basic-steps/#catcherror-catch-error-and-set-build-result-to-failure)
*  [error](https://www.jenkins.io/doc/pipeline/steps/workflow-basic-steps/#error-error-signal)
* [unstable](https://www.jenkins.io/doc/pipeline/steps/workflow-basic-steps/#unstable-set-stage-result-to-unstable)
* [warnError](https://www.jenkins.io/doc/pipeline/steps/workflow-basic-steps/#warnerror-catch-error-and-set-build-and-stage-result-to-unstable)

### En utilisant plusieurs agents

Dans tous les exemples précédents, un seul agent a été utilisé. Cela signifie que Jenkins attribuera un exécuteur dès qu'il sera disponible, indépendamment de son étiquette ou de sa configuration. Non seulement ce comportement peut être remplacé, mais Pipeline permet également d'utiliser plusieurs agents dans l'environnement Jenkins à partir du même fichier `Jenkinsfile`, ce qui peut s'avérer utile pour des cas d'utilisation plus avancés, tels que l'exécution de builds/tests sur plusieurs plateformes.

Dans l'exemple ci-dessous, l'étape "Build" sera effectuée sur un agent et les résultats construits seront réutilisés sur deux agents suivants, étiquetés respectivement "Linux" et "Windows", lors de l'étape "Test".

``` groovy title="<i>Jenkinsfile (Pipeline Déclaratif)</i>"
pipeline {
    agent none
    stages {
        stage('Build') {
            agent any
            steps {
                checkout scm
                sh 'make'
                stash includes: '**/target/*.jar', name: 'app' // (1)
            }
        }
        stage('Test on Linux') {
            agent { // (2)
                label 'linux'
            }
            steps {
                unstash 'app' // (3)
                sh 'make check'
            }
            post {
                always {
                    junit '**/target/*.xml'
                }
            }
        }
        stage('Test on Windows') {
            agent {
                label 'windows'
            }
            steps {
                unstash 'app'
                bat 'make check' // (4)
            }
            post {
                always {
                    junit '**/target/*.xml'
                }
            }
        }
    }
}
```

??? "Activer/désactiver le Pipeline Scripté (_Avancé_)"

    ``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
    stage('Build') {
        node {
            checkout scm
            sh 'make'
            stash includes: '**/target/*.jar', name: 'app' // (1)
        }
    }

    stage('Test') {
        node('linux') { // (2)
            checkout scm
            try {
                unstash 'app' // (3)
                sh 'make check'
            }
            finally {
                junit '**/target/*.xml'
            }
        }
        node('windows') {
            checkout scm
            try {
                unstash 'app'
                bat 'make check' // (4)
            }
            finally {
                junit '**/target/*.xml'
            }
        }
    }
    ```

1. L'étape `stash` permet de capturer les fichiers correspondant à un modèle d'inclusion (`**/target/*.jar`) afin de les réutiliser dans le _même_ pipeline. Une fois l'exécution du pipeline terminée, les fichiers stockés sont supprimés du contrôleur Jenkins.
2. Le paramètre dans `agent`/`node` permet toute expression de libellé Jenkins valide. Consultez la section [Syntaxe du Pipeline](./pipeline-syntaxe.md) pour plus de détails.
3. La commande `unstash` récupère le « stash » nommé depuis le contrôleur Jenkins vers l'espace de travail actuel du Pipeline.
4. Le script `bat` permet d'exécuter des scripts batch sur les plateformes Windows.

### Arguments d'étape facultatifs

Pipeline suit la convention du langage Groovy qui autorise l'omission des parenthèses autour des arguments de méthode.

De nombreuses étapes Pipeline utilisent également la syntaxe des paramètres nommés comme raccourci pour créer une carte dans Groovy, qui utilise la syntaxe `[key1: value1, key2: value2]`. Les instructions suivantes sont donc fonctionnellement équivalentes :

``` groovy
git url: 'git://example.com/amazing-project.git', branch: 'master'
git([url: 'git://example.com/amazing-project.git', branch: 'master'])
```

Pour plus de commodité, lorsque vous appelez des étapes ne prenant qu'un seul paramètre (ou un seul paramètre obligatoire), le nom du paramètre peut être omis, par exemple :

``` groovy
sh 'echo hello' /* short form  */
sh([script: 'echo hello'])  /* long form */
```

### Pipeline scripté avancé

Le Pipeline scripté est un langage spécifique à un domaine [^3] basé sur Groovy. La plupart des [syntaxes Groovy](http://groovy-lang.org/semantics.html) peuvent être utilisées dans le Pipeline scripté sans modification.

#### Exécution parallèle

L'exemple présenté dans la [section ci-dessus](#en-utilisant-plusieurs-agents) exécute des tests sur deux plateformes différentes dans une série linéaire. En pratique, si l'exécution de `make check` prend 30 minutes, la phase « Test » prendrait alors 60 minutes !

Heureusement, Pipeline dispose d'une fonctionnalité intégrée permettant d'exécuter certaines parties de Pipeline scripté en parallèle, mise en œuvre dans l'étape `parallel`, qui porte bien son nom.

Refactorisation de l'exemple ci-dessus pour utiliser l'étape `parallel` :

``` groovy title="<i>Jenkinsfile (Pipeline Scripté)</i>"
stage('Build') {
    /* .. snip .. */
}

stage('Test') {
    parallel linux: {
        node('linux') {
            checkout scm
            try {
                unstash 'app'
                sh 'make check'
            }
            finally {
                junit '**/target/*.xml'
            }
        }
    },
    windows: {
        node('windows') {
            /* .. snip .. */
        }
    }
}
```

Au lieu d'exécuter les tests sur les nœuds étiquetés « linux » et « windows » en série, ils s'exécuteront désormais en parallèle, à condition que la capacité requise existe dans l'environnement Jenkins.
<hr>

1. [en.wikipedia.org/wiki/Source_control_management](https://en.wikipedia.org/wiki/Source_control_management)
2. [en.wikipedia.org/wiki/Single_Source_of_Truth](https://en.wikipedia.org/wiki/Single_Source_of_Truth)
3. [en.wikipedia.org/wiki/Domain-specific_language](https://en.wikipedia.org/wiki/Domain-specific_language)
<hr>