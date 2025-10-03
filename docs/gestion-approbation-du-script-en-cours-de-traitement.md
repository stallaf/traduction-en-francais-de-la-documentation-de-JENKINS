# Approbation du Script en Cours de Traitement

<div class="couleur-introduction">
Jenkins et plusieurs plugins permettent aux utilisateurs d'exécuter des scripts Groovy dans Jenkins. Ces fonctionnalités de script sont fournies par :
</div>

* La [console de script](./gestion-console.md) ;
* [Jenkins Pipeline.](./pipeline-presentation.md) ;
* Le [plugin Extended Email](https://plugins.jenkins.io/email-ext) ;
* Le [plugin Groovy](https://plugins.jenkins.io/groovy), lors de l'utilisation de l'étape « Exécuter le script Groovy du système » ;
* Le [plugin JobDSL](https://plugins.jenkins.io/job-dsl) à partir de la version 1.60.

Afin de protéger Jenkins contre l'exécution de scripts malveillants, ces plugins exécutent les scripts fournis par l'utilisateur dans un [bac à sable Groovy](#sandbox-groovy) qui limite les API internes accessibles. Cette protection est assurée par le [plugin Script Security](https://plugins.jenkins.io/script-security). Dès qu'une méthode non sécurisée est utilisée dans l'un des scripts, l'administrateur peut utiliser l'action « In-process Script Approval » (Approbation de script en cours de traitement) qui apparaît dans **Manage Jenkins** (Gérer Jenkins) pour autoriser la méthode non sécurisée. Les méthodes non sécurisées ne doivent pas être activées sans une réflexion approfondie sur leur impact.

![Accéder à la configuration de l'approbation des scripts en cours d'exécution](https://www.jenkins.io/doc/book/resources/managing/manage-inprocess-script-approval.png)

## Pour commencer

Le [plugin Script Security](https://plugins.jenkins.io/script-security) est installé automatiquement par [l'assistant de configuration post-installation](./installation-presentation.md), bien qu'au départ, aucun script ou opération supplémentaire ne soit approuvé pour utilisation.

!!! warning "Avertissement"
    Les anciennes versions de ce plugin peuvent ne pas être sûres à utiliser. Veuillez consulter les avertissements de sécurité répertoriés sur la page du [plugin Script Security](https://plugins.jenkins.io/script-security) afin de vous assurer que le [plugin Script Security](https://plugins.jenkins.io/script-security) est à jour.

La sécurité des scripts en cours d'exécution est assurée par deux mécanismes différents : le [bac à sable Groovy](#sandbox-groovy) et [l'approbation des scripts](#approbation-du-script). Le premier, le bac à sable Groovy, est activé par défaut pour [Jenkins Pipeline](./pipeline-presentation.md), ce qui permet aux pipelines scriptés et déclaratifs fournis par l'utilisateur de s'exécuter sans intervention préalable de l'administrateur. Le second, l'approbation des scripts, permet aux administrateurs d'approuver ou de refuser les scripts non sandboxés, ou d'autoriser les scripts sandboxés à exécuter des méthodes supplémentaires.

Pour la plupart des systèmes, la combinaison de Groovy Sandbox et de la liste intégrée de [signatures de méthodes approuvées de Script Security](https://github.com/jenkinsci/script-security-plugin/tree/master/src/main/resources/org/jenkinsci/plugins/scriptsecurity/sandbox/whitelists) sera suffisante. Il est fortement recommandé aux administrateurs de ne s'écarter de ces paramètres par défaut qu'en cas d'absolue nécessité.

## Sandbox Groovy

Afin de réduire les interventions manuelles des administrateurs, la plupart des scripts s'exécutent par défaut dans un Sandbox Groovy, y compris tous les [Pipelines Jenkins](./pipeline-presentation.md). Le sandbox n'autorise qu'un sous-ensemble de méthodes Groovy jugées suffisamment sûres pour être exécutées sans autorisation préalable dans le cadre d'un accès « non fiable ». Les scripts utilisant le Sandbox Groovy sont **tous** soumis aux mêmes restrictions. Par conséquent, un Pipeline créé par un administrateur est soumis aux mêmes restrictions qu'un Pipeline autorisé par un utilisateur non administrateur.

Lorsqu'un script tente d'utiliser des fonctionnalités ou des méthodes non autorisées par le bac à sable, il est immédiatement interrompu, comme illustré ci-dessous avec  Pipeline Jenkins.

![Rejet de la méthode Sandbox](https://www.jenkins.io/doc/book/resources/managing/script-sandbox-rejection.png)

_Figure 1. Signature de méthode non autorisée rejetée lors de l'exécution via Blue Ocean_

Le Pipeline ci-dessus ne s'exécutera pas tant qu'un administrateur n'aura pas [approuvé la signature de la méthode](#approbation-la-signature-de-la-methode) via la page « In-process Script Approval » (Approbation du script en cours de traitement).

En plus d'ajouter des signatures de méthode approuvées, les utilisateurs peuvent également désactiver complètement le bac à sable Groovy, comme illustré ci-dessous. La désactivation du bac à sable Groovy nécessite que l'ensemble du script soit examiné et [approuvé manuellement](#approbation-dun-pipeline-scripté-non-sandboxé) par un administrateur.

![Création d'un Pipeline scripté et désactivation de l'option « Utiliser le bac à sable Groovy »](https://www.jenkins.io/doc/book/resources/managing/unchecked-groovy-sandbox-on-pipeline.png)

_Figure 2. Désactivation du bac à sable Groovy pour un Pipeline_

## Approbation des scripts

L'approbation manuelle de scripts entiers ou de signatures de méthodes par un administrateur offre aux administrateurs une flexibilité supplémentaire pour prendre en charge des utilisations plus avancées des scripts en cours de traitement. Lorsque [Sandbox Groovy](#sandbox-groovy) est désactivé ou qu'une méthode ne figurant pas dans la liste intégrée est invoquée, le plugin Script Security vérifie la liste des scripts et méthodes approuvés gérée par l'administrateur.
	
!!! info "Avertissement"
    Lorsqu'un script est approuvé, il l'est pour être utilisé dans n'importe quelle fonctionnalité ou plugin Jenkins qui s'intègre à l'approbation des scripts. L'approbation des scripts n'est pas liée à une tâche spécifique ou à une autre utilisation spécifique du script. Pour cette raison, il faut faire preuve de prudence lors de l'approbation d'un script afin de s'assurer qu'aucun paramètre fourni par l'utilisateur ne puisse être utilisé pour exploiter le contrôleur.

Pour les scripts qui doivent être exécutés en dehors du [bac à sable Groovy,](#sandbox-groovy) l'administrateur doit approuver **l'intégralité** du script dans la page **In-process Script Approval** (Approbation des scripts en cours)  :

![Approbation d'un Pipeline scripté hors bac à sable](https://www.jenkins.io/doc/book/resources/managing/inprocess-script-approval-pipeline.png)

_Figure 3. Approbation d'un Pipeline scripté hors bac à sable_

Pour les scripts qui utilisent le [bac à sable Groovy](#sandbox-groovy), mais qui souhaitent exécuter une signature de méthode actuellement non approuvée, Jenkins les interrompra également et demandera à l'administrateur d'approuver la signature de méthode spécifique avant que le script ne soit autorisé à s'exécuter :

![Approbation d'une nouvelle signature de méthode](https://www.jenkins.io/doc/book/resources/managing/inprocess-script-approval-method.png)

_Figure 4. Approbation d'une nouvelle signature de méthode_

## Approuver en supposant la vérification des autorisations

L'approbation du script offre trois options : Approuver, Refuser et « Approuver en supposant la vérification des autorisations ». Si l'objectif des deux premières options est évident, la troisième nécessite une compréhension supplémentaire des données internes auxquelles les scripts peuvent accéder et du fonctionnement des vérifications d'autorisations dans Jenkins.

Prenons l'exemple d'un script qui accède à la méthode `hudson.model.AbstractItem.getParent()`, qui en soi est inoffensive et renvoie un objet contenant soit le dossier, soit l'élément racine qui contient le pipeline ou le travail en cours d'exécution. Après l'appel de cette méthode, l'exécution de `hudson.model.ItemGroup.getItems()`, qui répertorie les éléments du dossier ou de l'élément racine, nécessite l'autorisation `Job/Read.`

Cela pourrait signifier que l'approbation de la signature de la méthode `hudson.model.ItemGroup.getItems()` permettrait à un script de contourner les vérifications d'autorisations intégrées.

Au lieu de cela, il est généralement préférable de cliquer sur **Approuver en supposant que la vérification des autorisations** amènera le moteur d'approbation des scripts à autoriser la signature de la méthode, en supposant que l'utilisateur qui exécute le script dispose des autorisations nécessaires pour exécuter la méthode, telles que l'autorisation `Job/Read` dans cet exemple.

