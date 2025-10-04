# Sécuriser Jenkins

!!! warning "Avertissement"
    Cette section est en cours d'élaboration. Vous souhaitez contribuer ? Consultez le [canal Gitter jenkinsci/docs](https://app.gitter.im/#/room/#jenkins/docs:matrix.org). Pour découvrir d'autres façons de contribuer au projet Jenkins, consultez [cette page sur la participation et la contribution](https://www.jenkins.io/participate).
    
La sécurisation de Jenkins comporte deux aspects :

* **Le contrôle d'accès,** qui garantit que les utilisateurs sont authentifiés lorsqu'ils accèdent à Jenkins et que leurs activités sont autorisées ;
* Protéger Jenkins contre les menaces externes

## Contrôle d'accès

Vous devez verrouiller l'accès à l'interface utilisateur de Jenkins afin que les utilisateurs soient authentifiés et se voient attribuer les autorisations appropriées. Ce paramètre est principalement contrôlé par deux axes :

*** Le domaine de sécurité**, qui détermine les utilisateurs et leurs mots de passe, ainsi que les groupes auxquels ils appartiennent ;
* **La stratégie d'autorisation**, qui détermine qui a accès à quoi.

Ces deux axes sont orthogonaux et doivent être configurés individuellement. Par exemple, vous pouvez choisir d'utiliser LDAP externe ou Active Directory comme domaine de sécurité, et vous pouvez choisir le mode « accès complet pour tous une fois connecté » comme stratégie d'autorisation. Vous pouvez également choisir de laisser Jenkins exécuter sa propre base de données utilisateur et effectuer le contrôle d'accès en fonction de la matrice des autorisations/utilisateurs.

* [Sécurité rapide et simple](https://wiki.jenkins.io/display/JENKINS/Quick+and+Simple+Security) --- si vous exécutez Jenkins comme `java -jar jenkins.war `et que vous n'avez besoin que d'une configuration très simple ;
* [Configuration de sécurité standard](https://wiki.jenkins.io/display/JENKINS/Standard+Security+Setup) --- présente la configuration la plus courante qui consiste à laisser Jenkins gérer sa propre base de données d'utilisateurs et à effectuer un contrôle d'accès plus fin ;
* [Frontend Apache pour la sécurité](https://wiki.jenkins.io/display/JENKINS/Apache+frontend+for+security) --- exécutez Jenkins derrière Apache et effectuez le contrôle d'accès dans Apache plutôt que dans Jenkin ;
* [Authentification des clients scriptés](https://wiki.jenkins.io/display/JENKINS/Authenticating+scripted+clients) --- si vous devez accéder par programmation à l'interface utilisateur web Jenkins sécurisée, utilisez l'authentification BASIC ;
* [Sécurité basée sur une matrice|Sécurité basée sur une matrice](https://wiki.jenkins.io/display/JENKINS/Matrix-based+security) --- Accorder et refuser des autorisations plus fines.

En plus du contrôle d'accès des utilisateurs, le [contrôle d'accès aux builds](./securite-autorisation-des-compilations.md) limite ce que les builds peuvent faire une fois lancés.
