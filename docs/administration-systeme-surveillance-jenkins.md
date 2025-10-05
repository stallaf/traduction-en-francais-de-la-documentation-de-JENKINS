# Surveillance de Jenkins

!!! info "Information"
    Cette page est en cours de développement, du contenu supplémentaire sera bientôt ajouté. Consultez le [WEBSITE-738](https://issues.jenkins.io/browse/WEBSITE-738) EPIC pour les tâches liées à cette page, vos contributions sont les bienvenues !

## Surveillance avec Datadog

* [Plugin Datadog pour Jenkins](https://plugins.jenkins.io/datadog) ;
* [Plugin Metrics-Datadog pour Jenkins](https://plugins.jenkins.io/metrics-datadog) ;
* [Jenkins sur Datadog](https://www.datadoghq.com/blog/monitor-jenkins-datadog).

## Surveillance avec Newrelic

* [Jenkins sur Newrelic](https://opensource.newrelic.com/projects/newrelic/nr-jenkins-plugin) ;
* [Développement d'un pipeline d'intégration Jenkins pour l'infrastructure New Relic Intégrations sur hôte](https://newrelic.com/blog/best-practices/how-use-jenkins-integration-tests).

## Surveillance avec Prometheus et Grafana

* [Plugin Prometheus pour Jenkins](https://plugins.jenkins.io/prometheus) ;
* [Blog pratique sur Medium](https://medium.com/@eng.mohamed.m.saeed/monitoring-jenkins-with-grafana-and-prometheus-a7e037cbb376).

## Surveillance avec JavaMelody

* [Plugin de surveillance pour Jenkins](https://plugins.jenkins.io/monitoring).

## Autres plugins de surveillance

* [ Versions Node Monitors](https://plugins.jenkins.io/versioncolumn) ;
* [Surveillance des agents pour les nœuds Unix](https://plugins.jenkins.io/systemloadaverage-monitor) ;
* [Surveillance des tâches/files d'attente/esclaves](https://plugins.jenkins.io/jqs-monitoring).

## Fil de ping

Jenkins installe un « fil de ping » sur chaque connexion à distance, telle que les connexions contrôleur/agent, quel que soit son mécanisme de transport (SSH, JNLP, etc.). Le niveau inférieur du protocole distant Jenkins est un protocole orienté messages, et un thread ping envoie périodiquement un message ping auquel le destinataire répond. Le thread ping mesure le temps nécessaire à la réception de la réponse et, si celui-ci est trop long (actuellement [4 minutes](https://github.com/jenkinsci/remoting/blob/master/src/main/java/hudson/remoting/Launcher.java), mais configurable), il suppose que la connexion a été perdue et lance la fermeture formelle.

Cela permet d'éviter un blocage infini, car certains modes de défaillance du réseau ne peuvent pas être détectés autrement. Le délai d'expiration est également fixé à une valeur suffisamment longue pour qu'une augmentation temporaire de la charge ou une longue pause de collecte des déchets ne déclenche pas la fermeture.

Le thread ping est installé à la fois sur le contrôleur et l'agent ; chaque côté envoie un ping à l'autre et tente de détecter le problème de son côté.

Le délai d'expiration du thread ping est signalé via` java.util.logging`. De plus, le contrôleur signalera également cette exception dans le journal de lancement de l'agent. Notez que certains lanceurs d'agents, notamment les agents SSH, écrivent toutes les sorties stdout/stderr de la JVM de l'agent dans ce même fichier journal, vous devez donc être prudent. Voir [JENKINS-25695](https://issues.jenkins.io/browse/JENKINS-25695).

## Désactivation du thread ping

Parfois, par exemple [pour diagnostiquer un problème de perte de connexion de l'agent](https://wiki.jenkins.io/display/JENKINS/Remoting+issue), vous pouvez vouloir désactiver le thread ping. Cela doit être fait à deux endroits.

Désactivez le contrôleur pour qu'il n'envoie plus de ping aux agents en définissant `hudson.slaves.ChannelPinger.pingIntervalSeconds` sur le contrôleur JVM à -1.

Vous pouvez également modifier la valeur en mémoire pour un Jenkins en cours d'exécution, si vous ne souhaitez pas redémarrer Jenkins.

Définissez `pingIntervalSeconds` et `pingTimeoutSeconds` sur le contrôleur JVM sur -1 :

``` groovy title="GROOVY"
Jenkins.instance.injector.getInstance(hudson.slaves.ChannelPinger.class).@pingIntervalSeconds = -1
Jenkins.instance.injector.getInstance(hudson.slaves.ChannelPinger.class).@pingTimeoutSeconds = -1
```

Ceci n'affectera que les agents nouvellement connectés. Les agents déjà connectés continueront d'exécuter les pings.

Pour désactiver les pings des agents vers le contrôleur, la propriété système :

``` bash title="BASH"
-Dhudson.remoting.Launcher.pingIntervalSec=-1
```

doit être définie sur agents. La procédure à suivre dépend du lanceur.

