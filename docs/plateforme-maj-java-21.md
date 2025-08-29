# Mise à niveau vers Java 17

Lors de la mise à niveau du JVM utilisé pour exécuter Jenkins de Java 17 à Java 21, il y a des détails que vous devez connaître et des précautions que vous devez prendre. REMARQUE : Java 21 est pris en charge à partir de LTS 2.426.1 et Jenkins Weekly 2.419.

## Sauvegarder Jenkins

Comme pour toute mise à niveau, nous recommandons : 

1.  [Sauvegarde de JENKINS_HOME](./administration-sauvegarde.md#jenkins-home).
2.  Test de la mise à niveau avec votre sauvegarde.
3.  Une fois tous les tests requis réussis, procédez à la mise à niveau sur votre contrôleur de production.

## Améliorez la version Jenkins Java en Java 21

_Mise à niveau de la version Jenkins Java de 17 à 21_

<iframe width="800" height="420" src="https://www.youtube.com/embed/8xQVGpWeIe0" title="Upgrading Jenkins Java Version From 17 to 21" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Pour vérifier la version Java actuellement utilisée dans votre contrôleur : 

1. Accédez à **_Manage Jenkins_** (Gestion de Jenkins), puis sélectionnez **_System Information_** (Informations Système) dans la section **_Status Information_** (Information du Statut).
2. Dans l'onglet **_System Properties_** (Propriétés Système), localisez `java.runtime.version` et affichez la valeur cachée pour voir votre version Java actuelle.

Pour vérifier la version Java actuellement utilisée dans votre agent : 

1. Sélectionnez un nom d'agent dans le widget **_Build Executor Status_** (Etat Exécuteur de Build), puis sélectionnez  **_System Information_** (Informations Système). 
2. Localisez le `java.runtime.version` et révélez la valeur cachée pour afficher la version Java actuelle.

Le script `checkNodes` détermine quelle version de Java exécute le processus du contrôleur, ainsi que la version de l'agent associé à ce contrôleur. Le script vérifie ensuite la version de Java qui est sur l'agent. Lorsque ces valeurs sont renvoyées, `OK` signifie que la version Java Agent correspond à la version Java du contrôleur. Sinon, le résultat affiche la version Java attendue et la version trouvée à la place.

Pour exécuter le script `checkNodes` : 

1. Accédez à **_Manage Jenkins_** (Gestion de Jenkins) et sélectionnez **_Script Console_** (Console de Script) dans la section **_Tools and Actions_** (Outils et Actions). 
2. Copiez le script suivant dans la zone de texte vide et sélectionnez **_Run_** (Exécuter) : 

``` java
/*** BEGIN META {
 "name" : "Check Nodes Version",
 "comment" : "Check the .jar version and the java version of the Nodes against the Master versions",
 "parameters" : [ ],
 "core": "1.609",
 "authors" : [
 { name : "Allan Burdajewicz" }
 ]
 } END META**/

import hudson.remoting.Launcher
import hudson.slaves.SlaveComputer
import jenkins.model.Jenkins

def expectedAgentVersion = Launcher.VERSION
def expectedJavaVersion = System.getProperty("java.version")
println "Master"
println " Expected Agent Version = '${expectedAgentVersion}'"
println " Expected Java Version = '${expectedJavaVersion}'"
Jenkins.instance.getComputers()
        .findAll { it instanceof SlaveComputer }
        .each { computer ->
    println "Node '${computer.name}'"
    if (!computer.getChannel()) {
        println " is disconnected."
    } else {
        def isOk = true
        def agentVersion = computer.getSlaveVersion()
        if (!expectedAgentVersion.equals(agentVersion)) {
            println " expected agent version '${expectedAgentVersion}' but got '${agentVersion}'"
            isOk = false
        }
        def javaVersion = computer.getSystemProperties().get("java.version")
        if (!expectedJavaVersion.equals(javaVersion)) {
            println " expected java version '${expectedJavaVersion}' but got '${javaVersion}'"
            isOk = false
        }

        if(isOk) {
            println " OK"
        }
    }
}
return;
```

## Passez à Java 21 sur votre contrôleur Jenkins

Les étapes suivantes sont issues de la vidéo liée en haut de cette page. 

1. Arrêtez le contrôleur Jenkins avec `systemctl stop jenkins`.
2. Installez la version Java correspondante avec `dnf -y install temurin-21-jdk` ou avec le gestionnaire de paquets utilisé par votre système.
3. Vérifiez la version Java avec `java -version`.
4. Modifiez la version Java par défaut du système en exécutant `update-alternatives --config java`, puis entrez le numéro correspondant à Java 21, par exemple `2` si c'est l'option correcte.
5. Redémarrez Jenkins avec `systemctl restart jenkins`.

## Mettre à niveau Jenkins

1.  [Sauvegarde de JENKINS_HOME](./administration-sauvegarde.md#jenkins-home).
2. Arrêtez le contrôleur Jenkins. 
3. Mettez à niveau le JVM sur lequel Jenkins fonctionne. 
    * Utilisez un gestionnaire de packages pour installer le nouveau JVM ;
    * Assurez-vous que le JVM par défaut est la version nouvellement installée ; 
        * Si ce n'est pas le cas, exécutez `systemctl edit jenkins` et définissez la variable d'environnement `JAVA_HOME ` ou la variable d'environnement `JENKINS_JAVA_CMD `. 
4. Mettez à niveau Jenkins vers la version la plus récente. 
    * La façon dont vous mettez à niveau Jenkins dépend de votre méthode d'installation d'origine Jenkins. 
!!! Success 
    Nous vous recommandons d'utiliser le gestionnaire de paquets de votre système (comme `apt` ou `yum`). 
5. Validez la mise à niveau pour confirmer que tous les plugins et travaux sont chargés. 
6. Mettez à niveau les plugins requis.

Lors de la mise à niveau de la version Java pour Jenkins et le JVM, il est important de mettre à niveau tous les plugins qui prennent en charge Java 17. Les mises à niveau du plugin assurent la compatibilité avec les versions les plus récentes de Jenkins. 

!!! info 
    Si vous découvrez un problème auparavant non déclaré, veuillez nous le faire savoir. Reportez-vous à notre [documentation sur le signalement des problèmes](https://www.jenkins.io/participate/report-issue/#issue-reporting) pour obtenir des conseils.

La façon dont vous améliorez Jenkins dépend de votre méthode d'installation d'origine Jenkins. 
Nous vous recommandons d'utiliser le gestionnaire de packages de votre système (comme APT ou YUM). 

## Version JVM sur les agents

Tous les agents doivent fonctionner sur la même version de JVM que le contrôleur, en raison de la manière dont les contrôleurs et les agents communiquent. Si vous mettez à niveau votre contrôleur Jenkins pour qu'il fonctionne sous Java 17, vous devez mettre à niveau la JVM sur vos agents.

La validation de la version de chaque agent peut être effectuée à l'aide du plugin [Versions Node Monitors](https://plugins.jenkins.io/versioncolumn). Ce plugin fournit des informations sur la version JVM de chaque agent sur l'écran de gestion des nœuds de votre contrôleur Jenkins. Ce plugin peut également être configuré pour déconnecter automatiquement tout agent dont la version JVM est incorrecte.

## Mise à niveau vers Java 21 sur les agents

Les étapes suivantes sont issues de la vidéo liée en haut de cette page. 

1. Dans la ligne de commande, connectez-vous à l'agent.
2. Entrez `dnf -y install temurin-21-jdk` ou utilisez la commande appropriée pour votre gestionnaire de paquets.
3. Vérifiez votre version Java à l'aide de `java -version`.
4. Modifiez la version Java par défaut à l'aide de la commande `update-alternatives --config java`, puis entrez la sélection correspondant à Java 21.
5. Vérifiez que la version Java a été mise à jour à l'aide de la commande` java -version`.
6. Dans la page de l'agent de votre contrôleur Jenkins, sélectionnez **_Disconnect_** (Déconnecter).
7. Après avoir déconnecté l'agent, reconnectez-le en sélectionnant **_Bring this node back online_** (Remettre ce nœud en ligne), puis en sélectionnant **_Lauch agent_** (Lancer l'agent).
