# Vue des Activités

<div class="couleur-introduction">
La vue Activité Blue Ocean affiche toutes les activités liées à un pipeline.
</div>

!!!  note "_Statut Blue Ocean_"
    Blue Ocean ne bénéficiera plus de mises à jour fonctionnelles. Blue Ocean continuera à fournir une visualisation facile à utiliser du Pipeline, mais ne sera plus amélioré. Il ne bénéficiera que de mises à jour sélectives pour les problèmes de sécurité importants ou les défauts fonctionnels.

## Barre de navigation

La vue Activité comprend la [barre de navigation](./blue-ocean-commencer.md#barre-de-navigation) standard en haut et une barre de navigation locale en dessous.

![Barres de navigation de la vue Activité](https://www.jenkins.io/doc/book/resources/blueocean/activity/navigation-bars.png)

La barre de navigation locale comprend :

*   **Pipeline Name** (Nom du Pipeline) : sélectionnez cette option pour afficher l'onglet Activité par défaut ;
* **Favorites Toggle** (Bascule Favoris) : sélectionnez l'icône Favoris ☆ à droite du nom du Pipeline pour ajouter une branche à la liste des favoris affichée dans la [liste des favoris du tableau de bord](./blue-ocean-tableau-de-bord.md#liste-des-favoris).

### Activité

L'onglet par défaut de la vue Activité affiche une liste des dernières exécutions terminées ou actives. Chaque ligne représente [l'état](./blue-ocean-tableau-de-bord.md#statut-dexécution) d'une exécution, son ID, les informations de validation et la date à laquelle elle s'est terminée.

![Affichage par défaut de l'onglet Activité](https://www.jenkins.io/doc/book/resources/blueocean/activity/overview.png)

* La sélection d'une exécution affiche les [détails de l'exécution du Pipeline](./blue-ocean-vue-detaillee-pipeline.md) afin de fournir une visualisation du pipeline ;
* Les exécutions actives peuvent être interrompues à partir de cette liste en sélectionnant l'icône **stop**, représentée par un ◾ dans un cercle ;
* Les exécutions terminées peuvent être réexécutées en sélectionnant l'icône de réexécution ↺;
* La liste peut être filtrée par branche à l'aide du menu déroulant « Branche » dans l'en-tête de la liste.  <img src="https://www.jenkins.io/doc/book/resources/blueocean/activity/branch-filter.png" alt="Filtre de branche dans la vue Activité de Blue Ocean" width="160" height="100" style="vertical-align: bottom;">

Cette vue ne permet pas de modifier les exécutions ni de les marquer comme favorites. Pour effectuer ces actions, sélectionnez l'onglet **Branches**.

#### Branches

L'onglet **Branches** affiche une liste de toutes les branches qui ont une exécution terminée ou active dans le Pipeline actuel. Chaque ligne de la liste correspond à une branche dans le contrôle de source, affichant [l'état général de la branche](./blue-ocean-tableau-de-bord.md#statut-dexécution) en fonction du statut de la dernière exécution, du nom de la branche, des informations de validation et de la date de fin de l'exécution.

[Onglet Branches de la vue Activité](https://www.jenkins.io/doc/book/resources/blueocean/activity/branches.png)

La sélection d'une branche affiche les [détails de l'exécution du Pipeline](./blue-ocean-vue-detaillee-pipeline.md) pour la dernière exécution terminée ou active de cette branche.

1. Les Pipelines dont la dernière exécution est terminée peuvent être relancés en sélectionnant l'icône **run** (exécution), représentée par un <i class="fa fa-play"></i> dans un cercle ;
    * Les exécutions actives peuvent être interrompues et affichent une icône **stop**, représentée par un cercle contenant un ◾;
1. En sélectionnant l'icône **history** (historique) <i class="fa fa-history"></i>, vous pouvez afficher l'historique des exécutions pour cette branche.
2. L'icône **edit** (édition), représentée par un <i class="fa fa-pencil"></i>, ouvre [l'éditeur de Pipeline](./blue-ocean-editeur-pipeline.md) pour cette branche.
3. L'icône **favorite** (favoris) ☆ ajoute une branche à votre liste de favoris sur le [tableau de bord](./blue-ocean-tableau-de-bord.md#liste-des-favoris). Sur le tableau de bord, une branche répertoriée dans les favoris affiche une étoile pleine ★. La désélection de l'étoile supprime la branche de la liste des favoris.

#### Pull Requests (Demandes d'extraction)

L'onglet Pull Requests (Demandes d'extraction) affiche une liste de toutes les demandes d'extraction pour le Pipeline actuel, qui ont une exécution terminée ou active. Chaque ligne de la liste correspond à une _pull request_ dans le contrôle de source, qui affiche le statut de la dernière exécution, l'ID de l'exécution, le résumé, l'auteur et la date d'achèvement de l'exécution.

![Vue des pull requests d'activité](https://www.jenkins.io/doc/book/resources/blueocean/activity/pull-requests.png)

Blue Ocean affiche les _pull requests_ et les branches séparément, mais les listes se comportent de manière similaire. La sélection d'une _pull request_ dans cette liste fait apparaître les [détails de l'exécution du Pipeline](./blue-ocean-vue-detaillee-pipeline.md) pour la dernière exécution terminée ou active de cette _pull request_.

* Les exécutions actives peuvent être interrompues à partir de cette liste en sélectionnant l'icône **stop**, représentée par un ◾ dans un cercle.
* Les _pull requests_ dont la dernière exécution est terminée peuvent être relancées en sélectionnant l'icône **run** (exécution), représentée par un <i class="fa fa-play"></i> dans un cercle.

La liste des _pull requests_ n'affiche pas [d'icônes d'état](./blue-ocean-vue-des-acitivites.md), et les _pull requests_ ne peuvent pas être modifiées ni marquées comme favorites.

!!! info " "
	Par défaut, lorsqu'une _pull request_ est fermée, Jenkins supprime le Pipeline de Jenkins, et les exécutions pour cette _pull request_ ne sont plus accessibles depuis Jenkins. Les pPpelines supprimés de cette manière devront être nettoyés à l'avenir. Cela peut être modifié en ajustant la configuration du travail du Pipeline multibranches sous-jacent. 

