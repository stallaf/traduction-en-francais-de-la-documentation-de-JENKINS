# Proxy inverse - Pomerium

<div class="couleur-introduction">
Vous pouvez sécuriser votre application Jenkins avec l'authentification JWT et des revendications personnalisées derrière Pomerium, un proxy inverse open source.
</div>

## Pourquoi utiliser Pomerium avec Jenkins ?

Vous pouvez configurer des autorisations basées sur les rôles dans Jenkins pour contrôler les privilèges d'un utilisateur à l'aide de la matrice d'autorisation intégrée à Jenkins. Cependant, cette méthode nécessite un nom d'utilisateur et un mot de passe pour se connecter et s'appuie sur la base de données utilisateur de Jenkins pour stocker les informations d'identification.

L'authentification JWT est une méthode plus sécurisée de vérification d'identité qui authentifie et autorise les utilisateurs auprès d'un fournisseur d'identité, éliminant ainsi le besoin de stocker ou de partager des identifiants pour accéder à votre application Jenkins.

Cependant, Jenkins ne prend pas en charge l'authentification JWT dès l'installation. Avec Pomerium, vous pouvez mettre en œuvre l'authentification JWT et appliquer des revendications à la politique d'autorisation de votre route afin de déterminer le rôle et les privilèges d'un utilisateur avant de lui accorder l'accès à Jenkins.

Une fois l'authentification JWT configurée, vous pouvez attribuer des autorisations dans Jenkins à un utilisateur spécifique, à tout utilisateur authentifié, à des utilisateurs anonymes ou à un groupe d'utilisateurs.

## Sécurisez Jenkins avec le proxy Pomerium

