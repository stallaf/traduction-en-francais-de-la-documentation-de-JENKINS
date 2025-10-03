# Contenu Utilisateur

Jenkins dispose d'un mécanisme appelé « Contenu utilisateur », qui permet aux administrateurs de placer des fichiers dans `$JENKINS_HOME/userContent`. Ces fichiers sont ensuite accessibles à partir de http://yourhost/jenkins/userContent. Ce mécanisme peut être considéré comme un mini-serveur HTTP permettant de fournir des images, des feuilles de style et d'autres ressources statiques que vous pouvez utiliser à partir de divers champs de description dans Jenkins.

* Notez que ces fichiers ne sont soumis à aucun contrôle d'accès autre que l'accès global/lecture ;
* Consultez le [plugin Git userContent](https://plugins.jenkins.io/git-userContent/) pour savoir comment gérer ces fichiers via un dépôt Git.