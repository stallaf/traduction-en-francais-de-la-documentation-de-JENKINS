# Proxy inverse - Nginx

<div class="couleur-introduction">
Si vous avez déjà des sites web sur votre serveur, il peut être utile d'exécuter Jenkins (ou le conteneur de servlets dans lequel Jenkins s'exécute) derrière <a href="https://nginx.org/">Nginx</a>, afin de pouvoir lier Jenkins à la partie d'un site web plus grand que vous possédez. Cette section présente certaines des approches permettant d'y parvenir.
</div>

Lorsqu'une requête arrive pour certaines URL, Nginx devient un proxy et transmet cette requête à Jenkins, puis renvoie la réponse au client.

Ce tutoriel vidéo de 9 minutes de Darin Pope configure Nginx en tant que proxy inverse.

_Configuration de Nginx en tant que proxy inverse._

<iframe width="640" height="360" src="https://www.youtube.com/embed/yixMeJGtLFk" title="How To Configure Nginx as a Reverse Proxy for Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Le fragment de configuration Nginx ci-dessous fournit un exemple de configuration de proxy inverse Nginx. Il suppose que le contrôleur Jenkins et le proxy inverse Nginx s'exécutent sur le même ordinateur.

``` cfg
upstream jenkins {
  keepalive 32; # connexions keepalive
  server 127.0.0.1:8080; # adresse IP et port Jenkins
}

# Requis pour les agents websocket Jenkins
map $http_upgrade $connection_upgrade {
  default upgrade;
  “” close;
}

server {
  listen          80;       # Écoute sur le port 80 pour les requêtes IPv4

  server_name     jenkins.example.com;  # remplacez « jenkins.example.com » par le nom de domaine de votre serveur

  # il s'agit du répertoire racine web de Jenkins
  # (mentionné dans la sortie de « systemctl cat jenkins »)
  root            /var/run/jenkins/war/;

  access_log      /var/log/nginx/jenkins.access.log;
  error_log       /var/log/nginx/jenkins.error.log;

  # transmettre les en-têtes de Jenkins que Nginx considère comme invalides
  ignore_invalid_headers off;

  location ~ « ^\/static\/[0-9a-fA-F]{8}\/(.*)$ » {
    # réécrire tous les fichiers statiques en requêtes vers la racine
    # Par exemple, /static/12345678/css/something.css deviendra /css/something.css
    rewrite « ^\/static\/[0-9a-fA-F]{8}\/(.*) » /$1 last;
  }

  location /userContent {
    # demander à nginx de traiter toutes les requêtes statiques vers le dossier userContent
    # remarque : il s'agit du répertoire $JENKINS_HOME
    root /var/lib/jenkins/;
    if (!-f $request_filename){
      # ce fichier n'existe pas, il s'agit peut-être d'un répertoire ou d'une URL /**view**
      rewrite (.*) /$1 last;
      break;
    }
    sendfile on;

  location / {
      sendfile off;
      proxy_pass         http://jenkins;
      proxy_redirect     default;
      proxy_http_version 1.1;

      # Requis pour les agents websocket Jenkins
      proxy_set_header   Connection        $connection_upgrade;
      proxy_set_header   Upgrade           $http_upgrade;

      proxy_set_header   Host              $http_host;
      proxy_set_header   X-Real-IP         $remote_addr;
      proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_set_header   X-Forwarded-Proto $scheme;
      proxy_max_temp_file_size 0;

      #il s'agit de la taille maximale de téléchargement
      client_max_body_size       10m;
      client_body_buffer_size    128k;

      proxy_connect_timeout      90;
      proxy_send_timeout         90;
      proxy_read_timeout         90;
      proxy_request_buffering    off; # Requis pour les commandes HTTP CLI
  }

}
```

Cela suppose que vous exécutez Jenkins sur le port 8080. N'oubliez pas de créer le dossier /var/log/nginx/jenkins.

