# Création d'un Pipeline

Blue Ocean facilite la création d'un projet Pipeline dans Jenkins.

Vous pouvez générer un Pipeline à partir d'un fichier `Jenkinsfile` existant dans le contrôle de source, ou vous pouvez utiliser [l'éditeur Blue Ocean Pipeline](./blue-ocean-editeur-pipeline.md) pour créer un Pipeline sous forme de fichier `Jenkinsfile` qui est validé dans le contrôle de source.

!!! info "_Statut de Blue Ocean_"
    Blue Ocean ne bénéficiera plus de mises à jour fonctionnelles. Blue Ocean continuera à fournir une visualisation facile à utiliser du Pipeline, mais il ne sera plus amélioré. Il ne bénéficiera que de mises à jour sélectives pour les problèmes de sécurité importants ou les défauts fonctionnels.

    D'autres options de visualisation du Pipeline, telles que les plugins [Pipeline: Stage View](https://plugins.jenkins.io/pipeline-stage-view/) et [Pipeline Graph View](https://plugins.jenkins.io/pipeline-graph-view/), sont disponibles et offrent certaines des mêmes fonctionnalités. Bien qu'elles ne remplacent pas complètement Blue Ocean, les contributions de la communauté sont encouragées pour le développement continu de ces plugins.

    Le [générateur d'extraits de syntaxe Pipeline](./pipeline-commencer.md#générateur-dextraits) aide les utilisateurs à définir les étapes du Pipeline avec leurs arguments. Il s'agit de l'outil privilégié pour la création de Pipelines Jenkins, car il fournit une aide en ligne pour les étapes du Pipeline disponibles dans votre contrôleur Jenkins. Il utilise les plugins installés sur votre contrôleur Jenkins pour générer la syntaxe du Pipeline. Reportez-vous à la page de [référence des étapes du Pipeline](https://www.jenkins.io/doc/pipeline/steps/) pour obtenir des informations sur toutes les étapes disponibles.

## Configuration de votre projet Pipeline

Pour commencer à configurer votre projet Pipeline dans Blue Ocean, sélectionnez le bouton **New Pipeline**  (Nouveau Pipeline) en haut à droite du tableau de bord Blue Ocean.

<img src="https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/new-pipeline-button.png" alt="Bouton Nouveau Pipeline" width="450" height="250">

Si votre instance Jenkins est nouvelle ou ne comporte aucun projet Pipeline ou autre élément configuré, Blue Ocean affiche un message **Welcome to Jenkins** [Bienvenue dans Jenkins) qui vous permet de sélectionner l'option **Create a new Pipeline** (Créer un nouveau Pipeline) pour commencer à configurer votre projet Pipeline.

<img src="https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/create-a-new-pipeline-box.png" alt="Boîte de message Bienvenue dans Jenkins - Créer un nouveau Pipeline" width="450" height="350">

Vous avez maintenant le choix de créer votre nouveau projet Pipeline à partir d'un :

* [Référentiel Git standard](#pour-un-referentiel-git) ;
* [Référentiel sur GitHub](#pour-un-referentiel-github) ou GitHub Enterprise ;
* [Référentiel sur Bitbucket Cloud](#pour-un-referentiel-bitbucket-cloud) ou Bitbucket Server.

### Pour un dépôt Git

Pour créer votre projet Pipeline pour un dépôt Git, cliquez sur le bouton **Git** sous **Where do you store your code** (Où stockez-vous votre code ?) 

<img src="https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/where-do-you-store-your-code.png" alt="Où stockez-vous votre code ?" width="450" height="350">

Dans la section **Connect to a Git repository** (Se connecter à un dépôt Git), entrez l'URL de votre dépôt Git dans le champ **Repository URL** (URL du dépôt).

<img src="https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/connect-to-a-git-repository.png" alt="Se connecter à un dépôt Git" width="450" height="350">

Vous devez maintenant spécifier un dépôt [local](#depot-local) ou [distant](#depot-distant) à partir duquel créer votre projet Pipeline.

#### Dépôt local

Si votre URL est un chemin d'accès à un répertoire local commençant par une barre oblique `/`, tel que `/home/cloned-git-repos/my-git-repo.git`, vous pouvez sélectionner l'option **Create Pipeline** (Créer un Pipeline).

Blue Ocean analyse alors les branches de votre référentiel local à la recherche d'un fichier `Jenkinsfile` et lance une exécution de Pipeline pour chaque branche contenant un fichier `Jenkinsfile`. Si Blue Ocean ne trouve pas de fichier `Jenkinsfile`, vous êtes invité à en créer un via [l'éditeur de Pipeline](./blue-ocean-editeur-pipeline.md).

Les référentiels locaux sont généralement limités à l'accès au système de fichiers et ne sont normalement disponibles qu'à partir du nœud contrôleur. Les référentiels locaux sont également connus pour nécessiter des noms de chemin d'accès plus complexes sous Windows que la plupart des utilisateurs ne souhaitent gérer. Il est conseillé aux utilisateurs d'exécuter les tâches sur des agents plutôt que directement sur le contrôleur. Par conséquent, vous devez utiliser un référentiel distant plutôt qu'un référentiel local pour profiter au mieux de Blue Ocean.

#### Dépôt  distant

Étant donné que l'éditeur de Pipeline enregistre les Pipelines modifiés dans des référentiels Git sous forme de fichiers 'Jenkinsfile', Blue Ocean ne prend en charge que les connexions à des référentiels Git distants via le protocole SSH.

Si votre URL correspond à un référentiel Git distant, assurez-vous qu'elle commence par :

* `ssh://` - qui s'affiche sous la forme `ssh://gituser@git-server-url/git-server-repos-group/my-git-repo.git`<br>
    ou<br>
* `user@host:path/to/git/repo.git` - qui s'affiche sous la forme `gituser@git-server-url:git-server-repos-group/my-git-repo.git`

Blue Ocean génère automatiquement une paire de clés SSH publique/privée ou vous fournit une paire existante pour l'utilisateur Jenkins actuel. Ces informations d'identification sont automatiquement enregistrées dans Jenkins avec les détails suivants pour cet utilisateur Jenkins :

* **Domaine** : `blueocean-private-key-domain` ;
* **ID** : `jenkins-generated-ssh-key` ;
* **Nom** : `<jenkins-username> (jenkins-generated-ssh-key)`.

Vous devez vous assurer que cette paire de clés SSH publique/privée est enregistrée sur votre serveur Git avant de continuer.

Si vous ne l'avez pas encore fait, suivez ces deux étapes.

1. Configurez le composant de clé publique SSH de cette paire de clés (que vous pouvez copier-coller depuis l'interface Blue Ocean) pour le compte utilisateur du serveur Git distant (par exemple, dans le fichier `authorized_keys` du répertoire `gituser/.ssh` de la machine).

    !!! info
        Ce processus permet à votre utilisateur Jenkins d'accéder aux référentiels auxquels le compte utilisateur de votre serveur Git a accès. Reportez-vous à la section « [Configuration du serveur](https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server) » de la [documentation Pro Git](https://git-scm.com/book/en/v2/).

2. Revenez à l'interface Blue Ocean.

Sélectionnez l'option **Create Pipeline** (Créer un Pipeline).

Blue Ocean analyse les branches de votre référentiel local à la recherche d'un fichier `Jenkinsfile` et lance une exécution de pipeline pour chaque branche contenant un fichier `Jenkinsfile`. Si Blue Ocean ne trouve pas de fichier `Jenkinsfile`, vous êtes invité à en créer un via [l'éditeur de Pipeline](./blue-ocean-editeur-pipeline.md).

### Pour un référentiel sur GitHub

Pour créer votre projet Pipeline directement pour un référentiel sur GitHub, sélectionnez l'option **GitHub** sous **Where do you store your code?.** (Où stockez-vous votre code ?).

<img src="https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/where-do-you-store-your-code.png" alt="Où stockez-vous votre code ?" width="450" height="350">

Dans la section **Connect to GitHub** (Se connecter à GitHub), entrez votre jeton d'accès GitHub dans le champ **Your GitHub access token** (Votre jeton d'accès GitHub).
Si vous avez précédemment configuré Blue Ocean pour se connecter à GitHub à l'aide d'un jeton d'accès personnel, Blue Ocean vous amène directement aux étapes de [choix du compte/de l'organisation et du référentiel GitHub](#choix-du-compte-github) ci-dessous :

<img src="https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/connect-to-github.png" alt="Se connecter à GitHub" width="450" height="350">

Si vous ne disposez pas d'un jeton d'accès GitHub, sélectionnez l'option **Create an access key here** (Créer une clé d'accès ici) pour ouvrir GitHub sur la page [Nouveau jeton d'accès personnel](#creer-votre-cle-dacces).

#### Créez votre cle d'accès

1. Dans le nouvel onglet, connectez-vous à votre compte GitHub. Sur la page **New Personal Access Token** (Nouveau jeton d'accès personnel) , spécifiez une brève **Token description** (description du jeton) pour votre cle d'accès GitHub, par exemple Blue Ocean ;

    !!! info
        Un jeton d'accès est généralement une chaîne alphanumérique qui représente votre compte GitHub, ainsi que les autorisations d'accès à diverses fonctionnalités et zones GitHub via votre compte GitHub. Le nouveau processus de création de jeton d'accès, lancé en sélectionnant ** Create an access key here** (Créer une clé d'accès ici), dispose des autorisations appropriées présélectionnées dont Blue Ocean a besoin pour accéder à votre compte GitHub et interagir avec celui-ci.

2. Faites défiler vers le bas jusqu'à la fin de la page, puis sélectionnez**Generate token** (Générer un jeton) ;
3. Sur la page **Personal access tokens** (Jetons d'accès personnels) qui s'affiche, copiez votre jeton d'accès nouvellement généré ;
4. De retour dans Blue Ocean, collez le jeton d'accès dans le champ **Your GitHub access token** (Votre jeton d'accès GitHub), puis sélectionnez **Connect** (Se connecter).

Votre utilisateur Jenkins actuel a désormais accès à votre compte GitHub et vous pouvez désormais [choisir votre compte/organisation GitHub et votre référentiel](#choix-du-compte-github). Jenkins enregistre ces informations d'identification avec les détails suivants pour cet utilisateur Jenkins :

* **Domaine** : `blueocean-github-domain` ;
* **ID** : `github` ;
* **Nom** : `<jenkins-username>/****** (jeton d'accès GitHub)`.

#### Choix du compte GitHub

Blue Ocean vous invite à choisir votre compte GitHub ou une organisation dont vous êtes membre. Il vous demande également de sélectionner le référentiel contenant votre projet Pipeline.

1. Dans la section **Which organization does the repository belong to?** (À quelle organisation le référentiel appartient-il ?), sélectionnez l'une des options suivantes :
    * Votre compte GitHub, pour créer un projet Pipeline pour l'un de vos propres référentiels GitHub ou pour un référentiel que vous avez _forké_ à partir d'un autre emplacement sur GitHub<br>
        ou<br>
    * L'organisation dont vous êtes membre, pour créer un projet Pipeline pour un référentiel GitHub situé au sein de cette organisation.
2. Dans la section **Choose a repository** (Choisissez un référentiel), sélectionnez le référentiel de votre compte ou organisation GitHub à partir duquel vous souhaitez créer votre projet Pipeline.

    !!! tip "Astuce"
        Si votre liste de référentiels est longue, vous pouvez utiliser l'option Rechercher pour filtrer vos résultats.

<img src="https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/choose-a-repository.png" alt="Choisissez un référentiel" width="400" height="350">

1. Cliquez sur **Create Pipeline** (Créer un pipeline).

Blue Ocean analyse les branches de votre référentiel local à la recherche d'un fichier Jenkinsfile et lance une exécution de Pipeline pour chaque branche contenant un fichier `Jenkinsfile`. Si Blue Ocean ne trouve pas de fichier `Jenkinsfile`, vous êtes invité à en créer un via [l'éditeur de Pipeline](#editeur-de-pipeline).

!!! info " "
    En réalité, un projet de Pipeline créé via Blue Ocean est en fait un « Pipeline multibranches ». Par conséquent, Jenkins recherche la présence d'au moins un fichier Jenkinsfile dans n'importe quelle branche de votre référentiel.

### Pour un référentiel sur Bitbucket Cloud

Pour créer votre projet Pipeline directement pour un référentiel Git ou Mercurial sur Bitbucket Cloud, sélectionnez le bouton **Bitbucket Cloud**sous **Where do you store your code?** (Où stockez-vous votre code ?).

<img src="https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/where-do-you-store-your-code.png" alt="Où stockez-vous votre code ?" width="400" height="350">

Dans la section **Connect to Bitbucket** (Se connecter à Bitbucket), entrez votre adresse e-mail et votre mot de passe Bitbucket dans les champs **Username** (Nom d'utilisateur) et **Password** (Mot de passe).

* Si vous avez précédemment configuré Blue Ocean pour vous connecter à Bitbucket avec votre adresse e-mail et votre mot de passe, Blue Ocean vous redirige directement vers les étapes de [sélection du compte/de l'équipe et du référentiel Bitbucket](#choix-du-compte-bitbucket) ci-dessous ;
* Si vous avez saisi ces informations d'identification, Jenkins les enregistre avec les détails suivants pour cet utilisateur Jenkins :
    * **Domaine** : `blueocean-bitbucket-cloud-domain` ;
    * **ID** : `bitbucket-cloud` ;
    * **Nom** : `+<bitbucket-user@email.address>/ (identifiants du serveur Bitbucket)`.

<img src="https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/connect-to-bitbucket.png" alt="Se connecter à Bitbucket" width="450" height="350">

Sélectionnez **Connect** (Se connecter) et votre utilisateur Jenkins actuel/connecté aura désormais accès à votre compte Bitbucket. Vous pouvez maintenant choisir votre compte/équipe Bitbucket et votre référentiel.

#### Choix du compte Bitbucket

Blue Ocean vous invite à choisir votre compte Bitbucket ou une équipe dont vous êtes membre, ainsi que le référentiel contenant votre projet à créer.

1. Dans la section **Which team does the repository belong to** (À quelle équipe appartient le référentiel ?), sélectionnez l'une des options suivantes :
    * Votre compte Bitbucket pour créer un projet Pipeline pour l'un de vos propres référentiels Bitbucket ou pour un référentiel que vous avez _forké_ à partir d'un autre emplacement sur Bitbucket ;
    * Une équipe dont vous êtes membre pour créer un projet Pipeline pour un référentiel Bitbucket situé au sein de cette équipe.
2. Dans la section **Choose a repository** (Choisissez un référentiel), sélectionnez le référentiel de votre compte ou équipe Bitbucket à partir duquel vous souhaitez créer votre projet Pipeline.

!!! tip "Astuce"
    Si votre liste de référentiels est longue, vous pouvez la filtrer à l'aide de l'option **Search** (Rechercher).

<img src="https://www.jenkins.io/doc/book/resources/blueocean/creating-pipelines/choose-a-repository.png" alt="Choisissez un référentiel" width="400" height="350">

3. Cliquez sur **Create Pipeline** (Créer un pipeline).

Blue Ocean analyse les branches de votre référentiel local à la recherche d'un fichier `Jenkinsfile` et lance une exécution Pipeline pour chaque branche contenant un fichier `Jenkinsfile`. Si Blue Ocean ne trouve pas de fichier `Jenkinsfile`, vous êtes invité à en créer un via [l'éditeur Pipeline](./blue-ocean-editeur-pipeline.md).

!!! info " "
    En réalité, un projet Pipeline créé via Blue Ocean est en fait un « Pipeline multibranches ». Par conséquent, Jenkins recherche la présence d'au moins un fichier Jenkinsfile dans n'importe quelle branche de votre référentiel. 

