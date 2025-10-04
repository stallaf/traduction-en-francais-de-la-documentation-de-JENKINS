# Contrôle d'Accès

Le contrôle d'accès Jenkins est divisé en deux parties :

* L'authentification (les utilisateurs prouvent leur identité) s'effectue à l'aide d'un domaine de sécurité. Le domaine de sécurité détermine l'identité de l'utilisateur et son appartenance à un groupe ;
* L'autorisation (les utilisateurs sont autorisés à effectuer certaines actions) s'effectue à l'aide d'une stratégie d'autorisation. Celle-ci contrôle si un utilisateur (directement ou via son appartenance à un groupe) dispose d'une autorisation.

Ces deux éléments peuvent être indépendants ou fonctionner en combinaison. Une configuration indépendante serait [Active Directory](https://plugins.jenkins.io/active-directory) ou [LDAP](https://plugins.jenkins.io/ldap) comme domaine de sécurité, et quelque chose comme [Matrix Authorization Strategy](https://plugins.jenkins.io/matrix-auth) comme stratégie d'autorisation. Un exemple de configuration de domaine de sécurité et de stratégie d'autorisation connexes est [l'authentification GitHub](https://plugins.jenkins.io/github-oauth) : bien que son domaine de sécurité puisse être utilisé avec une stratégie d'autorisation générique, il fournit également une stratégie d'autorisation qui recherche les autorisations de dépôt d'un utilisateur sur GitHub et accorde ou refuse des autorisations aux tâches connexes dans Jenkins en fonction de cela.

!!! info " "
    Jenkins peut être configuré pour que le contrôle d'accès de base s'effectue en dehors de celui-ci. Voici quelques exemples :
        * [Le conteneur de servlets Winstone/Jetty intégré](https://github.com/jenkinsci/winstone) fournit des options qui implémentent un domaine de sécurité de base en dehors de Jenkins ;
        * Si Jenkins fonctionne derrière un proxy inverse tel que Nginx ou Apache, ceux-ci peuvent limiter l'accès à Jenkins.
    L'avantage de ces approches est qu'elles n'autorisent aucun accès à Jenkins à moins qu'un utilisateur ne soit autorisé, ce qui réduit l'impact des problèmes de sécurité dans Jenkins ou les plugins, en particulier lorsqu'ils sont accessibles depuis Internet. L'inconvénient est le manque d'intégration avec les contrôles d'accès Jenkins et même une potentielle interférence avec ceux-ci (par exemple, lors de la tentative d'authentification de clients scriptés).

## Erreurs de configuration courantes

Lors de la configuration de l'authentification et de l'autorisation dans Jenkins, il est facile d'accorder accidentellement un accès beaucoup plus large que prévu. Consultez la [documentation sur l'accès accordé aux administrateurs](https://www.jenkins.io/doc/book/security/access-control/permissions/#administer) pour connaître l'impact de l'octroi involontaire d'une autorisation d'administration.

**Tout le monde peut tout faire**<br>
La stratégie d'autorisation **tout le monde peut tout faire** est très rarement un bon choix, car elle permet même à des utilisateurs anonymes d'administrer Jenkins. En règle générale, elle ne doit pas être utilisée. Pour des raisons de sécurité, ne comptez jamais sur le fait que l'URL Jenkins ne sera pas connue en dehors de votre équipe ou de votre organisation.

**Les utilisateurs connectés peuvent tout faire**<br>
L'utilisation de la stratégie d'autorisation **les utilisateurs connectés peuvent tout faire** peut être un choix judicieux tant que seuls les utilisateurs entièrement fiables disposent d'un compte pour accéder à Jenkins. Il s'agit du paramètre par défaut avec l'utilisateur administrateur unique de Jenkins lors de la configuration de Jenkins à l'aide de l'assistant de configuration.

Si vous passez à un domaine d'authentification qui permet aux utilisateurs non fiables d'avoir un compte, ces utilisateurs obtiendront un accès administratif à Jenkins si vous conservez cette stratégie d'autorisation. Par exemple, vous pouvez activer la création de comptes pour la base de données utilisateur de Jenkins ou divers autres domaines d'authentification, dont beaucoup (GitHub, Google, GitLab, etc.) permettent à n'importe qui de créer un compte.

**Utilisateurs anonymes et authentifiés**<br>
Comme pour les éléments précédents, vous ne devez généralement pas accorder d'autorisations importantes aux utilisateurs `anonymous` (l'utilisateur anonyme) ou authentifiés (tout utilisateur authentifié) lorsque vous utilisez une stratégie d'autorisation qui permet un contrôle plus fin (comme la [stratégie d'autorisation matricielle](https://plugins.jenkins.io/matrix-auth)). Accorder l'autorisation Global/Administrer à un utilisateur _anonymous_ revient à dire que _n'importe qui peut tout faire_, tandis qu'accorder cette autorisation à un utilisateur _authentifié_ revient essentiellement à dire que les _utilisateurs connectés peuvent tout faire_.

**Nœud intégré**<br>
Les utilisateurs disposant d'autorisations limitées [ne doivent pas pouvoir configurer des tâches qui s'exécutent sur le nœud intégré](./securite-isolation-du-controleur.md). Lors de la configuration d'une nouvelle instance Jenkins, de l'ajout d'utilisateurs et du changement de stratégie d'autorisation, il est important de configurer également des builds distribués et de limiter les tâches pouvant s'exécuter sur le nœud intégré.

