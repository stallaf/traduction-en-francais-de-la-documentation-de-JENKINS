# Abandonner une compilation

Lorsque vous cliquez sur cette icône [x] de l'interface utilisateur, les choses suivantes se produisent : 

1. Le navigateur envoie une demande au serveur ;
2. Le serveur interrompt (via Thread.interrupt ()) le thread (alias thread d'exécution) chargé d'effectuer une compilation ;
3. Le serveur fait un retour.

À ce stade, votre navigateur est de nouveau opérationnel, mais le processus d'interruption proprement dit se déroule de manière asynchrone à partir de là.

1. Le thread reçoit le signal d'interruption. La rapidité avec laquelle cela se produit dépend de ce que fait l'exécuteur au moment de l'interruption. Plus précisément, un thread d'exécution ne peut être interrompu qu'aux « points d'interruption » en raison de la conception Java.
    * L'attente de la fin d'un processus enfant (par exemple, la compilation exécutant Ant) est un point d'interruption. Cela signifie que si l'exécuteur était en train de faire cela, il est interrompu instantanément ;
    * L'attente d'un calcul sur un agent est un point d'interruption ;
    * L'attente d'un fichier ou d'une E/S réseau n'est pas un point d'interruption. Cela pose souvent le problème d'une compilation qui semble impossible à interrompre. Par exemple, la vérification d'un référentiel Subversion entre dans cette catégorie ;
    * Le calcul normal n'est pas non plus un point d'interruption ;
  2. L'exécuteur effectue une opération de nettoyage. Cela dépend de ce qu'il faisait au moment où il a remarqué l'interruption ;
    * S'il attendait l'achèvement d'un processus enfant, Jenkins recherchera tous les processus descendant et les tuera tous. Sur Unix, cela se fait via `java.lang.UnixProcess.destroyProcess`, qui envoie [SIGTERM](http://en.wikipedia.org/wiki/SIGTERM) sur les implémentations JDK basées sur UNIX. Sur Windows, cela se fait via API[] TermineProcess](http://msdn.microsoft.com/en-us/library/ms686714(VS.85).aspx) ;
    * S'il attendait l'achèvement d'un calcul dans un agent, le fil qui exécute le calcul à distance est interrompu de manière asynchrone. La rapidité avec laquelle ces fils sont interrompus dépend de ce que fait ce fil. Voir ci-dessus. 
2. L'exécuteur commence à dérouler la pile et finit par terminer le déroulement. À ce stade, la compilation est marquée comme abandonnée et l'exécuteur revient à l'état inactif.

Les tâches de Pipeline peuvent être arrêtées en envoyant une requête HTTP POST aux points de terminaison URL d'une compilation.

* `BUILD ID URL/stop` - interrompt un Pipeline ;
* `BUILD ID URL/term` - termine de force une compilation (à n'utiliser que si stop ne fonctionne pas) ;
* `BUILD ID URL/kill` - interrompt brutalement un Pipeline. Il s'agit de la méthode la plus destructive pour arrêter un Pipeline et elle ne doit être utilisée qu'en dernier recours.

## Si votre construction n'about pas

Vérifiez le vidage de thread `http://yourserver/jenkins/threadDump` et recherchez le thread d'exécution en question. Ceux-ci sont nommés d'après l'agent et le numéro d'exécution. Cela vous indiquera généralement où se trouve le thread et révélera souvent pourquoi il ne répond pas à une interruption.