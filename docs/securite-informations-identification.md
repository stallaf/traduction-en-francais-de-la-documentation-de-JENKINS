# Informations d'Identification

<div class="couleur-introduction">
Utilisez des informations d'identification pour sécuriser l'accès aux sites et applications externes pouvant interagir avec Jenkins, tels que les référentiels d'artefacts, les systèmes et services de stockage dans le cloud et les bases de données. Cette méthode est à la fois plus sûre et plus pratique que le codage en dur des noms d'utilisateur et mots de passe ou d'autres dispositifs d'authentification dans chaque Pipeline.

Cette page résume les meilleures pratiques pour sécuriser vos informations d'identification.
</div>

## Utilisation des identifiants

La mise en œuvre des identifiants comporte deux éléments :

* Configurez l'identifiant. Voir [Configuration des identifiants](./utilisation-identifiants.md#configuration-des-informations-didentification) sur la page [Utilisation des identifiants](./utilisation-identifiants.md) ;
* Appelez les identifiants dans le Pipeline pour accéder à une ressource externe. Voir [Gestion des identifiants](./pipeline-jenkinsfile.md#gestion-des-informations-didentification) sur la page [Utilisation d'un Jenkinsfile](./pipeline-jenkinsfile.md).

## Limiter l'accès aux identifiants

Limitez le nombre de personnes ayant accès aux identifiants. Cela minimise la surface d'attaque. Vous devez « construire à partir de la base ». En d'autres termes, commencez par ne donner accès à personne aux identifiants, puis accordez l'accès à des personnes et à des projets spécifiques plutôt que d'accorder l'accès à tout le monde et de supprimer ensuite certaines autorisations. Plus précisément :

* Limitez strictement le nombre de personnes disposant **d'informations d'identification > Créer des autorisations** ;
* Accordez l'accès uniquement aux personnes, projets et éléments spécifiques qui en ont réellement besoin pour effectuer leur travail ;
* Définissez chaque information d'identification au niveau le plus bas possible. Les informations d'identification définies pour le contrôleur sont disponibles pour tous les Pipelines exécutés par ce contrôleur. Les informations d'identification définies pour un dossier ne sont disponibles que pour les Pipelines exécutés à partir de ce dossier. Les écrans utilisés pour ajouter et gérer les informations d'identification pour les contrôleurs et les dossiers sont identiques, à l'exception de leur emplacement.

## Protéger les secrets

Les clés permettant de déchiffrer les secrets sont stockées dans le répertoire _$JENKINS_HOME/secrets/_. Ce répertoire dispose d'autorisations restrictives du système de fichiers Linux et doit être soigneusement protégé :

* Vous devez être en mesure de récupérer vos secrets si vous devez restaurer votre instance ;
* Si quelqu'un venait à accéder à vos sauvegardes ainsi qu'au secret, il aurait un accès complet à l'ensemble de votre instance Jenkins et à tout ce à quoi elle accède.

Les pratiques suivantes sont recommandées pour garantir la confidentialité de vos secrets :

* N'incluez jamais le répertoire _secrets_ dans une sauvegarde de votre instance ;
* Ne stockez pas les clés dans le SCM où leur contenu pourrait être compromis.
    * Une panne de disque ou une corruption des données de votre application pourrait entraîner la perte de toutes les informations cryptées de votre instance ;
    * Si quelqu'un compromet votre SCM et que vos secrets s'y trouvent, il aura un accès complet à votre instance Jenkins ;
* Conservez plutôt une copie des secrets dans un emplacement distinct des autres sauvegardes et copies de votre application. Le secret est petit et ne change jamais après sa création. Vous pouvez le rajouter manuellement lors de la restauration à partir de la sauvegarde.

## Autres remarques concernant les informations d'identification

Modifiez fréquemment les informations d'identification sous-jacentes (nom d'utilisateur/mot de passe, texte secret, etc.).