# Blue Ocean

<div class="couleur-introduction">
Ce chapitre couvre tous les aspects des fonctionnalités de Blue Ocean, notamment comment :
</div>

* [démarrer avec Blue Ocean](./blue-ocean-commencer.md), qui explique comment configurer Blue Ocean dans Jenkins et accéder à l'interface Blue Ocean ;
* [créer un nouveau projet Pipeline](./blue-ocean-creer-un-pipeline.md) dans Blue Ocean ;
* utiliser le [tableau de bord](./blue-ocean-tableau-de-bord.md) de Blue Ocean ;
* utiliser la [vue Activité](./blue-ocean-activites.md), où vous pouvez accéder à vos [données d'exécution actuelles et historiques](./blue-ocean-activites.md#activite), aux [branches](./blue-ocean-activites.md#branches) de votre Pipeline et à toutes les [_pull requests_](./blue-ocean-activites.md#pr) ouvertes ;
* utiliser la [vue des détails d'exécution du Pipeline](./blue-ocean-vue-activite-pipeline) pour accéder à des détails tels que la sortie de la console, pour un Pipeline ou un élément particulier ;
* utiliser [l'éditeur Pipeline](./blue-ocean-editeur-pipeline.md) pour modifier les Pipelines sous forme de code, que vous pouvez ensuite valider dans le contrôle de source.

Ce chapitre s'adresse aux utilisateurs Jenkins de tous niveaux, mais les débutants devront peut-être se reporter au chapitre [Pipeline](./pipeline-presentation.md) pour comprendre certains des sujets abordés ici.

Pour obtenir un aperçu du contenu du manuel de l'utilisateur Jenkins, consultez la [présentation du manuel de l'utilisateur](./index.md).

!!! info
    _Statut de Blue Ocean_<br>
    Blue Ocean ne bénéficiera plus de mises à jour fonctionnelles. Blue Ocean continuera à fournir une visualisation facile à utiliser du pipeline, mais ne sera plus amélioré. Il ne bénéficiera que de mises à jour sélectives pour les problèmes de sécurité importants ou les défauts fonctionnels.

    D'autres options de visualisation du Pipeline, telles que les plugins [Pipeline: Stage View](https://plugins.jenkins.io/pipeline-stage-view/) et [Pipeline Graph View,]( Pipeline Graph View) sont disponibles et offrent certaines de ces fonctionnalités. Bien qu'elles ne remplacent pas complètement Blue Ocean, les contributions de la communauté sont encouragées pour le développement continu de ces plugins.

    Le [générateur d'extraits de syntaxe Pipeline](./pipeline-commencer.md#générateur-dextraits) aide les utilisateurs à définir les étapes du Pipeline avec leurs arguments. Il s'agit de l'outil privilégié pour la création de Pipelines Jenkins, car il fournit une aide en ligne pour les étapes du pipeline disponibles dans votre contrôleur Jenkins. Il utilise les plugins installés sur votre contrôleur Jenkins pour générer la syntaxe Pipeline. Reportez-vous à la page de [référence des étapes du Pipeline](https://www.jenkins.io/doc/pipeline/steps/) pour obtenir des informations sur toutes les étapes disponibles.

## Qu'est-ce que Blue Ocean ?

Blue Ocean offre actuellement une visualisation facile à utiliser du Pipeline. Il a été conçu pour repenser l'expérience utilisateur de Jenkins, entièrement repensée pour [Jenkins Pipeline](./pipeline-presentation.md). Blue Ocean a été conçu pour réduire l'encombrement et améliorer la clarté pour tous les utilisateurs.

Cependant, Blue Ocean ne bénéficiera plus de mises à jour fonctionnelles ou d'améliorations. Il ne recevra que des mises à jour sélectives pour les problèmes de sécurité importants ou les défauts fonctionnels. Si vous débutez, vous pouvez toujours utiliser Blue Ocean, ou vous pouvez envisager d'autres options telles que les plugins [Pipeline : Stage View](https://plugins.jenkins.io/pipeline-stage-view/) et [Pipeline Graph View](https://plugins.jenkins.io/pipeline-graph-view/). Ceux-ci offrent certaines des mêmes fonctionnalités.

En résumé, les principales fonctionnalités de Blue Ocean sont les suivantes :

* **Visualisation sophistiquée** des Pipelines de livraison continue (CD), permettant une compréhension rapide et intuitive de l'état de votre Pipeline ;
* **L'éditeur de Pipeline** rend la création de Pipelines plus accessible, en guidant l'utilisateur à travers un processus visuel pour créer un Pipeline ;
* **Personnalisation** pour répondre aux besoins spécifiques de chaque membre de l'équipe en fonction de son rôle ;
* **Précision extrême** lorsque des interventions sont nécessaires ou que des problèmes surviennent. Blue Ocean indique les points qui nécessitent une attention particulière, facilitant ainsi la gestion des exceptions et augmentant la productivité ;
* **Intégration native pour les branches et les _pull requests_,** qui permet une productivité maximale des développeurs lorsqu'ils collaborent sur du code dans GitHub et Bitbucket.

Si vous souhaitez commencer à utiliser Blue Ocean, veuillez vous reporter à la [section « Démarrer avec Blue Ocean »](./blue-ocean-commencer.md).

<iframe width="853" height="480" src="https://www.youtube.com/embed/k_fVlU1FwP4" title="Jenkins Blue Ocean: Continuous Delivery for Every Team" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Foire aux questions

### Pourquoi Blue Ocean existe-t-il ?

Le monde DevOps est passé d'outils de développement purement fonctionnels à des outils faisant partie intégrante de l'« expérience développeur ». Il ne s'agit plus d'un seul outil, mais des nombreux outils que les développeurs utilisent tout au long de la journée et de la manière dont ils fonctionnent ensemble pour améliorer le flux de travail des développeurs.

Les éditeurs d'outils de développement tels que Heroku, Atlassian et GitHub ont relevé la barre en matière d'expérience développeur. Peu à peu, les développeurs se sont tournés vers des outils non seulement fonctionnels, mais également conçus pour s'intégrer de manière transparente à leur _workflow_ existant. Cette évolution s'accompagne d'exigences accrues en matière de conception et de fonctionnalités, les développeurs attendant désormais une expérience utilisateur exceptionnelle. Jenkins devait se mettre à la hauteur pour répondre à ces nouvelles attentes.

La création et la visualisation de Pipelines de livraison continue ont toujours été très appréciées par de nombreux utilisateurs de Jenkins. Cela s'est traduit par la création de plugins par la communauté Jenkins afin de répondre à leurs besoins. Cela indique également la nécessité de revoir la manière dont Jenkins exprime actuellement ces concepts et de considérer les Pipelines de livraison comme un thème central de l'expérience utilisateur Jenkins.

Il ne s'agit pas seulement des concepts de livraison continue, mais aussi des outils que les développeurs utilisent quotidiennement, tels que GitHub, Bitbucket, Slack, Puppet ou Docker. Cela va au-delà de Jenkins, car cela inclut le _workflow_ des développeurs autour de Jenkins, qui comprend plusieurs outils.

Les nouvelles équipes peuvent rencontrer des difficultés lorsqu'elles apprennent à assembler leur propre expérience Jenkins. Cependant, l'objectif d'améliorer leur délai de mise sur le marché en livrant des logiciels de meilleure qualité de manière plus cohérente reste le même. En tant que communauté d'utilisateurs et de contributeurs Jenkins, nous pouvons travailler ensemble pour définir l'expérience Jenkins idéale. Au fil du temps, les attentes des développeurs en matière d'expérience utilisateur évoluent, et le projet Jenkins doit être réceptif à ces attentes.

La communauté Jenkins a travaillé sans relâche pour créer l'outil d'automatisation logicielle le plus performant et le plus extensible qui soit. Ne pas révolutionner l'expérience des développeurs Jenkins aujourd'hui pourrait signifier qu'une option à code source fermé tentera d'en tirer profit à l'avenir.

Blue Ocean a été créé pour répondre aux exigences de son époque. Cependant, au fil du temps, des outils plus modernes ont fait leur apparition pour le remplacer. Aujourd'hui, le moment est venu pour d'autres plugins aux fonctionnalités similaires de prendre le relais. Par conséquent, tout nouveau développement ou amélioration de Blue Ocean a cessé. Si vous souhaitez contribuer à un plugin ayant un objectif similaire, vous devriez envisager les alternatives suggérées dans la section « [Qu'est-ce que Blue Ocean ?](#quest-ce-que-blue-ocean-) » ci-dessus.

<iframe width="853" height="480" src="https://www.youtube.com/embed/mn61VFdScuk" title="Jenkins World 2016 - Blue Ocean: A New User Experience for Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### D'où vient ce nom ?

Le nom Blue Ocean vient du livre [Blue Ocean Strategy](https://en.wikipedia.org/wiki/Blue_Ocean_Strategy). Cette stratégie consiste à examiner les problèmes dans un espace plus large et non contesté, plutôt que les problèmes stratégiques dans un espace contesté. Pour simplifier, considérez cette citation de la légende du hockey sur glace Wayne Gretzky : « Patinez vers l'endroit où le palet va se trouver, pas vers l'endroit où il était. »

#### Blue Ocean prend-il en charge les tâches freestyle ?

Blue Ocean vise à offrir une excellente expérience autour de Pipeline et à être compatible avec tous les travaux freestyle que vous avez déjà configurés dans votre contrôleur Jenkins. Cependant, vous ne bénéficierez pas des fonctionnalités conçues pour les Pipelines, par exemple la visualisation des Pipelines.

Blue Ocean a été conçu pour être extensible, afin que la communauté Jenkins puisse étendre ses fonctionnalités. Bien qu'aucune fonctionnalité supplémentaire ne soit ajoutée à Blue Ocean, il offre toujours la visualisation des Pipelines et d'autres fonctionnalités que les utilisateurs trouvent utiles.

### Qu'est-ce que cela signifie pour mes plugins ?

L'extensibilité est une fonctionnalité essentielle de Jenkins et la possibilité d'étendre l'interface utilisateur Blue Ocean est tout aussi importante. Le balisage `<ExtensionPoint name=..>` peut être utilisé dans le balisage de Blue Ocean, laissant ainsi de la place aux plugins pour contribuer à l'interface utilisateur. Cela signifie que les plugins peuvent avoir leurs propres points d'extension Blue Ocean. Blue Ocean lui-même est implémenté à l'aide de ces points d'extension.

Les extensions sont fournies par les plugins comme d'habitude. Les plugins doivent inclure du code JavaScript supplémentaire pour se connecter aux points d'extension de Blue Ocean. Les développeurs qui ont contribué à l'expérience utilisateur de Blue Ocean auront ajouté ce code JavaScript en conséquence.

### Quelles technologies sont actuellement utilisées ?

Blue Ocean est conçu comme un ensemble de plugins Jenkins. La principale différence réside dans le fait que Blue Ocean fournit à la fois son propre point de terminaison pour les requêtes HTTP et qu'il fournit du code HTML/JavaScript via un chemin différent, sans utiliser le balisage ou les scripts de l'interface utilisateur Jenkins existante. React.js et ES6 sont utilisés pour fournir les composants JavaScript de Blue Ocean. Inspiré par cet excellent projet open source, que vous pouvez consulter dans l'article de blog consacré à la [création de plugins pour les applications React](https://nylas.com/blog/react-plugins), un modèle `<ExtensionPoint>` a été mis en place pour permettre aux extensions de provenir de n'importe quel plugin Jenkins avec JavaScript. Si les extensions ne parviennent pas à se charger, leurs échecs sont isolés.

### Où puis-je trouver le code source ?

Le code source est disponible sur Github :

* [Blue Ocean](https://github.com/jenkinsci/blueocean-plugin) ;
* [Jenkins Design Language](https://github.com/jenkinsci/jenkins-design-language).

## Rejoignez la communauté Blue Ocean

Le développement de Blue Ocean étant gelé, nous ne prévoyons ni n'attendons de nouvelles contributions à son code source pour de nouvelles fonctionnalités. Cependant, il existe encore plusieurs façons de rejoindre la communauté :

1. Discutez avec la communauté et l'équipe de développement sur Gitter <img src="/assets/repo.svg" href="https://app.gitter.im/#/room/#jenkinsci_blueocean-plugin:gitter.im">.
2. Demandez des fonctionnalités ou signalez des bogues concernant le [composant blueocean-plugin dans JIRA](https://issues.jenkins.io/).
3. Abonnez-vous et posez vos questions sur la [liste de diffusion des utilisateurs de Jenkins](https://groups.google.com/g/jenkinsci-users).
4. Vous êtes développeur ? Nous avons [répertorié quelques problèmes](https://issues.jenkins.io/issues/?filter=16142) qui sont parfaits pour tous ceux qui souhaitent se lancer dans le développement de Blue Ocean. N'oubliez pas de passer sur le chat Gitter pour vous présenter !
