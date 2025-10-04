# Gestion de la Sécurité

<div class="couleur-introduction">
Jenkins est utilisé partout, des postes de travail sur les intranets d'entreprise aux serveurs haute performance connectés à l'Internet public. Afin de prendre en charge en toute sécurité cette large gamme de profils de sécurité et de menaces, Jenkins offre de nombreuses options de configuration permettant d'activer, de modifier ou de désactiver diverses fonctionnalités de sécurité.
</div>

À partir de Jenkins 2.0, de nombreuses options de sécurité ont été activées par défaut afin de garantir la sécurité des environnements Jenkins, sauf si un administrateur désactive explicitement certaines protections.

Cette section présente les différentes options de sécurité disponibles pour un administrateur Jenkins, explique les protections offertes et les compromis liés à la désactivation de certaines d'entre elles.

## Activation de la sécurité


À partir des versions Jenkins 2.214 et Jenkins LTS 2.222.1, la case à cocher « Enable Security » (Activer la sécurité) a été supprimée. La base de données utilisateur propre à Jenkins est utilisée comme domaine de sécurité par défaut.

Dans les versions antérieures à Jenkins 2.214 et Jenkins LTS 2.222.1, lorsque la case **Enable Security** (Activer la sécurité) est cochée, les utilisateurs peuvent se connecter avec un nom d'utilisateur et un mot de passe afin d'effectuer des opérations qui ne sont pas disponibles pour les utilisateurs anonymes. Les opérations qui nécessitent que les utilisateurs se connectent dépendent de la stratégie d'autorisation choisie et de sa configuration ; par défaut, les utilisateurs anonymes n'ont aucune autorisation, et les utilisateurs connectés ont un contrôle total. La case « Enable Security » doit toujours être cochée pour tout environnement Jenkins non local (test).

La section « Security » de l'interface utilisateur Web permet à un administrateur Jenkins d'activer, de configurer ou de désactiver les principales fonctionnalités de sécurité qui s'appliquent à l'ensemble de l'environnement Jenkins.

