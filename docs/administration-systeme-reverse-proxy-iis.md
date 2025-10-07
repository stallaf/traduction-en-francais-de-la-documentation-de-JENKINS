# Proxy inverse - IIS

<div class="couleur-introduction">
Si vous avez déjà des sites Web sur votre serveur, il peut être utile d'exécuter Jenkins (ou le conteneur de servlets dans lequel Jenkins s'exécute) derrière IIS, afin de pouvoir lier Jenkins à la partie d'un site Web plus grand que vous possédez. Cette section présente certaines des approches permettant d'y parvenir.
</div>

**Veillez à modifier l'adresse httpListenAddress de Jenkins, qui est par défaut 0.0.0.0, pour la remplacer par 127.0.0.1, ou configurez le pare-feu pour bloquer les requêtes sur le port auquel Jenkins est lié, sinon toute restriction au niveau IIS peut être facilement contournée en accédant directement au port Jenkins.**

## Configuration requise

* IIS 7.0 ou supérieur ;
    * IIS 8.5 ou supérieur si vous souhaitez utiliser [Certificate Rebind](https://docs.microsoft.com/en-us/iis/get-started/whats-new-in-iis-85/certificate-rebind-in-iis85) ;
* [URL Rewrite 2.1](https://www.iis.net/downloads/microsoft/url-rewrite) ou supérieur ;
    * Comme l'explique [l'annonce](https://blogs.iis.net/iisteam/url-rewrite-v2-1), cela introduit un indicateur de fonctionnalité permettant de désactiver le comportement par défaut non conforme à la norme RFC3986. C'est exactement ce que nous voulons ;
* [Application Request Routing](https://www.iis.net/downloads/microsoft/application-request-routing) 3.0 ou supérieur ;
* Accès au serveur.

## Exemple d'utilisation

Je dispose d'une installation Jenkins dédiée sur un serveur Windows Server 2012 R2 avec un nom commun **VRTJENKINS01** dans le domaine Active Directory **acme.example**, accessible via le nom de domaine complet **vrtjenkins01.acme.example**. De plus, Jenkins fonctionne sur le port *8080* et écoute déjà **127.0.0.1** au lieu de 0.0.0.0, et le serveur dispose de noms DNS supplémentaires : *jenkins* et **jenkins.acme.example**.

Je souhaite disposer d'une installation IIS qui agit comme un proxy inverse de terminaison TLS/SSL. En combinaison avec nos services de certificats Active Directory (ADCS, logiciel d'autorité de certification de Microsoft) internes, cela devrait faciliter considérablement la gestion des certificats, car Windows peut être configuré pour renouveler automatiquement les certificats, et la fonctionnalité de réaffectation des certificats IIS 8.5+ peut écouter les événements de renouvellement (qui contiennent les empreintes digitales de l'ancien et du nouveau certificat) et mettre à jour les liaisons pertinentes pour utiliser le nouveau certificat. Cela garantirait qu'après la demande manuelle initiale, il ne serait nécessaire de modifier manuellement les paramètres liés à TLS/SSL que lorsque l'ensemble des noms de sujet alternatifs sur le certificat présenté par IIS devrait changer.

IIS n'aura qu'à agir comme 1° un proxy inverse pour Jenkins 2° rediriger les URL non canoniques vers l'URL canonique : _https://jenkins.acme.example/_.

J'ai installé le rôle IIS (8.5) à l'aide de l'Assistant _Ajout de rôles et de fonctionnalités_ avec toutes les fonctionnalités par défaut et les fonctionnalités non par défaut suivantes :

* Redirection HTTP (sous Fonctionnalités HTTP courantes, pour rediriger \http(s)://jenkins/, etc. vers [https://jenkins.acme.example/](https://jenkins.acme.example/)) ;
* Protocole WebSocket (sous Développement d'applications, parce que j'en avais envie).

J'ai ensuite installé URL Rewrite et Application Request Routing.

## Temps de configuration

### Activation de la fonctionnalité de proxy inverse

1. Dans le _gestionnaire IIS (Internet Information Services)_, cliquez sur le serveur VRTJENKINS01.
2. Accédez à _Application Request Routing Cache_.
3. Dans le panneau _Actions_, cliquez sur _Server Proxy Settings…​_
4. Activez le proxy.
5. Désactivez _Reverse rewrite host in response header_.
    1. Ne vous inquiétez pas, cela fonctionnera, suivez simplement le reste des instructions.
6. Définissez le_ seuil du tampon de réponse (Ko)_ sur 0.
    1. Cela permet d'éviter les erreurs HTTP 502 sur les pages Replay de Jenkins.
7. Appliquez (à nouveau dans le panneau _Actions_).

### Configuration de TLS/SSL

Hors du champ d'application, il existe suffisamment de tutoriels sur le reste du Web pour cette partie. Le reste de ce tutoriel partira du principe qu'il a été configuré avec un certificat approuvé par le navigateur de votre choix.

### Configuration des règles de réécriture des réponses

1. Accédez au _site Web par défaut_.
2. Accédez à _Réécriture d'URL_.
3. Dans le panneau _Actions_, cliquez sur _Afficher les variables du serveur…_​
4. Ajoutez les éléments suivants s'ils ne sont pas déjà définis au niveau du serveur :
    1. Nom : **HTTP_FORWARDED**
5. Cliquez sur _Retour aux règles_.
6. Cliquez sur _Ajouter une ou plusieurs règles…​_
7. Sélectionnez _Proxy inverse_ et cliquez sur OK.
8. Entrez _jenkins.acme.example_ et cliquez sur OK.
9. Ouvrez la règle que vous venez de créer.
10. Sous _Conditions_, ajoutez :
    1. Entrée de condition : **{CACHE_URL}**
    2. Modèle : **^(http|ws)s://**
11. Sous _Variables du serveur_, ajoutez :
    1. Nom : **HTTP_FORWARDED**, Valeur : **for={REMOTE_ADDR};by={LOCAL_ADDR};host=« {HTTP_HOST} »;proto=« https »**, Remplacer : oui
        1. Jenkins fonctionne sous Jetty, Jetty prend en charge [RFC7239](https://tools.ietf.org/html/rfc7239), donc tout devrait bien se passer.
12. Sous Action, modifiez :
    1. Réécrire l'URL en **\{C:1}\://jenkins.acme.example:8080{UNENCODED_URL}**
        1. Notez qu'il n'y a pas de barre oblique entre le numéro de port et l'accolade ouvrante.
    2. **Décochez** la case **Ajouter une chaîne de requête**.
13. Appliquez les modifications.
14. Modifiez _C:\Windows\System32\drivers\etc\hosts_ afin que **jenkins.acme.example** pointe vers 127.0.0.1.
    1. Lors de la résolution des noms, Windows vérifie si le nom est son propre nom avant de consulter le fichier hosts. Cela signifie que l'ajout de _vrtjenkins01_ ou _vrtjenkins01.acme.example_ au fichier hosts n'aura aucun effet.
        1. Le fichier hosts sera toutefois consulté avant de consulter l'infrastructure DNS.

### Vous rencontrez l'erreur redoutée « Il semble que votre configuration de proxy inverse soit défectueuse. »

1. https://jenkins.acme.example/configure
2. Configurez l'_URL Jenkins_ comme suit : **https://jenkins.acme.example/** et enregistrez la modification.
3. Accédez à _Sécurité_ et activez _Activer la compatibilité proxy_ si vous avez déjà activé _Empêcher les exploits de type Cross Site Request Forgery_.
4. Accédez à https://jenkins.acme.example/manage.
5. Vous rencontrerez toujours l'erreur « Il semble que votre configuration de proxy inverse soit défectueuse. », comme prévu.
    1. Si vous ne rencontrez pas cette erreur à ce stade, c'est très étrange... Continuez quand même.
6. Cliquez avec le bouton droit sur le lien _Système_ et choisissez d'inspecter l'élément.
    1. Assurez-vous que vous êtes toujours sur la page Gérer, car vous en aurez besoin comme référent.
7. Modifiez la valeur de l'attribut _href_ pour qu'elle soit _administrativeMonitor/hudson.diagnosis.ReverseProxySetupMonitor/test_.
8. Ouvrez le lien que vous venez de modifier dans un nouvel onglet.
    1. Gardez cet onglet ouvert.
9. Observez l'erreur « https://jenkins.acme.example/manage vs http: » et savourez sa gloire.
    1. Une page blanche avec un code d'état HTTP 200 indique que tout va bien.
        1. Si vous obtenez cela à ce stade, c'est très étrange... Continuez quand même.

### Correction des erreurs

1. Dans le Gestionnaire IIS, accédez à _Pools d'applications_, puis modifiez _DefaultAppPool_ afin que la version _.NET CLR_ soit **No Managed Code** (Pas de code géré).
    1. Vous constaterez peut-être que cela n'est pas nécessaire (pour autant que vous puissiez en juger) pour votre configuration, car IIS agira uniquement comme un proxy inverse de déchargement TLS/SSL, nous n'en avons donc pas besoin.
2. Ensuite, allez dans _Sites → Site Web par défaut → Filtrage des requêtes_ et, dans le panneau _Actions_, choisissez _Modifier les paramètres de la fonctionnalité…_​ et activez **Autoriser le double échappement**.
    1. Cela permet à IIS de transférer les URL telles que https://jenkins.acme.example/%2525 vers Jenkins au lieu d'afficher une page d'erreur IIS.
3. Enfin, allez dans _Sites → Site Web par défaut → Éditeur de configuration_ et modifiez la _Section_ en _system.webServer/rewrite/rules_.
4. Vous devriez maintenant voir la propriété URL Rewrite 2.1 _useOriginalURLEncoding_ répertoriée. Si ce n'est pas le cas, installez URL Rewrite 2.1 à l'aide du programme d'installation x86 ou x64, et non celui de WebPI, puis reprenez à partir de là après un redémarrage.
5. Modifiez _useOriginalURLEncoding_ sur *False*.
    1. Comme annoncé dans URL Rewrite 2.1, cela modifiera la valeur de {UNENCODED_URL} pour la rendre conforme à la norme _RFC3986_ et utilisable à des fins de transfert de proxy inverse.
    2. Original comme dans le comportement pré-2.1.
6. Actualisez l'onglet que vous étiez censé garder ouvert ou recréez-le.
    1. Encore une fois, prenez le temps d'admirer le résultat.
7. Il devrait maintenant être blanc, et la page Gérer ne devrait plus afficher d'erreur !

### Continuer la configuration d'IIS

Voici quelques éléments qui pourraient vous intéresser, mais que je ne traiterai pas ici :

* En-têtes _Hypertext Strict Transport Security_ ;
* Redirection des URL non canoniques vers l'URL canonique (bon, j'ai en quelque sorte abordé ce sujet dans l'exemple web.config) ;
* L'en-tête X-UA-Compatibility afin qu'Internet Explorer 11 (ou 9, ou ...) ne se présente pas comme IE 7 pour les sites intranet ;
* Utilisation d'IIS Crypto pour configurer les suites de chiffrement
* ...

### Un fichier web.config fonctionnel

**web.config**

``` xml title="XML"
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules useOriginalURLEncoding="false">
        <rule name="CanonicalHostNameRule2" stopProcessing="true">
          <match url="(.*)" />
          <conditions trackAllCaptures="true">
            <add input="{CACHE_URL}" pattern="^(http|ws)://" />
            <add input="{HTTP_HOST}"
                 pattern="^jenkins$|^jenkins\.acme\.example$|
                          ^vrtjenkins01$|^vrtjenkins01\.acme\.example$" />
          </conditions>
          <action type="Redirect"
                  url="{C:1}s://jenkins.acme.example{UNENCODED_URL}"
                  appendQueryString="false"
                  redirectType=« Permanent » />
        </rule>
        <rule name="CanonicalHostNameRule1" stopProcessing="true">
          <match url="(.*)" />
          <conditions trackAllCaptures="true">
            <add input="{CACHE_URL}" pattern="^(https|wss)://" />
            <add input="{HTTP_HOST}" pattern="^jenkins$|^vrtjenkins01$|
                                              ^vrtjenkins01\.acme\.example$" />
          </conditions>
          <action type="Redirect"
                  url="{C:1}://jenkins.acme.example{UNENCODED_URL}"
                  appendQueryString="false" redirectType="Permanent" />
        </rule>
        <rule name="ReverseProxyInboundRule1" stopProcessing="true">
          <match url="(.*)" />
          <action type="Rewrite"
                  url="{C:1}://jenkins.acme.example:8080{UNENCODED_URL}"
                  appendQueryString="false" />
          <serverVariables>
            <set name="HTTP_FORWARDED"
                 value="for={REMOTE_ADDR};
                        by={LOCAL_ADDR};
                        host=&quot;{HTTP_HOST}&quot;;
                        proto=&quot;https&quot;" />
          </serverVariables>
          <conditions trackAllCaptures="true">
            <add input="{CACHE_URL}" pattern="^(http|ws)s://" />
            <add input="{HTTP_HOST}" pattern="^jenkins\.acme\.example$" />
          </conditions>
        </rule>
      </rules>
    </rewrite>
    <security>
      <requestFiltering allowDoubleEscaping="true" />
    </security>
  </system.webServer>
</configuration>
```
