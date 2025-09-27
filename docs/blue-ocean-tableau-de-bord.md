# Tableau de bord

<div class="couleur-introduction">
Le « Tableau de bord » de Blue Ocean est la vue par défaut qui s'affiche à l'ouverture de Blue Ocean. Il présente une vue d'ensemble de tous les projets Pipeline configurés sur un contrôleur Jenkins.
</div>
	
!!! info "_Statut de Blue Ocean_"
    Blue Ocean ne bénéficiera plus de mises à jour fonctionnelles. Blue Ocean continuera à fournir une visualisation facile à utiliser de Pipeline, mais ne sera plus amélioré. Il ne bénéficiera que de mises à jour sélectives pour les problèmes de sécurité importants ou les défauts fonctionnels.

Le tableau de bord se compose d'une [barre de navigation](#barre-de-navigation) bleue en haut, de la [liste des Pipelines](#liste-de-pipelines) et de la [liste des favoris](#liste-de-favoris).

![Aperçu du tableau de bord Blue Ocean](https://www.jenkins.io/doc/book/resources/blueocean/dashboard/overview.png)

## Barre de navigation

Le tableau de bord comprend une [barre de navigation](./blue-ocean-commencer.md#barre-de-navigation) bleue située en haut de l'interface.

La barre est divisée en deux sections :

* Une section commune en haut ;
* Une section contextuelle en dessous ;
    * La section contextuelle change en fonction de la page Blue Ocean que vous consultez.

Lorsque vous consultez le tableau de bord, la section contextuelle de la barre de navigation comprend :

* **Search pipeline** (Rechercher des Pipelines) : ce champ permet aux utilisateurs de filtrer la [liste des Pipelines](#liste-de-pipelines) afin qu'elle corresponde au texte que vous saisissez dans ce champ.
* **New Pipeline** (Nouveau pipeline) : cette option lance le processus de [création d'un Pipeline](./blue-ocean-creation-pipeline.md).

## Liste de Pipelines

La liste des **Pipelines** est la liste par défaut du tableau de bord. C'est la seule liste affichée lors du premier accès à Blue Ocean.

La liste affiche l'état général de chaque Pipeline configuré dans le contrôleur Jenkins. La liste peut inclure d'autres éléments configurés dans le contrôleur Jenkins. Les informations suivantes sont affichées pour chaque Pipeline répertorié :

* Le **NOM** de l'élément ;
* La **[SANTÉ](#sante-du-pipeline)** de l'élément ;
* Le nombre de **BRANCHES** et de _pull requests_ (**PR**) qui réussissent ou échouent dans le référentiel de contrôle de source du Pipeline ;
* Une ☆, indiquant si l'élément a été ajouté manuellement à votre [liste actuelle de favoris](#liste-de-favoris) Jenkins.

Sélectionner le symbole ☆ d'un élément permet de basculer entre :

* L'ajout de la branche par défaut du référentiel de l'élément à votre liste de favoris, indiquée par un symbole ★ plein ;
* La suppression de la branche par défaut de l'élément de cette liste, indiquée par un symbole ☆ en contour.

La sélection d'un élément dans la liste Pipelines affiche la vue Activité de cet élément.

## Liste des favoris

La liste des favoris apparaît au-dessus de la [liste Pipelines](#liste-de-pipelines) par défaut du tableau de bord lorsqu'au moins un Pipeline/élément est présent dans votre liste de favoris.

Cette liste fournit des informations et des actions clés pour un sous-ensemble essentiel de vos éléments accessibles dans la [liste Pipelines](#liste-de-pipelines). Ces informations clés comprennent [l'état d'exécution](#etat-dexecution) actuel d'un élément et la branche de son référentiel, le nom de la branche, la partie initiale du hachage de validation et la dernière heure d'exécution. Cette liste comprend également des options permettant d'exécuter ou de réexécuter l'élément sur la branche indiquée.

Vous ne devez ajouter un élément ou une branche à votre liste de favoris que s'il nécessite une surveillance régulière. L'ajout d'une branche spécifique d'un élément à votre liste de favoris peut être effectué via la [Vue Activité](./blue-ocean-activite.md) de l'élément.

Blue Ocean ajoute automatiquement les branches et les _PRs_ à cette liste lorsqu'elles incluent une exécution contenant des modifications que vous avez effectuées.

Vous pouvez également supprimer manuellement des éléments de votre liste de favoris en désélectionnant le ★ dans cette liste. Lorsqu'il n'y a plus d'éléments dans cette liste, celle-ci est supprimée du tableau de bord. Sélectionnez le ★ favori d'un élément pour faire réapparaître la liste dans votre tableau de bord.

La sélection d'un élément dans la liste des favoris ouvre les [détails de l'exécution du Pipeline](./blue-ocean-details-execution-pipeline.md ) pour la dernière exécution sur la branche ou la _PR_ indiquée.

### Icônes d'état

Blue Ocean représente l'état général d'un pipeline/élément ou d'une de ses branches à l'aide d'icônes météo. Ces icônes changent en fonction du nombre de compilations récentes qui ont été validées.

Les icônes d'état du tableau de bord représentent l'état général du Pipeline, tandis que celles de [l'onglet Branches de la vue Activité](./blue-ocean-activite.md#branches) représentent l'état général de chaque branche.

|       Icône         | 	        Santé        |
| --------------------| ------------------------ |
| ![Sunny](https://www.jenkins.io/doc/book/resources/blueocean/icons/weather/sunny.svg)| <br><br>**Ensoleillé, plus de 80 % des exécutions réussies** |
| ![Partially Sunny](https://www.jenkins.io/doc/book/resources/blueocean/icons/weather/partially-sunny.svg) | <br><br>**Partiellement ensoleillé, 61 % à 80 % des exécutions réussies** |
| ![Cloudy](https://www.jenkins.io/doc/book/resources/blueocean/icons/weather/cloudy.svg) | <br>**Nuageux, 41 % à 60 % des exécutions réussies** |
| ![Raining](https://www.jenkins.io/doc/book/resources/blueocean/icons/weather/raining.svg) | <br>*Pluvieux, 21 % à 40 % des exécutions réussies** |
| ![Storm](https://www.jenkins.io/doc/book/resources/blueocean/icons/weather/storm.svg) | <br>**Tempête, moins de 21 % des exécutions réussies** |
	
### Statut d'exécution

Blue Ocean représente le statut d'exécution d'un pipeline/élément ou d'une de ses branches à l'aide d'un ensemble cohérent d'icônes.

|       Icône         | 	        Statut        |
| --------------------| ------------------------ |
| ![En cours](https://www.jenkins.io/doc/book/resources/blueocean/dashboard/status-in-progress.png) | <br>**En cours** |
| ![Réussi](https://www.jenkins.io/doc/book/resources/blueocean/dashboard/status-passed.png) | <br>**Réussi** |
| ![Instable](https://www.jenkins.io/doc/book/resources/blueocean/dashboard/status-unstable.png) | <br>**Instable** |
| ![Échec](https://www.jenkins.io/doc/book/resources/blueocean/dashboard/status-failed.png) | <br>**Échec** |
| ![Abandonné](https://www.jenkins.io/doc/book/resources/blueocean/dashboard/status-aborted.png) | <br>**Abandonné** |
	

