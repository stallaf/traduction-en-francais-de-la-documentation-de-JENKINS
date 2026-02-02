# Proxy inverse - Squid

<div class="couleur-introduction">
Dans les situations où vous souhaitez disposer d'une URL conviviale pour accéder à Jenkins (et non au port 8080), il peut être judicieux d'exécuter Jenkins derrière Squid, afin de pouvoir accéder à Jenkins sur le port 80 ou 443. Cette section présente certaines des approches permettant d'y parvenir.
Squid 2.6
</div>

## Utilisation de Squid 2.6 :

``` cfg
acl all src 0.0.0.0/0.0.0.0
acl localhost src 127.0.0.1/255.255.255.255
acl manager proto cache_object
acl to_localhost dst 127.0.0.0/8
acl valid_dst dstdomain .VOTRE_DOMAINE ci

cache_replacement_policy heap LFUDA
memory_replacement_policy heap GDSF

cache_dir ufs /var/spool/squid 512 16 256
cache_mem 512 Mo
maximum_object_size 12000 Ko

## http --> https redirect
## n'oubliez pas de mettre à jour « Jenkins URL » sur https://ci.YOUR_DOMAIN/configure
#acl httpPort myport 80
#http_access deny httpPort
#deny_info https://ci.YOUR_DOMAIN/ httpPort

cache_peer localhost parent 8080 0 originserver name=myAccel
coredump_dir /var/spool/squid
hierarchy_stoplist cgi-bin
http_access allow localhost
http_access allow manager localhost
http_access allow valid_dst
http_access deny all
http_access deny manager

## mkdir /etc/squid/ssl/ && cd /etc/squid/ssl/
## pour générer votre certificat auto-signé
## openssl genrsa -out jenkins.key 1024
## openssl req -new -key jenkins.key -x509 -out jenkins.crt -days 999
http_port 80 vhost
#https_port 443 cert=/etc/squid/ssl/jenkins.crt key=/etc/squid/ssl/jenkins.key vhost
http_reply_access allow all
icp_access allow all

refresh_pattern -i \.jp(e?g|gif|png|ico)   300  20%  600 override-expire

# Combiner les TROIS LIGNES suivantes en une SEULE LIGNE pour Squid
logformat combined %>a %ui %un \[%tl\]
          « %rm %ru HTTP/%rv » %Hs %<st
          « %{Referer}>h » « %{User-Agent}>h » %Ss:%Sh
strip_query_terms off
access_log /var/log/squid/access.log combined
visible_hostname ci.VOTRE_DOMAINE
```

Cela suppose que vous exécutez Jenkins sur le port 8080 de localhost. Mais vous pouvez l'avoir sur un autre serveur / un autre port (ajustez la ligne commençant par cache_peer).

Bien sûr, remplacez YOUR_DOMAIN par votre domaine.

### Avec ssl

Supprimez un niveau de commentaire.

``` cfg
sed s/^#// /etc/squid/squid.conf
```

Remarque : Si vous utilisez le plugin client swarm, les nœuds peuvent signaler :

``` cfg
Causé par : sun.security.validator.ValidatorException :
    Échec de la création du chemin PKIX : sun.security.provider.certpath.SunCertPathBuilderException :
        impossible de trouver un chemin de certification valide vers la cible demandée
        à sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:285)
        at sun.security.validator.PKIXValidator.engineValidate(PKIXValidator.java:191)
        at sun.security.validator.Validator.validate(Validator.java:218)
        at c.s.n.s.i.s.X509TrustManagerImpl.validate(X509TrustManagerImpl.java:126)
        à c.s.n.s.i.s.X509TrustManagerImpl.checkServerTrusted(X509TrustManagerImpl.java:209)
        à c.s.n.s.i.s.X509TrustManagerImpl.checkServerTrusted(X509TrustManagerImpl.java:249)
        à c.s.n.s.i.s.ClientHandshaker.serverCertificate(ClientHandshaker.java:1014)
        ... 13 autres
Causé par : sun.security.provider.certpath.SunCertPathBuilderException :
        impossible de trouver un chemin de certification valide vers la cible demandée
```

Vous pouvez éviter ce message en utilisant l'argument `-noCertificateCheck` avec `agent.jar.` Cela désactivera la vérification du certificat serveur par l'agent.

## Chemin d'accès contextuel

Le chemin d'accès contextuel est le préfixe d'un chemin d'accès URL. Le contrôleur Jenkins et le proxy inverse doivent utiliser le même chemin d'accès contextuel. Par exemple, si l'URL du contrôleur Jenkins est https://www.example.com/jenkins/, l'argument `--prefix=/jenkins` doit être inclus dans les arguments de ligne de commande du contrôleur Jenkins.

Définissez le chemin d'accès au contexte lorsque vous utilisez les paquets Linux en exécutant `systemctl edit jenkins` et en ajoutant ce qui suit :

``` cfg
[Service]
Environment="JENKINS_PREFIX=/jenkins"
```

Définissez le chemin d'accès au contexte sur les contrôleurs Windows en incluant l'argument de ligne de commande `--prefix` dans le fichier `jenkins.xml `du répertoire d'installation.

Assurez-vous que Jenkins s'exécute sur le chemin d'accès contextuel où votre proxy inverse sert Jenkins. Vous aurez moins de difficultés si vous respectez ce principe.

L'argument de ligne de commande `--prefix` n'est pas nécessaire si le chemin d'accès contextuel est vide. Par exemple, l'URL https://jenkins.example.com/ a un chemin d'accès contextuel vide.
