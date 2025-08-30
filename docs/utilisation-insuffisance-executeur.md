# Insuffisance de l'Exécuteur

Si vous voyez une petite icône d'horloge noire dans la file d'attente de compilation, comme illustré ci-dessous, cela signifie que votre tâche est inutilement bloquée dans la file d'attente.

![image](https://www.jenkins.io/images/using/starvation.png)

L'info-bulle du lien du nom de la tâche à côté de l'icône de l'horloge devrait vous indiquer exactement pourquoi la compilation échoue, mais les symptômes courants sont les suivants :

1. **_Agents are offline_** (Les agents sont hors ligne) : votre build doit s'exécuter sur un agent particulier, mais celui-ci est hors ligne. Rendez-vous sur http://server/jenkins/computer/AGENTNAME pour comprendre pourquoi et le remettre en ligne. Mieux encore, utilisez des étiquettes et ne liez pas les builds à des agents spécifiques, afin qu'un seul agent hors ligne n'empêche pas vos builds de s'exécuter.
2. **_Waiting for an available executor on an agent:_** (En attente d'un exécuteur disponible sur un agent) : votre build doit s'exécuter sur un agent particulier, mais celui-ci est déjà entièrement occupé à créer d'autres éléments, et votre build attend « trop longtemps » par rapport au temps nécessaire à son exécution. En d'autres termes, il n'est pas logique d'attendre 5 minutes alors que le build lui-même se termine en 2 minutes. Utilisez des étiquettes afin que les builds puissent s'exécuter sur n'importe quelle machine qui répond aux exigences du système. Vous pouvez ainsi ajouter davantage d'agents pour améliorer le délai d'exécution.
3. **_Waiting for an available executor on a label:_** (Attente d'un exécuteur disponible sur une étiquette) : tous les agents qui ont l'étiquette donnée sont entièrement occupés à d'autres tâches. Il est temps d'ajouter davantage d'agents.