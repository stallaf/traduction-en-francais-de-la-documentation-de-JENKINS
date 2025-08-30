# Palette de Commandes

Jenkins propose désormais une nouvelle expérience de recherche connue sous le nom de palette de commandes. Situé dans le coin supérieur droit de la page à côté de votre nom, la palette de commande vous permet de tout chercher dans Jenkins.

![Icône de recherche mise à jour de la palette de commandes.](https://www.jenkins.io/doc/book/resources/using/command-palette/search-icon.png)

Par exemple, entrez « git #23 console » pour afficher les résultats de la page de sortie de la console de « git job build #23 ». La fonctionnalité Command Palette utilise la saisie semi-automatique pour vous aider à trouver ce que vous cherchez. Les résultats peuvent inclure des termes de recherche supplémentaires pour affiner les résultats, des pipelines, des vues, des fonctionnalités supplémentaires installées dans votre environnement Jenkins, telles que la bibliothèque de conception, et des pages spécifiques dans les tâches en fonction des plugins installés, tels que le plugin [Warnings](https://plugins.jenkins.io/warnings-ng). Les résultats de recherche offrent également plus de clarté sous la forme d'icônes qui indiquent les fonctionnalités Jenkins et l'état des tâches.

La palette de commandes est également accessible via des commandes de clavier pour macOS (CMD + K) et Windows/Linux (Ctrl + K).

![Résultats de l'auto-complétion de la saisie dans la palette de commande.](https://www.jenkins.io/doc/book/resources/using/command-palette/auto-complete-results.png)

### Recherche insensible à la casse

Accédez à la page des préférences de votre profil (/jenkins/user/<votre_profil>/preferences) et activez l'option de recherche insensible à la casse, si vous le souhaitez.

![Cas à cocher d'insensibilité au cas dans les préférences des utilisateurs.](https://www.jenkins.io/doc/book/resources/using/command-palette/case-sensitivity.png)

Veuillez noter que la recherche insensible à la cas n'est pas disponible pour les utilisateurs anonymes (non connectés).

### Support Opeensearch

Cette fonction de recherche est également exposée au navigateur via [OpenSesearch](http://en.wikipedia.org/wiki/OpenSearch), vous pouvez donc installer cette fonction de recherche et d'auto-complétion dans votre zone de recherche de navigateur, ce qui facilite la navigation de Jenkins.

Par exemple dans Firefox, vous pouvez ajouter la recherche Jenkins aux moteurs de recherche de votre navigateur via la liste déroulante à droite de la barre d'adresse:

![image](https://www.jenkins.io/doc/book/resources/using/command-palette/add-to-firefox.png)

### Vos commentaires sont les bienvenus.

Il y a toujours matière à amélioration dans la manière dont Jenkins associe les termes de recherche aux pages réelles. Vos commentaires sont les bienvenus.
