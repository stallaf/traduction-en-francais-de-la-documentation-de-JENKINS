# Configuration en tant que Code

<div class="couleur-introduction">
La fonctionnalité Jenkins Configuration as Code (JCasC) définit les paramètres de configuration Jenkins dans un fichier YAML lisible par l'utilisateur qui peut être stocké en tant que code source. Elle capture essentiellement les paramètres et les valeurs de configuration utilisés lors de la configuration de Jenkins à partir de l'interface utilisateur Web. La configuration peut ensuite être modifiée en éditant ce fichier, puis en l'appliquant.
</div>

!!! info " "
    Traditionnellement, les administrateurs Jenkins expérimentés créaient des scripts `init` Apache Groovy pour automatiser la configuration de leurs contrôleurs Jenkins. Cette méthode fonctionne, mais nécessite une compréhension approfondie des API Jenkins et la capacité à écrire des scripts Groovy. Ces scripts sont puissants et peuvent presque tout faire, mais ils offrent également peu de protections contre les erreurs de configuration.

JCasC offre la commodité et la flexibilité de configurer les contrôleurs sans utiliser l'interface utilisateur. Il ne nécessite pas une compréhension des paramètres de configuration plus approfondie que celle requise pour configurer Jenkins via l'interface utilisateur et fournit certaines vérifications des valeurs fournies.

Le fichier de configuration JCasC peut être enregistré dans un SCM, ce qui vous permet de déterminer qui a apporté quelles modifications à la configuration et de revenir à une configuration précédente si nécessaire.

