# Proxy inverse - Apache

<div class="couleur-introduction">
Si vous avez déjà des sites web sur votre serveur, il peut être utile d'exécuter Jenkins (ou le conteneur de servlets dans lequel Jenkins s'exécute) derrière Apache, afin de pouvoir lier Jenkins à la partie d'un site web plus grand que vous possédez. Cette section présente certaines des approches permettant d'y parvenir.
</div>

**Veillez à modifier la valeur par défaut 0.0.0.0 de Jenkins httpListenAddress et à la remplacer par 127.0.0.1, sinon les restrictions au niveau d'Apache peuvent être facilement contournées en accédant directement au port Jenkins.**

Il existe plusieurs alternatives pour configurer Jenkins avec Apache. Choisissez la technique qui répond le mieux à vos besoins :

* [mod_proxy](#mod-proxy) ;
* [mod_proxy avec HTTPS](#mod-proxy-avec-https) ;
* [mod_rewrite](#mod-rewrite) ;
* [mod_proxy_unix_sockets](#mod-proxy-unix-sockets).

Ce tutoriel de 6 minutes de Darin Pope configure Apache httpd sur Alma Linux en tant que proxy inverse avec [mod_proxy](#mod-proxy).

_Configurer le serveur HTTP Apache en tant que proxy inverse_

<iframe width="640" height="360" src="https://www.youtube.com/embed/E3_g5wYZlfk" title="How To Configure Apache HTTP Server as a Reverse Proxy for Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## mod_proxy

[mod_proxy](http://httpd.apache.org/docs/2.0/mod/mod_proxy.html) fonctionne en faisant en sorte qu'Apache effectue un « proxy inverse » : lorsqu'une requête arrive pour certaines URL, Apache devient un proxy et transmet cette requête à Jenkins, puis renvoie la réponse de Jenkins au client.

Les modules Apache suivants doivent être installés :

``` cfg
a2enmod proxy
a2enmod proxy_http
a2enmod headers
# Requis pour websocket
a2enmod proxy_wstunnel
a2enmod rewrite
```

Une configuration type pour mod_proxy ressemblerait à ceci :

``` cfg
ProxyPass         /jenkins  http://localhost:8081/jenkins nocanon
ProxyPassReverse  /jenkins  http://localhost:8081/jenkins
ProxyRequests     Off
AllowEncodedSlashes NoDecode

# Requis pour les agents websocket Jenkins
RewriteEngine on
RewriteCond %{HTTP:Upgrade} websocket [NC]
RewriteCond %{HTTP:Connection} upgrade [NC]
RewriteRule ^/jenkins/?(.*) « ws://localhost:8081/jenkins/$1 » [P,L]

# Remplacement de l'autorisation du proxy inverse local
# La plupart des distributions Unix refusent le proxy par défaut.
# Voir /etc/apache2/mods-enabled/proxy.conf dans Ubuntu.
<Proxy http://localhost:8081/jenkins*>
  Order deny,allow
  Allow from all
</Proxy>
```

Cela suppose que vous exécutez Jenkins sur le port 8081.

[Le chemin d'accès contextuel](#chemin-dacces-contextuel) fournit plus de détails sur les exigences du proxy inverse pour le chemin d'accès contextuel Jenkins.

Lorsque vous exécutez Jenkins sur un serveur dédié et que vous utilisez / comme contexte, veillez à ajouter une barre oblique à la fin de toutes les URL dans les paramètres proxy d'Apache. Sinon, vous risquez de rencontrer des erreurs de proxy. Ainsi

``` cfg
ProxyPass / http://localhost:8080/ nocanon
```

au lieu de

``` cfg
ProxyPass / http://localhost:8080 nocanon     # ne fonctionnera pas
```

Notez que cela ne s'applique pas à la directive `ProxyPassMatch`, qui se comporte différemment de `ProxyPass`. Vous trouverez ci-dessous un exemple de `ProxyPassMatch` pour proxy toutes les URL autres que `/.well-known` (une URL requise par letsencrypt) :

``` cfg
ProxyPassMatch  ^/(?\!.well-known)  http://localhost:8080 nocanon
```

La directive _ProxyRequests Off_ empêche Apache de fonctionner comme un serveur proxy de transfert (à l'exception de _ProxyPass_). Il est conseillé de l'inclure, sauf si le serveur doit fonctionner comme un proxy.

Les options `nocanon` de `ProxyPass` et `AllowEncodedSlashes NoDecode` sont toutes deux nécessaires au bon fonctionnement de certaines fonctionnalités de Jenkins.

Si vous exécutez Apache sur une machine Security-Enhanced Linux (SE-Linux), il est essentiel de configurer SE-Linux correctement en exécutant la commande suivante en tant qu'administrateur :

``` cfg
setsebool -P httpd_can_network_connect true
```

Si cette commande n'est pas exécutée, Apache ne sera pas autorisé à transférer les requêtes proxy vers Jenkins et seul un message d'erreur s'affichera.

Comme Jenkins compresse déjà sa sortie, vous ne pouvez pas utiliser le filtre proxy-html normal pour modifier les URL :

``` cfg
SetOutputFilter proxy-html
```

Vous pouvez plutôt utiliser ce qui suit :

``` cfg
SetOutputFilter INFLATE;proxy-html;DEFLATE
ProxyHTMLURLMap http://your_server:8080/jenkins /jenkins
```

Mais comme Jenkins semble bien fonctionner, il est préférable de ne pas utiliser SetOutputFilter et ProxyHTMLURLMap.

Si Jenkins rencontre parfois des problèmes avec des pages indésirables aléatoires, la commande suivante peut vous aider :

``` cfg
SetEnv proxy-nokeepalive 1
```

Certains plug-ins déterminent les URL à partir des requêtes client provenant de l'en-tête Host. Si vous rencontrez des problèmes avec des URL incorrectes, vous pouvez essayer d'activer la directive `ProxyPreserveHost`, qui est désactivée par défaut :

``` cfg
ProxyPreserveHost On
```

## mod_proxy avec HTTPS

Vous pouvez ajouter une directive `ProxyPassReverse` supplémentaire pour rediriger les URL non SSL générées par Jenkins vers le côté SSL. En supposant que votre serveur web est `your.host.com`, il suffit d'ajouter les lignes suivantes dans la définition de l'hôte virtuel SSL :

``` cfg
ProxyRequests     Off
ProxyPreserveHost On
AllowEncodedSlashes NoDecode

<Proxy http://localhost:8081/jenkins*>
  Order deny,allow
  Allow from all
</Proxy>

ProxyPass         /jenkins  http://localhost:8081/jenkins nocanon
ProxyPassReverse  /jenkins  http://localhost:8081/jenkins
ProxyPassReverse  /jenkins  http://your.host.com/jenkins
```

Une autre option consiste à réécrire les en-têtes Location qui contiennent des URL non SSL générées par Jenkins. Si vous souhaitez accéder à Jenkins à partir de https://www.example.com/jenkins, vous pouvez également ajouter les éléments suivants dans la définition de l'hôte virtuel SSL :

``` cfg
ProxyRequests     Off
ProxyPreserveHost On
ProxyPass /jenkins/ http://localhost:8081/jenkins/ nocanon
AllowEncodedSlashes NoDecode

<Location /jenkins/>
  ProxyPassReverse /
  Order deny,allow
  Allow from all
</Location>

Modification de l'en-tête Location ^http://www.example.com/jenkins/ https://www.example.com/jenkins/
```

Mais il peut également suffire d'utiliser un simple transfert comme ci-dessus (premier extrait HTTPS) et d'ajouter ...

``` cfg
RequestHeader set X-Forwarded-Proto « https »
RequestHeader set X-Forwarded-Port « 443 »
```

... dans la configuration du site HTTPS, comme le fait la démo Docker (ci-dessous). (`X-Forwarded-Port` n'est pas interprété par Jenkins avant JENKINS-23294, il peut donc être souhaitable de configurer le conteneur de servlets pour spécifier le port d'origine.)

``` cfg
NameVirtualHost *:80
NameVirtualHost *:443

<VirtualHost *:80>
    ServerAdmin  webmaster@localhost
    Redirect permanent / https://www.example.com/
</VirtualHost>

<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cert.pem
    ServerAdmin  webmaster@localhost
    ProxyRequests     Off
    ProxyPreserveHost On
    AllowEncodedSlashes NoDecode
    <Proxy *>
        Order deny,allow
        Allow from all
    </Proxy>
    ProxyPass         /  http://localhost:8080/ nocanon
    ProxyPassReverse  /  http://localhost:8080/
    ProxyPassReverse  /  http://www.example.com/
    RequestHeader set X-Forwarded-Proto « https »
    RequestHeader set X-Forwarded-Port « 443 »

    # Requis pour les agents websocket Jenkins
RewriteEngine on
RewriteCond %{HTTP:Upgrade} websocket [NC]
RewriteCond %{HTTP:Connection} upgrade [NC]
RewriteRule ^/?(.*) « ws://localhost:8080/$1 » [P,L]
</VirtualHost>
```

## mod_rewrite

Le module Apache mod_rewrite peut être utilisé pour configurer un proxy inverse Apache pour Jenkins.

Les modules Apache suivants doivent être installés :

``` cfg
a2enmod rewrite
a2enmod proxy
a2enmod proxy_http
# Requis pour les agents websocket Jenkins
a2enmod proxy_wstunnel
```

Une configuration mod_rewrite type ressemblerait à ceci :

``` cfg
# Requis pour les agents websocket Jenkins
RewriteCond %{HTTP:Upgrade} websocket [NC]
RewriteCond %{HTTP:Connection} upgrade [NC]
RewriteRule ^/jenkins/?(.*) « ws://localhost:8081/jenkins/$1 » [P,L]

# Utilisez le dernier indicateur car aucune réécriture ne peut être appliquée après le proxy pass.
# NE s'assure que les barres obliques ne sont pas recodées.
# Apache ne recode pas les espaces, nous demandons à Apache de les recoder avec le drapeau B.
# BNP indique à Apache d'utiliser %20 au lieu de + pour recoder l'espace.
RewriteRule       ^/jenkins(.*)$  http://localhost:8081/jenkins$1 [P,L,NE,B=\,BNP]

ProxyPassReverse  /jenkins        http://localhost:8081/jenkins
ProxyRequests     Off

AllowEncodedSlashes NoDecode

# Remplacement de l'autorisation du proxy inverse local
# La plupart des distributions Unix refusent le proxy par défaut.
# Voir /etc/apache2/mods-enabled/proxy.conf dans Ubuntu
<Proxy http://localhost:8081/jenkins*>
  Order deny,allow
  Allow from all
</Proxy>

# Si vous utilisez HTTPS, ajoutez les directives suivantes
# RequestHeader set X-Forwarded-Proto « https »
# RequestHeader set X-Forwarded-Port « 443 » 
```

Cela suppose que vous exécutiez Jenkins sur le port 8081. Le chemin d'accès contextuel de Jenkins doit être le même entre Apache et Jenkins. Jenkins ne peut pas s'exécuter sur http://example.com:8081/ci et être proxy inversé sur http://example.com/jenkins . Le [chemin d'accès contextuel](#chemin-dacces-contextuel) fournit plus de détails sur les exigences du proxy inversé pour le chemin d'accès contextuel de Jenkins.

_ProxyRequests Off_ empêche Apache de fonctionner comme un serveur proxy direct (à l'exception de _ProxyPass_). Il est conseillé de l'inclure, sauf si le serveur doit fonctionner comme un proxy.

## mod_proxy_unix_sockets

Apache peut être configuré pour proxyfier les requêtes vers Jenkins via un socket de domaine Unix au lieu d'un port TCP. Cette approche renforce la sécurité en éliminant l'exposition réseau de Jenkins, et est particulièrement utile lorsque Jenkins et Apache s'exécutent sur le même hôte.

Pour éviter l'utilisation répétitive de `sudo` dans les commandes ci-dessous, passez d'abord à un shell root en exécutant :

``` bash title="BASH"
sudo -i
```

### Systèmes basés sur Debian

``` bash title="BASH"
# Activez les modules à l'aide de `a2enmod`.
a2enmod proxy
a2enmod proxy_http

# Redémarrez Apache pour appliquer les modifications
systemctl restart httpd

# Autorisez le trafic HTTP et HTTPS à travers le pare-feu
ufw allow http
ufw allow https
ufw reload
ufw status

# Modifiez le service systemd Jenkins pour désactiver TCP et activer un socket Unix
systemctl edit jenkins

# Ajoutez la substitution suivante
[Service]
ExecStart=
# Définissez le nouveau ExecStart avec la socket Unix
ExecStart=/usr/bin/jenkins --pluginroot=/var/cache/jenkins/plugins --httpPort=-1 --httpUnixDomainPath=/var/run/jenkins/jenkins.socket

# Recharger et redémarrer Jenkins
systemctl daemon-reload
systemctl restart jenkins

# Vérifier que le socket existe
ls -l /var/run/jenkins/jenkins.socket

# S'assurer qu'Apache peut accéder au socket
sudo chown jenkins:jenkins /var/run/jenkins/jenkins.socket
sudo chmod 700 /var/run/jenkins/jenkins.socket

# Créer un nouvel hôte virtuel pour proxyfier les requêtes vers le socket Unix
nano /etc/apache2/sites-available/jenkins.conf

# Ajouter ce qui suit
<VirtualHost *:80>
    ServerName localhost

    # Proxy vers le socket de domaine Unix
    ProxyPass / unix:/var/run/jenkins/jenkins.socket|http://localhost/
    ProxyPassReverse / unix:/var/run/jenkins/jenkins.socket|http://localhost/

    # Conserver l'en-tête hôte
    ProxyPreserveHost On

    # Journalisation
    ErrorLog ${APACHE_LOG_DIR}/jenkins_error.log
    CustomLog ${APACHE_LOG_DIR}/jenkins_access.log combined
</VirtualHost>

# Activer le site et redémarrer Apache
a2ensite jenkins.conf
systemctl restart apache2

# Si Apache doit partager l'accès au socket avec Jenkins, mettez à jour son utilisateur/groupe
nano /etc/apache2/envvars
# Modifiez :
export APACHE_RUN_USER=jenkins
export APACHE_RUN_GROUP=jenkins

# Redémarrez Apache
systemctl restart apache2
```

### Systèmes basés sur RHEL

La vidéo suivante de Darin Pope propose un tutoriel de 16 minutes sur la configuration des sockets de domaine Unix avec le serveur HTTP Apache et Jenkins fonctionnant sous AlmaLinux.

_Configurer à l'aide des sockets de domaine Unix avec le serveur HTTP Apache et Jenkins_

<iframe width="640" height="360" src="https://www.youtube.com/embed/KxFJh4184vk" title="Using Unix Domain Sockets With Apache HTTP Server and Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

``` bash title="BASH"
# Assurez-vous que les modules sont chargés en modifiant la configuration du module.
dnf install httpd -y  # Installez Apache s'il n'est pas déjà présent.

# Autorisez le trafic HTTP et HTTPS à travers le pare-feu.
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --add-service=https --permanent
firewall-cmd --reload

# Modifiez le service systemd Jenkins pour désactiver TCP et activer un socket Unix.
systemctl edit jenkins

# Ajoutez la substitution suivante.
[Service]
ExecStart=/usr/lib/jenkins/jenkins --pluginroot=/var/cache/jenkins/plugins --httpPort=-1 --httpUnixDomainPath=/var/run/jenkins/jenkins.socket

# Rechargez et redémarrez Jenkins.
systemctl daemon-reload
systemctl restart jenkins

# Vérifiez que le socket existe.
ls -l /var/run/jenkins/jenkins.socket

# Assurez-vous qu'Apache peut accéder au socket.
sudo chown jenkins:jenkins /var/run/jenkins
sudo chmod 700 /var/run/jenkins
systemctl restart jenkins

# Créer un nouvel hôte virtuel pour proxyfier les requêtes vers le socket Unix
mv welcome.conf welcome.old
vi /etc/httpd/conf.d/jenkins.conf

# Ajouter ce qui suit
User jenkins
Group jenkins
ProxyPass / unix:/var/run/jenkins/jenkins.socket|http://localhost/ nocanon
ProxyPassReverse / unix:/var/run/jenkins/jenkins.socket|http://localhost/

# Pour autoriser SE Linux à autoriser le trafic vers le socket
chcon -t httpd_sys_rw_content_t /var/run/jenkins/jenkins.socket

# Redémarrer Apache
systemctl restart apache2
```

Une fois la configuration terminée, accédez à [http://localhost/](http://localhost/) dans un navigateur. Si vous rencontrez des problèmes, vérifiez les journaux pour voir s'il y a des erreurs à l'aide des commandes correspondantes :

``` bash title="BASH"
tail -f /var/log/apache2/jenkins_error.log  #Debian
tail -f /var/log/apache2/jenkins_access.log #Debian
```

``` bash title="BASH"
tail -f /var/log/httpd/jenkins_error.log  #RHEL
tail -f /var/log/httpd/jenkins_access.log #RHEL
```

## Chemin d'accès contextuel

Le chemin d'accès contextuel est le préfixe d'un chemin d'accès URL. Le contrôleur Jenkins et le proxy inverse **doivent utiliser le même chemin d'accès contextuel**. Par exemple, si l'URL du contrôleur Jenkins est https://www.example.com/jenkins/, l'argument `--prefix=/jenkins `doit être inclus dans les arguments de ligne de commande du contrôleur Jenkins.

Définissez le chemin d'accès contextuel lorsque vous utilisez les paquets Linux en exécutant `systemctl edit jenkins` et en ajoutant ce qui suit :

``` cfg
[Service]
Environment="JENKINS_PREFIX=/jenkins"
```

Définissez le chemin d'accès contextuel sur les contrôleurs Windows en incluant l'argument de ligne de commande `--prefix` dans le fichier `jenkins.xml` du répertoire d'installation.

Assurez-vous que Jenkins s'exécute dans le chemin d'accès contextuel où votre proxy inverse sert Jenkins. Vous aurez moins de difficultés si vous respectez ce principe.

L'argument de ligne de commande `--prefix` n'est pas nécessaire si le chemin d'accès contextuel est vide. Par exemple, l'URL https://jenkins.example.com/ a un chemin d'accès contextuel vide.

## Proxy des commandes CLI avec le transport HTTP(S)

L'utilisation du protocole CLI simple avec le transport HTTP(S) pour accéder à Jenkins via un proxy inverse Apache ne fonctionne pas. Pour plus d'informations, consultez [JENKINS-47279 - Le transport HTTP(S) full-duplex avec le protocole CLI simple ne fonctionne pas avec le proxy inverse Apache](https://issues.jenkins.io/browse/JENKINS-47279). Pour contourner ce problème, vous pouvez utiliser le [CLI via SSH](./gestion-cli.md#utilisation-de-la-cli-via-ssh).

Si vous utilisez Apache, vérifiez que _nocanon_ est défini sur _ProxyPass_ et que _AllowEncodedSlashes_ est défini.

_AllowEncodedSlashes_ n'est pas hérité dans les configurations Apache, cette directive doit donc être placée dans la définition _VirtualHost_.


