# CLI Jenkins

<div class="couleur-introduction">
Jenkins dispose d'une interface de ligne de commande intégrée qui permet aux utilisateurs et aux administrateurs d'accéder à Jenkins à partir d'un script ou d'un environnement shell. Cela peut être pratique pour la création de scripts de tâches routinières, les mises à jour en masse, le dépannage, etc.
</div>

L'interface de ligne de commande est accessible via SSH ou avec le client CLI Jenkins, un fichier `.jar` distribué avec Jenkins.

!!! info " "
    Ce document suppose que vous utilisez Jenkins 2.54 ou une version plus récente. Les anciennes versions du client CLI sont considérées comme non sécurisées et ne doivent pas être utilisées.

    La prise en charge de WebSocket est disponible lorsque vous utilisez à la fois le serveur et le client 2.217 ou une version plus récente.

## Utilisation de la CLI via SSH

Dans une nouvelle installation Jenkins, le service SSH est désactivé par défaut. Les administrateurs peuvent choisir de définir un port spécifique ou demander à Jenkins de choisir un port aléatoire dans la page [Sécurité](./gestion-securite.md#serveur-ssh). Pour déterminer le port SSH attribué aléatoirement, inspectez les en-têtes renvoyés sur une URL Jenkins, par exemple :

``` console
% curl -Lv https://JENKINS_URL/login 2>&1 | grep -i “x-ssh-endpoint”
< X-SSH-Endpoint: localhost:53801
%
```

Avec le port SSH aléatoire (`53801` dans cet exemple) et [l'authentification ](#authentification)configurée, tout client SSH moderne peut exécuter en toute sécurité des commandes CLI.

### Authentification

Quel que soit l'utilisateur utilisé pour l'authentification avec le contrôleur Jenkins, il doit disposer des autorisations `Overall/Read` (Global/Lecture) pour pouvoir accéder à la CLI. L'utilisateur peut avoir besoin d'autorisations supplémentaires en fonction des commandes exécutées.

L'authentification en mode SSH repose sur l'authentification par clé publique/privée basée sur SSH. Pour ajouter une clé publique SSH pour l'utilisateur approprié, accédez à **Users** (Utilisateurs) dans la section **Security** (Sécurité) de **Manage Jenkins** (Gérer Jenkins), sélectionnez l'utilisateur que vous souhaitez mettre à jour, puis sélectionnez **Security**  <span class="fa fa-cadenas"></span> (Sécurité).

![Option Sécurité dans les paramètres utilisateur](https://www.jenkins.io/doc/book/resources/managing/user-security-settings.png)

Vous pouvez également vous rendre à l'adresse `<votre_url_jenkins>/user/<nom_d'utilisateur_jenkins>/security`. Une fois sur la page des paramètres de sécurité, vous pouvez coller une clé publique SSH dans la zone de texte appropriée.

![Ajout de clés publiques SSH pour un utilisateur](https://www.jenkins.io/doc/book/resources/managing/cli-adding-ssh-public-keys.png)

### Commandes courantes

Jenkins dispose d'un certain nombre de commandes CLI intégrées qui se trouvent dans chaque environnement Jenkins, telles que `build` ou `list-jobs`. Les plugins peuvent également fournir des commandes CLI. Pour obtenir la liste complète des commandes disponibles dans un environnement Jenkins donné, exécutez la commande CLI `help` :

``` console
% ssh -l kohsuke -p 53801 localhost help
```

La liste de commandes suivante n'est pas exhaustive, mais elle constitue un point de départ utile pour l'utilisation de la CLI Jenkins.

#### build

L'une des commandes CLI les plus courantes et les plus utiles est `build`, qui permet à l'utilisateur de déclencher toute tâche ou pipeline pour lesquels il dispose d'une autorisation.

L'invocation la plus basique déclenche simplement la tâche ou le pipeline et se termine, mais grâce à des options supplémentaires, l'utilisateur peut également passer des paramètres, interroger le SCM ou même suivre la sortie console de la compilation ou de l'exécution du pipeline déclenché.

``` console
% ssh -l kohsuke -p 53801 localhost help build

java -jar jenkins-cli.jar build JOB [-c] [-f] [-p] [-r N] [-s] [-v] [-w]
Lance une compilation et, éventuellement, attend qu'elle soit terminée.  Outre son utilisation générale dans les scripts,
cette commande peut être utilisée pour invoquer une autre tâche à partir d'une
compilation d'une tâche.  Avec l'option -s, cette commande modifie le code de sortie 
en fonction du résultat de la compilation (le code de sortie 0 indique une réussite) et l'interruption de la commande interrompra la tâche.  Avec l'option -f, cette commande modifie le code de sortie en fonction du résultat de la compilation (le code de sortie 0 indique une réussite) mais, contrairement à -s, l'interruption de la commande n'interrompra pas la tâche (le code de sortie 125 indique que la commande a été interrompue).  Avec l'option -c, une compilation ne s'exécutera que s'il y a eu un changement SCM.
 JOB : Nom du travail à construire.
 -c  : Vérifie les modifications SCM avant de démarrer la construction, et s'il n'y a pas de
       modification, quitte sans effectuer de construction.
 -f  : Suit la progression de la construction. Comme -s, mais les interruptions ne sont pas transmises
       à la construction.
 -p  : Spécifie les paramètres de construction au format clé=valeur.
 -s  : Attendre la fin/l'abandon de la commande. Les interruptions sont transmises
       à la construction.
 -v  : Affiche la sortie console de la construction. À utiliser avec -s
 -w  : Attendre le démarrage de la commande
% ssh -l kohsuke -p 53801 localhost build build-all-software -f -v
Lancement de build-all-software #1
Lancé à partir de la ligne de commande par admin
Construction dans l'espace de travail /tmp/jenkins/workspace/build-all-software
[build-all-software] /bin/sh -xe /tmp/hudson1100603797526301795.sh
+ echo hello world
hello world
Terminé : SUCCÈS
Construction build-all-software #1 terminée : SUCCÈS
%
```

#### console

La commande `console` est tout aussi utile. Elle permet de récupérer la sortie console pour la build ou l'exécution de pipeline spécifiée. Si aucun numéro de build n'est fourni, la commande `console` affiche la sortie console de la dernière build terminée.

``` console
% ssh -l kohsuke -p 53801 localhost help console

java -jar jenkins-cli.jar console JOB [BUILD] [-f] [-n N]
Produit la sortie console d'une build spécifique vers stdout, comme si vous exécutiez « cat build.log ».
 JOB   : nom du job.
 BUILD : numéro de build ou permalien pointant vers la build. Par défaut, la dernière
         build
 -f    : Si la build est en cours, restez à proximité et ajoutez la sortie de la console au fur et à mesure qu'elle
         apparaît, comme « tail -f ».
 -n N  : Affichez les N dernières lignes.
% ssh -l kohsuke -p 53801 localhost console build-all-software
Démarré à partir de la ligne de commande par kohsuke
Construction dans l'espace de travail /tmp/jenkins/workspace/build-all-software
[build-all-software] /bin/sh -xe /tmp/hudson1100603797526301795.sh
+ echo hello world
yes
Terminé : SUCCÈS
```

#### who-am-i (qui suis-je ?)

La commande `who-am-i` permet d'afficher les informations d'identification et les autorisations dont dispose l'utilisateur actuel. Cela peut être utile pour déboguer l'absence de commandes CLI due à un manque d'autorisations.

``` console
% ssh -l kohsuke -p 53801 localhost help who-am-i

java -jar jenkins-cli.jar who-am-i
Affiche vos identifiants et autorisations.
% ssh -l kohsuke -p 53801 localhost who-am-i
Authentifié en tant que : kohsuke
Autorités :
  authentifié
%
```

## Utilisation du client CLI

Bien que le CLI basé sur SSH soit rapide et couvre la plupart des besoins, il peut arriver que le client CLI distribué avec Jenkins soit plus adapté. Par exemple, le transport par défaut pour le client CLI est HTTP, ce qui signifie qu'aucun port supplémentaire ne doit être ouvert dans un pare-feu pour son utilisation.

### Comparaison entre SSH et le client CLI

SSH et jenkins-cli.jar donnent tous deux accès à un ensemble de commandes qui vous permettent d'interagir avec Jenkins à partir d'une ligne de commande, mais ils présentent quelques différences :

* Jenkins SSH ne nécessite aucun fichier jar personnalisé côté client, ce qui facilite l'accès à Jenkins à partir de diverses sources ;
* Le client SSH a été conçu comme un outil générique pouvant servir à plusieurs fins. Il n'offre pas de moyen simple d'effectuer des tâches de base courantes et spécifiques aux environnements Jenkins. L'utilisation de `jenkins-cli.jar` à la place du client ssh peut augmenter la productivité et améliorer l'expérience de développement.

### Téléchargement du client

Le client CLI peut être téléchargé directement à partir d'un contrôleur Jenkins à l'URL /`jnlpJars/jenkins-cli.jar`, soit [JENKINS_URL/jnlpJars/jenkins-cli.jar](https://jenkins_url/jnlpJars/jenkins-cli.jar).

Bien qu'un fichier `.jar` CLI puisse être utilisé avec différentes versions de Jenkins, si des problèmes de compatibilité surviennent pendant l'utilisation, veuillez télécharger à nouveau le dernier fichier `.jar` à partir du contrôleur Jenkins.

### Utilisation du client

La syntaxe générale pour invoquer le client est la suivante :

``` console
java -jar jenkins-cli.jar [-s JENKINS_URL] [options globales...] commande [options de commande...] [arguments...]
```

La `JENKINS_URL` peut être spécifiée via la variable d'environnement `$JENKINS_URL`. Vous pouvez afficher un résumé des autres options générales en exécutant le client sans aucun argument.

### Modes de connexion du client

Il existe trois modes de base dans lesquels le client peut être utilisé, sélectionnables par option globale : `-webSocket`, `-http` et `-ssh`.

#### Mode de connexion WebSocket

Il s'agit du mode par défaut, mais vous pouvez passer explicitement l'option `-webSocket` pour plus de clarté. L'avantage est qu'un transport plus standard est utilisé, ce qui évite les problèmes avec de nombreux proxys inversés ou la nécessité d'une configuration proxy spéciale.

#### Mode de connexion HTTP

À partir de Jenkins 2.391, le mode par défaut est `-webSocket`. Pour utiliser le mode HTTP, vous devez passer explicitement l'option `-http`.

L'authentification se fait de préférence avec une option `-auth`, qui prend un argument `username:apitoken`. Obtenez votre jeton API à partir de `/me/configure` :

``` console
java -jar jenkins-cli.jar [-s JENKINS_URL] -auth kohsuke:abc1234ffe4a commande ...
```

(Les mots de passe réels sont également acceptés, mais cela est déconseillé.)

Vous pouvez également faire précéder l'argument de `@` pour charger le même contenu à partir d'un fichier :

``` console
java -jar jenkins-cli.jar [-s JENKINS_URL] -auth @/home/kohsuke/.jenkins-cli command ...
``` 

!!! warning " "
    Pour des raisons de sécurité, l'utilisation d'un fichier pour charger les informations d'authentification est la méthode d'authentification recommandée.

Une autre méthode d'authentification consiste à configurer des variables d'environnement de la même manière que `$JENKINS_URL` est utilisé. Le `username` peut être spécifié via la variable d'environnement `$JENKINS_USER_ID`, tandis que `apitoken` peut être spécifié via la variable `$JENKINS_API_TOKEN`. Les deux variables doivent être définies en même temps.

``` console
export JENKINS_USER_ID=kohsuke
export JENKINS_API_TOKEN=abc1234ffe4a
java -jar jenkins-cli.jar [-s JENKINS_URL] commande ...
```

Si ces variables d'environnement sont configurées, vous pouvez toujours remplacer la méthode d'authentification en utilisant des informations d'identification différentes avec l'option` -auth`, qui a la priorité sur celles-ci.

En général, aucune configuration système particulière n'est nécessaire pour activer les connexions CLI basées sur HTTP. Si vous exécutez Jenkins derrière un proxy inverse HTTP(S), assurez-vous qu'il ne met pas en mémoire tampon les corps de requête ou de réponse.

!!! warning " "
    Ce mode est connu pour ne pas fonctionner de manière fiable, voire pas du tout, lors de l'utilisation de certains proxys inversés. Préférez le mode WebSocket.

#### Mode de connexion SSH

L'authentification s'effectue via une paire de clés SSH. Vous devez également sélectionner l'ID utilisateur Jenkins :

``` console
java -jar jenkins-cli.jar [-s JENKINS_URL] -ssh -user kohsuke command ...
```

Dans ce mode, le client agit essentiellement comme une commande `ssh` native.

Par défaut, le client tentera de se connecter à un port SSH sur le même hôte que celui utilisé dans `JENKINS_URL`. Si Jenkins se trouve derrière un proxy inverse HTTP, cela ne fonctionnera généralement pas. Exécutez donc Jenkins avec la propriété système `-Dorg.jenkinsci.main.modules.sshd.SSHD.hostName=ACTUALHOST` pour définir un nom d'hôte ou une adresse IP pour le point de terminaison SSH.

### Problèmes courants avec le client CLI

Un certain nombre de problèmes courants peuvent survenir lors de l'exécution du client CLI.

#### La clé du serveur n'a pas été validée.

Vous pouvez obtenir l'erreur ci-dessous et trouver une entrée de journal juste en dessous concernant des `mismatched keys` (clés non correspondantes) :

``` console
org.apache.sshd.common.SshException : La clé du serveur n'a pas été validée.
    à org.apache.sshd.client.session.AbstractClientSession.checkKeys(AbstractClientSession.java:523)
    at org.apache.sshd.common.session.helpers.AbstractSession.handleKexMessage(AbstractSession.java:616)
    ...
```

Cela signifie que votre configuration SSH ne reconnaît pas la clé publique présentée par le serveur. C'est souvent le cas lorsque vous exécutez Jenkins en mode développement et que plusieurs instances de l'application sont exécutées sous le même port SSH au fil du temps.

Dans un contexte de développement, accédez à votre fichier `~/.ssh/known_hosts` (ou `C:/Users/<votre_nom>/.ssh/known_hosts` pour Windows) et supprimez la ligne correspondant à votre port SSH actuel (par exemple `[localhost]:3485`). Dans un contexte de production, vérifiez auprès de l'administrateur Jenkins si la clé publique du serveur a récemment changé. Si tel est le cas, demandez à l'administrateur de suivre les étapes décrites ci-dessus.

#### UsernameNotFoundException

Si votre client affiche une trace de pile qui ressemble à :

``` console
org.acegisecurity.userdetails.UsernameNotFoundException: <nom_que_vous_avez_utilisé>
    ...
```

Cela signifie que vos clés SSH ont été reconnues et validées par rapport aux utilisateurs enregistrés, mais que le nom d'utilisateur n'est pas valide pour le domaine de sécurité utilisé actuellement par votre application. Cela peut se produire lorsque vous avez initialement utilisé la base de données Jenkins, configuré vos utilisateurs, puis basculé vers un autre domaine de sécurité (comme LDAP, etc.) où les utilisateurs définis n'existent pas encore.

Pour résoudre le problème, assurez-vous que vos utilisateurs existent dans votre domaine de sécurité configuré.

#### Journaux de dépannage

Pour obtenir plus d'informations sur le processus d'authentification :

* Allez dans **Manage Jenkins > System Log > Add new log recorder** (Gérer Jenkins > Journal système > Ajouter un nouvel enregistreur de journal) ;
* Entrez le nom de votre choix et cliquez sur **Ok** ;
* Cliquez sur **Add** (Ajouter) ;
* Saisissez `org.jenkinsci.main.modules.sshd.PublicKeyAuthenticatorImpl` (ou saisissez `PublicKeyAuth`, puis sélectionnez le nom complet) ;
* Définissez le niveau sur **ALL** (TOUT) ;
* Répétez les trois étapes précédentes pour `hudson.model.User` ;
* Cliquez sur **Save** (Enregistrer).

Lorsque vous essayez de vous authentifier, vous pouvez actualiser la page et voir ce qui se passe en interne.