Le plugin [Configuration as Code](https://plugins.jenkins.io/configuration-as-code) doit être installé sur le contrôleur Jenkins que vous utiliserez pour créer votre configuration JCasC. Si vous ne voyez pas la vignette **Configuration as Code** (Configuration as Code) dans la section **System Configuration** (Configuration du système) de la page **Manage Jenkins** (Gérer Jenkins) de votre tableau de bord, vous devez installer le plugin. 

## Affichage du fichier JCasC

Une fois le plugin Configuration as Code installé, vous verrez **Configuration as Code** (Configuration as Code) dans la section **System Configuration** (Configuration du système) de la page **Manage Jenkins** (Gérer Jenkins) de votre tableau de bord. Cliquez sur ce lien, puis sur **View Configuration** (Afficher la configuration) pour afficher le fichier YAML.

Ce fichier est une exportation de la configuration actuelle de ce contrôleur. Dans la plupart des cas, il est prêt à être utilisé sans modification, même si vous souhaitez généralement le personnaliser avant de le déployer. Vous pouvez envoyer la version non modifiée vers SCM afin de la conserver dans votre historique.

!!! info " "
    La page **Configuration as Code** UI affiche le chemin d'accès complet du fichier YAML utilisé et propose une zone dans laquelle vous pouvez spécifier un autre fichier à utiliser. Reportez-vous aux informations ci-dessous pour savoir comment modifier l'emplacement utilisé pour les fichiers YAML JCasC.

Le fichier YAML JCasC par défaut comporte quatre sections :

* La section `jenkins` définit l'objet Jenkins racine, avec des configurations qui peuvent être définies à l'aide des écrans **Manage Jenkins >> System** (Gérer Jenkins >> Système) et **Manage Jenkins >> Configure Nodes and Clouds** (Gérer Jenkins >> Configurer les nœuds et les clouds).
* La section `tool` définit les outils de construction qui peuvent être configurés dans l'écran **Manage Jenkins >> Tools** (Gérer Jenkins >> Outils).
* La section `unclassified` définit toutes les autres configurations, y compris la configuration des plugins installés.
* La section `credentials` définit les informations d'identification qui peuvent être configurées dans l'écran **Manage Jenkins >> Manage Credentials** (Gérer Jenkins >> Gestion d'Identification). Vous pouvez supprimer cette section de votre fichier YAML ; cette opération est décrite dans la section [Comment installer Jenkins à l'aide d'Ansible et de JCasC](https://youtu.be/ANU7jkxbZSM?t=1673).

## Syntaxe du fichier YAML

JCasC définit la configuration du contrôleur dans un fichier YAML. YAML est un langage de sérialisation populaire pour les informations de configuration, avec une syntaxe simple, facile à lire et précise.

Quelques points clés concernant la syntaxe YAML :

* Les fichiers YAML sont sensibles à la casse ;
* L'indentation est très importante et spécifique ;
* Chaque élément est une paire clé/valeur ;
    * La clé est suivie d'un deux-points (`:`) et d'un espace ;
    * YAML convertit certaines chaînes en d'autres types, sauf si elles sont entre guillemets ;
        * Les valeurs telles que `true`, `false`, `Yes` et `No` sont converties en valeurs booléennes ;
        * Les valeurs telles que `2` et `3.0` sont converties en valeurs à virgule flottante ;
* Une valeur peut être une liste :
    * Chaque élément de liste se trouve sur une ligne distincte commençant par un tiret (`-`) ;
    * Chaque élément de la liste dans le fichier doit commencer avec la même indentation ;
        * Utilisez des espaces, jamais des tabulations, pour l'indentation ;
    * Ne laissez jamais de ligne vide dans un fichier YAML, cela causerait des dysfonctionnements !

Consultez la [fiche de référence YAML](https://yaml.org/refcard.html) pour plus de détails sur la syntaxe des fichiers YAML.

## Enregistrement des fichiers YAML dans le SCM

Pour tirer le meilleur parti de JCasC, les fichiers YAML doivent être stockés dans le SCM. Cela vous permet de disposer d'un historique que vous pouvez utiliser pour suivre les modifications apportées et vous permet de revenir facilement à une version antérieure du fichier si nécessaire.

JCasC n'exige pas que le fichier soit stocké dans SCM et n'impose donc aucune règle à ce sujet. La pratique la plus courante consiste à créer un référentiel SCM dans lequel vous stockez tous vos fichiers JCasC.

Si vous stockez vos fichiers YAML JCasC dans SCM, vous devez valider le premier fichier par défaut généré avant d'apporter des modifications au fichier.

## Modification du fichier JCasC

Pour modifier le fichier YAML JCasC, utilisez l'éditeur de texte de votre choix pour modifier le fichier répertorié sur la page **Manage Jenkins >> Configuration as Code** (Gérer Jenkins >> Configuration en tant que Code) UI. Par défaut, il s'agit de _$JENKINS_HOME/jenkins.yaml_.

Pour vous familiariser avec le processus, vous pouvez modifier la valeur du « message système » affiché sur le tableau de bord Jenkins.

1. Ouvrez le fichier YAML JCasC avec l'éditeur de texte de votre choix.
2. Recherchez la ligne `systemMessage` vers le haut du fichier :
``` yaml
jenkins:
  systemMessage: « Jenkins configured automatically by Jenkins Configuration as Code plugin\n\n »
  # (Jenkins configuré automatiquement par le plugin Jenkins Configuration as Code)
```
3. Modifiez le texte entre les guillemets pour y insérer votre nouveau texte.
4. Écrivez/enregistrez le fichier.
5. Cliquez sur le bouton **Reload existing configuration**  (Recharger la configuration existante) pour appliquer les modifications.
6. Affichez le « message système » modifié sur votre tableau de bord.

!!! info " "
    Il n'est pas nécessaire de redémarrer Jenkins pour appliquer les modifications JCasC, mais vous devriez essayer de redémarrer Jenkins avec les modifications avant d'enregistrer le fichier YAML modifié dans le SCM, en particulier lorsque vous effectuez des modifications de configuration plus importantes.

Une fois que vous avez effectué et testé les modifications souhaitées, transférez le fichier YAML JCasC modifié vers votre SCM.

## Configuration d'un plugin avec JCasC

Pour configurer un plugin avec JCasC :

1. Utilisez l'interface utilisateur du système actuel pour installer et configurer le plugin.
2. Cliquez sur **Apply >> Save** (Appliquer >> Enregistrer) pour enregistrer la configuration.
3. Utilisez **Manage Jenkins >> Configuration as Code >> View Configuration** (Gérer Jenkins >> Configuration en tant que code >> Afficher la configuration) pour afficher le fichier JCasC avec le plugin configuré.
4. Cliquez sur **Download Configuration** (Télécharger la configuration) pour enregistrer le fichier de configuration modifié localement.
5. Modifiez le fichier YAML JCasC pour modifier la configuration, si nécessaire.
6. Enregistrez le fichier.
7. Cliquez sur **Reload existing configuration** (Recharger la configuration existante) pour charger les modifications locales sur le serveur Jenkins.
8. Vérifiez les modifications dans l'interface utilisateur.
9. Une fois que vous avez testé de manière approfondie la configuration du plugin, transférez le fichier YAML modifié vers votre SCM.

Consultez le blog [Configurer des plugins avec JCasC](https://www.jenkins.io/blog/2021/05/20/configure-plugins-with-jcasc/) pour obtenir des instructions détaillées et une vidéo de démonstration intégrée de ce processus.

## Emplacement du fichier YAML

Par défaut, le fichier YAML pour la configuration CasC se trouve dans `$JENKINS_HOME/jenkins.yaml`. L'emplacement et le nom du fichier utilisé sont affichés sur la page **Configuration as Code** (Configuration as Code) UI. Vous pouvez spécifier un autre fichier à afficher en saisissant le chemin d'accès complet dans le champ Path ou URL.

Vous pouvez spécifier un emplacement ou un nom de fichier différent pour la création du fichier YAML JCasC en procédant de l'une des manières suivantes :

* Remplissez la variable d'environnement `CASC_JENKINS_CONFIG` pour qu'elle pointe vers une liste séparée par des virgules qui définit l'emplacement des fichiers de configuration ;
* Utilisez la propriété Java `casc.jenkins.config` pour contrôler le nom et l'emplacement du fichier. Cela est utile lors de l'installation de Jenkins via un outil de gestion des paquets. La plupart des systèmes de gestion des paquets prennent en charge les fichiers de configuration qui sont conservés lors des mises à niveau. Il est préférable de ne pas modifier un fichier installé par un gestionnaire de paquets, car il pourrait être écrasé par une mise à jour.
Sur les systèmes Linux, vous pouvez exécuter `systemctl edit jenkins` et ajouter ce qui suit :

``` cfg
[Service]
Environment="JAVA_OPTS=-Dcasc.jenkins.config=/jenkins/casc_configs"
```

L'emplacement et le nom du fichier peuvent être spécifiés comme suit :

* Chemin d'accès à un dossier contenant un ensemble de fichiers de configuration, tel que `/var/jenkins_home/casc_configs` ;
* Chemin d'accès complet à un fichier unique, tel que `/var/jenkins_home/casc_configs/jenkins.yaml` ;
* Une URL pointant vers un fichier hébergé sur le Web, tel que [https://acme.org/jenkins.yaml](https://acme.org/jenkins.yaml).

La valeur de la variable `CASC_JENKINS_CONFIG` est décompressée selon les règles suivantes :

* Si un élément de `CASC_JENKINS_CONFIG` pointe vers un dossier, le plugin parcourt récursivement le dossier pour trouver le ou les fichiers avec l'extension .yml, .yaml, .YAML ou .YML ;
* Il exclut les fichiers cachés ou les fichiers qui contiennent un dossier caché (tel que `/ jenkins/casc_configs/.dir1/config.yaml`) dans n'importe quelle partie du chemin complet ;
* Il suit les liens symboliques pour les fichiers et les répertoires ;
* L'ordre de parcours n'a pas d'importance pour le résultat final, car tous les fichiers de configuration découverts DOIVENT être supplémentaires. Si un fichier tente de remplacer les valeurs de configuration d'un autre fichier, cela crée un conflit et déclenche une exception `ConfiguratorException`.

### Configuration CasC et modifications de l'interface utilisateur

La configuration d'un contrôleur Jenkins doit être mise en œuvre soit avec CasC, soit avec l'interface utilisateur, mais pas avec les deux. Le système permet aux administrateurs de modifier les options de configuration sur l'interface utilisateur même si elles ont été configurées par CasC, mais ces modifications sont écrasées au prochain redémarrage du contrôleur.

Vous pouvez installer le [plugin Extended Read Permission](https://github.com/jenkinsci/extended-read-permission-plugin), qui vous permet d'accorder aux utilisateurs un accès en lecture seule aux paramètres de configuration. Pour plus d'informations, consultez [JEP-224 : Configuration système en lecture seule](https://github.com/jenkinsci/jep/blob/master/jep/224/README.adoc).

Un petit groupe d'administrateurs peut disposer d'un accès en écriture aux champs de configuration de l'interface utilisateur. Ils doivent comprendre que JCasC écrasera les modifications qu'ils apportent à l'interface utilisateur.

## Pour plus d'informations

### Informations générales

* [Look Ma! No Hands! — Manage Jenkins Configuration as Code](https://www.youtube.com/watch?v=47D3H1BZi4o) est une vidéo de la présentation DevOps World 2018 qui a introduit la fonctionnalité JCasC ;
* [Configure Plugins with JCasC](Configure Plugins with JCasC) est un article de blog accompagné d'une vidéo qui montre comment configurer un plugin avec JCasC ;
* [How to Install Jenkins Using Ansible and JCasC](https://www.youtube.com/watch?v=ANU7jkxbZSM) est une présentation vidéo qui détaille l'utilisation de JCasC.

### Détails de mise en œuvre

La plupart des informations détaillées relatives à JCasC sont disponibles dans le référentiel Github.

* [Détails de mise en œuvre](https://github.com/jenkinsci/configuration-as-code-plugin/blob/master/docs/IMPLEMENTATION.md) :
    * Le répertoire [demos](https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos) contient des exemples de fichiers _*.yaml_ permettant de configurer des composants et des plugins Jenkins spécifiques, ainsi qu'un fichier _README_ dans chaque répertoire décrivant les configurations pour ce composant ;
    * [Comment créer le job « seed » initial avec Job DSL](https://github.com/jenkinsci/configuration-as-code-plugin/blob/master/docs/seed-jobs.md) ;
    * [Scénarios d'utilisation](https://github.com/jenkinsci/configuration-as-code-plugin/blob/master/docs/usageScenarios.md) ;
    * [Déclenchement du rechargement de la configuration](https://github.com/jenkinsci/configuration-as-code-plugin/blob/master/docs/features/configurationReload.md) ;
    * [Exportation des configurations](https://github.com/jenkinsci/configuration-as-code-plugin/blob/master/docs/features/configExport.md).

### Informations pour les développeurs et les responsables de maintenance des plugins

* [Documentation pour les développeurs ](https://github.com/jenkinsci/configuration-as-code-plugin/blob/master/docs/DEVELOPER.md) JCasC ;
* [Exigences JCasC - guide pour les responsables de maintenance des plugins](https://github.com/jenkinsci/configuration-as-code-plugin/blob/master/docs/REQUIREMENTS.md).

