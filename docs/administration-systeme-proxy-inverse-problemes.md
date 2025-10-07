# Proxy inversé - Problèmes

## Symptômes

Un message d'erreur s'affiche dans la page « Manage Jenkins » (Gérer Jenkins) :

`It appears that your reverse proxy setup is broken` (Il semble que votre configuration de proxy inverse soit défectueuse).

!!! info " "
    Ce message peut également s'afficher si vous n'accédez pas à Jenkins via un proxy inverse : assurez-vous que l'URL Jenkins configurée dans la configuration système correspond à l'URL que vous utilisez pour accéder à Jenkins.

## Contexte

Pour qu'un proxy inverse fonctionne correctement, il doit réécrire à la fois la requête et la réponse. La réécriture de la requête implique la réception d'un appel HTTP entrant, puis la transmission d'une requête à Jenkins (parfois avec certains en-têtes HTTP modifiés, parfois non). Une erreur de configuration de la réécriture de la requête est facile à détecter, car aucune page ne s'affiche.

Mais un proxy inverse correct implique également **l'une des deux options** suivantes : SOIT

* **réécrire la réponse** avec un en-tête « Location » dans la réponse, qui est utilisé lors des redirections. Jenkins envoie `Location: http://actual.server:8080/jenkins/foobar` et le proxy inverse doit le réécrire en `Location: http://nice.name/jenkins/foobar`. Malheureusement, une configuration incorrecte est plus difficile à détecter ; OU
* **Définissez les en-têtes** `X-Forwarded-Host` (et éventuellement `X-Forwarded-Port`) sur la requête transférée. Jenkins analysera ces en-têtes et générera toutes les redirections et autres liens sur la base de ces en-têtes. Selon votre proxy inverse, il peut être plus facile de définir `X-Forwarded-Host` et `X-Forwarded-Port` respectivement sur le nom d'hôte et le port dans l'en-tête `Host` d'origine, ou il peut être plus facile de simplement transmettre l'en-tête `Host` d'origine en tant que `X-Forwarded-Host` et de supprimer l'en-tête `X-Forwarded-Port` # de la requête. Vous devrez également définir l'en-tête `X-Forwarded-Proto` si votre proxy inverse passe de `https` à `http` ou vice-versa.

Jenkins dispose d'une surveillance proactive pour s'assurer que cela est correctement configuré. Il utilise XmlHttpRequest pour demander une URL spécifique dans Jenkins (via un chemin relatif, ce qui garantit que la requête aboutira toujours à condition qu'elle soit correctement réécrite), qui redirigera ensuite l'utilisateur vers une autre page dans Jenkins (cela ne fonctionne correctement que si vous avez correctement configuré la réécriture de la réponse), qui renvoie alors 200.

Ce message d'erreur indique que ce test échoue. La cause la plus probable est que la réécriture de la réponse est mal configurée. Consultez les [exemples de configuration](./administration-systeme-configuration-proxy-inverse.md) pour obtenir des conseils supplémentaires sur la configuration d'un proxy inverse.

Veillez à définir l'en-tête `X-Forwarded-Proto` si votre proxy inverse est accessible via HTTPS et que Jenkins lui-même est accessible via HTTP, c'est-à-dire en proxysant HTTPS vers HTTP.

## Chemin d'accès contextuel

Le chemin d'accès contextuel est le préfixe d'un chemin d'accès URL. Le contrôleur Jenkins et le proxy inverse doivent utiliser le même chemin d'accès contextuel. Par exemple, si l'URL du contrôleur Jenkins est https://www.example.com/jenkins/, l'argument `--prefix=/jenkins` doit être inclus dans les arguments de ligne de commande du contrôleur Jenkins.

Définissez le chemin d'accès contextuel lorsque vous utilisez les paquets Linux en exécutant `systemctl edit jenkins` et en ajoutant ce qui suit :

``` cfg
[Service]
Environment="JENKINS_PREFIX=/jenkins"
```

Définissez le chemin d'accès contextuel sur les contrôleurs Windows en incluant l'argument de ligne de commande `--prefix` dans le fichier `jenkins.xml` du répertoire d'installation.

Assurez-vous que Jenkins s'exécute sur le chemin d'accès contextuel où votre proxy inverse sert Jenkins. Vous aurez moins de difficultés si vous respectez ce principe.

L'argument de ligne de commande `--prefix` n'est pas nécessaire si le chemin d'accès au contexte est vide. Par exemple, l'URL https://jenkins.example.com/ a un chemin d'accès au contexte vide.

La modification du chemin d'accès au contexte de Jenkins avec un proxy inverse comporte de nombreux risques. De nombreuses URL doivent être réécrites. Même si vous réécrivez toutes les URL dans les fichiers HTML, vous risquez d'en oublier certaines dans les ressources JavaScript, CSS ou XML.

Bien qu'il soit techniquement possible d'utiliser des règles de réécriture pour modifier le chemin d'accès contextuel, vous devez savoir que cela demanderait beaucoup de travail pour trouver et corriger tout ce qui se trouve dans vos règles de réécriture et que le proxy inverse passerait la plupart de son temps à réécrire les réponses de Jenkins. Il est beaucoup plus simple de modifier Jenkins pour qu'il s'exécute dans le chemin d'accès contextuel attendu par votre proxy inverse. Par exemple, si votre proxy inverse transfère les requêtes vers https://manchu.example.org/foobar/ à Jenkins, vous pouvez simplement utiliser `java -jar jenkins.war --prefix=/foobar` pour démarrer Jenkins en utilisant `/foobar` comme chemin d'accès contextuel.

## Diagnostic approfondi

Pour un diagnostic plus approfondi, essayez d'utiliser cURL :

``` shell title="SH"
BASE=administrativeMonitor/hudson.diagnosis.ReverseProxySetupMonitor
curl -iL -e http://your.reverse.proxy/jenkins/manage \
            http://your.reverse.proxy/jenkins/${BASE}/test
```

(en supposant que votre Jenkins se trouve à l'adresse http://your.reverse.proxy/jenkins/ et qu'il est ouvert à un accès en lecture anonyme)

