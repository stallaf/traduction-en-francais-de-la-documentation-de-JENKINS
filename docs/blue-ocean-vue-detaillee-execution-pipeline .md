# Vue détaillée de l'exécution des Pipelines

<div class="couleur-introduction">
La vue Détails de l'exécution du Pipeline Blue Ocean affiche les informations relatives à une seule exécution du Pipeline et permet aux utilisateurs de modifier ou de rejouer cette exécution. Vous trouverez ci-dessous un aperçu détaillé des différentes parties de la vue Détails de l'exécution.
</div>

!!! info "_Statut de Blue Ocean_"
    Blue Ocean ne fera l'objet d'aucune nouvelle mise à jour fonctionnelle. Blue Ocean continuera à fournir une visualisation facile à utiliser du Pipeline, mais ne sera plus amélioré. Il ne fera l'objet que de mises à jour sélectives en cas de problèmes de sécurité ou de défauts fonctionnels importants.

![Aperçu des détails de l'exécution du Pipeline](https://www.jenkins.io/doc/book/resources/blueocean/pipeline-run-details/overview.png)

1. **Run Status** (Statut de l'exécution) : icône indiquant le statut de cette exécution du Pipeline. La couleur de la barre de navigation correspond à l'icône de statut.
2. **Pipeline Name** (Nom du Pipeline) : nom du Pipeline de cette exécution.
3. **Run Number** (Numéro d'exécution) : numéro d'identification de cette exécution du Pipeline. Les numéros d'identification sont uniques pour chaque branche et chaque _pull request_ d'un Pipeline.
4. **View Tabs** (Onglets d'affichage) : accédez aux vues **Pipelines**, **Modifications**, **Tests** et **Artefacts** à l'aide de l'un des [onglets](#onglets) de cette exécution. La vue par défaut est « [Pipeline](#pipeline) ».
5. **Re-run Pipeline** (Réexécuter le Pipeline) : réexécute le Pipeline de cette exécution.
6. **Edit Pipeline** (Editer le Pipeline) : ouvre le Pipeline de cette exécution dans [l'éditeur de Pipeline](./blue-ocean-editeur-pipeline.md).
7. **Configure** (Configurer) : ouvre la page de configuration du Pipeline dans Jenkins.
8. **Go to Classic** (Aller à l'interface classique) : bascule vers l'interface utilisateur « classique » des détails de cette exécution.
9. **Close Details** (Fermer les détails) : ferme la vue Détails et ramène l'utilisateur à la vue Activité de ce Pipeline.
10. **Branch or Pull Request** (Branche ou demande d'extraction) : la branche ou la demande d'extraction pour cette exécution.
11. **Commit Id** (Commit Id) : l'ID de validation pour cette exécution.
12. **Duration** (Durée) : la durée de cette exécution.
13. **Completed Time** (Heure d'achèvement) : l'heure à laquelle cette exécution a été achevée.
14. **Change Author** (Auteur des modifications) : les noms des auteurs ayant apporté des modifications lors de cette exécution.
15. **Tab View** (Affichage de l'onglet) : affiche les informations pour l'onglet sélectionné.

## État d'exécution du Pipeline

Blue Ocean permet de visualiser facilement l'état d'exécution actuel du Pipeline, en modifiant la couleur de la barre de menu supérieure en fonction de l'état :

* Bleu pour « En cours » ;
* Vert pour « Réussi » ;
* Jaune pour « Instable » ;
* Rouge pour « Échec » ;
* Gris pour « Abandonné ».

## Cas particuliers

Blue Ocean est optimisé pour fonctionner avec les Pipelines dans le contrôle de source, mais peut afficher des détails pour d'autres types de projets. Blue Ocean propose les mêmes [onglets](#onglets) pour tous les types de projets pris en charge, mais ces onglets affichent des informations différentes selon le type.

### Pipelines en dehors du contrôle de source

Pour les Pipelines qui ne sont pas basés sur le contrôle de source, Blue Ocean affiche les champs « _Commit Id_ », « _Branch_ » et « _Changes_ », mais ceux-ci restent vides. Dans ce cas, la barre de menu supérieure ne comprend pas l'option « _Edit_ ».

### Projets Freestyle

Pour les projets Freestyle, Blue Ocean propose les mêmes [onglets](#onglets), mais l'onglet Pipeline affiche uniquement la sortie du journal de la console. Les options « Rerun » (Réexécuter) ou « Edit » (Modifier) ne sont pas disponibles dans la barre de menu supérieure.

### Projets Matrix

Les projets Matrix ne sont pas pris en charge dans Blue Ocean. L'affichage d'un projet Matrix redirige vers la vue « Classic UI » (Interface utilisateur classique) de ce projet.

## Onglets

Chaque onglet de la vue Détails de l'exécution fournit des informations sur un aspect spécifique d'une exécution.

### Pipeline

Pipeline est l'onglet par défaut et donne une vue d'ensemble du flux de son exécution. Il affiche chaque étape et chaque branche parallèle, les étapes de ces phases et la sortie de la console de ces étapes. L'image ci-dessous montre une exécution réussie. Si une étape particulière échoue pendant l'exécution, cet onglet affiche automatiquement le journal de la console de l'étape qui a échoué. L'image ci-dessous montre également une exécution qui a échoué.

![Exécution](https://www.jenkins.io/doc/book/resources/blueocean/pipeline-run-details/pipeline-failed.png)

### Modifications

L'onglet **Changes** (Modifications) affiche les informations relatives à toutes les modifications apportées entre la dernière exécution terminée et l'exécution en cours. Cela inclut l'ID de validation de la modification, l'auteur de la modification, le message et la date d'achèvement.

![Liste des modifications pour une exécution](https://www.jenkins.io/doc/book/resources/blueocean/pipeline-run-details/changes-one-change.png)

### Tests

L'onglet **Tests** affiche des informations sur les résultats des tests pour cette exécution. Cet onglet ne contient des informations que si une étape de publication des résultats des tests est présente, telle que l'étape « Publier les résultats des tests JUnit » (`junit`). Si aucun résultat n'est enregistré, ce tableau affiche un message. Si tous les tests sont réussis, cet onglet indique le nombre total de tests réussis. En cas d'échecs, l'onglet affiche les détails du journal des échecs, comme indiqué ci-dessous.

![Résultats des tests pour une exécution instable](https://www.jenkins.io/doc/book/resources/blueocean/pipeline-run-details/tests-unstable.png)

Lorsqu'une exécution précédente comporte des échecs et que l'exécution actuelle corrige ces échecs, cet onglet note les corrections et affiche leurs journaux.

![Résultats des tests pour une exécution corrigée](https://www.jenkins.io/doc/book/resources/blueocean/pipeline-run-details/tests-fixed.png)

### Artefacts

L'onglet **Artefacts** affiche une liste de tous les artefacts enregistrés à l'aide de l'étape « Archiver les artefacts » (`archiveArtifacts`). Sélectionnez un élément dans la liste pour le télécharger. Le journal de sortie complet de l'exécution peut être téléchargé à partir de cette liste.

![Liste des artefacts d'une exécution](https://www.jenkins.io/doc/book/resources/blueocean/pipeline-run-details/artifacts-list.png)

