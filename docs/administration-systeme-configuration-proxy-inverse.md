# Configuration proxy inverse

<div class="couleur-introduction">
Un <a href="https://en.wikipedia.org/wiki/Reverse_proxy">« proxy inverse »</a> permet à un autre fournisseur HTTP ou HTTPS de communiquer avec les navigateurs Web au nom de Jenkins. Ce fournisseur alternatif peut offrir des fonctionnalités supplémentaires, telles que le chiffrement SSL. Il peut également décharger Jenkins de certaines tâches, comme la fourniture d'images statiques.
</div>

## Directives générales

Jenkins surveille activement la configuration du proxy inverse. Jenkins signale [« Votre configuration de proxy inverse est défectueuse »](./administration-systeme-proxy-inverse-problemes.md) lorsqu'il détecte un problème de configuration du proxy inverse. Reportez-vous à la section de [dépannage](./administration-systeme-proxy-inverse-problemes.md) si Jenkins signale que votre configuration de proxy inverse est défectueuse.

### Contexte

Les proxys inversés reçoivent les requêtes HTTP entrantes et les transmettent à Jenkins. Ils reçoivent la réponse HTTP sortante de Jenkins et transmettent ces requêtes au demandeur d'origine. Un proxy inverse correctement configuré réécrit à la fois la requête HTTP et la réponse HTTP.

Lorsque la réécriture des requêtes HTTP est mal configurée, les pages ne s'affichent pas du tout. Reportez-vous à la section [**Exemples de configuration par type de serveur**](#exemples-de-configuration-par-type-de-serveur) si votre proxy inverse n'affiche aucune page Jenkins.

## Exemples de configuration par type de serveur

Jenkins fonctionne avec de nombreux proxys inversés différents. Cette section fournit des exemples pour des proxys inversés spécifiques, bien que la plupart des informations s'appliquent également à d'autres proxys inversés.

* [Exécution de Jenkins avec Apache](./administration-systeme-reverse-proxy-apache.md) ;
* [Exécution de Jenkins avec Nginx](./administration-systeme-reverse-proxy-nginx.md) ;
* [Exécution de Jenkins avec Lighttpd](./administration-systeme-reverse-proxy-lighttpd.md) ;
* [Exécution de Jenkins avec HAProxy](./administration-systeme-reverse-proxy-haproxy.md) ;
* [Exécution de Jenkins avec Pomerium](./administration-systeme-reverse-proxy-pomerium.md) ;
* [Exécution de Jenkins avec Squid](./administration-systeme-reverse-proxy-squid.md) ;
* [Exécution de Jenkins avec IIS](./administration-systeme-reverse-proxy-iis.md) ;
* [Exécution de Jenkins avec iptables](./administration-systeme-reverse-proxy-iptables.md) ;


