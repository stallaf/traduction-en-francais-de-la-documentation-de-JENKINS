# Gestion de Jenkins

<div class="couleur-traduction">
La plupart des tâches administratives standard peuvent être effectuées à partir des écrans de la section **Manage Jenkins** (Gérer Jenkins) du tableau de bord. Dans ce chapitre, nous examinons ces écrans et expliquons comment les utiliser.
</div>

Les vignettes affichées sur la page **Manage Jenkins** (Gérer Jenkins)  sont regroupées de manière logique. Nous abordons ici les pages qui font partie de l'installation standard. Des plugins peuvent ajouter des pages à cet écran.

La partie supérieure de l'écran **Manage Jenkins** (Gérer Jenkins) peut contenir des « moniteurs » qui vous alertent lorsqu'une nouvelle version du logiciel Jenkins ou une mise à jour de sécurité est disponible. Chaque moniteur comprend une description du problème signalé et des liens vers des informations supplémentaires à ce sujet.

Une aide en ligne est disponible sur la plupart des pages **Manage Jenkins** (Gérer Jenkins) :

* Pour accéder à l'aide, sélectionnez l'icône `?` à droite de chaque champ ;
* Cliquez à nouveau sur l'icône `?` pour masquer le texte d'aide.

D'autres sujets liés à l'administration du système sont abordés dans [Administration du système Jenkins](./administration-systeme-presentation.md).

## Groupe Configuration du système

Écrans permettant de configurer les ressources pour votre contrôleur Jenkins.

[Système](./gestion-configuration-du-system.md)<br>
Configurez les paramètres globaux et les chemins d'accès pour le contrôleur Jenkins.

[Outils](./gestion--outils.md)<br>
Configurez les outils, leur emplacement et les programmes d'installation automatiques.

[Plugins](./gestion-plugins.md)<br>
Ajoutez, mettez à jour, supprimez, désactivez/activez les plugins qui étendent les fonctionnalités de Jenkins.

[Nœuds et clouds](./gestion-noeuds.md)<br>
Ajoutez, supprimez, contrôlez et surveillez les nœuds utilisés pour les agents sur lesquels s'exécutent les tâches de construction.

[Configuration en tant que code](gestion-configuration-comme-code.md)
Configurez votre contrôleur Jenkins à l'aide d'un fichier YAML lisible par l'utilisateur plutôt que via l'interface utilisateur. Il s'agit d'une fonctionnalité facultative qui n'apparaît dans ce groupe que lorsque le plugin est installé sur votre contrôleur.

## Groupe de sécurité

Écrans permettant de configurer les fonctionnalités de sécurité de votre contrôleur Jenkins. Consultez la section [Sécurisation de Jenkins](./securisation-presentation.md) pour obtenir des informations générales sur la gestion de la sécurité Jenkins.

[Sécurité](./gestion-configuration-systeme.md)<br>
Définissez les paramètres de configuration qui sécurisent votre contrôleur Jenkins.

[Gérer les informations d'identification](./utilisation-identifiants.md#ajout-de-nouvelles-informations-didentification-globales)<br>
Configurez les informations d'identification qui permettent un accès sécurisé aux sites et applications tiers qui interagissent avec Jenkins.

**Fournisseurs d'informations d'identification**
Configurez les fournisseurs et les types d'informations d'identification.

[Utilisateurs](./gestion-utilisateurs.md)<br>
Gérez les utilisateurs définis dans la base de données des utilisateurs Jenkins. Cette option n'est pas utilisée si vous utilisez un domaine de sécurité différent, tel que LDAP ou AD.

## Groupe Informations d'état

[Informations système](./gestion-information-systeme.md)<br>
Affiche des informations sur l'environnement Jenkins.

[Journal système](./administration-systeme-affichage-des-journaux.md)<br>
Journal Jenkins contenant toutes les sorties `java.util.logging` liées à Jenkins.

**Statistiques de charge**
Affiche des informations sur l'utilisation des ressources sur votre contrôleur Jenkins.

[À propos de Jenkins](./gestion-apropos-de-jenkins.md)<br>
Fournit des informations sur la version et la licence de votre contrôleur Jenkins.

## Groupe Dépannage

**Gérer les anciennes données**
Supprime les informations de configuration relatives aux plugins qui ont été supprimés du contrôleur.

## Groupe Outils et actions

Écrans pour les tâches de gestion courantes et les outils qui vous permettent d'effectuer des tâches administratives sans utiliser l'interface utilisateur.

**Recharger la configuration à partir du disque**
Supprime toutes les données chargées en mémoire et recharge tout à partir du système de fichiers. Cette fonction est utile lorsque vous modifiez les fichiers de configuration directement sur le disque.

[CLI Jenkins](./gestion-cli.md)<br>
Comment utiliser la CLI Jenkins à partir d'un shell ou d'un script.

[Console de script](./gestion-script-en-console)<br>
Exécutez un script Apache Groovy pour l'administration, le dépannage et les diagnostics.

**Préparation à l'arrêt**
Empêche le démarrage de nouvelles compilations afin que le système puisse être arrêté en toute sécurité. Affiche une bannière rouge avec un message personnalisé afin que les utilisateurs sachent ce qui va se passer.

![Bandeau rouge avec un message personnalisé](https://www.jenkins.io/doc/book/resources/managing/prepare-for-shutdown.png)

!!! info " "
    Cela ne demande pas à Jenkins de s'arrêter ; cette action empêche simplement le démarrage de nouvelles compilations. Si vous devez arrêter ou redémarrer Jenkins, vous devez utiliser la ligne de commande ou les points de terminaison `/restart` et `/safeRestart`. Il existe également un plugin appelé [Safe Restart](https://plugins.jenkins.io/saferestart/) qui ajoute un lien `Restart Safely` (Redémarrer en toute sécurité) dans l'interface utilisateur. 

## Groupe non classé

Écrans pour les plugins qui n'ont pas encore déclaré la catégorie de leur page.

