# Commencez avec Blue Ocean

<div class="couleur-introduction">
Cette section décrit comment démarrer avec Blue Ocean dans Jenkins. Elle comprend des instructions pour [configurer Blue Ocean](#installer-blue-ocean) sur votre contrôleur Jenkins, [accéder à l'interface utilisateur Blue Ocean](#acceder-a-blue-ocean) et [revenir à l'interface utilisateur classique de Jenkins](#revenir-a-linterface-classique).
</div>

!!! info "_Statut de Blue Ocean_"
    Blue Ocean ne bénéficiera plus de mises à jour fonctionnelles. Blue Ocean continuera à fournir une visualisation facile à utiliser du Pipeline, mais ne sera plus amélioré. Il ne bénéficiera que de mises à jour sélectives pour les problèmes de sécurité importants ou les défauts fonctionnels.


### Installation de Blue Ocean

Vous pouvez installer Blue Ocean à l'aide des méthodes suivantes :

* En tant que suite de plugins sur un [contrôleur Jenkins existant](#sur-un-controleur-jenkins-existant) ;
* En tant que partie intégrante de [Jenkins dans Docker](#dans-le-cadre-de-jenkins-dans-docker).

### Sur un contrôleur Jenkins existant

Lorsque Jenkins est installé sur la plupart des plateformes, le plugin [Blue Ocean](https://plugins.jenkins.io/blueocean) et tous les plugins dépendants nécessaires, qui compilent la suite de plugins Blue Ocean, ne sont pas installés par défaut.

Les plugins peuvent être installés sur un contrôleur Jenkins par tout utilisateur Jenkins disposant de l'autorisation **Administrer**. Celle-ci est définie via la sécurité basée sur Matrix. Les utilisateurs Jenkins disposant de cette autorisation peuvent également configurer les autorisations des autres utilisateurs sur leur système. Pour plus d'informations, consultez la section [Autorisation](./securite-gestion-de-la-securite.md#autorisation ) de la rubrique [Gestion de la sécurité](./securite-gestion-de-la-securite.md).

Pour installer la suite de plugins Blue Ocean sur votre contrôleur Jenkins :

1. Assurez-vous d'être connecté à Jenkins en tant qu'utilisateur disposant des droits **Administrateur** ;
2. Depuis la page d'accueil de Jenkins, sélectionnez **_Manage Jenkins_**  (Gérer Jenkins) à gauche, puis **_Plugins_**  (Plugins) ;
3. Sélectionnez l'onglet **_Available_** (Disponible) et saisissez `blue ocean` dans la zone de texte **_Filter_** (Filtrer). Cela permet de filtrer la liste des plugins en fonction du nom et de la description.

![Liste des plugins Blue Ocean disponibles après filtrage.](https://www.jenkins.io/doc/book/resources/blueocean/intro/blueocean-plugins-filtered.png)

4. Cochez la case à gauche de **_Blue Ocean_**, puis sélectionnez l'option **_Download now and install after restart_** (Télécharger maintenant et installer après redémarrage) (recommandé) ou l'option **_Install without restart_** (Installer sans redémarrer) en bas de la page.

!!! info " "
    * Il n'est pas nécessaire de sélectionner d'autres plugins dans cette liste. Le plugin **Blue Ocean** principal sélectionne et installe automatiquement tous les plugins dépendants, composant ainsi la suite de plugins Blue Ocean.
    * Si vous sélectionnez l'option **_Install without restart_** (Installer sans redémarrer), vous devez redémarrer Jenkins pour bénéficier de toutes les fonctionnalités de Blue Ocean.

Pour plus d'informations, consultez la page [Gestion des plugins](./gestion-gestion-des-plugins.md). Blue Ocean ne nécessite aucune configuration supplémentaire après l'installation. Les Pipelines et projets existants continueront de fonctionner comme d'habitude.

!!! info " "
	La première fois que vous [créez un Pipeline](./blue-ocean-creer-un-pipeline.md) dans Blue Ocean pour un serveur Git spécifique, Blue Ocean vous demande vos identifiants Git afin de vous permettre de créer des Pipelines dans les référentiels. Cela est nécessaire car Blue Ocean peut ajouter un fichier `Jenkinsfile` à vos référentiels. 

### Dans le cadre de Jenkins dans Docker

La suite de plugins Blue Ocean n'est pas fournie avec l'image Docker Jenkins officielle, [jenkins/jenkins](https://hub.docker.com/r/jenkins/jenkins/), disponible dans le référentiel [Docker Hub](https://hub.docker.com/).

Pour en savoir plus sur l'exécution de Jenkins et Blue Ocean dans Docker, consultez la section [Docker](./installation-docker.md) de la page d'installation de Jenkins.

## Accéder à Blue Ocean

Une fois que Blue Ocean est installé dans l'environnement Jenkins et que vous vous êtes connecté à l'interface utilisateur classique de Jenkins, vous pouvez accéder à l'interface utilisateur de Blue Ocean en sélectionnant **_Open Blue Ocean_** (Ouvrir Blue Ocean) dans la partie gauche de l'écran.

<img src="/assets/open-blue-ocean-link.png" alt="Lien Open Blue Ocean" width="200" height="40">

Vous pouvez également accéder directement à Blue Ocean en ajoutant `/blue `à la fin de l'URL de votre serveur Jenkins. Par exemple : `https://jenkins-server-url/blue`.

Si votre contrôleur Jenkins :

* comporte déjà des projets Pipeline ou d'autres éléments, le [tableau de bord Blue Ocean](./bue-ocean-tableau-de-bord.md) s'affiche.
* est nouveau ou ne comporte aucun projet ou autre élément configuré, Blue Ocean affiche un volet **Welcome to Jenkins** (Bienvenue dans Jenkins) avec un bouton **Create a new Pipeline** (Créer un nouveau Pipeline). Vous pouvez sélectionner ce bouton pour commencer à créer un nouveau projet Pipeline. Pour plus d'informations, consultez la page [Création d'un Pipeline](./blue-ocean-creer-un-pipeline.md) pour plus d'informations sur la création d'un projet Pipeline dans Blue Ocean.

![Bienvenue dans Jenkins - Créer un nouveau pipeline boîte de message](https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/create-a-new-pipeline-box.png)

## Barre de navigation

L'interface utilisateur Blue Ocean comporte une barre de navigation en haut de l'interface, qui vous permet d'accéder aux différentes vues et fonctionnalités.

La barre de navigation est divisée en deux sections :

* Une section commune en haut de la plupart des vues Blue Ocean ;
* Une section contextuelle en dessous.

La section contextuelle est spécifique à la page Blue Ocean que vous consultez actuellement.

La section commune de la barre de navigation comprend les boutons suivants :

* **Jenkins** : sélectionnez l'icône Jenkins pour accéder au [tableau de bord](./blue-ocean-tableau-de-bord.md) ou recharger cette page si vous l'affichez déjà ;
* **Pipelines** : cette option vous permet également d'accéder au tableau de bord. Si vous êtes déjà sur le tableau de bord, cette option recharge la page. Ce bouton a une fonction différente lorsque vous affichez la page des [détails d'exécution d'un Pipeline](./blue-ocean-details-de-l-execution-du-pipeline.md).
* **Administration** : cette option vous permet d'accéder à la page [**Manage Jenkins**](./gestion-jenkins.md) (Gérer Jenkins) de l'interface utilisateur classique de Jenkins. Ce bouton n'est pas disponible si vous ne disposez pas de l'autorisation **Administer** (Administrer). Pour plus d'informations, consultez la section [Autorisation](./gestion-securite.md#autorisation) de la page Gestion de la sécurité ;
* Icône **Go to classic** (Aller à l'interface classique) : cette option vous permet de revenir à l'interface utilisateur classique de Jenkins. Pour en savoir plus, consultez la section [Passage à l'interface utilisateur classique](#passer-a-linterface-utilisateur-classique) ;
* **Logout** (Déconnexion) : cette option déconnecte votre utilisateur Jenkins actuel et vous ramène à la page de connexion Jenkins.

Les vues qui utilisent la barre de navigation commune ajoutent une autre barre en dessous. Cette deuxième barre comprend des options spécifiques à cette vue. Certaines vues remplacent la barre de navigation commune par une barre spécialement adaptée à cette vue.

## Passer à l'interface utilisateur classique

Blue Ocean ne prend pas en charge certaines fonctionnalités héritées ou administratives de Jenkins qui sont nécessaires à certains utilisateurs.

Si vous avez besoin d'accéder à ces fonctionnalités, sélectionnez l'icône **Go to classic** (Aller à la version classique) en haut d'une section commune de la [barre de navigation](#barre-de-navigation) de Blue Ocean.

<img src="/assets/go-to-classic-icon.png" alt="Icône Aller à la version classique">

En sélectionnant ce bouton, vous accédez à la page équivalente dans l'interface utilisateur classique de Jenkins ou à la page de l'interface utilisateur classique la plus pertinente qui correspond à la page actuelle dans Blue Ocean.