# Travailler avec des Projets

Jenkins utilise des projets (également appelés « tâches ») pour effectuer son travail. Les projets sont définis et exécutés par les utilisateurs de Jenkins. Jenkins propose plusieurs types de projets, notamment :

* [Pipeline](./pipeline-presentation.md) ;
* [Multibranch Pipeline](./pipeline-multibranches.md) ;
* [Organization folders](./pipeline-multibranches.md#dossiers-organisation) ;
* Freestyle ;
* [Multi-configuration (matrix)](https://plugins.jenkins.io/matrix-project) ;
* [Maven](https://plugins.jenkins.io/maven-plugin) ;
* [External job](https://plugins.jenkins.io/external-monitor-job).

Darin Pope présente un résumé des différences entre les projets Pipeline et les projets Freestyle dans cette vidéo.

_Comparaison entre les projets Pipeline et Freestyle_
<iframe width="800" height="420" src="https://www.youtube.com/embed/IOUm1lw7F58" title="What Is The Difference Between Freestyle and Pipeline in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Copier des projets

Copiez un projet existant en cliquant sur le lien « New Item » (Nouvel élément) dans le panneau latéral. Entrez le nom du projet de destination dans le champ « Item name » (Nom de l'élément). Insérez le nom du projet source dans le champ « Copy from » (Copier depuis). Le projet existant sera copié dans un nouveau projet portant le nom saisi dans le champ « Nom de l'élément ».

_Copier un projet_
<iframe width="800" height="420" src="https://www.youtube.com/embed/MNzNPCJJqaI" title="How To Clone a Jenkins Job" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Renommer des projets

Renommez un projet existant en cliquant sur l'action « Renamme » (Renommer) sur la page du projet. Lorsqu'un travail est renommé, les autres travaux qui font référence à ce travail par son nom doivent être mis à jour pour correspondre au nouveau nom.

_Renommer un projet_
<iframe width="800" height="420" src="https://www.youtube.com/embed/zO3xnCwbv_c" title="Rename a Job in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Déplacer des projets

Déplacez un projet existant vers un autre dossier en cliquant sur l'action « Move » (Déplacer) sur la page du projet. Lorsqu'un travail est déplacé, les autres travaux qui font référence à ce travail par son nom doivent être mis à jour pour correspondre au nouveau nom.

_Déplacer un projet_
<iframe width="800" height="420" src="https://www.youtube.com/embed/Mof_YRGZLd8" title="Move a Jenkins Job to a Folder" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