Veillez à ne pas terminer la valeur `proxy_pass` par une barre oblique, car cela causerait des problèmes tels que ceux décrits dans [cette discussion](https://community.jenkins.io/t/reverse-proxy-test-does-not-handle-url-correctly/1294/16).

## Chemin d'accès contextuel

Le chemin contextuel est le préfixe d'un chemin URL. Le contrôleur Jenkins et le proxy inverse doivent utiliser le même chemin contextuel. Par exemple, si l'URL du contrôleur Jenkins est https://www.example.com/jenkins/, l'argument `--prefix=/jenkins` doit être inclus dans les arguments de ligne de commande du contrôleur Jenkins.

Définissez le chemin d'accès contextuel lorsque vous utilisez les paquets Linux en exécutant `systemctl edit jenkins` et en ajoutant ce qui suit :

``` cfg
[Service]
Environment="JENKINS_PREFIX=/jenkins"
```

Définissez le chemin d'accès contextuel sur les contrôleurs Windows en incluant l'argument de ligne de commande `--prefix` dans le fichier `jenkins.xml` du répertoire d'installation.

Assurez-vous que Jenkins s'exécute sur le chemin d'accès contextuel où votre proxy inverse sert Jenkins. Vous aurez moins de difficultés si vous respectez ce principe.

L'argument de ligne de commande` --prefix` n'est pas nécessaire si le chemin d'accès contextuel est vide. Par exemple, l'URL https://jenkins.example.com/ a un chemin d'accès contextuel vide.

Si vous rencontrez des problèmes avec certains chemins (par exemple des dossiers) avec **Blue Ocean**, vous devrez peut-être ajouter l'extrait suivant à votre configuration de proxy :

``` cfg
if ($request_uri ~* « /blue(/.*) ») {
    proxy_pass http://YOUR_SERVER_IP:YOUR_JENKINS_PORT/blue$1;
    break;
```

## Autorisations

Pour autoriser Nginx à lire le dossier racine Web Jenkins, ajoutez l'utilisateur `nginx` au groupe Jenkins :

``` sh title="SH"
usermod -aG jenkins nginx
```

Si la dernière commande a échoué parce que l'utilisateur `nginx` n'est pas défini dans le système, vous pouvez essayer d'ajouter l'utilisateur `www-data` au groupe Jenkins :

``` sh title="SH"
usermod -aG jenkins www-data
```

Si vous rencontrez des délais d'attente lorsque vous essayez d'exécuter de longues commandes CLI via un proxy dans Jenkins, vous pouvez augmenter le paramètre `proxy_read_timeout` si nécessaire. Les anciennes versions de Jenkins peuvent ne pas respecter le paramètre `proxy_read_timeout`.

Si vous rencontrez l'erreur suivante lorsque vous essayez d'exécuter des commandes CLI longues dans Jenkins et que Jenkins s'exécute derrière Nginx, cela est probablement dû au fait que Nginx a expiré la connexion CLI. Vous pouvez augmenter le paramètre `proxy_read_timeou`t si nécessaire afin que la commande s'exécute correctement.

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
Causé par : java.io.IOException : EOF 
        à sun.net.www.http.ChunkedInputStream.readAheadBlocking(ChunkedInputStream.java:565)
        ...
    à hudson.cli.FlightRecorderInputStream.analyzeCrash(FlightRecorderInputStream.java:82)
    à hudson.cli.PlainCLIProtocol$EitherSide$Reader.run(PlainCLIProtocol.java:153)
Causé par : java.io.IOException : EOF prématuré
    à sun.net.www.http.ChunkedInputStream.readAheadBlocking(ChunkedInputStream.java:565)
    ...
    à java.io.DataInputStream.readInt(DataInputStream.java:387)
    à hudson.cli.PlainCLIProtocol$EitherSide$Reader.run(PlainCLIProtocol.java:111)
```
