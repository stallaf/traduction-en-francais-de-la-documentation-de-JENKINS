# Editeur Pipeline

<div class="couleur-introduction">
L'éditeur Blue Ocean Pipeline est un moyen simple pour tout le monde de se lancer dans la création de Pipelines dans Jenkins. Il est également idéal pour les utilisateurs actuels de Jenkins qui souhaitent commencer à adopter Pipeline.
</div>

!!! info "_Statut de Blue Ocean_"
    Blue Ocean ne bénéficiera plus de mises à jour fonctionnelles. Blue Ocean continuera à fournir une visualisation facile à utiliser de Pipeline, mais ne sera plus amélioré. Il ne bénéficiera que de mises à jour sélectives pour les problèmes de sécurité importants ou les défauts fonctionnels.

    Le [générateur d'extraits de syntaxe Pipeline](./pipeline-commencer.md#générateur-dextraits) aide les utilisateurs à définir les étapes Pipeline avec leurs arguments. Il s'agit de l'outil privilégié pour la création de pipelines Jenkins, car il fournit une aide en ligne pour les étapes Pipeline disponibles dans votre contrôleur Jenkins. Il utilise les plugins installés sur votre contrôleur Jenkins pour générer la syntaxe Pipeline. Reportez-vous à la page de [référence des étapes Pipeline](https://www.jenkins.io/doc/pipeline/steps/) pour obtenir des informations sur toutes les étapes Pipeline disponibles.

L'éditeur permet aux utilisateurs de créer et de modifier des Pipelines déclaratifs et d'effectuer des actions telles que l'ajout d'étapes ou la configuration de tâches parallèles, en fonction de leurs besoins. Lorsque l'utilisateur a terminé sa configuration, l'éditeur enregistre le Pipeline dans un référentiel de code source sous forme de fichier `Jenkinsfile`. Si le Pipeline nécessite des modifications supplémentaires, Blue Ocean permet de revenir facilement à l'éditeur visuel pour modifier le Pipeline à tout moment.

<iframe width="640" height="360" src="https://www.youtube.com/embed/FhDomw6BaHU" title="Jenkins Minute - Creating Your First Pipeline in Blue Ocean" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

!!! info "_Limitations de l'éditeur Blue Ocean Pipeline_"
    L'éditeur Blue Ocean Pipeline est une excellente alternative pour les utilisateurs de Jenkins, mais ses fonctionnalités présentent certaines limitations :

    * L'éditeur utilise uniquement des Pipelines déclaratifs basés sur SCM ;
    * Les identifiants utilisateur doivent disposer d'une autorisation d'écriture ;
    * L'éditeur n'est pas entièrement compatible avec Declarative Pipeline ;
    * Le pipeline est réorganisé et les commentaires sont supprimés.

## Démarrer l'éditeur

Pour utiliser l'éditeur, l'utilisateur doit d'abord créer un pipeline dans Blue Ocean ou disposer d'au moins un pipeline existant dans Jenkins. Si vous modifiez un Pipeline existant, les informations d'identification de celui-ci doivent autoriser l'envoi des modifications vers le référentiel cible.

L'éditeur peut être lancé via :

* L'option **New Pipeline** (Nouveau Pipeline) du tableau de bord Blue Ocean ;
* L'onglet **Branches** dans la vue Activité ;
* **Edit** (Modifier) <i class="fa fa-pencil"></i> dans la vue des détails d'exécution du pipeline.

## Barre de navigation

L'éditeur de Pipeline comprend la barre de navigation standard en haut, avec une barre de navigation locale en dessous. La barre de navigation locale comprend :

* **Pipeline Name** (Nom du Pipeline) : cela inclut également le nom de la branche ;
* **Cancel** (Annuler) : annule les modifications apportées au Pipeline ;
* **Save** (Enregistrer) : ouvre la boîte de dialogue d'enregistrement du Pipeline.

## Paramètres du Pipeline

Par défaut, le volet situé à droite de l'éditeur affiche les **Pipeline Settings** (paramètres du Pipeline). Si vous sélectionnez une étape existante ou ajoutez une étape, le volet de [configuration des étapes](#configuration-des-etapes) s'affiche à la place. Pour revenir au volet Paramètres du Pipeline, sélectionnez un espace vide dans l'arrière-plan de l'éditeur. Le volet Paramètres du Pipeline comporte deux sections configurables.

### Agent

La section **Agent** contrôle l'agent utilisé par le Pipeline. Elle exécute le même processus que la [directive agent](./pipeline-syntaxe.md#agent). Le champ Image permet aux utilisateurs de configurer l'image de conteneur à exécuter lorsque le Pipeline est actif.

### Environnement

La section **Environnement** permet aux utilisateurs de configurer les variables d'environnement pour le Pipeline. Il s'agit du même processus que la [directive environnement](./pipeline-syntaxe.md#environnement).

## Éditeur d'étapes

Le volet gauche affiche l'interface utilisateur de l'éditeur d'étapes, qui permet aux utilisateurs de créer ou d'ajouter des étapes à un Pipeline.

![Vue de l'éditeur d'étapes d'un nouveau Pipeline.](https://www.jenkins.io/doc/book/resources/blueocean/editor/stage-editor-basic.png)

* Pour ajouter une étape au Pipeline, sélectionnez l'icône <i class="fa fa-plus"></i> à droite d'une étape existante. Sélectionner l'icône <i class="fa fa-plus"></i> sous une étape existante ajoute une étape parallèle.
* Pour supprimer les étapes indésirables, utilisez le [menu « Plus » dans le volet de configuration des étapes](#configuration-detapes).

Après avoir défini le nom de l'étape et l'avoir enregistré, le nom s'affiche au-dessus de l'étape. Les étapes qui contiennent des informations incomplètes ou non valides affichent un <i class="fa fa-triangle-exclamation"></i> . Les Pipelines peuvent présenter des erreurs de validation pendant l'édition, mais l'enregistrement est bloqué jusqu'à ce que les erreurs soient corrigées.

![Éditeur d'étapes avec erreur affichée.](https://www.jenkins.io/doc/book/resources/blueocean/editor/stage-editor-error.png
)

### Configuration des étapes

Lorsque vous sélectionnez une étape dans l'éditeur, le volet **Stage Configuration** (Configuration des étapes) s'affiche à droite de l'écran. Vous pouvez y modifier le nom de l'étape, supprimer l'étape et ajouter des échelons à une étape.

![Volet Configuration des étapes](https://www.jenkins.io/doc/book/resources/blueocean/editor/stage-configuration.png)

Le nom de l'étape peut être défini en haut du volet Configuration des étapes. Le menu plus, représenté par trois points à droite du nom de l'étape, permet aux utilisateurs de supprimer l'étape actuellement sélectionnée. Sélectionnez **Add step** (Ajouter une étape) pour afficher la liste des types d'étapes disponibles. Après avoir sélectionné un type d'étape, la page affiche le volet Configuration de l'étape.

![Liste des étapes filtrée par « fichier »](https://www.jenkins.io/doc/book/resources/blueocean/editor/step-list.png)

### Configuration de l'étape

Cet affichage est basé sur le type d'étape sélectionné et contient les champs ou contrôles nécessaires.

![Configuration de l'étape pour l'étape JUnit](https://www.jenkins.io/doc/book/resources/blueocean/editor/step-configuration.png)

Veillez à donner un nom significatif à l'étape, car le nom conserve sa configuration d'origine. La seule façon de donner un nom différent est de supprimer l'étape et de la recréer. Le menu plus, représenté par trois points à droite du nom de l'étape, permet aux utilisateurs de supprimer l'étape actuelle. Les champs contenant des informations incomplètes ou non valides affichent un . Toute erreur de validation doit être corrigée avant l'enregistrement.

![Configuration de l'étape avec erreur](https://www.jenkins.io/doc/book/resources/blueocean/editor/step-error.png)

## Boîte de dialogue Enregistrer le Pipeline

Les modifications apportées à un Pipeline doivent être enregistrées dans le contrôle de source avant d'être exécutées. La boîte de dialogue **Save Pipeline** (Enregistrer le Pipeline) contrôle l'enregistrement des modifications dans le contrôle de source.

![Boîte de dialogue Enregistrer le Pipeline](https://www.jenkins.io/doc/book/resources/blueocean/editor/save-pipeline.png)

Vous pouvez éventuellement saisir une description des modifications avant de les enregistrer. La boîte de dialogue propose également des options permettant d'enregistrer les modifications dans la même branche ou de créer une nouvelle branche dans laquelle les enregistrer. Sélectionner **Save & run** (Enregistrer et exécuter) enregistre toutes les modifications apportées au Pipeline en tant que nouveau _commit_, lance une nouvelle exécution du Pipeline basée sur ces modifications et accède à la [vue Activité](./blue-ocean-activite) de ce Pipeline.



