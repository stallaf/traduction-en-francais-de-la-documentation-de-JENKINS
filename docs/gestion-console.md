# Console de Script

<div classd="couleur-introduction">
Jenkins dispose d'une console de script qui permet d'exécuter des scripts Groovy arbitraires dans le runtime du contrôleur Jenkins ou dans le runtime des agents.
</div>

!!! danger "Important"
    Il est très **important** de comprendre tous les points suivants, car ils ont une incidence sur l'intégrité de votre installation Jenkins. Concernant la console de script Jenkins :

    * Son accès est contrôlé par l'autorisation `Administrer` ;
    * Il s'agit d'un shell Groovy basé sur le Web dans le runtime Jenkins. Groovy est un langage très puissant qui offre la possibilité de faire pratiquement tout ce que Java peut faire, notamment :
        * Créer des sous-processus et exécuter des commandes arbitraires sur le contrôleur Jenkins et les agents ;
        * Il peut même lire les fichiers auxquels le contrôleur Jenkins a accès sur l'hôte (comme `/etc/passwd`) ;
        * Décrypter les informations d'identification configurées dans Jenkins.
    * N'offre aucun contrôle administratif pour empêcher un utilisateur (ou un administrateur) d'affecter toutes les parties de l'infrastructure Jenkins une fois qu'il est en mesure d'exécuter la console de script. Accorder à un utilisateur Jenkins normal l'accès à la console de script revient essentiellement à lui donner des droits d'administrateur dans Jenkins.
    * Peut configurer n'importe quel paramètre Jenkins. Il peut désactiver la sécurité, reconfigurer la sécurité, voire ouvrir une porte dérobée sur le système d'exploitation hôte complètement en dehors du processus Jenkins. En raison de l'importance cruciale que de nombreuses organisations accordent à Jenkins dans leur infrastructure, ce point est particulièrement important car il permettrait à un attaquant de se déplacer latéralement au sein de l'infrastructure sans grand effort.
    * Il est si puissant parce qu'il était à l'origine conçu comme une interface de débogage pour les développeurs Jenkins, mais qu'il est depuis devenu une interface utilisée par les administrateurs Jenkins pour configurer Jenkins et déboguer les problèmes d'exécution de Jenkins.

    Considérations supplémentaires en matière de sécurité pour les scripts administratifs :

    * Les scripts peuvent contourner tous les contrôles de sécurité et accéder à toutes les données ;
    * Les informations d'identification accessibles via les scripts sont déchiffrées ;
    * Les modifications apportées via des scripts peuvent ne pas être vérifiées dans les journaux normaux.
    * Les scripts peuvent modifier Jenkins aux niveaux les plus bas, ce qui peut le rendre instable.

    En raison de la puissance offerte par la console de scripts Jenkins, Jenkins et ses agents ne doivent jamais être exécutés en tant qu'utilisateur `root` (sous Linux) ou administrateur système sur tout autre type de système d'exploitation. Les vidéos liées à cette page présentent et discutent des avertissements de sécurité.

    **Veillez à sécuriser votre contrôleur Jenkins.**

## Contextes multiples

La console de script Jenkins peut être exécutée soit sur le contrôleur, soit sur n'importe quel agent configuré.

### Exécution de la console de script sur le contrôleur

Cette fonctionnalité est accessible depuis _« Manage Jenkins » > « Script Console »_ (Gérer Jenkins > Console de script).  Vous pouvez également y accéder en vous rendant sur la sous-URL `/script` de votre contrôleur Jenkins.

### Exécution de la console de script sur les agents

Accédez à _« Manage Jenkins » > « Manage Nodes »_ (Gérer Jenkins > Gérere les Noeuds).  Sélectionnez n'importe quel nœud pour afficher la page d'état.  Dans le menu de gauche, un élément de menu permet d'ouvrir une « Script Console » sur cet agent spécifique.

### Exécuter des scripts à partir de la console de script du contrôleur sur les agents

Il est également possible d'exécuter des scripts à partir de la console de script du contrôleur sur des agents individuels.  Le script suivant est un exemple d'exécution d'un script sur des agents à partir de la console de script du contrôleur.

**Le script exécute le code sur l'agent à partir de la console de script principale**

