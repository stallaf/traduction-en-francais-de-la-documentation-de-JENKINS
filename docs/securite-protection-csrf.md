# Protection CSRF

<div class="couleur-introduction">
[La falsification de requête intersite](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF ou XSRF) est un type de vulnérabilité de sécurité dans les applications Web. Sans protection contre la CSRF, un utilisateur ou un administrateur Jenkins visitant un autre site Web permettrait à l'opérateur de ce site d'effectuer des actions dans Jenkins en tant que victime.
</div>

## Protection CSRF dans Jenkins

La protection CSRF utilise un jeton (appelé _crumb_ dans Jenkins) qui est créé par Jenkins et envoyé à l'utilisateur. Tout envoi de formulaire ou action similaire entraînant des modifications, comme le déclenchement de builds ou la modification de la configuration, nécessite la fourniture du crumb. Le crumb contient des informations identifiant l'utilisateur pour lequel il a été créé, de sorte que les envois avec le jeton d'un autre utilisateur seraient rejetés. Tout cela se passe en arrière-plan et n'a aucun impact visible, sauf dans de rares circonstances, par exemple après l'expiration de la session d'un utilisateur et sa reconnexion.

!! info " "
    La documentation de cette page s'applique à Jenkins 2.222 ou à une version plus récente.

## Configuration de la protection CSRF

Dans _Manage Jenkins » Security » CSRF Protection_, les administrateurs peuvent configurer la protection CSRF.

![Protection CSRF dans Security](https://www.jenkins.io/doc/book/resources/security/configure-global-security-prevent-csrf.png)
Le _Default Crumb Issuer_ encode les informations suivantes dans le [hash](https://en.wikipedia.org/wiki/Cryptographic_hash_function) utilisé comme crumb :

* Le nom d'utilisateur pour lequel le crumb a été généré ;
* L'ID de session Web dans lequel le crumb a été généré ;
* L'adresse IP de l'utilisateur pour lequel le crumb a été généré ;
* Un [sel](https://en.wikipedia.org/wiki/Salt_(cryptography)) unique à cette instance Jenkins.

Toutes ces informations doivent correspondre lorsqu'un crumb est renvoyé à Jenkins pour que cette soumission soit considérée comme valide.

La seule option prise en charge, _Activer la compatibilité proxy_, supprime les informations relatives à l'adresse IP de l'utilisateur du jeton. Cela peut être utile lorsque Jenkins fonctionne derrière un proxy inverse et que l'adresse IP d'un utilisateur telle que vue par Jenkins change régulièrement.

!!! info " "
    L'ID de session Web a été ajouté dans Jenkins 2.176.2 et 2.186 pour provoquer l'expiration du crumb. Voir [l'avis de sécurité](https://www.jenkins.io/security/advisory/2019-07-17/#SECURITY-626) et le [guide de mise à niveau](https://www.jenkins.io/doc/upgrade-guide/2.176/#SECURITY-626).

Les plugins peuvent fournir d'autres émetteurs de crumbs qui utilisent d'autres critères pour déterminer si un crumb est valide. Le [Strict Crumb Issuer](https://plugins.jenkins.io/strict-crumb-issuer) fournit une implémentation alternative de crumb issuer qui est plus personnalisable.

## Utilisation avec des clients scriptés

Les requêtes envoyées à l'aide de la méthode `POST` sont soumises à la protection CSRF dans Jenkins et doivent généralement fournir un crumb. Cela s'applique également aux clients scriptés qui **s'authentifient à l'aide d'un nom d'utilisateur et d'un mot de passe**. Étant donné que le crumb inclut l'ID de session Web, les clients doivent procéder comme suit :

* Envoyez une requête aux points de terminaison `/crumbIssuer/api` pour demander un crumb. Notez l'en-tête de réponse `Set-Cookie` ;
* Pour toutes les requêtes suivantes, fournissez le crumb et le cookie de session en plus du nom d'utilisateur et du mot de passe.

Vous pouvez également vous **authentifier à l'aide de votre nom d'utilisateur et de votre jeton API**. Les requêtes s'authentifiant avec un [jeton API](./administration-systeme-authentification-des-clients-scriptes.md) sont exemptées de la protection CSRF dans Jenkins.

## Désactivation de la protection CSRF

Les plugins obsolètes qui envoient des requêtes HTTP à Jenkins peuvent ne pas fonctionner lorsque la protection CSRF est activée. Dans ce cas, il peut être nécessaire de désactiver temporairement la protection CSRF.

!!! warning "Avertissement"
    Il est **fortement recommandé** de laisser la protection CSRF **activée**, y compris sur les instances fonctionnant sur des réseaux privés entièrement fiables.

Pour désactiver la protection CSRF, définissez la propriété système `hudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION` sur `true` au démarrage. En savoir plus sur les [propriétés système](./gestion-proprietes-systeme.md) dans Jenkins.
