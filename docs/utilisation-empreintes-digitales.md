# Empreintes Digitales

## Suivi de l'utilisation des fichiers dans les tâches Jenkins à l'aide d'empreintes digitales

Lorsque vous avez des projets interdépendants sur Jenkins, il devient souvent difficile de savoir quelle version d'un fichier est utilisée par quelle version d'une dépendance sur ce fichier. Jenkins prend en charge **l'empreinte digitale des fichiers** pour suivre les dépendances.

Par exemple, supposons que vous ayez un projet _TOP_ (supérieur) qui dépend d'un projet _MIDDLE_ (moyen), lequel dépend à son tour d'un projet _BOTTOM_ (inférieur). Vous travaillez sur le projet BOTTOM. L'équipe TOP a signalé que le fichier `bottom.jar` qu'elle utilise provoque une exception NPE, que vous (membre de l'équipe BOTTOM) pensiez avoir corrigée dans BOTTOM #32. Jenkins peut vous indiquer quelles versions MIDDLE et TOP utilisent (ou n'utilisent pas) votre fichier `bottom.jar` #32.

## Comment le configure ?

Pour que cela fonctionne, tous les projets pertinents doivent être configurés pour **enregistrer les empreintes digitales** des fichiers JAR (dans le cas ci-dessus, `bottom.jar`).

Par exemple, si vous souhaitez simplement suivre les versions BOTTOM utilisées par les versions TOP, configurez TOP et BOTTOM pour enregistrer l'empreinte digitale de `bottom.jar`. Si vous souhaitez également savoir quelles versions MIDDLE utilisent quel `bottom.jar`, configurez également MIDDLE.

Étant donné que l'enregistrement des empreintes digitales est une opération bon marché, la chose la plus simple à faire est d'enregistrer aveuglément toutes les empreintes digitales comme ceci :: 

1. fichiers jar générés par votre projet ;
2. fichiers jar utilisés par votre projet.

L'utilisation du disque est davantage influencée par le nombre de fichiers dont l'empreinte est enregistrée que par la taille des fichiers ou le nombre de versions dans lesquelles ils sont utilisés. Par conséquent, à moins de disposer d'un espace disque important, il est déconseillé d'enregistrer l'empreinte de tous les fichiers `**/*`.

### Configurer une tâche pour enregistrer les empreintes digitales d'un fichier ou un ensemble de fichiers

Accédez à votre projet, cliquez sur **-Configure-** (Configurer) dans la barre de navigation de gauche, puis faites défiler vers la section **_Post-build Actions -** (Actions post-construction) de la tâche.

Cliquez sur le bouton pour ajouter une **_Post-build_** (Action post-construction).

Sélectionnez **_Record fingerprints of files to track usage_** (Enregistrer les empreintes digitales des fichiers pour suivre l'utilisation).

Les champs de configuration des actions post-construction vous offrent une option de modèle pour faire correspondre les fichiers que vous souhaitez identifier, ainsi que plusieurs cases à cocher pour effectuer l'identification de vos fichiers.

Le type de tâche Maven le fait automatiquement pour ses dépendances et artefacts.

## Comment ça marche ?

L'empreinte digitale d'un fichier est simplement une somme de contrôle MD5. Jenkins gère une base de données des sommes de contrôle MD5 et, pour chaque somme de contrôle MD5, Jenkins enregistre les versions des projets qui l'ont utilisée. Cette base de données est mise à jour à chaque fois qu'une version est exécutée et que les fichiers sont marqués d'une empreinte.

Pour éviter une utilisation excessive du disque, Jenkins ne stocke pas le fichier réel. Au lieu de cela, il stocke simplement les sommes de contrôle MD5 et leurs usages. Ces fichiers peuvent être vus dans :

**$JENKINS_HOME/fingerprints**

Les plugins peuvent stocker des informations supplémentaires dans ces enregistrements. Par exemple, le plugin [Deployment Notification Plugin](https://plugins.jenkins.io/deployment-notification/) suit les fichiers déployés sur les serveurs via chef/puppet grâce à des empreintes digitales.

## Comment puis-je l'utiliser ?

Voici quelques scénarios typiques qui bénéficient de cette fonctionnalité :

Vous développez le projet BOTTOM et vous voulez savoir qui utilise BOTTOM # 13 dans lequel se construit : 

1. Allez dans la page de  construction BOTTOM # 13.
2. Cliquez sur l'icône "empreinte digitale" de `bottom.jar` dans les artefacts de construction.
3. Vous verrez tous les projets et builds qui l'utilisent.

Vous développez le projet TOP et vous souhaitez savoir quelle version de `bottom.jar` et `middle.jar` vous utilisez dans TOP #10.

1. Accédez à la page de construction TOP # 10.
2. Cliquez sur "Voir les empreintes digitales". 
3. Vous verrez tous les fichiers empreintes digitales dans TOP # 10, ainsi que leur origine.

Vous avez le projet TOP qui construit un fichier jar. Vous avez également le projet TOP-TEST qui s'exécute après le projet TOP et effectue des tests d'intégration approfondis sur les derniers bits TOP. Vous souhaitez connaître les résultats des tests de TOP #7.

1. Accédez à la page de construction TOP # 7.
2. Cliquez sur l'icône "empreinte digitale" de `top.jar` dans les artefacts de construction. 
3. Vous verrez toutes les constructions TOP-TEST qui l'ont utilisé. 
4. Cliquez dessus et vous serez transporté sur la page de construction TOP-TEST appropriée, qui vous montrera les rapports de test. 
5. S'il n'y a pas de versions TOP-TEST affichées, cela signifie que la construction de TOP-TEST ne s'est pas déroulée avec TOP # 7. Peut-être que cela a sauté TOP # 7 (cela peut se produire s'il y a beaucoup de constructions TOP dans un court laps de temps), ou peut-être qu'une nouvelle construction de TOP-TEST est en cours.