``` groovy title="GROOVY"
import hudson.util.RemotingDiagnostics
import jenkins.model.Jenkins

String agentName = “nom de votre agent”
//script groovy que vous souhaitez exécuter sur un agent
groovy_script = “”'
println System.getenv(« PATH »)
println « uname -a ».execute().text
'“”.trim()

String result
Jenkins.instance.slaves.find { agent ->
    agent.name == agentName
}.with { agent ->
    result = RemotingDiagnostics.executeGroovy(groovy_script, agent.channel)
}
```

### Lecture et écriture de fichiers

Les fichiers peuvent être lus et écrits directement sur le contrôleur ou les agents via la console de script du contrôleur.

**Écrire un fichier sur le contrôleur Jenkins**

``` groovy title="GROOVY"
new File(“/tmp/file.txt”).withWriter(“UTF-8”) { writer ->
    try {
        writer << “hello world\n”
    } finally {
        writer.close()
    }
}
```

**Lecture d'un fichier à partir du contrôleur Jenkins**

``` groovy title="GROOVY"
new File(“/tmp/file.txt”).text
```

**Écriture d'un fichier sur l'agent via le canal de l'agent**

``` groovy title="GROOVY"
import hudson.FilePath
import hudson.remoting.Channel
import jenkins.model.Jenkins

String agentName = “some-agent”
String filePath = '/tmp/file.txt'

Channel agentChannel = Jenkins.instance.slaves.find { agent ->
    agent.name == agentName
}.channel

new FilePath(agentChannel, filePath).write().with { os ->
    try {
        os << “hello world\n”
    } finally {
        os.close()
    }
}
```

**Lire un fichier depuis l'agent via le canal de l'agent**

``` groovy title="GROOVY"
import hudson.FilePath
import hudson.remoting.Channel
import jenkins.model.Jenkins

import java.io.BufferedReader
import java.io.InputStreamReader
import java.nio.charset.StandardCharsets
import java.util.stream.Collectors

String agentName = “some-agent”
String filePath = '/tmp/file.txt'

Channel agentChannel = Jenkins.instance.slaves.find { agent ->
    agent.name == agentName
}.channel

String fileContents = “”
new FilePath(agentChannel, filePath).read().with { is ->
    try {
        fileContents = new BufferedReader(
            new InputStreamReader(is, StandardCharsets.UTF_8))
                .lines()
                .collect(Collectors.joining(« \n »))
    } finally {
        is.close()
    }
}

// imprimer le contenu du fichier depuis l'agent
println “===”
println(fileContents)
println '==='
```

## Capacités administratives

La console de script fournit de puissantes fonctions administratives pour gérer Jenkins.

Vous trouverez ci-dessous des exemples de tâches administratives courantes :

### Gestion des utilisateurs

``` groovy title="GROOVY"
Jenkins.instance.securityRealm.allUsers.each { user ->
    println user.id + « : » + user.fullName
}

import hudson.model.User
import jenkins.model.Jenkins

User user = User.get(“new-user”, true)
user.fullName = « New User »
user.save()
```

### Configuration du système

``` groovy title="GROOVY"
Jenkins.instance.systemMessage = « New system message »
Jenkins.instance.save()

println Jenkins.VERSION
```

### Modifications des paramètres système

``` groovy title="GROOVY"
System.getProperty(« hudson.remoting.Launcher.pingIntervalSec ») // afficher le paramètre actuel
System.clearProperty(« hudson.remoting.Launcher.pingIntervalSec ») // définir la valeur par défaut
System.setProperty(« hudson.remoting.Launcher.pingIntervalSec », « ») // désactiver l'en-tête
System.setProperty(« hudson.remoting.Launcher.pingIntervalSec », 0)
```

### Gestion des plugins

``` groovy title="GROOVY"
Jenkins.instance.pluginManager.plugins.each {
    println « ${it.shortName}: ${it.version} »
}
Jenkins.instance.pluginManager.getPlugin(“git”).disable()
```

### Gestion des nœuds/agents

``` groovy title="GROOVY"
Jenkins.instance.nodes.each { node ->
    println « ${node.name}: ${node.numExecutors} executors »
}
Jenkins.instance.getNode(“agent-name”).computer.doDoDelete()
```

### Tuer une tâche bloquée

