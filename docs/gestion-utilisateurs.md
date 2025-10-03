# Utilisateurs

<div class="couleur-introduction">
La gestion des utilisateurs dans Jenkins implique la création, la configuration et la maintenance des comptes utilisateurs pour les personnes qui interagissent avec l'instance Jenkins. Cette page fournit un guide étape par étape sur la manière de gérer efficacement les utilisateurs.
</div>

_Jenkins prend en charge les systèmes d'authentification externes tels que GitHub, GitLab, Google et LDAP. Si une méthode d'authentification externe est configurée, la gestion des utilisateurs peut être effectuée en dehors de Jenkins. Ce guide s'applique uniquement lorsque vous utilisez la **base de données utilisateur interne** de Jenkins._

## Que signifie gérer les utilisateurs ?

Dans Jenkins, la gestion des utilisateurs désigne le processus de création, de configuration et de maintenance des comptes utilisateurs. Cela comprend :

* La création de nouveaux utilisateurs ;
* La configuration des informations utilisateur telles que le nom complet, l'adresse e-mail et le mot de passe ;
* La modification des informations utilisateur ;
* La suppression d'utilisateurs.

Les administrateurs disposant des autorisations appropriées peuvent effectuer ces actions.

## Accéder à la page des utilisateurs

Pour gérer les utilisateurs dans Jenkins, vous devez disposer de privilèges administratifs. Suivez ces étapes pour accéder à la section de gestion des utilisateurs :

1. Connectez-vous à votre instance Jenkins avec un compte disposant des autorisations administratives.
2. Accédez à la page **Manage Jenkins** (Gérer Jenkins) depuis le tableau de bord Jenkins.
![Accéder à la page Manage Jenkins (Gérer Jenkins)](https://www.jenkins.io/doc/book/resources/managing/manager-users-home-page.png)
3. Sélectionnez **Users** (Utilisateurs) dans la section **Security** (Sécurité).
![Accéder à la section Users (Utilisateurs)](https://www.jenkins.io/doc/book/resources/managing/select-users.png)

## Créer un utilisateur

Pour créer un nouvel utilisateur dans Jenkins :

1. Sur la page **Users** (Utilisateurs), sélectionnez **Create User **(Créer un utilisateur).
![Accéder au bouton Create User (Créer un utilisateur)](https://www.jenkins.io/doc/book/resources/managing/create-users-click.png)
2. Remplissez les informations requises
    * **Username** (Nom d'utilisateur) : identifiant unique de l'utilisateur ;
    * **Password** (Mot de passe) : mot de passe sécurisé de l'utilisateur ;
    * **Confirm Password** (Confirmer le mot de passe) : ressaisissez le mot de passe ;
    * **Full Name** (Nom complet) : nom complet de l'utilisateur ;
    * **Email Address** (Adresse e-mail) : adresse e-mail de l'utilisateur ;
![Accéder à la page Créer un utilisateur](https://www.jenkins.io/doc/book/resources/managing/create-users.png)
3. Sélectionnez **Create User** (Créer un utilisateur) pour enregistrer le nouvel utilisateur.
Par exemple :
![Exemple de page Créer un utilisateur](https://www.jenkins.io/doc/book/resources/managing/create-users-example.png)

## Configuration des paramètres utilisateur

Une fois qu'un utilisateur est créé, vous pouvez configurer ses paramètres :

1. Sur la page **Manage Users** (Gérer les utilisateurs), sélectionnez l'utilisateur que vous souhaitez configurer.
![Page Configuration de l'utilisateur](https://www.jenkins.io/doc/book/resources/managing/select-user.png)
2. Mettez à jour les informations suivantes selon vos besoins :
    * **Full Name:** (Nom complet) : modifiez le nom complet de l'utilisateur ;
    * **Description** (Description) : mettez à jour la description de l'utilisateur ;
    * **Credentials** (Identifiants) : modifiez les identifiants de l'utilisateur.
![Configurer la page](https://www.jenkins.io/doc/book/resources/managing/account-preference.png)
3. Sélectionnez **Save** (Enregistrer) pour appliquer les modifications. == Modification des informations utilisateur

Les administrateurs peuvent modifier les informations utilisateur à tout moment :

1. Accédez à la page **Manage Users** (Gérer les utilisateurs).
2. Sélectionnez l'utilisateur dont vous souhaitez modifier les informations.
3. Mettez à jour les champs pertinents tels que le nom complet, l'adresse e-mail ou le mot de passe.
4. Sélectionnez Enregistrer pour appliquer les modifications.

## Suppression d'un utilisateur

Il existe deux méthodes pour supprimer un utilisateur de Jenkins. Voyons les deux :

### Première méthode

1. Accédez à **Users** (Utilisateurs).
2. Recherchez l'utilisateur que vous souhaitez supprimer.
3. Sélectionnez l'option **Delete** (Supprimer) à côté du nom de l'utilisateur.
![Suppression d'un utilisateur](https://www.jenkins.io/doc/book/resources/managing/delete-users.png)
4. Confirmez la suppression.

### Deuxième méthode

1. Accédez à **Manage Jenkins** (Gérer Jenkins).
2. Accédez ensuite à **Users** [Utilisateurs].
3. Sélectionnez l'icône de corbeille située à l'extrême droite de l'utilisateur que vous souhaitez supprimer.
![Suppression d'un utilisateur Alternative](https://www.jenkins.io/doc/book/resources/managing/delete-user-alternative.png)
4. Confirmez la suppression.

!!! warning " "
    La suppression d'un utilisateur entraîne la suppression définitive de son compte Jenkins. Assurez-vous qu'il n'a pas de tâches ou de responsabilités actives avant de poursuivre.

## Lien vers les paramètres de sécurité

La gestion des utilisateurs est étroitement liée aux paramètres de sécurité de Jenkins. Pour plus d'informations sur la configuration des autorisations et des permissions, consultez la documentation [Contrôle d'accès - Autorisations](./securite-controle-acces.md#permissions).

## Dépannage

Voici quelques problèmes courants que vous pouvez rencontrer lors de la gestion des utilisateurs :

* **User creation fails :** (La création d'un utilisateur échoue) : assurez-vous que tous les champs obligatoires sont remplis et que le nom d'utilisateur est unique ;
* **User permissions are incorrect :** (Les autorisations utilisateur sont incorrectes) : vérifiez les paramètres de sécurité sous Manage Jenkins > Configure Global Security ;
* **Cannot delete a user :** (Impossible de supprimer un utilisateur) : si un utilisateur est associé à des tâches actives, envisagez de désactiver son compte plutôt que de le supprimer.

## Conclusion

La gestion des utilisateurs dans Jenkins est un processus simple qui consiste à créer des utilisateurs, à configurer leurs paramètres et à s'assurer qu'ils disposent des autorisations appropriées. En suivant ce guide, vous pouvez gérer efficacement les utilisateurs dans votre instance Jenkins.


