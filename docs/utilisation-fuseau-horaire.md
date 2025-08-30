# Changer le fuseau horaire

Si votre contrôleur Jenkins s'exécute dans un endroit différent du vôtre (par exemple : le serveur est à New York mais vous êtes à  Paris), le fuseau horaire de New York sera très probablement utilisé. Cela peut être assez ennuyeux si vous avez besoin de comparer les dates de construction.

Pour voir le fuseau horaire actuellement défini, accédez à `jenkins_server/systemInfo` et consultez la propriété système `user.timezone`.

![Fuseau horaire du serveur Jenkins](https://www.jenkins.io/doc/book/resources/using/jenkins-server-timezone.png)

Vous voudrez peut-être modifier le fuseau horaire affiché pour correspondre à votre propre fuseau horaire. En allant sur votre page de configuration utilisateur, vous pouvez définir le `User Defined Time Zone` (fuseau horaire défini par l'utilisateur) pour correspondre au vôtre.

![Changer le fuseau horaire](https://www.jenkins.io/doc/book/resources/using/change-time-zone.png)