``` groovy title="GROOVY"
Jenkins.instance.getItemByFullName(« <projectName »).getBuildByNumber(<BuildNumber).finish(
    hudson.model.Result.ABORTED, new java.io.IOException(« Aborting build »)
);
```

## Meilleures pratiques pour le système Groovy

Lorsque vous utilisez la console de script pour des tâches administratives :

* Testez toujours les scripts dans un environnement hors production avant de les exécuter ;
* [Sauvegardez votre configuration](./administration-systeme-sauvegarde.md) Jenkins avant d'exécuter des scripts ;
* Utilisez [l'API Jenkins](https://www.jenkins.io/doc/developer/security/misc/) plutôt que la manipulation directe du système de fichiers lorsque cela est possible ;
* Limitez la portée des scripts afin que seules les opérations nécessaires soient effectuées ;
* Ajoutez des commentaires aux scripts pour référence future ;
* Envisagez d'utiliser des plugins tels que [Job DSL](https://plugins.jenkins.io/job-dsl) ou [Configuration as Code](https://plugins.jenkins.io/configuration-as-code) pour les configurations répétables ;
* Vérifiez attentivement les scripts afin de vous assurer qu'ils s'exécutent avec tous les privilèges système ;
* Surveillez l'exécution des scripts, car les scripts à exécution longue peuvent avoir un impact sur les performances.

### Exemple : modèle de script sécurisé

``` groovy title="GROOVY"
try {
    Jenkins.instance.doSomething()
    Jenkins.instance.save()

    println « Opération terminée avec succès »
} catch(Exception e) {
    println « Erreur : ${e.message} »
}
```

## Accès à distance

Un administrateur Jenkins peut exécuter des scripts Groovy à distance en envoyant une requête HTTP POST à l'URL `/script/` ou `/scriptText/`.

**Exemple de curl via bash**

``` shell title="SHELL"
curl -d « script=<votre_script_ici> » https://jenkins/script
# ou pour obtenir le résultat sous forme de texte brut (sans HTML)
curl -d « script=<votre_script_ici> » https://jenkins/scriptText
```

De plus, [l'interface CLI](./gestion-cli.md) Jenkins offre la possibilité d'exécuter des scripts Groovy à distance à l'aide de la commande `groovy` ou d'exécuter Groovy de manière interactive via `groovysh`. Cependant, une fois encore, curl peut être utilisé pour exécuter des scripts Groovy en utilisant la substitution de commande bash. Dans l'exemple suivant, `somescript.groovy` est un script Groovy dans le répertoire de travail actuel.

**Curl soumettant un fichier Groovy via bash**

``` shell title="SHELL"
curl --data-urlencode « script=$(< ./somescript.groovy) » https://jenkins/scriptText
```

Si la sécurité est configurée dans Jenkins, curl peut alors être fourni avec des options d'authentification à l'aide de l'option `curl --user`.

**Curl soumettant un fichier Groovy en fournissant un nom d'utilisateur et un jeton API via bash**

``` shell title="SHELL"
curl --user “username:api-token” --data-urlencode \
  « script=$(< ./somescript.groovy) » https://jenkins/scriptText
```

Voici la commande équivalente utilisant python, et non curl.

**Python soumettant un fichier groovy fournissant le nom d'utilisateur et le jeton API**

``` py title="PYTHON"
with open(“somescript.groovy”, “r”) as fd:
    data = fd.read()
r = requests.post(“https://jenkins/scriptText”, auth=(“username”, “api-token”), data={“script”: data})
```

## Raccourci clavier sur la console de script pour soumettre

Vous pouvez envoyer un script sans souris. Jenkins dispose d'un raccourci clavier qui permet d'envoyer à l'aide du clavier.

* Windows / Linux : Ctrl + Entrée ;
* Mac : Commande + Entrée.

## Tutoriels vidéo et supports pédagogiques supplémentaires

Voici quelques vidéos enregistrées sur la console de script Jenkins :

* [Jenkins World 2017 : Maîtriser la console de script Jenkins](https://www.youtube.com/watch?v=qaUPESDcsGg) - 44 minutes - exemples d'utilisation et discussion sur la sécurité ;
* [LA Jenkins Area Meetup 2016 - Hacking on Jenkins Internals - Jenkins Script Console ](https://www.youtube.com/watch?v=T1x2kCGRY1w)- 39 minutes - exemples d'utilisation.

Pour développer vos compétences en matière de rédaction de scripts dans la console de script, nous vous recommandons les références suivantes :

* [Apprendre Groovy](http://groovy-lang.org/learn.html) - L'apprentissage de Groovy est utile pour bien plus que la rédaction de scripts.  Groovy est également pertinent pour d'autres fonctionnalités de Jenkins telles que les [Pipelines et les bibliothèques de pipelines partagées](./pipeline-presentation.md), le [plugin Groovy](https://plugins.jenkins.io/groovy), le [plugin Job DSL](https://plugins.jenkins.io/job-dsl) et de nombreux autres plugins qui utilisent Groovy (voir la section [Plugins-enabling-Groovy-usage](#plugins-permettant-lutilisation-de-groovy)).
* [Écrire des scripts Groovy pour Jenkins avec la complétion de code ](http://www.mdoninger.de/2011/11/07/write-groovy-scripts-for-jenkins-with-code-completion.html)- L'essentiel consiste à créer un projet Maven dans votre IDE et à dépendre de org.jenkins-ci.main:jenkins-core (et de tout autre plugin que vous souhaitez utiliser). Vous pouvez ensuite écrire un script Groovy avec la complétion de code des objets et méthodes de l'API Jenkins.

## Exemples de scripts Groovy

### Scripts obsolètes

En raison de la nature des scripts Groovy qui accèdent directement au code source Jenkins, les scripts de la console de script sont facilement obsolètes par rapport au code source Jenkins. Il est possible d'exécuter un script et d'obtenir des exceptions parce que les méthodes publiques et les interfaces dans le noyau Jenkins ou les plugins Jenkins ont changé. Gardez cela à l'esprit lorsque vous essayez des exemples. Jenkins est facilement démarré à partir d'une machine de développement locale via la commande suivante :

**Démarrer une copie locale de Jenkins**

``` shel title="SHELL"
export JENKINS_HOME="./my_jenkins_home"
java -jar jenkins.war
```

Utilisez CTRL+C pour arrêter Jenkins. Il n'est pas recommandé d'essayer les exemples de la console de script dans un contrôleur Jenkins de production.

Les référentiels suivants offrent des exemples concrets de scripts Groovy pour Jenkins.

* [Référentiel CloudBees jenkins-scripts](https://github.com/cloudbees/jenkins-scripts) ;
* [Référentiel Jenkins CI jenkins-scripts sous le répertoire scriptler/](https://github.com/jenkinsci/jenkins-scripts) (scripts pour le [plugin Scriptler](https://plugins.jenkins.io/scriptler)) ;
 * [Référentiel jenkins-script-console-scripts de Sam Gleske](https://github.com/samrocketman/jenkins-script-console-scripts) ;
* [Référentiel jenkins-bootstrap-shared de Sam Gleske sous le répertoire scripts/](https://github.com/samrocketman/jenkins-bootstrap-shared).

Parcourez tous les scripts Groovy du [plugin Scriptler](https://plugins.jenkins.io/scriptler) et partagez vos scripts avec le [plugin Scriptler](https://plugins.jenkins.io/scriptler).

* Activer le [plugin Chuck Norris](https://wiki.jenkins.io/display/JENKINS/Activate+Chuck+Norris+Plugin) — Ce script active le plugin Chuck Norris pour toutes les tâches de votre serveur Jenkins ;
* [Ajouter une installation Maven, une installation d'outil, modifier la configuration du système](https://wiki.jenkins.io/JENKINS/74416647.html) ;
* [Ajouter une nouvelle étiquette aux agents répondant à une condition](https://wiki.jenkins.io/display/JENKINS/Add-a-new-label-to-agents-meeting-a-condition.html) — Ce script montre comment modifier l'appartenance à une étiquette des nœuds d'agent. Dans ce cas, nous créons une nouvelle étiquette si l'étiquette existante contient une chaîne. Il a été testé à partir de la fenêtre de commande Jenkins ;
* [Ajouter un plugin de notification à chaque tâche](https://wiki.jenkins.io/display/JENKINS/Add+notification+plugin+to+every+job) — Ce script ajoutera le plugin de notification à chaque tâche ;
* [Autoriser la revendication de build cassé sur toutes les tâches](https://wiki.jenkins.io/display/JENKINS/Allow+broken+build+claiming+on+every+jobs) — Grâce au script simple suivant, vous pouvez activer l'option sur toutes les tâches de votre serveur en une seule fois ;
* [Mise à jour groupée de la branche Mercurial qui est extraite](https://wiki.jenkins.io/display/JENKINS/Batch-Update+Mercurial+branch+that+is+checked+out) — Mises à jour pour plusieurs tâches dont la branche sera extraite de Hg ;
* [Renommer en masse des projets](https://wiki.jenkins.io/display/JENKINS/Bulk+rename+projects) ;
* [Modifier les options JVM dans toutes les tâches Maven des tâches Freestyle](https://wiki.jenkins.io/display/JENKINS/Change+JVM+Options+in+all+Maven+tasks+of+Freestyle+Jobs) — Ce script recherche toutes les tâches Maven enregistrées dans les tâches Freestyle et remplace les options JVM par une nouvelle valeur ;
* [Modification de la configuration de publication via SSH](https://wiki.jenkins.io/display/JENKINS/Change+publish+over+SSH+configuration  ;)
* [Modification du SCMTrigger pour chaque projet afin de le désactiver pendant la nuit et le week-end](https://wiki.jenkins.io/display/JENKINS/Change+SCMTrigger+for+each+project+to+disable+during+the+night+and+the+week-end) — Ce script vous permet de modifier facilement tous les travaux s'exécutant toutes les minutes afin qu'ils soient désactivés entre 21h00 et 7h00, ainsi que le samedi et le dimanche ;
* [Modifier le numéro de version dans le chemin SVN](https://wiki.jenkins.io/display/JENKINS/Change+Version-Number+in+SVN-path) ;
* [Cloner tous les projets dans une vue](https://wiki.jenkins.io/display/JENKINS/Clone+all+projects+in+a+View) — Ce script énumère tous les projets appartenant à une vue spécifique et les clone ;
* [Convertir les notifications par e-mail standard pour utiliser le plugin Mail-Ext Publisher ](https://wiki.jenkins.io/display/JENKINS/Convert+standard+mail+notifications+to+use+the+Mail-Ext+Publisher+plugin) — Ce script remplace les notifications par e-mail dans tous les projets par le plugin Mail-Ext Publisher et réutilise les destinataires existants ;
* [Supprimer les fichiers tmp restants dans les fichiers de l'espace de travail](https://wiki.jenkins.io/display/JENKINS/Delete+.tmp+files+left+in+workspace-files) — Ce script supprime tous les fichiers tmp restants dans le répertoire des fichiers de l'espace de travail après la compilation. Sur les serveurs Windows, cela semble assez courant ;
* [Supprimer l'espace de travail pour tous les travaux désactivés](https://wiki.jenkins.io/display/JENKINS/Delete+workspace+for+all+disabled+jobs) — Supprime l'espace de travail pour tous les travaux désactivés afin d'économiser de l'espace ;
* [Désactiver tous les travaux ](https://wiki.jenkins.io/display/JENKINS/Disable+all+jobs) — Ce script désactive tous les travaux sur votre serveur Jenkins ;
* [Afficher les informations sur les nœuds](https://wiki.jenkins.io/display/JENKINS/Display+Information+About+Nodes) — Ce script affiche un ensemble d'informations sur tous les nœuds agents ;
* [Afficher les paramètres des tâches](https://wiki.jenkins.io/display/JENKINS/Display+job+parameters) — Ce script affiche les paramètres de toutes les tâches ainsi que leurs valeurs par défaut (le cas échéant) ;
* [Afficher les tâches regroupées par étapes de compilation utilisées](https://wiki.jenkins.io/display/JENKINS/Display+jobs+group+by+the+build+steps+they+use) ;
* [Afficher la liste des projets compilés il y a plus d'un jour](https://wiki.jenkins.io/display/JENKINS/Display-list-of-projects-that-were-built-more-than-1-day-ago..html) — Ce script affiche la liste des projets compilés il y a plus d'un jour ;
* [Afficher les destinataires des notifications par e-mail](https://wiki.jenkins.io/display/JENKINS/Display+mail+notifications+recipients) — Ce script affiche pour toutes les tâches la liste des destinataires utilisés pour les notifications ;
* [Afficher l'état des moniteurs](https://wiki.jenkins.io/display/JENKINS/Display+monitors+status) — Jenkins utilise des moniteurs pour valider divers comportements. Si vous en supprimez un, Jenkins ne vous proposera jamais de le réactiver. Ce script vous permet de vérifier l'état de tous les moniteurs et de les réactiver ;
* [Afficher le nombre de tâches à l'aide du sondage SCM depuis Freestyle, Pipeline et Maven](https://wiki.jenkins.io/display/JENKINS/138454178.html) ;
* [Afficher les déclencheurs de minuterie](https://wiki.jenkins.io/display/JENKINS/Display+timer+triggers) — Ce script affiche les déclencheurs de minuterie pour toutes les tâches afin de mieux les organiser ;
* [Afficher l'emplacement des outils sur tous les nœuds](https://wiki.jenkins.io/display/JENKINS/Display+Tools+Location+on+All+Nodes) — Ce script permet d'obtenir l'emplacement des outils Jenkins sur tous vos agents ;
* [Activer le plugin Timestamper sur toutes les tâches ](https://wiki.jenkins.io/display/JENKINS/Enable+Timestamper+plugin+on+all+jobs)— Grâce au script simple suivant, vous pouvez activer cette option sur toutes les tâches de votre serveur en une seule fois ;
* [Tâches ayant échouées](https://wiki.jenkins.io/display/JENKINS/Failed+Jobs) — Ce script affiche la liste de toutes les tâches ayant échoué. Addon : les redémarrer ;
* [Rechercher les builds en cours d'exécution depuis plus de N secondes](https://wiki.jenkins.io/display/JENKINS/Find+builds+currently+running+that+has+been+executing+for+more+than+N+seconds) ;
* [Accorder l'autorisation d'annulation aux utilisateurs et groupes disposant d'une autorisation de build ](https://wiki.jenkins.io/display/JENKINS/Grant+Cancel+Permission+for+user+and+group+that+have+Build+permission) — Ce script passe en revue tous les groupes et utilisateurs dans les paramètres de sécurité globaux et par tâche ;
* [Invalider les sessions HTTP Jenkins](https://wiki.jenkins.io/display/JENKINS/Invalidate+Jenkins+HTTP+sessions) — Ce script peut surveiller et invalider les sessions HTTP s'il y en a beaucoup d'ouvertes sur votre serveur ;
* [Exécuter manuellement la rotation des journaux sur toutes les tâches ](https://wiki.jenkins.io/display/JENKINS/Manually+run+log+rotation+on+all+jobs) — Exécute la rotation des journaux sur toutes les tâches pour libérer de l'espace ;
* [Surveiller et redémarrer les agents hors ligne](https://wiki.jenkins.io/display/JENKINS/Monitor+and+Restart+Offline+Slaves) — Ce script peut surveiller et redémarrer les nœuds hors ligne s'ils ne sont pas déconnectés manuellement ;
* [Scripts de surveillance](https://wiki.jenkins.io/display/JENKINS/Monitoring+Scripts) — Plusieurs scripts permettant d'afficher des données sur les sessions HTTP, les threads, la mémoire, la JVM ou les MBeans, lors de l'utilisation du plugin Monitoring ;
* [My Test Grovvy](https://wiki.jenkins.io/display/JENKINS/My+Test+Grovvy) ;
* [Script Groovy système paramétré](https://wiki.jenkins.io/display/JENKINS/Parameterized+System+Groovy+script) — Ce script montre comment obtenir des paramètres dans un script Groovy système ;
* [Présélectionner le nom d'utilisateur dans Maven Release Build](https://wiki.jenkins.io/display/JENKINS/Preselect+username+in+Maven+Release+Build) ;
* [Imprimer une liste des informations d'identification et de leurs identifiants](https://wiki.jenkins.io/display/JENKINS/Printing+a+list+of+credentials+and+their+IDs) ;
* [Supprimer tous les modules désactivés dans les tâches Maven ](https://wiki.jenkins.io/display/JENKINS/Remove+all+disabled+modules+in+Maven+jobs) — Pour supprimer tous les modules désactivés dans les tâches Maven ;
* [Supprimer les actions d'artefacts déployés](https://wiki.jenkins.io/display/JENKINS/Remove+Deployed+Artifacts+Actions) — Ce script est utilisé pour supprimer la liste des artefacts déployés qui est inutilement stockée pour chaque build par le plugin Artifact Deployer ;
* [Supprimer les données de build BuildByBranch du plugin Git](https://wiki.jenkins.io/display/JENKINS/Remove+Git+Plugin+BuildsByBranch+BuildData) — Ce script permet de supprimer la liste statique BuildByBranch qui est inutilement stockée pour chaque build par le plugin Git ;
* [Définir GitBlitRepositoryBrowser avec des paramètres personnalisés sur tous les dépôts](https://wiki.jenkins.io/display/JENKINS/Set+GitBlitRepositoryBrowser+with+custum+settings+on+all+repos) — Ce script permet de mettre à jour le navigateur de dépôt. Il peut être adapté à n'importe quel autre navigateur, pas seulement gitblit ;
* [Mettre à jour les tâches Maven pour utiliser la tâche post-compilation afin de déployer les artefacts](https://wiki.jenkins.io/display/JENKINS/Update+maven+jobs+to+use+the+post+build+task+to+deploy+artifacts) — Ce script met à jour toutes les tâches Maven ayant un objectif de déploiement en installant et en activant l'étape post-compilation afin de déployer les artefacts à la fin de la compilation ;
* [Mettre à jour le navigateur SVN](https://wiki.jenkins.io/display/JENKINS/Update+SVN+Browser) ;
* [Effacer les espaces de travail de toutes les tâches](https://wiki.jenkins.io/display/JENKINS/Wipe+out+workspaces+of+all+jobs) — Ce script efface les espaces de travail de toutes les tâches sur votre serveur Jenkins ;
* [Effacer les espaces de travail pour un ensemble de tâches sur tous les nœuds](https://wiki.jenkins.io/display/JENKINS/Wipe+workspaces+for+a+set+of+jobs+on+all+nodes) — Le script efface les espaces de travail de certaines tâches sur tous les nœuds.

## Plugins permettant l'utilisation de Groovy

* [Plugin Config File Provider](https://plugins.jenkins.io/config-file-provider) Ajoute la possibilité de fournir des fichiers de configuration (c'est-à-dire settings.xml pour Maven, XML, Groovy, fichiers personnalisés, etc.) chargés via l'interface utilisateur Jenkins, qui seront copiés dans l'espace de travail de la tâche ;
* [Plugin Global Post Script](https://plugins.jenkins.io/global-post-script) — Exécute un script Groovy configuré globalement après chaque build de chaque tâche gérée par Jenkins. Cela est généralement utile lorsque vous devez effectuer une action en fonction d'un ensemble de paramètres partagés, par exemple déclencher des tâches en aval gérées par le même Jenkins ou des tâches distantes en fonction des paramètres transmis aux tâches paramétrées ;
* [Plugin Groovy](https://plugins.jenkins.io/groovy) ;
* [Plugin Groovy Postbuild](https://plugins.jenkins.io/groovy-postbuild) — Ce plugin exécute un script Groovy dans la JVM Jenkins. En général, le script vérifie certaines conditions et modifie le résultat de la compilation en conséquence, place des badges à côté de la compilation dans l'historique des compilations et/ou affiche des informations sur la page de résumé de la compilation ;
* [Plugin Groovy Remote Control](https://plugins.jenkins.io/groovy-remote) — Ce plugin fournit le récepteur Groovy Remote Control et permet de contrôler des applications externes à partir de Jenkins ;
* [Plugin Matrix Groovy Execution Strategy](https://plugins.jenkins.io/matrix-groovy-execution-strategy) — Un plugin permettant de décider de l'ordre d'exécution et des combinaisons valides des projets matriciels ;
* [Plugin Scriptler](https://plugins.jenkins.io/scriptler) — Scriptler vous permet de stocker/modifier des scripts Groovy et de les exécuter sur n'importe quel nœud... plus besoin de copier/coller le code Groovy.