Remarque : ce guide suppose que vous utilisez la version 0.21 de Pomerium | Veuillez vous reporter au [guide Pomerium sur la sécurisation de Jenkins](https://www.pomerium.com/docs/guides/jenkins) pour obtenir la dernière version de ce guide.

### Avant de commencer

Pour suivre ce guide, vous avez besoin :

1. de [Docker](https://docs.docker.com/get-docker/) et[ Docker Compose](https://docs.docker.com/compose/install/) ;
2. d'un [fournisseur d'identité](https://www.pomerium.com/docs/identity-providers) (IdP).

Si ce n'est pas déjà fait, suivez le guide de [démarrage rapide Pomerium Core ](https://www.pomerium.com/docs/quickstart)pour vous familiariser avec l'exécution de Pomerium dans un environnement conteneurisé. Ce guide est gratuit et ne devrait pas vous prendre plus de cinq minutes pour mettre en place le proxy open source Pomerium.

## Exécutez Jenkins avec Docker Compose

Créez un projet et ajoutez un fichier appelé `docker-compose.yaml`.

Dans le fichier `docker-compose.yaml`, ajoutez le code suivant :

``` YAML title="YAML titre=DOCKER-COMPOSE.YAML"
jenkins:
  networks:
    main: {}
  image: jenkins/jenkins:lts-jdk21
  privileged: true
  user: root
  ports:
    - 8080:8080
    - 50000:50000

  volumes:
    # Chemin d'accès au fichier Jenkins_home -- stocke les configurations, les journaux de compilation et les artefacts
    - ./home/jenkins_compose/jenkins_configuration:/var/jenkins_home
    # « sock » est le socket Unix que le démon Docker écoute par défaut
    - ./var/run/docker.sock:/var/run/docker.sock
```

Maintenant, exécutez `docker compose up`.

## Configurez votre contrôleur Jenkins.

Si votre conteneur Jenkins est correctement configuré, l'assistant de configuration vous guide à travers plusieurs invites avant que vous puissiez accéder à votre tableau de bord.

Pour configurer votre contrôleur Jenkins :

1. Accédez à `localhost:8080` et entrez le mot de passe utilisateur admin pour continuer. Vous trouverez le mot de passe utilisateur admin dans vos journaux Docker ou dans `/var/jenkins_home/secrets/initialAdminPassword`.
2. Installez les plugins suggérés.
3. Créez un utilisateur administrateur. Vous pouvez créer votre premier utilisateur administrateur ou sélectionner  **Skip and continue as admin** (Ignorer et continuer en tant qu'administrateu). Si vous ignorez et continuez en tant qu'administrateur, le nom d'utilisateur par défaut est **admin** et le mot de passe est le mot de passe administrateur.
4. Dans la fenêtre **Instance Configuration** (Configuration de l'instance), acceptez le nom d'hôte par défaut.

Une fois les invites de l'assistant de configuration terminées, vous pouvez accéder au tableau de bord Jenkins.

Maintenant, exécutez `docker compose stop` afin de pouvoir configurer Pomerium.

## Configurez Pomerium

### Créez un fichier de configuration Pomerium

Dans le dossier racine de votre projet, créez un fichier `config.yaml`.

Dans votre fichier `config.yaml`, ajoutez le code suivant :

``` yaml title="YAML titre=CONFIG.YAML"
authenticate_service_url : https://authenticate.localhost.pomerium.io

idp_provider : REPLACE_ME
idp_provider_url : REPLACE_ME
idp_client_id : REPLACE_ME
idp_client_secret : REPLACE_ME

signing_key : REPLACE_ME

routes :
 - from: https://verify.localhost.pomerium.io
   to: http://verify:8000
   pass_identity_headers: true
   policy:
     - allow:
         and:
    email:
          is: user@example.com
 - from: https://jenkins.localhost.pomerium.io
   to: http://jenkins:8080/
   host_rewrite_header: true
   pass_identity_headers: true
   policy:
     - allow:
         and:
           - domain:
               is: example.com
           - user:
               is: username
```

Ensuite, vous :

* Mettez à jour les variables de configuration du [fournisseur d'identité](https://www.pomerium.com/docs/identity-providers) avec les vôtres ;
* Remplacez `user@example.com` par l'adresse e-mail associée à votre IdP ;
* Remplacez `example.com` par le nom de domaine de votre organisation ;
* Remplacez `username` par le nom d'utilisateur associé à votre IdP ;
* Générez une clé de signature.

Pour générer une `clé de signature`, utilisez les commandes ci-dessous :

``` bash title="BASH"
# Génère une clé de signature P-256 (ES256)
openssl ecparam  -genkey  -name prime256v1  -noout  -out ec_private.pem
# Affiche la valeur encodée en base64 de la clé de signature
cat ec_private.pem | base64
```

Ajoutez la clé de signature encodée en base64 à la variable `signing_key` dans votre fichier `config.yaml`.

### Exécutez les services Pomerium avec Docker Compose

Dans votre fichier `docker-compose.yaml`, remplacez le code du fichier par les services Pomerium et Jenkins ci-dessous :

``` cfg
version : “3”
réseaux :
 principal : {}
services :
 pomerium :
   image : pomerium/pomerium:latest
   volumes :
     - ./config.yaml:/pomerium/config.yaml:ro
   ports :
     - 443:443
   réseaux :
     principal :
       alias :
       - authenticate.localhost.pomerium.io
 vérifier :
   réseaux :
     principal : {}
   image : pomerium/verify:latest
   exposer :
     - 8000
 jenkins :
   réseaux :
     principal : {}
   image : jenkins/jenkins:lts-jdk21
   privilégié : vrai
   utilisateur : root
   ports :
     - 8080:8080
     - 50000:50000
   volumes :
     # Chemin d'accès au fichier Jenkins_home -- stocke les configurations, les journaux de compilation et les artefacts
     - ./home/jenkins_compose/jenkins_configuration:/var/jenkins_home
     # « sock » est le socket Unix que le démon Docker écoute par défaut
```

Exécutez `docker compose up` et accédez à la route Jenkins externe à l'adresse [https://jenkins.localhost.pomerium.io](https://jenkins.localhost.pomerium.io/).

Jenkins vous demandera de vous connecter avec votre nom d'utilisateur et votre mot de passe. Connectez-vous pour accéder au tableau de bord Jenkins.

## Installez les plugins Jenkins

Ensuite, vous devez ajouter des plugins pour activer l'authentification JWT et contourner la validation TLS.

Installez le [plugin **JWT Auth**](https://plugins.jenkins.io/jwt-auth/) :

1. Sélectionnez **Manage Jenkins** (Gérer Jenkins).
2. Sous **System Configuration** (Configuration du système), sélectionnez **Manage Plugins** (Gérer les plugins).
3. Sélectionnez **Available Plugins** (Plugins disponibles).
4. Dans la barre de recherche, saisissez **JWT Auth.**
5. Sélectionnez le plugin JWT Auth et **Install without restart** (Installer sans redémarrer).

Installez le [plugin **skip-certificate-check**](https://plugins.jenkins.io/skip-certificate-check/) :

1. Sélectionnez **Available Plugins** (Plugins disponibles).
2. Dans la barre de recherche, saisissez **skip-certificate-check**.
3. Sélectionnez le plugin skip-certificate-check et **Install without restart** (Installer sans redémarrer).

Une fois les deux plugins installés, arrêtez vos conteneurs.

## Configurez l'authentification JWT.

Accédez à votre route Jenkins externe.

Pour configurer l'authentification JWT :

1. Accédez à **Manage Jenkins** (Gérer Jenkins).
2. Sous **Security** (Sécurité), sélectionnez **Configure Global Security** (Configurer la sécurité globale.
3. Sous **Authentication > Security Realm,** (Authentification > Domaine de sécurité) , sélectionnez **JWT Header Authentication Plugin** (Plugin d'authentification d'en-tête JWT).

Sous **Global JWT Auth Settings**, vous verrez des champs de formulaire dans lesquels vous pouvez saisir les revendications JWT. Pomerium transmet les [informations d'identité](https://www.pomerium.com/docs/capabilities/getting-users-identity#jwt-verification) associées à un utilisateur dans une attestation JWT signée qui est incluse dans les requêtes en amont dans un en-tête `X-Pomerium-Jwt-Assertion`.

Une fois le plugin [JWT Auth](https://plugins.jenkins.io/jwt-auth) installé, Jenkins peut recevoir et analyser l'en-tête d'attestation pour authentifier les utilisateurs. Il vous suffit de lui donner les instructions appropriées pour trouver l'en-tête et les revendications JWT.

Saisissez les informations suivantes dans le champ **Global JWT Auth Settings** (Paramètres d'authentification JWT globaux) :

| Champ         | Valeur            | 
| ------------------ | ------------------ | 
| Header name (Nom d'en-tête) | `x-pomerium-jwt-assertion` |
| Username claim name (Nom de revendication du nom d'utilisateur) | `name` or `email` |
| Groups claim name (Nom de revendication des groupes) | `groups` |
| Groups claim list separator (Séparateur de liste de revendications des groupes) | `,` |
| Email claim name (Nom de revendication de l'e-mail) | `email` |
| Acceptable issuers (Émetteurs acceptables) | `authenticate.corp.example.com` |
| Acceptable audiences (Audiences acceptables) | `jenkins.corp.example.com` |
| JWKS JSON URL | [https://jenkins.corp.example.com/.well-known/pomerium/jwks.json](https://jenkins.corp.example.com/.well-known/pomerium/jwks.json) |

Veuillez noter les détails suivants concernant les champs ci-dessus :

* **Username claim name** (Le nom de revendication du nom d'utilisateur) peut être votre nom ou votre adresse e-mail ;
* **Acceptable issuers** (Les émetteurs acceptables) doivent être l'URL du domaine d'authentification qui a émis le JWT. La revendication iss indique à l'application cible qui est l'autorité émettrice et fournit des informations contextuelles sur le sujet ;
* **Acceptable audiences** (Les audiences acceptables) doivent être l'URL de l'application cible. La revendication aud définit à quelle application le JWT est destiné ;
* L'**URL JSON JWKS** ajoute `/.well-known/pomerium/jwks.json` à l'URL de la route externe. Le point de terminaison JWKS fournit à Jenkins la clé publique de l'utilisateur afin de vérifier sa signature JWT.

Vous pouvez accéder à la route `verify` externe définie dans votre politique pour afficher vos revendications JWT.

Dans le menu déroulant **Authorization** (Autorisation), configurez les autorisations Jenkins afin que les utilisateurs **Anonymous** (anonymes) disposent des privilèges **Administer** (d'administration).

1. Sélectionnez **Matrix-based security** (Sécurité basée sur une matrice).
2. Sous **Overall** Général, attribuez les privilèges **Administer** (d'administration) aux utilisateurs **Anonymous** (anonymes) et aux **Authenticated Users** (utilisateurs authentifiés).

Si l'authentification JWT ne vous authentifie pas correctement, Jenkins vous connecte en tant qu'utilisateur anonyme. Avec les privilèges d'administration, vous pouvez dépanner les paramètres JWT en tant qu'utilisateur anonyme et réessayer.

Sélectionnez **Save** (Enregistrer) pour appliquer les paramètres de sécurité.

### Testez l'authentification JWT

Redémarrez votre conteneur. Si l'authentification JWT a fonctionné, votre nom apparaît dans le tableau de bord à la place de « admin ». Pour obtenir plus de détails sur la requête, ajoutez `/whoAmI` à l'URL. Par exemple, [https://jenkins.localhost.pomerium.io/whoAmI](https://jenkins.localhost.pomerium.io/whoAmI).

## Mettez à jour vos paramètres d'autorisation Jenkins

Vous pouvez maintenant configurer vos paramètres d'autorisation Jenkins :

1. Sélectionnez **Matrix-based security** (Sécurité basée sur une matrice).
2. Sélectionnez **Add user…** (Ajouter un utilisateur…) et saisissez le nom ou l'adresse e-mail associé(e) à votre IdP (la valeur dépend de la revendication que vous avez saisie pour le **Username claim name** (nom de revendication du nom d'utilisateur)).

Attribuez-vous les privilèges **Administer** (d'administrateur) et tous les privilèges qui vous semblent appropriés pour les **Authenticated Users** (utilisateurs authentifiés) et les utilisateurs **Anonymous** (anonymes).

Sélectionnez **save** (enregistrer) pour appliquer les modifications de sécurité.

## Étapes suivantes : ajoutez plus de contexte à vos politiques

Vous pouvez ajuster la politique d'autorisation dans Jenkins pour limiter ou élargir les privilèges dont disposent les utilisateurs authentifiés et anonymes, mais vous pouvez également étendre vos politiques d'autorisation avec Pomerium.

Par exemple :

* Vous pouvez créer une politique qui autorise uniquement les utilisateurs à accéder à Jenkins à certaines heures de la journée ou certains jours de la semaine, ou limiter l'accès à certains appareils ;
* Vous pouvez importer des revendications de groupes personnalisés depuis votre IdP et n'autoriser l'accès qu'aux membres du groupe.
