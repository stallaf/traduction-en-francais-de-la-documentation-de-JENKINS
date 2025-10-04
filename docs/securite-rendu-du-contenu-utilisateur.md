# Rendu du Contenu Utilisateur

<div class="couleur-traduction">
De nombreuses fonctionnalités de Jenkins, ainsi que beaucoup d'autres dans les plugins, fournissent des fichiers qui peuvent être consultés ou téléchargés à partir de Jenkins. Parmi les exemples intégrés, on peut citer le navigateur d'espace de travail, les artefacts archivés, les paramètres de fichier pour les builds ou le répertoire `/userContent/`. Les plugins tels que [Javadoc](https://plugins.jenkins.io/javadoc), [HTML Publisher](https://plugins.jenkins.io/htmlpublisher) ou [Maven Integration](https://plugins.jenkins.io/maven-plugin) (lors de la publication du site Maven) offrent des fonctionnalités importantes qui fournissent du code HTML contrôlé par les utilisateurs à partir de Jenkins.

Cela peut présenter un risque, car des attaques de type [Cross-Site Scripting](https://owasp.org/www-community/attacks/xss/) pourraient être introduites dans ces fichiers HTML par des personnes ayant une influence sur les builds et qui ne sont pas forcément dignes de confiance.
</div>

## Politique de sécurité du contenu

Par défaut, Jenkins fournit des fichiers pouvant provenir de sources moins fiables avec un en-tête de réponse HTTP `Content-Security-Policy` stricte. Ce paramètre par défaut empêche tous les éléments JavaScript et autres éléments actifs, et n'autorise que les CSS et les images fournis à partir d'autres fichiers dans Jenkins.

Bien que cela soit sûr, cela empêche également de nombreuses fonctionnalités utiles de fonctionner, telles que les rapports HTML riches et dynamiques créés pendant les builds. Il est possible de [configurer Content-Security-Policy](./securite-configurer-content-security-policy.md). Il s'agit souvent d'un compromis difficile entre fonctionnalité et sécurité, qui ne doit donc être effectué qu'avec beaucoup de prudence.

## URL racine des ressources

Au lieu d'assouplir la `Content-Security-Policy,` les administrateurs peuvent configurer Jenkins pour qu'il serve des fichiers provenant de sources potentiellement moins fiables à partir d'un autre domaine. Cette option peut être configurée dans _Manage Jenkins » System_, dans la section _Serve resource files from another domain_.

Si l'URL racine de la ressource est définie, Jenkins redirigera les requêtes pour les fichiers de ressources créés par l'utilisateur vers des URL commençant par l'URL configurée ici. Ces URL ne définiront pas l'en-tête CSP, ce qui permettra à JavaScript et aux fonctionnalités similaires de fonctionner. Pour que cette option fonctionne comme prévu, les contraintes et considérations suivantes s'appliquent :

* L'URL racine de la ressource doit être un choix alternatif valide pour l'URL Jenkins afin que les requêtes soient traitées correctement ;
* L'URL Jenkins doit être définie et différente de cette URL racine de la ressource (en fait, un nom d'hôte différent est requis) ;
* Une fois définie, Jenkins ne servira que les requêtes d'URL de ressources via l'URL racine de la ressource. Toutes les autres requêtes recevront des réponses HTTP 404 Not Found.

Une fois cette URL correctement configurée, Jenkins redirigera les requêtes vers les espaces de travail, les artefacts archivés et les collections similaires de contenu généralement généré par les utilisateurs vers des URL commençant par l'URL racine de la ressource. Au lieu d'un chemin tel que `job/name_here/ws`, les URL de ressources contiendront un jeton codant ce chemin, l'utilisateur pour lequel l'URL a été créée et la date de création. Ces URL de ressources accèdent aux fichiers statiques comme si l'utilisateur pour lequel elles ont été créées y accédait : si l'autorisation d'accès à ces fichiers est supprimée pour l'utilisateur, les URL de ressources correspondantes ne fonctionneront plus non plus. **Ces URL sont accessibles à tous sans authentification jusqu'à leur expiration. Le partage de ces URL revient donc à partager directement les fichiers.**

### Considérations relatives à la sécurité

#### Authentification

Les URL de ressources ne nécessitent pas d'authentification (les utilisateurs n'auront pas de session valide pour l'URL racine de la ressource). Le partage d'une URL de ressource avec un autre utilisateur, même s'il ne dispose pas des autorisations globales/de lecture pour Jenkins, lui permettra d'accéder à ces fichiers jusqu'à l'expiration des URL.

#### Expiration

Les URL de ressources expirent après 30 minutes par défaut. Les URL de ressources expirées redirigent les utilisateurs vers leurs URL Jenkins équivalentes, afin que l'utilisateur puisse se réauthentifier, si nécessaire, puis être redirigé vers une nouvelle URL de ressource qui sera valide pendant 30 minutes supplémentaires. Cela sera généralement transparent pour l'utilisateur s'il dispose d'une session Jenkins valide.  Sinon, ils devront s'authentifier à nouveau auprès de Jenkins. Cependant, lors de la navigation sur des pages avec des cadres HTML, comme les sites Javadoc, le formulaire de connexion ne peut pas apparaître dans un cadre. Dans ce cas, les utilisateurs devront recharger le cadre de niveau supérieur pour faire apparaître le formulaire de connexion.

Pour modifier la durée d'expiration des URL de ressources, définissez la propriété système `jenkins.security.ResourceDomainRootAction.validForMinutes` sur la valeur souhaitée en minutes. Une expiration plus rapide peut rendre l'utilisation de ces URL plus difficile, tandis qu'une expiration plus tardive augmente le risque que des utilisateurs non autorisés accèdent aux URL partagées avec eux par des utilisateurs autorisés.

#### Authenticité

Les URL de ressources encodent l'URL, l'utilisateur pour lequel elles ont été créées et leur horodatage de création. De plus, cette chaîne contient un [HMAC](https://en.wikipedia.org/wiki/HMAC) pour garantir l'authenticité de l'URL. Cela empêche les attaquants de falsifier des URL qui leur donneraient accès aux fichiers de ressources comme s'ils étaient un autre utilisateur.

