# Réinitialiser le mot de passe administrateur Jenkins 

En tant qu'administrateur système, vous pouvez réinitialiser le mot de passe administrateur Jenkins. Si vous avez oublié ou perdu le mot de passe, cette page vous explique comment réinitialiser le mot de passe administrateur.

<iframe width="640" height="360" src="https://www.youtube.com/embed/_VhOMyWDIcY" title="How To Reset Jenkins Admin Password" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Réinitialiser le mot de passe administrateur

1. Connectez-vous à votre contrôleur Jenkins.
2. Arrêtez le processus Jenkins. Vous pouvez utiliser cette commande : ```systemctl stop jenkins```.
3. Modifiez le fichier de configuration Jenkins (```config.xml```) dans votre répertoire ```jenkins/``` ou ```$JENKINS_HOME```.
4. Recherchez **useSecurity** et remplacez manuellement la valeur **true** par **false**.
5. Enregistrez votre fichier et fermez-le.
6. Redémarrez le service Jenkins pour appliquer vos modifications. Vous pouvez utiliser cette commande : ```systemctl start jenkins```. Après avoir redémarré Jenkins, accédez à votre contrôleur et connectez-vous.
7. Dans le tableau de bord, sélectionnez **Manage Jenkins** (Gérer Jenkins) dans le volet de navigation situé à gauche de la page.
8. Dans la page **Manage Jenkins** (Gérer Jenkins), sous la section **Security** (Sécurité), sélectionnez **Configure Global Security** (Configurer la Sécurité Globale).
9. Sous **Security Realm** (Domaine de Sécurité), sélectionnez **Jenkins' own user database** (la base de données utilisateur propre à Jenkins) dans le menu déroulant. Assurez-vous que l'option **Allow users to sign up** (Permettre aux utilisateurs de s'inscrire) n'est pas cochée et enregistrez vos modifications. Vous serez redirigé vers la page Manage Jenkins.
10. Sur la page **Manage Jenkins** (Gérer Jenkins), sélectionnez **Users** (Utilisateurs).
11. Vous verrez une liste affichant les identifiants utilisateur. Sélectionnez l'identifiant utilisateur pour lequel vous souhaitez modifier le mot de passe.
12. Sélectionnez Configure à l'aide de l'icône en forme d'engrenage ou du menu déroulant de l'identifiant utilisateur. Localisez la section Password pour modifier votre mot de passe.

Après avoir modifié le mot de passe, vous pourrez vous reconnecter à votre contrôleur Jenkins en utilisant le même nom d'utilisateur et le nouveau mot de passe que vous venez de définir.

!!! note
    Veillez à ne pas laisser le contrôleur Jenkins sans protection. Réactivez donc la sécurité en suivant ces étapes.

## Activer la sécurité

1. Connectez-vous à votre compte administrateur.
2. Dans le tableau de bord, sélectionnez **Manage Jenkins** (Gérer Jenkins) dans le volet de navigation situé à gauche de la page.
3. Sur la page **Manage Jenkins** (Gérer Jenkins), dans la section **Security** (Sécurité), sélectionnez **Configure Global Security** (Configurer la Sécurité Globale).
4. Définissez **Authorization** (Autorisation) sur **Logged-in users can do anything** (Les utilisateurs connectés peuvent tout faire.). Décochez l'option **Allow anonymous read access** (Autoriser l'accès en lecture anonyme.). Sélectionnez **Save** (Sauvegarder).
