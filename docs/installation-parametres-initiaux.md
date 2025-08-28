# Paramètres Initiaux

La plupart des modifications de configuration de Jenkins peuvent être effectuées via l'interface utilisateur Jenkins ou via la [configuration en tant que plugin de code](https://plugins.jenkins.io/configuration-as-code). Il existe certaines valeurs de configuration qui ne peuvent être modifiées que pendant le démarrage de Jenkins. Cette section décrit ces paramètres et comment vous pouvez les utiliser.

## Paramètres Jenkins

L'initialisation de Jenkins peut également être contrôlée par des paramètres d'exécution passés en tant qu'arguments. Les arguments de ligne de commande permettent d'ajuster les paramètres réseau, de sécurité, de surveillance et autres.

### Paramètres réseau

La configuration réseau de Jenkins est généralement contrôlée par des arguments en ligne de commande. Les arguments de configuration de mise en réseau sont :

| Paramètres de ligne de commande | Description |
| ------------------------------------------------------ | ----------------- |
| `--httpPort=$HTTP_PORT`        | Exécute le listener Jenkins sur le port $HTTP_PORT à l'aide du protocole http standard. Le port par défaut est 8080. Pour désactiver cette option (parce que vous utilisez _https_), utilisez le port `-1`. Cette option n'a aucun impact sur l'URL racine générée dans la logique Jenkins (interface utilisateur, fichiers d'agent entrants, etc.). Elle est définie par l'URL Jenkins spécifiée dans la configuration globale.
| `--httpListenAddress=$HTTP_HOST`    | Lie Jenkins à l'adresse IP représentée par $HTTP_HOST. La valeur par défaut est 0.0.0.0, c'est-à-dire à l'écoute sur toutes les interfaces disponibles. Par exemple, pour n'écouter que les requêtes provenant de localhost, vous pouvez utiliser : `--httpListenAddress=127.0.0.1`.
| `--httpsPort=$HTTPS_PORT` | Utilise le protocole HTTPS sur le port $HTTPS_PORT. Cette option n'a aucune incidence sur l'URL racine générée dans la logique Jenkins (interface utilisateur, fichiers d'agent entrants, etc.). Elle est définie par l'URL Jenkins spécifiée dans la configuration globale.
| `--httpsListenAddress=$HTTPS_HOST`  | Lie Jenkins pour écouter les requêtes HTTPS sur l'adresse IP représentée par $HTTPS_HOST.
| `--http2Port=$HTTP_PORT`    | Utilise le protocole HTTP/2 sur le port $HTTP_PORT. Cette option n'a aucune incidence sur l'URL racine générée dans la logique Jenkins (interface utilisateur, fichiers d'agent entrants, etc.). Elle est définie par l'URL Jenkins spécifiée dans la configuration global
| `--http2ListenAddress=$HTTPS_HOST`  | Lie Jenkins pour écouter les requêtes HTTP/2 sur l'adresse IP représentée par $HTTPS_HOST.
| --prefix=$PREFIX  | Exécute Jenkins pour inclure le préfixe $PREFIX à la fin de l'URL. Par exemple, définissez _--prefix=/jenkins pour rendre Jenkins accessible à l'adresse http://myServer:8080/jenkins_.
| `--sessionTimeout=$TIMEOUT` |Définit la valeur du délai d'expiration de la session http à $SESSION_TIMEOUT minutes. La valeur par défaut est celle spécifiée par l'application web, puis 60 minutes. |

### Paramètres divers

D'autres options d'initialisation de Jenkins sont également contrôlées par des arguments en ligne de commande. Les arguments de configuration divers sont :

| Paramètres de ligne de commande | Description |
| ------------------------------------------------------ | ----------------- |
| `--argumentsRealm.passwd.$USER=$PASS` | Attribue le mot de passe à l'utilisateur $USER. Si la sécurité Jenkins est activée, vous devez vous connecter en tant qu'utilisateur disposant d'un rôle d'administrateur pour configurer Jenkins.
| `--argumentsRealm.roles.$USER=admin`  | Attribue le rôle d'administrateur à l'utilisateur $USER. L'utilisateur peut configurer Jenkins même si la sécurité est activée dans Jenkins. Pour plus d'informations, consultez la section [Sécurisation de Jenkins](./securite-securiser-jenkins.md).
| `--paramsFromStdIn`   | Lit les paramètres à partir de l'entrée standard (stdin). Lorsque les paramètres sont transmis via la ligne de commande, ils peuvent être consultés à l'aide de `ps(1)` sous Unix ou Process Explorer sous Windows tant que le processus continue de s'exécuter. Cela n'est pas souhaitable lors de la transmission de paramètres sensibles tels que `--httpsKeyStorePassword`. Avec le paramètre `--paramsFromStdIn`, vous pouvez remplacer par exemple :<br>`java -jar jenkins.war --httpPort=-1 --httpsPort=443 --httpsKeyStore=path/to/keystore --httpsKeyStorePassword=keystorePassword`<br>  par <br>`echo “--httpPort=-1 --httpsPort=443 --httpsKeyStore=path/to/keystore --httpsKeyStorePassword=keystorePassword” | java -jar jenkins.war --paramsFromStdIn`.
| `--useJmx`    | Activer [l'extension de gestion Java Jetty (JMX)](https://www.eclipse.org/jetty/documentation/current/#jmx-chapter) |

Jenkins passe tous les paramètres de ligne de commande au conteneur de servlet Winstone. Plus d'informations sur les paramètres de la ligne de commande Jenkins Winstone sont disponibles à partir de la référence des [paramètres de ligne de commande Winstone](https://github.com/jenkinsci/winstone#command-line-options). 

!!! danger "Soyez prudent avec les paramètres de ligne de commande"

    Jenkins ignore les paramètres de ligne de commande qu'il ne comprend pas au lieu de produire une erreur. Soyez prudent lorsque vous utilisez des paramètres de ligne de commande et assurez-vous d'avoir l'orthographe correcte. Par exemple, le paramètre nécessaire pour définir l'utilisateur administrateur de Jenkins est `--argumentsRealm` et non `--argumentRealm`.
 
### Propriétés de Jenkins

Certains comportements de Jenkins sont configurés à l'aide de propriétés Java. Les propriétés Java sont définies à partir de la ligne de commande qui a lancé Jenkins. Les affectations de propriétés utilisent le format `-DsomeName=someValue` pour attribuer la valeur `someValue` à la propriété nommée `someName`. Par exemple, pour attribuer la valeur `true` à une propriété `testName`, l'argument de ligne de commande serait `-DtestName=true`.

Reportez-vous à la liste détaillée des [propriétés de Jenkins](./gestion-jenkins.md#fonctionnalite) pour plus d'informations.

## Configuration de HTTP

### HTTPS avec un certificat existant

Si vous configurez Jenkins à l'aide du serveur Winstone intégré et souhaitez utiliser un certificat existant pour HTTPS :

``` bash title="BASH"
--httpPort=-1 \
--httpsPort=443 \
--httpsKeyStore=path/to/keystore \
--httpsKeyStorePassword=keystorePassword
```

### Utilisation de HTTP/2

Le protocole HTTP/2 permet aux serveurs web de réduire la latence sur les connexions cryptées en mettant en pipeline les requêtes, en multiplexant les requêtes et en permettant aux serveurs, dans certains cas, d'envoyer des données avant même de recevoir une requête du client. Le serveur Jetty utilisé par Jenkins prend en charge HTTP/2 avec l'ajout de l'extension TLS ALPN (Application-Layer Protocol Negotiation).

!!! info
    L'activation de HTTP/2 active implicitement TLS même si aucun port HTTPS n'est défini. À partir de Jenkins 2.339, qui utilise Winstone 5.23, vous devez également spécifier un fichier de stockage de clés HTTPS.

``` bash title="BASH"
--httpPort=-1 \
--http2Port=9090 \
--httpsKeyStore=path/to/keystore \
--httpsKeyStorePassword=keystorePassword
```

### Certificats HTTPS avec Windows

Ces instructions utilisent une installation Jenkins standard sur Windows Server. Elles supposent que vous disposez d'un certificat signé par une autorité de certification telle que Digicert. Si vous créez votre propre certificat, ignorez les étapes 3, 4 et 5.

Ce processus utilise l'outil keytool de Java. Utilisez l'outil `keytool` fourni avec votre installation Java.

**Étape 1 :** Créez un nouveau keystore sur votre serveur. Cela placera un fichier « keystore » dans votre répertoire actuel.

``` bat title="BAT"
C:\>keytool -genkeypair -keysize 2048 -keyalg RSA -alias jenkins -keystore keystore
Enter keystore password:
Re-enter new password:
What is your first and last name?
[Unknown]: server.example.com
What is the name of your organizational unit?
[Unknown]: A Unit
What is the name of your organization?
[Unknown]: A Company
What is the name of your City or Locality?
[Unknown]: A City
What is the name of your State or Province?
[Unknown]: A State
What is the two-letter country code for this unit?
[Unknown]: US
Is CN=server.example.com, OU=A Unit, O=A Company, L=A City, ST=A State, C=US correct?
[no]: yes

Enter key password for <jenkins>
(RETURN if same as keystore password):
```

**Étape 2 :** Vérifiez que le keystore a bien été créé (votre empreinte digitale variera).

``` bat title="BAT"
C:\>keytool -list -keystore keystore
Enter keystore password:

Keystore type: JKS
Keystore provider: SUN

Your keystore contains 1 entry

jenkins, May 6, 2015, PrivateKeyEntry,
Certificate fingerprint (SHA1): AA:AA:AA:AA:AA:AA:AA:AA:AA:AA ...
```

**Étape 3 :** Créez la demande de certificat. Cela créera un fichier «certreq.csr» dans votre répertoire actuel.

``` bat title="BAT"
C:\>keytool -certreq -alias jenkins -keyalg RSA ^
-file certreq.csr ^
-ext SAN=dns:server-name,dns:server-name.your.company.com ^
-keystore keystore
Enter keystore password:
```

**Étape 4 :** Utilisez le contenu du fichier `certreq.csr` pour générer un certificat de votre fournisseur de certificat. Demandez un certificat SHA-1 (SHA-2 n'est pas testé mais fonctionnera probablement). Si vous utilisez DigiCert, téléchargez le certificat résultant comme autre format "a .p7b bundle of all the certs in a .p7b file".

**Étape 5 :** Ajoutez le .p7b résultant dans le gesté que vous avez créé ci-dessus.

``` bat title="BAT"
C:\>keytool -import ^
-alias jenkins ^
-trustcacerts ^
-file response_from_digicert.p7b ^
-keystore keystore
Enter keystore password:
Certificate reply was installed in keystore
```

**Étape 6 :** Copiez le fichier «keystore» dans votre répertoire Jenkins Secrets. Sur une installation standard, il se trouve à l'emplacement suivant :

``` cfg
C:\Program Files (x86)\Jenkins\secrets
```

**Étape 7 :** Modifiez la section <rarguments> de votre fichier `C:\Program Files (x86)\Jenkins\jenkins.xml`  pour refléter le nouveau certificat. REMARQUE : Cet exemple désactive `--httpPort=-1` et place le serveur sur `8443` via `--httpsPort=8443`.

``` xml title="XML"
<arguments>
  -Xrs
  -Xmx256m
  -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle
  -jar "%BASE%\jenkins.war"
  --httpPort=-1
  --httpsPort=8443
  --httpsKeyStore="%BASE%\secrets\keystore"
  --httpsKeyStorePassword=your.password.here
</arguments>
```

**Étape 8 :** Redémarrez le service Jenkins pour initialiser la nouvelle configuration.

``` bat title="BAT"
net stop jenkins
net start jenkins
```

**Étape 9 :** Après 30 à 60 secondes, Jenkins aura terminé le processus de démarrage et vous devriez pouvoir accéder au site Web à _https://server.example.com:8443_. Vérifiez que le certificat semble bon via les outils de votre navigateur. Si le service se termine immédiatement, il y a une erreur quelque part dans votre configuration. Des informations d'erreur utiles peuvent être trouvées dans :

``` cfg
C:\Program Files (x86)\Jenkins\jenkins.err.log
C:\Program Files (x86)\Jenkins\jenkins.out.log
```