## Autorisations

À un niveau très basique, l'autorisation _Global/Lecture_ fournit aux utilisateurs un accès basique à Jenkins. Cette autorisation est une condition préalable à un accès plus complet à Jenkins. Sans cette autorisation, seules quelques fonctionnalités explicitement destinées à être utilisées sans authentification sont disponibles.

Le niveau d'autorisation le plus élevé est « Global/Administration ». Avec cette autorisation, les utilisateurs peuvent télécharger et installer des plugins et ont accès à la [console de script](./gestion-console.md).

Entre ces deux extrêmes, il existe un contrôle plus fin des autorisations impliquant d'autres autorisations. Les autorisations dans Jenkins ont une portée : elles peuvent être accordées globalement, sur un élément (comme un dossier ou une tâche), sur une build, etc. Chaque fois qu'un utilisateur tente d'effectuer une action protégée par des autorisations, la stratégie d'autorisation vérifie si l'utilisateur actuel dispose de l'autorisation spécifique (par exemple, _Job/Read_) sur l'objet spécifique (par exemple, une tâche). La manière exacte dont les autorisations sont attribuées et si et comment elles sont héritées est contrôlée par la stratégie d'autorisation spécifique.

À titre d'exemple, [Matrix Authorization Strategy](https://plugins.jenkins.io/matrix-auth) propose deux stratégies d'autorisation différentes :

* L'une fournit une configuration globale unique de toutes les autorisations. Un utilisateur bénéficiant de l'autorisation _Item/Read_ se verra accorder cette autorisation partout ;
* L'autre fournit une configuration basée sur les projets. Dans ce modèle, les autorisations peuvent être accordées de manière globale (comme dans la stratégie précédente) ou uniquement sur des dossiers, des tâches ou des agents spécifiques. Les autorisations sont héritées par défaut, mais cela peut également être personnalisé, de sorte que les utilisateurs disposant d'une autorisation _Item/Read_ au niveau global ou sur un dossier parent puissent être exclus de l'accès à une tâche.

Pour plus de détails sur les différentes autorisations dans Jenkins et le niveau d'accès qu'elles accordent, consultez la section [Autorisations](https://www.jenkins.io/doc/book/security/access-control/permissions/).

## Désactivation du contrôle d'accès

Consultez la section [Désactiver le contrôle d'accès](https://www.jenkins.io/doc/book/security/access-control/disable/).



