# Isolation du Contrôleur

Ce qui se passe exactement pendant une compilation est souvent contrôlé par des personnes moins fiables qu'un administrateur Jenkins :

* Les utilisateurs Jenkins disposant d'une autorisation Job/Configure (Tâche/Configurer) ;
* Les auteurs de scripts de compilation (`pom.xml`, `Makefile`, etc.) ;
* Les auteurs de code (par exemple, le code de test exécuté pendant une compilation).

Ils ont tous un certain contrôle sur les commandes exécutées pendant une compilation.

Pour garantir la stabilité du contrôleur Jenkins, les compilations doivent être exécutées sur d'autres nœuds que le nœud intégré. Ce concept est appelé _distributed compilations _ (compilations distribuées) dans Jenkins. Pour en savoir plus, [cliquez ici](https://www.jenkins.io/doc/book/scaling/architecting-for-scale/). La configuration de compilations distribuées dans Jenkins est un excellent point de départ pour protéger le contrôleur Jenkins contre les scripts de compilation malveillants (ou simplement défectueux), mais il faut veiller à ce que les protections soient efficaces.

!!! danger "Attention"
    La plupart des environnements Jenkins se développent au fil du temps, ce qui nécessite une évolution de leurs modèles de confiance à mesure que l'environnement se développe. Envisagez de programmer des « vérifications » régulières afin de déterminer si des paramètres de sécurité désactivés doivent être réactivés.

## Ne pas effectuer de build sur le nœud intégré

Par défaut, Jenkins est configuré pour exécuter les compilations sur le nœud intégré. Cela facilite la prise en main de Jenkins, mais n'est pas recommandé à long terme : tous les compilations exécutés sur le nœud intégré ont le même niveau d'accès au système de fichiers du contrôleur que le processus Jenkins.

Il est donc fortement recommandé de ne pas exécuter de compilations sur le nœud intégré, mais d'utiliser plutôt des agents (configurés de manière statique ou fournis par des _clouds_) pour exécuter les compilations.

Pour empêcher les compilations de s'exécuter directement sur le nœud intégré, accédez à _Manage Jenkins » Nodes and Clouds_. Sélectionnez _Built-In Node_ dans la liste, puis sélectionnez _Configure_ dans le menu. Définissez le nombre d'exécuteurs sur 0 et enregistrez. Veillez également à configurer des clouds ou des agents de build pour exécuter les compilations, sinon ceux-ci ne pourront pas démarrer.

Vous pouvez également utiliser un plugin tel que [Job Restrictions Plugin](https://plugins.jenkins.io/job-restrictions) pour limiter les tâches pouvant être exécutées sur certains nœuds (comme le nœud intégré), indépendamment de ce que vos utilisateurs moins fiables peuvent utiliser comme expression de label dans la configuration de leurs tâches.
	
!!! info " "
    Si vous ne disposez d'aucun autre ordinateur pour exécuter des agents, vous pouvez également exécuter un processus d'agent en tant qu'utilisateur d'un autre système d'exploitation sur le même système afin d'obtenir un effet d'isolation similaire. Dans ce cas, assurez-vous que le processus d'agent n'a pas accès au système de fichiers (ni en lecture ni en écriture) du répertoire d'accueil Jenkins et qu'il ne peut pas utiliser `sudo` ou des méthodes similaires pour élever ses propres permissions.

## Agent → Contrôleur Contrôle 

Le contrôleur et les agents Jenkins peuvent être considérés comme un processus distribué qui s'exécute sur plusieurs processus et machines distincts. Cela permet à un agent de demander au processus contrôleur les informations dont il dispose, par exemple le contenu de fichiers, etc., et même de demander au contrôleur d'exécuter certaines commandes à la demande de l'agent.

Ainsi, même si le fait de ne pas construire sur le nœud intégré est une bonne pratique générale pour se protéger contre les bogues et les attaquants moins sophistiqués, un processus d'agent pris en charge par un utilisateur malveillant serait toujours en mesure d'obtenir des données ou d'exécuter des commandes sur le contrôleur Jenkins. Pour éviter cela, le système de contrôle d'accès Agent → Contrôleur empêche les processus d'agent d'envoyer des commandes malveillantes au contrôleur Jenkins.

Ce système est toujours activé depuis Jenkins 2.326 (voir [Modifications de sécurité Agent → Contrôleur dans 2.326](#jep-235)). Dans Jenkins 2.325 et les versions antérieures, il est activé par défaut, mais peut être désactivé dans l'interface utilisateur web en décochant la case sur la page _Gérer Jenkins » Sécurité_.

!!! danger "Attention"
	Il est fortement recommandé de ne pas désactiver le système de contrôle d'accès Agent → Contrôleur.

![Sécurité - Activer le contrôle d'accès Agent ⇒ Contrôleur](https://www.jenkins.io/doc/book/resources/security/configure-global-security-agent-controller-toggle.png)

Au lieu de désactiver le contrôle d'accès Agent → Contrôleur, dans Jenkins 2.325 et versions antérieures, les administrateurs peuvent autoriser de manière sélective un accès plus étendu. Consultez [la documentation](https://www.jenkins.io/doc/book/security/controller-isolation/agent-to-controller/) pour plus de détails.

