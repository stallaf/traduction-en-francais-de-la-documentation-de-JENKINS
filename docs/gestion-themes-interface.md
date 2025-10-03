# Thèmes pour l'Interface Utilisateur

<div class="couleur-introduction">
Il est possible de personnaliser l'apparence de Jenkins à l'aide de thèmes personnalisés. Cette fonctionnalité ne fait pas partie du noyau Jenkins, mais elle est prise en charge par des plugins.
</div>

## Utilisation des thèmes intégrés

Il existe plusieurs plugins qui fournissent des thèmes intégrés, les plus populaires étant

* [Dark Theme Plugin](https://plugins.jenkins.io/dark-theme) - fournit un thème sombre pour Jenkins. Prend en charge la configuration sous forme de code pour sélectionner la configuration du thème ;
* [Plugin Material Theme](https://plugins.jenkins.io/material-theme) - portage du [thème Material de Jenkins](http://afonsof.com/jenkins-material-theme/) d'Afonso F pour utiliser Theme Manager ;
* [Plugin Solarized Theme](https://plugins.jenkins.io/solarized-theme) - fournit des thèmes Solarized (clair et sombre).

L'installation de l'un de ces plugins installera également leur dépendance commune : le [plugin Theme Manager](https://plugins.jenkins.io/theme-manager). Ce plugin permet aux administrateurs de définir le thème par défaut pour une installation Jenkins via _Manage Jenkins > System > Built-in Themes_ et les utilisateurs peuvent définir leur thème préféré dans leurs paramètres personnels. Vous pouvez également configurer ce plugin à l'aide du [plugin Configuration-as-Code](https://plugins.jenkins.io/configuration-as-code). Pour plus d'informations, consultez la documentation du plugin.

### Utilisation de thèmes personnalisés

Pour pouvoir personnaliser entièrement l'apparence de Jenkins, vous pouvez installer le [plugin Simple Theme](https://plugins.jenkins.io/simple-theme-plugin). Il permet de personnaliser l'interface utilisateur de Jenkins en fournissant des fichiers CSS et Javascript personnalisés. Il prend également en charge le remplacement du favicon.

Pour configurer un thème, vous pouvez aller dans _Manage Jenkins > System > Theme_ et entrer l'URL de votre feuille de style et/ou de votre fichier Javascript. Vous pouvez également configurer ce plugin à l'aide du [plugin Configuration-as-Code](https://plugins.jenkins.io/configuration-as-code). Consultez la documentation du plugin pour obtenir des instructions d'utilisation détaillées et des liens vers des exemples de thèmes.

### Personnalisation de l'écran de connexion

Depuis Jenkins 2.128, les thèmes configurés à l'aide du [plugin Simple Theme](https://plugins.jenkins.io/simple-theme-plugin) ne permettent pas de personnaliser l'écran de connexion ([annonce](https://www.jenkins.io/blog/2018/06/27/new-login-page/)). Pour personnaliser l'écran de connexion, vous pouvez installer le [plugin Login Theme](https://plugins.jenkins.io/login-theme).

## Politique de prise en charge des thèmes

!!! warning " "
    Les thèmes Jenkins sont fournis « tels quels », sans garantie d'aucune sorte, implicite ou explicite. Les mises à jour du noyau Jenkins, des plugins et d'autres composants peuvent nuire à la compatibilité des thèmes sans préavis.

À l'heure actuelle, le projet Jenkins ne fournit pas de spécifications pour les mises en page/CSS, et nous ne pouvons garantir la compatibilité ascendante ou descendante. Nous essayons de refléter les changements majeurs dans les journaux de modifications (voir par exemple les changements « développeur » dans le [journal de modifications](https://www.jenkins.io/changelog/) Jenkins), mais les changements mineurs peuvent ne pas y figurer.

### Pourquoi ?

Des efforts continus sont déployés pour améliorer l'apparence, l'accessibilité et l'expérience utilisateur de Jenkins. Ce domaine est essentiel pour le projet. Plusieurs initiatives sont prévues dans la [feuille de route Jenkins](https://www.jenkins.io/project/roadmap/), coordonnées par le groupe d'intérêt spécial [Jenkins User Experience SIG](https://www.jenkins.io/sigs/ux/).

Les modifications majeures apportées à l'interface utilisateur impliquent des changements incompatibles dans les mises en page et la structure CSS, ce qui est critique pour les plugins de thème. Historiquement, Jenkins n'avait pas de politique de support explicite pour les thèmes, et nous ne voulons pas imposer d'exigences de compatibilité qui créeraient des obstacles à la refonte de l'interface principale de Jenkins. Plus tard, une fois que la refonte de l'interface utilisateur de Jenkins aura atteint son objectif et que l'interface utilisateur sera plus stable, nous pourrions envisager de créer des spécifications pour l'extensibilité des thèmes afin de rendre ceux-ci plus stables et de maintenir la compatibilité.

## Signaler et corriger les problèmes

Pour les thèmes intégrés, les utilisateurs sont invités à signaler les problèmes de compatibilité détectés aux responsables de la maintenance des thèmes et à leur soumettre des correctifs.

Nous rejetons généralement les rapports de bogues concernant le noyau Jenkins/les plugins qui impliquent des éléments d'interface utilisateur défectueux avec un thème personnalisé. Nous prendrons en considération les demandes d'extraction qui rétablissent la compatibilité et n'entravent pas l'évolution future de l'interface utilisateur Web.

!!! info " "
    Si un thème extérieur à l'organisation GitHub [jenkinsci](https://github.com/jenkinsci) n'est plus maintenu, il est possible de le dupliquer et de créer une nouvelle version. Pour les thèmes hébergés au sein de l'organisation `jenkinsci`, nous avons mis en place un [processus d'adoption](https://www.jenkins.io/doc/developer/plugin-governance/adopt-a-plugin/) qui s'applique également aux thèmes.

## Informations pour les développeurs de thèmes

Nous encourageons les utilisateurs de Jenkins à créer des thèmes et à les partager. Ces thèmes peuvent être un excellent moyen d'expérimenter des améliorations de l'interface utilisateur, et nous serions heureux d'étudier les améliorations qui en découlent pour le thème Jenkins par défaut.

Afin d'améliorer l'expérience utilisateur, veuillez tenir compte des recommandations suivantes :

* Versionnez les thèmes avec des balises sur Git et maintenez des journaux de modifications avec des références explicites aux changements dans les versions prises en charge (voir par exemple notre documentation sur la rédaction de versions comme l'un des moyens d'automatiser les journaux de modifications) ;
* Définissez explicitement une [licence open source approuvée par l'OSI](https://opensource.org/licenses) afin que les utilisateurs puissent les modifier et les redistribuer librement.
    * Il s'agit également d'une condition préalable à l'hébergement de thèmes dans les organisations GitHub de Jenkins et, à l'avenir, sur les marchés de thèmes ou d'autres moteurs de promotion similaires.

Si vous souhaitez partager une anecdote sur les thèmes Jenkins, n'hésitez pas à en faire part au groupe [Advocacy&Outreach SIG ](https://www.jenkins.io/sigs/advocacy-and-outreach/)!