![Sécurité](https://www.jenkins.io/doc/book/resources/security/configure-global-security.png)

### Port TCP

Jenkins peut utiliser un port TCP pour communiquer avec des agents entrants (anciennement appelés « JNLP »), tels que les agents Windows. À partir de Jenkins 2.0, ce port est désactivé par défaut.

Pour les administrateurs souhaitant utiliser des agents TCP entrants, les deux options de port sont les suivantes :

* **Random** (Aléatoire) : le port TCP est choisi au hasard afin d'éviter les collisions sur le [contrôleur](./glossaire.md#controleur) Jenkins. L'inconvénient des ports aléatoires est qu'ils sont choisis lors du démarrage du contrôleur Jenkins, ce qui rend difficile la gestion des règles de pare-feu autorisant le trafic TCP ;
* **Fixed** (Fixe) : le port est choisi par l'administrateur Jenkins et reste le même à chaque redémarrage du [contrôleur]Jenkins. Cela facilite la gestion des règles de pare-feu autorisant les agents TCP à se connecter au contrôleur.

À partir de Jenkins 2.217, les agents entrants peuvent être configurés pour utiliser le transport WebSocket afin de se connecter à Jenkins. Dans ce cas, aucun port TCP supplémentaire ne doit être activé et aucune configuration de sécurité particulière n'est nécessaire.

### Contrôle d'accès

Le contrôle d'accès est le principal mécanisme permettant de sécuriser un environnement Jenkins contre toute utilisation non autorisée. Deux aspects de la configuration sont nécessaires pour configurer le contrôle d'accès dans Jenkins :

* **Un domaine de sécurité** qui indique à l'environnement Jenkins comment et où extraire les informations sur les utilisateurs (ou les identités). Également connu sous le nom d'« authentification » ;
* **La configuration de l'autorisation** qui indique à l'environnement Jenkins quels utilisateurs et/ou groupes peuvent accéder à quels aspects de Jenkins, et dans quelle mesure.

En utilisant à la fois les configurations Security Realm et Authorization, il est possible de configurer des schémas d'authentification et d'autorisation très souples ou très rigides dans Jenkins.

De plus, certains plugins, tels que le plugin [Role-based Authorization Strategy](https://plugins.jenkins.io/role-strategy), peuvent étendre les capacités de contrôle d'accès de Jenkins afin de prendre en charge des schémas d'authentification et d'autorisation encore plus nuancés.

#### Domaine de sécurité

Par défaut, Jenkins prend en charge plusieurs Security Realms différents :

**Déléguer au conteneur de servlets**<br>
Pour déléguer l'authentification à un conteneur de servlets exécutant le contrôleur Jenkins, tel que [Jetty](https://www.eclipse.org/jetty/). Si vous utilisez cette option, veuillez consulter la documentation relative à l'authentification du conteneur de servlets.

**Base de données utilisateur propre à Jenkins**<br>
Utilisez le magasin de données utilisateur intégré à Jenkins pour l'authentification au lieu de déléguer à un système externe. Cette option est activée par défaut avec les nouvelles installations Jenkins 2.0 ou ultérieures et convient aux environnements de petite taille.

**LDAP**<br>
Déléguez toute l'authentification à un serveur LDAP configuré, y compris les utilisateurs et les groupes. Cette option est plus courante pour les installations de plus grande envergure dans les organisations qui ont déjà configuré un fournisseur d'identité externe tel que LDAP. Elle prend également en charge les installations Active Directory.

!!! info " "
    Cette fonctionnalité est fournie par le [plugin LDAP](https://plugins.jenkins.io/ldap) qui n'est peut-être pas installé sur votre instance.

**Base de données des utilisateurs/groupes Unix**<br>
Délègue l'authentification à la base de données des utilisateurs au niveau du système d'exploitation Unix sous-jacent sur le contrôleur Jenkins. Ce mode permet également de réutiliser les groupes Unix pour l'autorisation. Par exemple, Jenkins peut être configuré de manière à ce que « tous les membres du groupe des `developpers` aient un accès administrateur ». Pour prendre en charge cette fonctionnalité, Jenkins s'appuie sur [PAM](https://en.wikipedia.org/wiki/Pluggable_Authentication_Modules), qui peut nécessiter une configuration externe à l'environnement Jenkins.

!! danger " "
    Unix permet à un utilisateur et à un groupe d'avoir le même nom. Afin de lever toute ambiguïté, utilisez le préfixe `@` pour forcer l'interprétation du nom comme un groupe. Par exemple, `@dev` désignerait le groupe `dev` et non l'utilisateur `dev`.

Les plugins peuvent fournir des domaines de sécurité supplémentaires qui peuvent être utiles pour intégrer Jenkins dans des systèmes d'identité existants, tels que :

* [Active Directory](https://plugins.jenkins.io/active-directory) ;
* [Authentification GitHub](https://plugins.jenkins.io/github-oauth).

#### Autorisation

Le domaine de sécurité, ou authentification, indique _qui_ peut accéder à l'environnement Jenkins. L'autre élément du puzzle est **Authorization**, qui indique ce _à quoi_ ils peuvent accéder dans l'environnement Jenkins. Par défaut, Jenkins prend en charge plusieurs options d'autorisation différentes :

**Tout le monde peut tout faire**<br>
Tout le monde dispose d'un contrôle total sur Jenkins, y compris les utilisateurs anonymes qui ne se sont pas connectés. N'utilisez pas ce paramètre pour autre chose que les contrôleurs Jenkins de test locaux.

**Mode hérité**<br>
Se comporte exactement comme Jenkins <1.164. À savoir, si un utilisateur a le rôle « admin », il se verra accorder le contrôle total du système, et sinon (y compris les utilisateurs anonymes) n'aura qu'un accès en lecture. **N'utilisez pas ce paramètre** pour autre chose que les contrôleurs Jenkins de test locaux.

**Les utilisateurs connectés peuvent tout faire**<br>
Dans ce mode, chaque utilisateur connecté obtient le contrôle total de Jenkins. En fonction d'une option avancée, les utilisateurs anonymes obtiennent un accès en lecture à Jenkins, ou aucun accès. Ce mode est utile pour forcer les utilisateurs à se connecter avant d'effectuer des actions, afin de disposer d'une piste d'audit des actions des utilisateurs.

**Sécurité basée sur une matrice**<br>
Ce schéma d'autorisation permet un contrôle granulaire des utilisateurs et des groupes autorisés à effectuer certaines actions dans l'environnement Jenkins (voir la capture d'écran ci-dessous).

**Stratégie d'autorisation matricielle basée sur les projets**<br>
Ce schéma d'autorisation est une extension de la sécurité basée sur une matrice qui permet de définir des listes de contrôle d'accès (ACL) supplémentaires **pour chaque projet** séparément dans l'écran de configuration du projet. Cela permet d'accorder à des utilisateurs ou groupes spécifiques l'accès uniquement à des projets spécifiques, plutôt qu'à tous les projets de l'environnement Jenkins. Les ACL définies avec l'autorisation matricielle basée sur les projets sont additives, de sorte que les autorisations d'accès définies dans l'écran Sécurité seront combinées avec les ACL spécifiques au projet.

!!!info " "
    La sécurité matricielle et la stratégie d'autorisation matricielle basée sur les projets sont fournies par le [plugin Matrix Authorization Strategy](https://plugins.jenkins.io/matrix-auth) et peuvent ne pas être installées sur votre Jenkins.

Pour la plupart des environnements Jenkins, la sécurité basée sur Matrix offre le plus haut niveau de sécurité et de flexibilité. Elle est donc recommandée comme point de départ pour les environnements de « production ».

![Sécurité - Autorisation Matrix](https://www.jenkins.io/doc/book/resources/security/configure-global-security-matrix-authorization.png)
_Figure 1. Sécurité basée sur Matrix_

Le tableau ci-dessus peut être assez large, car chaque colonne représente une autorisation fournie par le noyau Jenkins ou un plugin. En passant la souris sur une autorisation, vous obtiendrez plus d'informations à son sujet.

Chaque ligne du tableau représente un utilisateur ou un groupe (également appelé « rôle »). Cela inclut des entrées spéciales nommées « anonymous » (anonyme) et « authenticated » (authentifié). L'entrée « anonymous » représente les autorisations accordées à tous les utilisateurs non authentifiés qui accèdent à l'environnement Jenkins. L'entrée « authenticated » peut être utilisée pour accorder des autorisations à tous les utilisateurs authentifiés qui accèdent à l'environnement.

Les autorisations accordées dans la matrice sont cumulatives. Par exemple, si un utilisateur « kohsuke » fait partie des groupes « développeurs » et « administrateurs », les autorisations accordées à « kohsuke » seront alors la somme de toutes les autorisations accordées à « kohsuke », « développeurs », « administrateurs », « authentifiés » et « anonymes ».

### Formatage du balisage

Voir [Formatage du balisage](./securite-formatage-des-balises.md).

## Protection CSRF

Voir [Protection CSRF](./securite-protection-csrf.md).

## Contrôle d'accès agent/maître

Voir [Isolation du contrôleur des builds](./securite-isolation-du-controleur.md).

