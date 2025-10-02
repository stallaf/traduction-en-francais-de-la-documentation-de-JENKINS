# Script Groovy Hook

!!! warning "Avertissement"
    Cette section est en cours d'élaboration. Vous souhaitez contribuer ? Consultez le [canal Gitter jenkinsci/docs](https://app.gitter.im/#/room/#jenkins/docs:matrix.org). Pour découvrir d'autres façons de contribuer au projet Jenkins, consultez cette [page sur la participation et la contribution](https://www.jenkins.io/participate).

    À plusieurs endroits dans Jenkins, une série de « scripts hook » sont exécutés pour permettre à certaines actions de se produire en réaction à des événements clés.

Ces scripts sont écrits en Groovy et s'exécutent dans la même JVM que Jenkins, ce qui leur permet d'accéder pleinement au modèle de domaine de Jenkins. Pour un hook HOOK donné, les emplacements suivants sont recherchés :

* `WEB-INF/HOOK.groovy dans jenkins.war` ;
* `WEB-INF/HOOK.groovy.d/*.groovy` dans l'ordre lexical dans `jenkins.war` ;
* `$JENKINS_HOME/HOOK.groovy` ;
* `$JENKINS_HOME/HOOK.groovy.d/*.groovy` dans l'ordre lexical.

`HOOK.groovy.d `est adapté pour éviter les conflits : plusieurs entités peuvent insérer des éléments dans le _hook_ sans craindre d'écraser le code des autres.

Les événements suivants utilisent ce mécanisme en remplaçant `HOOK` dans `HOOK.groovy.d` ou `HOOK.groovy` par l'un des types mentionnés ci-dessous :

* **init** : script post-initialisation ;
* **boot-failure** : hook d'échec de démarrage.

## Script post-initialisation (hook init)

Vous pouvez créer un fichier de script Groovy `$JENKINS_HOME/init.groovy`, ou tout autre fichier `.groovy` dans le répertoire `$JENKINS_HOME/init.groovy.d/`, afin d'exécuter certaines tâches supplémentaires immédiatement après le démarrage de Jenkins. Les scripts Groovy sont exécutés à la fin de l'initialisation de Jenkins. Ce script peut accéder aux classes de Jenkins et à tous les plugins. Vous pouvez par exemple écrire quelque chose comme :

``` groovy
import jenkins.model.Jenkins;

// démarrer dans un état qui n'effectue aucune compilation.
Jenkins.instance.doQuietDown();
```

La sortie est consignée dans le fichier journal Jenkins. Pour les utilisateurs basés sur Debian, il s'agit de /var/log/jenkins/jenkins.log

!!! info " "
    Si vous utilisez l'image Docker Jenkins, `$JENKINS_HOME/init.groovy.d/` n'existe pas. Vous devez plutôt placer les fichiers de script Groovy dans `/usr/share/jenkins/ref/init.groovy.d/`, car ce répertoire est copié au démarrage du conteneur Docker. Pour plus de détails, consultez la [documentation Installation d'autres outils](https://github.com/jenkinsci/docker?tab=readme-ov-file#installing-more-tools).

## _Hook_ d'échec de démarrage

Lorsque Jenkins rencontre un problème fatal lors du démarrage, il invoque le script _hook_ « boot-failure » pour permettre la prise de mesures correctives automatiques (telles que la notification d'une personne, le déclenchement d'alertes, le redémarrage, etc.

Ces scripts obtiennent la cause du problème sous la forme de la variable « exception » lorsqu'ils sont exécutés.
