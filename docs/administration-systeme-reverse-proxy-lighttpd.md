# Proxy inverse - Lighttpd

<div class="couleur-introduction">
Si vous avez déjà des sites web sur votre serveur, il peut être utile d'exécuter Jenkins (ou le conteneur de servlets dans lequel Jenkins s'exécute) derrière <a href="https://www.lighttpd.net/">Lighttpd</a>. Cela vous permet de lier Jenkins à une partie d'un site web plus grand que vous possédez ou de centraliser la terminaison TLS en un seul endroit. Cette section présente certaines des approches permettant d'y parvenir.
</div>

Le [mod_proxy](https://redmine.lighttpd.net/projects/lighttpd/wiki/Mod_proxy) fonctionne en faisant fonctionner Lighttpd comme un « proxy inverse ». Cela signifie que lorsqu'une requête arrive pour certaines URL, Lighttpd devient un proxy et transmet cette requête à Jenkins, puis renvoie la réponse de Jenkins au client.

Il existe deux alternatives pour configurer Jenkins avec Lighttpd. Sélectionnez la technique qui répond le mieux à vos besoins :

* [Par hôte](#par-hote) ;
* [Par chemin](#par-chemin).

[Par défaut](https://www.lighttpd.net/2018/11/28/1.4.52/), Lighttpd forcera la normalisation de l'URL, ce qui entraînera une incompatibilité avec les en-têtes utilisés par un test de proxy inverse Jenkins interne. Cela se traduira par l'affichage du message « Il semble que votre configuration de proxy inverse soit défectueuse » sur la page de gestion. Pour éviter cela, désactivez explicitement l'option `url-path-2f-decode`.

## Par hôte

Dans la configuration ci-dessous, il n'y a pas de chemin d'accès contextuel pour l'URL Jenkins.

Lorsqu'une requête pour le domaine` jenkins.example.com `arrive, Lighttpd transmet cette requête à Jenkins.

``` cfg
server.http-parseopts = (
  « url-path-2f-decode » => « disable »)


server.modules += ( « mod_proxy » )

$HTTP[« host »] == « jenkins.example.com » {
  proxy.balance = « hash »
  proxy.server = (
    « » => (
      « jenkins » => (
        « host » => « 127.0.0.1 »,
        « port » => « 8080 »
      )
    )
  )
}
```

Cela suppose que vous exécutez Jenkins sur le port 8080.

## Par chemin

Dans la configuration ci-dessous, il existe un chemin spécifique `/jenkins` pour Jenkins. Ce chemin doit être configuré dans Lighttpd avec `$HTTP[« url »]`. Dans Jenkins, définissez le chemin de contexte en modifiant le fichier de configuration jenkins.xml et en ajoutant `--prefix=/jenkins` (ou similaire) à l'entrée <arguments>.

Lorsqu'une requête est envoyée, par exemple à [http://localhost/jenkins](http://localhost/jenkins), Lighttpd transmet cette requête à Jenkins.

``` cfg
server.http-parseopts = (
  "url-path-2f-decode" => "disable"
)

server.modules += ( "mod_proxy" )

$HTTP["url"] =~ "^/jenkins(.*)$" {
  proxy.balance = "hash"
  proxy.server = (
    "" => (
      "jenkins" => (
        "host" => "127.0.0.1",
        "port" => 8080
      )
    )
  )
}
```

Cela suppose que vous exécutez Jenkins sur le port 8080.

