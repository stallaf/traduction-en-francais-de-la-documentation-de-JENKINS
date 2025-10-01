# Fonctionnalités Jenkins contrôlées par les propriétés système

<div class="couleur-introduction">
Jenkins dispose de plusieurs fonctionnalités « cachées » qui peuvent être activées à l'aide des propriétés système. Cette page en répertorie un grand nombre et explique comment les configurer sur votre contrôleur.
</div>

Certaines propriétés système liées à la bibliothèque Remoting utilisée pour la communication entre le contrôleur et les agents sont documentées dans le r[éférentiel de ce composant](https://github.com/jenkinsci/remoting/blob/master/docs/configuration.md).

## Utilisation

Les propriétés système sont définies en passant `-Dproperty=value` à la ligne de commande `java` pour démarrer Jenkins. Veillez à passer tous ces arguments avant l'argument `-jar`, sinon ils seront ignorés. Exemple :

``` sh
java -Dhudson.footerURL=http://example.org -jar jenkins.war
```

La liste suivante répertorie les propriétés et la version de Jenkins dans laquelle elles ont été introduites.

* `Property` (Propriété) - Nom de la propriété Java ;
* **Default** (Par défaut) - Valeur par défaut si elle n'est pas explicitement définie ;
* **Since** (Depuis) - Version de Jenkins dans laquelle la propriété a été introduite ;
* **Description** - Autres remarques.

### Compatibilité

Nous ne garantissons **PAS** que les propriétés du système resteront inchangées et fonctionnelles indéfiniment. Ces commutateurs sont souvent de nature expérimentale et peuvent être modifiés sans préavis. Si vous les trouvez utiles, veuillez remplir un ticket afin qu'ils soient promus au rang de fonctionnalité officielle.

## Propriétés dans Jenkins Core

!!! info " "
    En raison du très grand nombre de propriétés système utilisées, souvent ajoutées simplement comme « soupape de sécurité » ou « issue de secours » au cas où un changement causerait des problèmes, cette liste n'est pas exhaustive.

`debug.YUI`<br>
<span class="tag">Development</span><br>
**Depuis** : décembre 2006<br>
**Par défaut** : `false`<br>
**Description** :<br>
Indique s'il faut utiliser les fichiers JS minifiés (`false`) ou de débogage (`true`) pour la bibliothèque YUI.

`exécutable-war`<br>
<span class="tag">Packaging</span><br>
**Depuis** : _Non documenté_<br>
**Par défaut** : Chemin d'accès à `jenkins.war` lorsqu'il est invoqué en tant que `java -jar jenkins.war`, indéfini dans le cas contraire.<br>
**Description** :<br>
Il s'agit du chemin d'accès à `jenkins.war`, défini par `executable.Main` lorsqu'il est invoqué à l'aide de `java -jar jenkins.war.` Cela permet à Jenkins de trouver son propre fichier `.war` et, par exemple, de le remplacer pour appliquer une mise à jour. S'il n'est pas défini, Jenkins ne proposera pas, par exemple, de se mettre à jour.

`historyWidget.descriptionLimit`<br>
<span class="tag">Feature</span> <span class="tag">UI</span><br>
**Depuis** : 2.223<br>
**Par défaut** : `100`<br>
**Description** :<br>
Définit une limite pour les caractères affichés dans le champ de description de chaque ligne de build dans la colonne Historique des builds. Un nombre entier positif (par exemple `300`) définit la limite. Une fois la limite atteinte, (...​) s'affiche. La valeur `-1` désactive la limite et autorise un nombre illimité de caractères dans la description du build. La valeur `0` n'affiche aucune description.

`hudson.bundled.plugins`<br>
<span class="tag">Development</span><br>
**Depuis** : _Non documenté_<br>
**Par défaut** : Indéfini<br>
**Description** :<br>
Spécifiez un emplacement de stockage pour les plugins supplémentaires fournis avec le plugin pendant le développement du plugin (`hpi:run`). Il n'y a aucune raison pour que cela soit défini par un administrateur.

`hudson.ClassicPluginStrategy.noBytecodeTransformer`<br>
<span class="tag">Escape Hatch</span><span class="tag">Obsolete</span><br>
**Depuis** : 1.538<br>
**Par défaut** :  `false`<br>
**Description** :<br>
Désactive le transformateur de bytecode qui conserve la compatibilité lors de l'exécution après la modification des API Java publiques. N'a aucun effet depuis la version 2.296, car le transformateur de bytecode a été supprimé.

`hudson.ClassicPluginStrategy.useAntClassLoader`<br>
<span class="tag">Escape Hatch</span><br>
**Depuis** : 1.316<br>
**Par défaut** : `false` (jusqu'à 2.309 et depuis 2.348), `true` (de 2.310 à 2.347)<br>
**Description** :<br>
Inutilisé entre 1.527 et 2.309. Depuis 2.310, peut être défini sur `false` pour utiliser `URLClassLoader` à la place. Il s'agit de la valeur par défaut depuis 2.347.

`hudson.cli.CLI.pingInterval`<br>
<span class="tag">Tunning</span><br>
**Depuis** : 2.199<br>
**Par défaut** : `3000`<br>
**Description** :<br>
Intervalle de ping HTTP CLI côté client en millisecondes. Définir sur le client CLI (`java -jar jenkins-cli.jar`), pas sur le processus du serveur Jenkins.

`hudson.cli.CLIAction.ALLOW_WEBSOCKET`<br>
<span class="tag">Escape Hatch</span><span class="tag">Security</span><br>
**Depuis** : 2.426.3 / 2.442<br>
**Par défaut** : _non défini_<br>
**Description** :<br>
Escape hatch pour [SECURITY-3315](https://www.jenkins.io/security/advisory/2024-01-24/#SECURITY-3315). Le comportement par défaut (lorsqu'il n'est pas défini) consiste à appliquer une vérification de l'en-tête `Origin` lorsqu'une tentative d'accès à la CLI via WebSocket est effectuée. Définissez cette option sur `true` pour autoriser l'accès à la CLI via des connexions WebSocket sans vérification de l'en-tête `Origin` (valeur par défaut avant l'introduction de cette propriété système). Définissez cette option sur `false` pour interdire complètement l'accès à la CLI via des connexions WebSocket.

`hudson.cli.CLICommand.allowAtSyntax`<br>
<span class="tag">Escape Hatch</span><span class="tag">Security</span><br>
**Depuis** : 2.426.3 / 2.442<br>
**Par défaut** : `false`<br>
**Description** :<br>
Échappatoire pour [SECURITY-3314](https://www.jenkins.io/security/advisory/2024-01-24/#SECURITY-3314).

`hudson.ConsoleNote.INSECURE`<br>
<span class="tag">Escape Hatch</span><span class="tag">Security</span><br>
**Depuis** : 2.32.2 / 2.44<br>
**Par défaut** : `false`<br>
**Description** :<br>
Indique s'il faut charger les notes de console non signées. Voir SECURITY-382 [dans l'avis de sécurité Jenkins 2017-02-01](https://www.jenkins.io/security/advisory/2017-02-01/#persisted-cross-site-scripting-vulnerability-in-console-notes).

`hudson.consoleTailKB`<br>
<span class="tag">Tunning</span><br>
**Depuis** : mars 2009<br>
**Par défaut** : `150`<br>
**Description** :<br>
Nombre de Ko de journal de console à afficher dans la vue console par défaut. Cette propriété n'avait aucun effet entre Jenkins 2.4 (inclus) et 2.98/2.89.3 (exclus), voir JENKINS-48593.

`hudson.diagnosis.HudsonHomeDiskUsageChecker.freeSpaceThreshold`<br>
<span class="tag">Tunning</span><br>
**Depuis** : 1.339<br>
**Par défaut** : `1073741824` (1 Go, jusqu'à 2,39), `10737418240` (10 Go, à partir de 2,40)<br>
**Description** : Si l'espace disque disponible, en octets, sur le disque contenant le répertoire d'accueil Jenkins est inférieur à cette valeur et que le disque est plein à 90 % ou plus, un avertissement sera affiché à l'intention des administrateurs.

`hudson.diyChunking`<br>
<span class="tag">Feature</span><br>
**Depuis** : mai 2009<br>
**Par défaut** : `false`<br>
**Description** :<br>
Définissez cette option sur `true` si le conteneur de servlets ne prend pas en charge l'encodage par morceaux.

`hudson.DNSMultiCast.disabled`<br>
<span class="tag">Escape Hatch</span><span class="tag">Obsolete</span><br>
**Depuis** : 1.359<br>
**Par défaut** : `false` jusqu'à la version 2.218, `true` dans la version 2.219<br>
**Description** :<br>
Définissez cette option sur `true` pour désactiver la multidiffusion DNS. N'a aucun effet depuis la version 2.220, car cette fonctionnalité a été supprimée. Voir [SECURITY-1641](https://www.jenkins.io/security/advisory/2020-01-29/#SECURITY-1641).

`hudson.FilePath.VALIDATE_ANT_FILE_MASK_BOUND`<br>
<span class="tag">Tunning</span><br>
**Depuis** : 1.592<br>
**Par défaut** : `10000`<br>
**Description** :<br>
Nombre maximal d'opérations pour valider un masque de fichier (par exemple, modèle pour archiver des artefacts).

`hudson.footerURL`<br>
<span class="tag">Feature</span><br>
**Depuis** : 1.416<br>
**Par défaut** : `https://jenkins.io`<br>
**Description** :<br>
Permet de modifier l'URL affichée au bas de l'interface utilisateur de Jenkins.

`hudson.Functions.autoRefreshSeconds`<br>
<span class="tag">Obsolete</span><span class="tag">Tunning</span><br>
**Depuis** : 1.365<br>
**Par défaut** : `10`<br>
**Description** :<br>
Nombre de secondes entre les rechargements lorsque l'actualisation automatique est activée. Obsolète depuis la suppression de cette fonctionnalité dans Jenkins 2.223.

`hudson.Functions.hidingPasswordFields`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 2.205<br>
**Par défaut** : `true`<br>
**Description** :<br>
Jenkins 2.205 et les versions plus récentes tentent d'empêcher les navigateurs de proposer le remplissage automatique des champs de mot de passe à l'aide d'un contrôle de mot de passe personnalisé. En définissant cette option sur `false`, vous revenez au comportement hérité qui consiste à utiliser principalement des champs de mot de passe standard.

`hudson.lifecycle`<br>
<span class="tag">Packaging</span><br>
**Depuis** : _non documenté_<br>
**Par défaut** : déterminé automatiquement en fonction de l'environnement, voir `hudson.lifecycle.Lifecycle`
**Description** :<br>
Spécifiez le nom complet de la classe pour l'implémentation du cycle de vie afin de remplacer la valeur par défaut. Consultez la [documentation](https://www.jenkins.io/doc/developer/extensions/jenkins-core/#lifecycle) pour connaître les noms des classes.

`hudson.logging.LogRecorderManager.skipPermissionCheck`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 2.121.3 / 2.138<br>
*Par défaut* : `false`<br>
**Description** :<br>
Désactive le renforcement de la sécurité pour l'accès à LogRecorderManager Stapler. Potentiellement dangereux, [voir l'avis de sécurité du 05/12/2018](https://www.jenkins.io/security/advisory/2018-12-05/#SECURITY-595).

`hudson.Main.development`<br>
<span class="tag">Development</span><br>
**Depuis** : _non documenté_<br>
**Par défaut** : `false` en production, `true` en développement<br>
**Description** :<br>
Cette option est définie sur `true` par les outils de développement afin d'identifier quand Jenkins s'exécute via `jetty:run` ou `hpi:run`. Elle peut être utilisée pour distinguer l'utilisation en développement de l'utilisation en production ; elle est principalement utilisée pour contourner l'assistant de configuration lors de l'exécution avec un répertoire d'accueil Jenkins vide pendant le développement.

`hudson.Main.timeout`<br>
<span class="tag">Tunning</span><br>
**Depuis** : _non documenté_<br>
**Par défaut** : `15000`<br>
**Description** :<br>
Lorsque vous utilisez `jenkins-core.ja`r à partir de l'interface CLI, il s'agit du délai d'expiration de la connexion à Jenkins pour signaler un résultat de compilation.

`hudson.markup.MarkupFormatter.previewsAllowGET`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 2.263.2 / 2.275<br>
**Par défaut** : `false`<br>
**Description** :<br>
Contrôle si les URL implémentant les aperçus du formateur de balisage sont accessibles via GET. Voir [l'avis de sécurité du 13 janvier 2021](https://www.jenkins.io/security/advisory/2021-01-13/#SECURITY-2153).

`hudson.markup.MarkupFormatter.previewsSetCSP`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 2.263.2 / 2.275<br>
**Par défaut** : `true`<br>
**Description** :<br>
Contrôle si des en-têtes Content-Security-Policy restrictifs doivent être définis sur les URL implémentant les aperçus du formateur de balisage. Voir [l'avis de sécurité du 13 janvier 2021](https://www.jenkins.io/security/advisory/2021-01-13/#SECURITY-2153).

`hudson.matrix.MatrixConfiguration.useShortWorkspaceName`<br>
<span class="tag">Feature</span><br>
**Depuis** : _non documenté_<br>
**Par défaut** : `false`<br>
**Description** :<br>
Utilise des noms plus courts mais cryptiques dans les répertoires de l'espace de travail de construction de la matrice. Évite les problèmes liés à la limite de 256 caractères pour les chemins d'accès dans Cygwin, les problèmes de profondeur des chemins d'accès sous Windows et les problèmes de métacaractères shell avec les expressions d'étiquette sur la plupart des plateformes. Voir [JENKINS-25783](https://issues.jenkins.io/browse/JENKINS-25783).

`hudson.model.AbstractItem.skipPermissionCheck`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 2.121.3 / 2.138<br>
**Par défaut** : `false`<br>
**Description** :<br>
Désactive le renforcement de la sécurité lié au routage Stapler pour AbstractItem. Potentiellement dangereux, [voir l'avis de sécurité du 05/12/2018](https://www.jenkins.io/security/advisory/2018-12-05/#SECURITY-595). 

`hudson.model.Api.INSECURE`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><span class="tag">Obsolete</span><br>
**Depuis** : 1.502<br>
**Par défaut**: `false`<br>
**Description** : <br>
Définissez cette option sur `true` pour autoriser l'accès à l'API distante Jenkins de manière non sécurisée. Voir SECURITY-47. Obsolète, utilisez plutôt [Secure Requester Whitelist](https://plugins.jenkins.io/secure-requester-whitelist/), par exemple.

`hudson.model.AsyncAperiodicWork.logRotateMinutes`<br>
<span class="tag">Tunning</span><br>
**Depuis** : 1.651<br>
**Par défaut** : `1440`<br>
**Description** :<br>
Nombre de minutes après lesquelles essayer de faire tourner le fichier journal utilisé par toute extension AsyncAperiodicWork. Pour un contrôle précis d'une extension spécifique, vous pouvez utiliser la propriété système _`FullyQualifiedClassName`_`.logRotateMinutes` afin de n'affecter qu'une extension spécifique. _Il n'est pas prévu que vous ayez besoin de modifier ces valeurs par défaut_.

`hudson.model.AsyncAperiodicWork.logRotateSize`<br>
<span class="tag">Tunning</span><br>
**Depuis** : 1.651<br>
**Par défaut** : `-1`<br>
**Description** :<br>
Lors du démarrage d'une nouvelle exécution d'une extension AsyncAperiodicWork, si cette valeur est non négative et que le fichier journal existant est plus grand que le nombre d'octets spécifié, le fichier journal sera roté. Pour un contrôle plus précis d'une extension spécifique, vous pouvez utiliser la propriété système _`FullyQualifiedClassName`_`.logRotateSize` afin de n'affecter qu'une extension spécifique. _Il n'est pas prévu que vous ayez besoin de modifier ces valeurs par défaut_.

`hudson.model.AsyncPeriodicWork.logRotateMinutes`<br>
<span class="tag">Tunning</span><br>
**Depuis** : 1.651<br>
**Par défaut** : `1440`<br>
**Description** :<br>
Nombre de minutes après lesquelles essayer de faire tourner le fichier journal utilisé par toute extension AsyncPeriodicWork. Pour un contrôle précis d'une extension spécifique, vous pouvez utiliser la propriété système _`FullyQualifiedClassName`_.`logRotateMinutes` afin de n'affecter qu'une extension spécifique. _Il n'est pas prévu que vous ayez besoin de modifier ces valeurs par défaut_.

Certaines implémentations peuvent être configurées individuellement (voir _FullyQualifiedClassName_ ci-dessus) :

* `hudson.model.WorkspaceCleanupThread` ;
* `hudson.model.FingerprintCleanupThread` ;
* `hudson.slaves.ConnectionActivityMonitor` ;
* `jenkins.DailyCheck` ;
* j`enkins.model.BackgroundGlobalBuildDiscarder` ;
* `jenkins.telemetry.Telemetry$TelemetryReporter`.

`hudson.model.AsyncPeriodicWork.logRotateSize`<br>
<span class="tag">Tunning</span><br>
**Depuis** : 1.651<br>
**Par défaut** : -1<br>
**Description** :<br>
Lors du démarrage d'une nouvelle exécution d'une extension AsyncPeriodicWork, si cette valeur est non négative et que le fichier journal existant est plus grand que le nombre d'octets spécifié, le fichier journal sera roté. Pour un contrôle précis d'une extension spécifique, vous pouvez utiliser la propriété système _`FullyQualifiedClassName`_`.logRotateSize` afin de n'affecter qu'une extension spécifique. _Il n'est pas prévu que vous ayez besoin de modifier ces valeurs par défaut_.

Certaines implémentations peuvent être configurées individuellement (voir _FullyQualifiedClassName _ci-dessus) :

* `hudson.model.WorkspaceCleanupThread` ;
* `hudson.model.FingerprintCleanupThread` ;
* `hudson.slaves.ConnectionActivityMonitor` ;
* `jenkins.DailyCheck` ;
* `jenkins.model.BackgroundGlobalBuildDiscarder` ;
* ` jenkins.telemetry.Telemetry$TelemetryReporter`.

`hudson.model.DirectoryBrowserSupport.allowAbsolutePath`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><span class="tag">Obsolete</span><br>
**Depuis** : 2.303.2 / 2.315<br>
**Par défaut **: `false`<br>
**Description** :<br>
Déprécié : échappatoire pour [SECURITY-2481](https://www.jenkins.io/security/advisory/2021-10-06/#SECURITY-2481). N'a aucun effet depuis la version 2.463.

`hudson.model.DirectoryBrowserSupport.allowSymlinkEscape`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 2.138.4 / 2.154<br>
**Par défaut**: `false`<br>
**Description** :<br>
Échappatoire pour [SECURITY-904](https://www.jenkins.io/security/advisory/2018-12-05/#SECURITY-904) et [SECURITY-1452](https://www.jenkins.io/security/advisory/2021-01-13/#SECURITY-1452).

`hudson.model.DirectoryBrowserSupport.allowTmpEscape`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 2.375.4 / 2.394<br>
**Par défaut** : `false`<br>
**Description** :<br>
Échappatoire pour [SECURITY-1807](https://www.jenkins.io/security/advisory/2023-03-08/#SECURITY-1807).

`hudson.model.DirectoryBrowserSupport.CSP`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 1.625.3 / 1.641<br>
**Par défaut** : `sandbox` ; `default-src “none”` ;  `image-src “self”` ; `style-src “self”`.<br>
**Description** :<br>
Détermine l'en-tête Content Security Policy envoyé pour les fichiers statiques servis par Jenkins. N'affecte que les contrôleurs qui n'ont pas d'URL racine de ressource configurée. Voir [Configuration de la politique de sécurité du contenu](https://www.jenkins.io/doc/book/system-administration/security/configuring-content-security-policy/) pour plus de détails.

`hudson.model.DownloadService$Downloadable.defaultInterval`<br>
<span class="tag">Tunning</span><br>
**Depuis** : 1.5<br>
**Par défaut** : `86400000` (1 jour)<br>
**Description** :<br>
Intervalle entre les téléchargements périodiques des fichiers téléchargeables, généralement les métadonnées d'installation des outils.

`hudson.model.DownloadService.never`<br>
<span class="tag">Obsloete</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 1.319<br>
**Par défaut** : `false`<br>
**Description** :<br>
Supprime le téléchargement périodique des fichiers de données pour les plugins via le téléchargement par navigateur. Depuis Jenkins 2.200, cela n'a plus d'effet.

`hudson.model.DownloadService.noSignatureCheck`<br>
<span class="tag">Security</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 1.482<br>
**Par défaut** : `false`<br>
**Description** :<br>
Ignorer la vérification de la signature du site de mise à jour. Définir cette option sur `true` peut être dangereux.

`hudson.model.Hudson.flyweightSupport`<br>
<span class="tag">Obsolete</span><span class="tag">Feature</span><span class="tag">Escape Hatch</span><br>
**Depuis** : 1.318<br>
**Par défaut** : `false` avant 1.337 ; `true` à partir de 1.337 ; inutilisé depuis 1.598
**Description** :<br>
Les tâches parentales Matrix et autres tâches flyweight (par exemple, le plugin Build Flow) ne consomment pas d'exécuteur lorsque cette option est définie sur `true`. Inutilisée depuis 1.598, la prise en charge flyweight est désormais toujours activée.

`hudson.model.Hudson.initLogLevel`<br>
<span class="tag">Obsolete</span><br>
**Depuis** : _non documenté_<br>
**Par défaut** : _non documenté_<br>
**Description** :<br>
Déprécié : solution de repli compatible avec les versions antérieures pour` jenkins.model.Jenkins.initLogLevel`. Supprimé depuis la version 2.272.

`hudson.model.Hudson.killAfterLoad`<br>
<span class="tag">Obsolete</span><br>
**Depuis** : _non documenté_<br>
**Par défaut** : _non documenté_<br>
**Description** :<br>
Déprécié : solution de repli compatible avec les versions antérieures pour `jenkins.model.Jenkins.killAfterLoad`. Supprimé depuis la version 2.272.

`hudson.model.Hudson.logStartupPerformance`<br>
<span class="tag">Obsolete</span><br>
**Depuis** : _non documenté_<br>
**Par défaut** : _non documenté_<br>
**Description** :<br>
Déprécié : solution de repli rétrocompatible pour `jenkins.model.Jenkins.logStartupPerformance.` Supprimé depuis la version 2.272.

`hudson.model.Hudson.parallelLoad`<br>
<span class="tag">Obsolete</span><br>
**Depuis** : _non documenté_<br>
**Par défaut** : _non documenté_<br>
**Description** :<br>
Déprécié : solution de repli compatible avec les versions antérieures pour `jenkins.model.Jenkins.parallelLoad`. Supprimé depuis la version 2.272.

