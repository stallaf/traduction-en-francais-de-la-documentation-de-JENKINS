# Services et Ports Exposés

<div class="couleur-introduction">
Jenkins est une application assez complexe qui ne se contente pas de fournir une interface Web. Cette section documente les services qu'elle expose fréquemment sur le réseau.
</div>

!!! info " "
    Les plugins peuvent exposer d'autres services sur le réseau. Seuls les plugins les plus couramment installés sont pris en compte ci-dessous.

## Interface utilisateur Web

L'interface utilisateur Web est fournie via HTTP (ou HTTPS), par défaut sur le port 8080. Jenkins s'exécute parfois derrière un proxy inverse qui peut personnaliser ou filtrer les requêtes et les réponses.

Pour obtenir un aperçu des URL accessibles aux utilisateurs sans aucune autorisation dans Jenkins, consultez la section [Contrôle d'accès](./securite-controle-acces.md).

## Port d'écoute de l'agent TCP

Jenkins peut exposer un port TCP qui permet aux agents entrants de s'y connecter. Il peut être activé, désactivé et configuré dans _Manage Jenkins » Security_.

Les deux modes pris en charge (lorsqu'ils sont activés) sont les suivants :

1. **Random** (Aléatoire) : le port TCP est choisi au hasard pour éviter les collisions sur le [contrôleur](./glossaire.md#controleur) Jenkins. L'inconvénient des ports aléatoires est qu'ils sont choisis lors du démarrage du contrôleur Jenkins, ce qui rend difficile la gestion des règles de pare-feu autorisant le trafic TCP.
2. **Fixed** (Fixe) : le port est choisi par l'administrateur Jenkins et reste le même à chaque redémarrage du contrôleur Jenkins. Cela facilite la gestion des règles de pare-feu autorisant les agents TCP à se connecter au contrôleur.

Ce service est désactivé par défaut dans la plupart des paquets, mais les images Docker l'exposent sur le port 50000.

Les requêtes HTTP sur ce port obtiennent une réponse minimale en texte brut avec quelques informations de diagnostic. Elle se présente comme suit :

``` cfg
Jenkins-Agent-Protocols : JNLP4-connect, Ping  #(1)
Jenkins-Version : 2.289 #(2)
Jenkins-Session : b4783dfa #(3)
Client : 172.17.0.1 #(4)
Serveur : 172.17.0.3 #(5)
Remoting-Minimum-Version : 3.14 #(6)
```

1.  Liste des protocoles TCP entrants actuellement activés.
2. Version actuelle de Jenkins.
3. Session de l'exécution Jenkins actuelle. Cela n'a aucun rapport avec les cookies de session Web. Cette valeur est remplacée par une nouvelle valeur à chaque démarrage de Jenkins. Elle est utilisée dans diverses URL pour faciliter la mise en cache des ressources statiques et peut être utilisée par les clients pour savoir si Jenkins a redémarré.
4. Adresse IP du client telle que vue par Jenkins.
5. dresse IP du contrôleur Jenkins.
6. Version minimale requise de la bibliothèque [Remoting](https://github.com/jenkinsci/remoting/) pour que les agents puissent se connecter à ce contrôleur Jenkins.

Les agents entrants peuvent être configurés pour utiliser le transport WebSocket afin de se connecter à Jenkins. Dans ce cas, aucun port TCP supplémentaire ne doit être activé et aucune configuration de sécurité particulière n'est nécessaire.

## Serveur SSH

Le plugin [SSH Server](https://plugins.jenkins.io/sshd) peut être configuré pour ouvrir un port SSH sur le contrôleur Jenkins.

!!! info " "
    La fonctionnalité de ce plugin faisait partie de Jenkins (noyau) jusqu'à la version Jenkins 2.281.

L'authentification peut se faire par nom d'utilisateur/mot de passe ou par clé publique. Dans ce dernier cas, les clés publiques peuvent être stockées dans les profils des utilisateurs Jenkins.

Ce serveur SSH ne fournit pas de shell OS standard une fois connecté. Il fournit plutôt une interface de type shell pour la [CLI Jenkins](./gestion-cli.md).

De plus, certains plugins peuvent fournir des fonctionnalités qui nécessitent l'activation du serveur SSH, comme le plugin [GIT Server](https://plugins.jenkins.io/git-server).

Ce service est désactivé par défaut.

## Découverte de services UDP et mDNS

Les versions 2.219 et antérieures de Jenkins incluaient des fonctionnalités qui permettaient aux utilisateurs de découvrir Jenkins sur leur réseau. Ces fonctionnalités ont été supprimées sans remplacement dans Jenkins 2.220 pour des raisons de sécurité.
