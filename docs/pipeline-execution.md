# Exécution des Pipelines

## Exécution d'un pipeline

### Multibranche

Pour plus d'informations  [consultez la documentation Multibranch](./pipeline-multibranche).

### Paramètres

Pour plus d'informations [voir la documentation JenkinsFile](./pipeline-jenkinsfile.md#gestion-des-paramètres).

## Redémarrer ou relancer un Pipeline

Il existe plusieurs façons de relancer ou de redémarrer un Pipeline terminé.

### Rejouer

Pour plus d'informations, [consultez la documentation Replay](./pipeline-developpement.md#rejouer).

### Redémarrer à partir d'une étape

Vous pouvez redémarrer n'importe quel Pipeline déclaratif terminé à partir de n'importe quelle étape de niveau supérieur qui s'est exécutée dans ce Pipeline. Cela vous permet de réexécuter un Pipeline à partir d'une étape qui a échoué en raison de considérations transitoires ou environnementales, par exemple. Toutes les entrées du Pipeline seront identiques. Cela inclut les informations SCM, les paramètres de compilation et le contenu de tout appel d'étape de `stash` dans le Pipeline d'origine, le cas échéant.

#### Mode d'emploi

Aucune configuration supplémentaire n'est nécessaire dans le fichier Jenkinsfile pour vous permettre de redémarrer les étapes de vos Pipelines déclaratifs. Cette fonctionnalité fait partie intégrante des Pipelines déclaratifs et est disponible automatiquement.

##### Redémarrer à partir de l'interface utilisateur classique

Une fois votre Pipeline terminé, qu'il ait réussi ou échoué, vous pouvez vous rendre dans le panneau latéral d'exécution dans l'interface utilisateur classique et sélectionner « Redémarrer à partir de l'étape ».

![Barre latérale de l'interface utilisateur Jenkins Classic pour les options de construction  Exemple-Pipeline  Affichage: statut, modifications, sortie de la console, modifier les informations de construction, supprimer la construction n ° 1, calendrier, aperçu du pipeline, console de pipeline, redémarrer à partir de la scène, de la relecture, des étapes de pipeline, des espaces de travail et une version précédente.](https://www.jenkins.io/doc/book/resources/pipeline/restart-stages-sidebar.png)

Vous serez invité à choisir parmi une liste d'étapes de niveau supérieur qui ont été exécutées lors de l'exécution initiale, dans l'ordre dans lequel elles ont été exécutées. Les étapes qui ont été ignorées en raison d'un échec antérieur ne pourront pas être redémarrées, mais celles qui ont été ignorées parce qu'une condition `when` n'était pas remplie pourront l'être. L'étape parent d'un groupe d'étapes `parallel` ou d'un groupe de `stages` imbriquées à exécuter séquentiellement ne sera pas non plus disponible : seules les étapes de niveau supérieur sont autorisées.

![Jenkins Classic UI pour redémarrer la construction n ° 1 à partir d'une étape sélectionnée. La section affiche un menu déroulant intitulé «Nom de scène» avec l'option sélectionnée «Fetch» ​​et un bouton bleu intitulé «Exécuter» pour initier le redémarrage.](https://www.jenkins.io/doc/book/resources/pipeline/restart-stages-dropdown.png)

Une fois que vous avez choisi une étape à partir de laquelle redémarrer et cliqué sur « Soumettre », une nouvelle compilation, avec un nouveau numéro de compilation, sera lancée. Toutes les étapes précédant l'étape sélectionnée seront ignorées, et le Pipeline commencera à s'exécuter à partir de l'étape sélectionnée. À partir de ce moment, le Pipeline fonctionnera normalement.

##### Redémarrer à partir de l'interface utilisateur Blue Ocean

Le redémarrage des étapes peut également être effectué dans l'interface utilisateur Blue Ocean. Une fois votre Pipeline terminé, qu'il ait réussi ou échoué, vous pouvez cliquer sur le nœud qui représente l'étape. Vous pouvez ensuite cliquer sur le lien `Restart` (Redémarrer) pour cette étape.

![Visualisation du Pipeline Jenkins dans le tableau de bord Blue Ocean montrant les étapes d'une exécution Pipeline réussie. Les étapes incluent le démarrage, la construction, certains tests, les tests de navigateur (avec Chrome, Firefox, Internet Explorer et Safari), tester d'autres choses, une analyse statique et un déploiement. Chaque étape est marquée d'une coche verte indiquant une réussite. En bas, il y a une étape de déploiement/finale avec un journal de messages et des options pour redémarrer les journaux de déploiement et de téléchargement.](https://www.jenkins.io/doc/book/resources/pipeline/pipeline-restart-stages-blue-ocean.png)

!!! note "<i>Statut de Blue Ocean</i>"

    Blue Ocean ne bénéficiera plus de mises à jour fonctionnelles. Blue Ocean continuera à fournir une visualisation facile à utiliser du Pipeline, mais ne sera plus amélioré. Il ne bénéficiera que de mises à jour sélectives pour les problèmes de sécurité importants ou les défauts fonctionnels.

    D'autres options de visualisation du Pipeline, telles que les plugins [Pipeline:  Stage View](https://plugins.jenkins.io/pipeline-stage-view/) et [Pipeline Graph View,](https://plugins.jenkins.io/pipeline-graph-view/) sont disponibles et offrent certaines de ces fonctionnalités. Bien qu'elles ne remplacent pas complètement Blue Ocean, les contributions de la communauté sont encouragées pour le développement continu de ces plugins.

#### Conserver une `stash` (réserve) pour une utilisation avec des étapes redémarrées

Normalement, lorsque vous exécutez l'étape de `stash` (mise en cache) dans votre Pipeline, le cache d'artefacts résultant est effacé à la fin du Pipeline, quel que soit le résultat de celui-ci. Étant donné que les artefacts en `stash` (cache) ne sont pas accessibles en dehors de l'exécution du Pipeline qui les a créés, cela n'a pas entraîné de restrictions d'utilisation. Mais avec le redémarrage déclaratif des étapes, vous pouvez souhaiter pouvoir `unstash` (déstocker) les artefacts d'une étape qui s'est exécutée avant l'étape à partir de laquelle vous redémarrez.

Pour ce faire, une propriété de tâche vous permet de configurer le nombre maximal d'exécutions terminées dont les artefacts stockés doivent être conservés pour être réutilisés dans une exécution redémarrée. Vous pouvez spécifier un nombre compris entre 1 et 50 pour le nombre d'exécutions à conserver.

Cette propriété de tâche peut être configurée dans la section `options` de votre Pipeline déclaratif, comme indiqué ci-dessous :

``` groovy
options {
    preserveStashes() // (1)
    // ou
    preserveStashes(buildCount: 5) // (2)
}
```

1. Le nombre par défaut d'exécutions à conserver est 1, soit la dernière build terminée.
2. Si un nombre compris entre 1 et 50 est spécifié pour `buildCount`, le Pipeline échouera avec une erreur de validation.

Lorsqu'un Pipeline est terminé, il vérifie si les artefacts stockés des exécutions précédemment terminées doivent être supprimés.

## Planification des tâches dans Jenkins

La fonction de planification vous permet de programmer l'exécution automatique de tâches pendant les heures creuses ou les périodes d'inactivité. La planification des tâches peut vous aider à adapter votre environnement à mesure que l'utilisation de Jenkins augmente. Cette vidéo vous donne un aperçu de la fonction de planification et de ses différentes options de configuration.

<iframe width="800" height="420" src="https://www.youtube.com/embed/JhvVJtYFUm0" title="How to Schedule a Jenkins Job to Run Every Hour" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
