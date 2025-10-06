# Proxy inverse - HAProxy

<div class="couleur-introduction">
Dans les situations où vous souhaitez disposer d'une URL conviviale, de différents ports publics ou mettre fin aux connexions SSL avant qu'elles n'atteignent Jenkins, il peut s'avérer utile d'exécuter Jenkins (ou le conteneur de servlets dans lequel Jenkins s'exécute) derrière HAProxy. Cette section présente certaines des approches permettant d'y parvenir.
</div>

Ce tutoriel vidéo de 6 minutes de Darin Pope explique comment configurer un proxy inverse HAProxy.

_Configuration d'un proxy inverse HAProxy_.

##HTTP simple

À l'aide de HAProxy 2.6.7, voici un exemple de fichier HAProxy.cfg pour le proxy sur HTTP simple :

``` cfg
# Si vous disposez déjà d'un fichier haproxy.cfg, vous pouvez probablement laisser les sections
# global et defaults telles quelles, mais vous devrez peut-être augmenter les
# délais d'expiration afin que les commandes CLI longues puissent fonctionner.
global
    maxconn 4096
    log stdout local0 debug
valeurs par défaut
   log global
   option httplog
   option dontlognull
   option forwardfor
   maxconn 20
   timeout connect 5s
   timeout client 60s
   timeout server 60s

frontend http-in
  log stdout format raw local0 debug #pour une journalisation supplémentaire
  bind *:80
  mode http
  acl préfixé-avec-jenkins  path_beg /jenkins
  # Utilisez le préfixe de redirection http-request pour ajouter le préfixe /jenkins à l'emplacement de l'URL
  # afin de vous assurer que l'URL de base jenkins (chemin d'accès contextuel) fonctionne correctement.
  http-request redirect code 301 prefix /jenkins unless prefixed-with-jenkins
  use_backend jenkins if prefixed-with-jenkins

backend jenkins
  log stdout format raw local0 debug
  mode http
  server jenkins1 127.0.0.1:8080 check
  http-request replace-path /jenkins(/)?(.*) /\2
  http-request set-header X-Forwarded-Port %[dst_port]
  http-request add-header X-Forwarded-Proto https if { ssl_fc }
  http-request set-header X-Forwarded-Host %[req.hdr(Host)]
```

Cela suppose que Jenkins s'exécute localement sur le port 8080.

Cela suppose que vous utilisez le chemin d'accès [/jenkins ](http://localhost/jenkins) à la fois pour le site exposé depuis HAProxy et pour Jenkins lui-même. Si ce n'est pas le cas, vous devrez ajuster la configuration. Reportez-vous à la documentation HAProxy sur le [routage du trafic](https://www.haproxy.com/documentation/hapee/latest/traffic-routing/) pour plus d'informations.

Si vous rencontrez l'erreur suivante lorsque vous essayez d'exécuter des commandes CLI longues dans Jenkins, et que Jenkins s'exécute derrière HAProxy, cela est probablement dû au fait que HAProxy a expiré la connexion CLI. Vous pouvez augmenter les paramètres de `timeout client ` et du `timeout server ` si nécessaire afin que la commande s'exécute correctement.

``` cfg
AVERTISSEMENT : null
hudson.cli.DiagnosedStreamCorruptionException
Lecture : 0x00 0x00 0x00 0x1e 0x07
           « Démarrage du test de proxy inverse n° 68 »
           0x00 0x00 0x00 0x01 0x07 0x0a
Lecture anticipée :
Problème de diagnostic :
    java.io.IOException : EOF prématuré
        à sun.net.www.http.ChunkedInputStream.readAheadBlocking(ChunkedInputStream.java:565)
        ...
    à hudson.cli.FlightRecorderInputStream.analyzeCrash(FlightRecorderInputStream.java:82)
    à hudson.cli.PlainCLIProtocol$EitherSide$Reader.run(PlainCLIProtocol.java:153)
Causé par : java.io.IOException : EOF prématuré
    à sun.net.www.http.ChunkedInputStream.readAheadBlocking(ChunkedInputStream.java:565)
    ...
    à java.io.DataInputStream.readInt(DataInputStream.java:387)
```

### Avec SSL

À l'aide de HAProxy 2.6.7, voici un exemple de fichier HAProxy.cfg permettant de se connecter au proxy à l'aide de SSL, de mettre fin à la connexion SSL, puis de communiquer avec Jenkins à l'aide du protocole HTTP simple :

``` cfg
# Si vous disposez déjà d'un fichier haproxy.cfg, vous pouvez probablement laisser les sections
# global et defaults telles quelles, mais vous devrez peut-être augmenter les
# délais d'expiration afin que les commandes CLI longues puissent fonctionner.

global
    maxconn 4096
    log stdout local0 debug

defaults
   log global
   option httplog
   option dontlognull
   option forwardfor
   maxconn 20
   timeout connect 5s
   timeout client 5m
   timeout server 5m

frontend http-in
  log stdout format raw local0 debug
  bind *:80
  bind *:443 ssl crt /usr/local/etc/haproxy/ssl/server.pem
  mode http
  acl prefixed-with-jenkins  path_beg /jenkins
  http-request redirect code 301 prefix /jenkins unless prefixed-with-jenkins
  redirect scheme https if !{ ssl_fc } # Rediriger les requêtes http vers https
  use_backend jenkins if prefixed-with-jenkins

backend jenkins
  log stdout format raw  local0 debug
  mode http
  server jenkins1 127.0.0.1:8080 check
  http-request replace-path /jenkins(/)?(.*) /\2
  http-request set-header X-Forwarded-Port %[dst_port]
  http-request add-header X-Forwarded-Proto https if { ssl_fc }
  http-request set-header X-Forwarded-Host %[req.hdr(Host)]
```

## Chemin d'accès contextuel

Le chemin d'accès contextuel est le préfixe d'un chemin d'accès URL. Le contrôleur Jenkins et le proxy inverse doivent utiliser le même chemin d'accès contextuel. Par exemple, si l'URL du contrôleur Jenkins est https://www.example.com/jenkins/, l'argument `--prefix=/jenkins` doit être inclus dans les arguments de ligne de commande du contrôleur Jenkins.

Définissez le chemin d'accès au contexte lorsque vous utilisez les paquets Linux en exécutant `systemctl edit jenkins` et en ajoutant ce qui suit :

``` cfg
[Service]
Environment="JENKINS_PREFIX=/jenkins"
```

Définissez le chemin d'accès au contexte sur les contrôleurs Windows en incluant l'argument de ligne de commande` --prefix` dans le fichier `jenkins.xml` du répertoire d'installation.

Assurez-vous que Jenkins s'exécute sur le chemin d'accès contextuel où votre proxy inverse sert Jenkins. Vous aurez moins de difficultés si vous respectez ce principe.

L'argument de ligne de commande` --prefix `n'est pas nécessaire si le chemin d'accès contextuel est vide. Par exemple, l'URL https://jenkins.example.com/ a un chemin d'accès contextuel vide.
