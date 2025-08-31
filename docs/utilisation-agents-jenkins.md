# Utilisation des agents Jenkins

L'architecture Jenkins est conçue pour les environnements de construction distribués. Il nous permet d'utiliser différents environnements pour chaque projet de construction équilibrant la charge de travail entre plusieurs agents exécutant des travaux en parallèle.

Le contrôleur Jenkins est le nœud d'origine dans l'installation de Jenkins. Le contrôleur Jenkins administre les agents Jenkins et orchestre leur travail, y compris les tâches de planification sur les agents et les agents de surveillance. Les agents peuvent être connectés au contrôleur Jenkins à l'aide d'ordinateurs locaux ou cloud.

Les agents nécessitent une installation Java et une connexion réseau au contrôleur Jenkins. Consultez la vidéo de 3 minutes ci-dessous pour une brève explication des agents de Jenkins.

_Qu'est-ce qu'un agent Jenkins_
<iframe width="640" height="360" src="https://www.youtube.com/embed/4KghHJEz5no" title="What Is a Jenkins Agent?" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Configuration des agents avec Docker

Les agents Jenkins peuvent être lancés dans des machines physiques, des machines virtuelles, des grappes Kubernetes et avec des images Docker. Cette section relie les agents Docker à Jenkins via SSH.

### Environnement

Pour exécuter ce guide, vous aurez besoin d'une machine avec : 

* Installation Java ;
* Installation Jenkins ;
* Installation Docker ;
* Paire de clés SSH.

!!! info
    Si vous avez besoin d'aide pour installer Java, Jenkins et Docker, veuillez visiter la section [Installation de Jenkins](./installation-docker.md).

### Générer une paire de clés ssh

Pour générer la paire de clés SSH, vous devez exécuter un outil de ligne de commande nommé ` ssh-keygen` sur une machine à laquelle vous avez accès. Cela pourrait être : 

* la machine sur laquelle fonctionne votre contrôleur Jenkins ;
* l'hôte (si vous utilisez des conteneurs) ;
* une machine sur laquelle vous avez un agent qui s'exécute ; 
* ou même votre machine de développeur.

!!! info
    La génération de clés SSH peut être effectuée sur n'importe quel système d'exploitation :

    * Sur Windows, vous pouvez utiliser n'importe quelle installation OpenSSH telle que [Windows OpenSSH](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse), le `ssh-keygen` qui est inclus avec [git for Windows](https://gitforwindows.org/) ou [Cygwin](https://cygwin.com/) ; 
    * Sur Unix (Linux, macOS, BSD, etc.), vous pouvez également utiliser toute installation OpenSSH emballée avec votre système. 

!!! note
    Notez que vous devrez être en mesure de copier la valeur clé sur votre contrôleur et votre agent, alors vérifiez donc que vous pouvez copier un contenu de fichier dans le presse-papiers au préalable. 

1. Dans une fenêtre de terminal, exécutez la commande: `ssh-keygen -f ~ / .ssh / jenkins_agent_key`.
2. Fournissez une phrase de passe à utiliser avec la clé (elle peut être vide).
3. Confirmez que la sortie ressemble à ceci : 

``` bash title="BASH"
ubuntu@desktop:~$ ssh-keygen -f ~/.ssh/jenkins_agent_key
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/jenkins_agent_key
Your public key has been saved in /home/ubuntu/.ssh/jenkins_agent_key.pub
The key fingerprint is:
SHA256:XqxxjqsLlvDD0ZHm9Y2iR7zC6IbsUlMEHo3ffy8TzGs
The key's randomart image is:
+---[RSA 3072]----+
|  o+             |
| ...o  .         |
|  .o .+ .        |
|    o+.+ o o     |
|  ... o.So* .    |
|  o+ = +.X=      |
| o oO + *..+     |
|. oo.o o .E .    |
| o... oo.. o     |
+----[SHA256]-----+
```

### Créer des informations d'identification SSH Jenkins

1. Depuis votre tableau de bord Jenkins, accédez à **_Manage Jenkins_** (gérer Jenkins). 
2. Dans la section Sécurité, sélectionnez **_Credentials_** (informations d'identification). 
![Option d'identification à partir de la page Gérer Jenkins.](https://www.jenkins.io/doc/book/resources/node/manage-credentials.png) 
3. Sous **_Stores scoped to Jenkins_** (Magasins pris en charge par Jenkins), sélectionnez `Add Credentials` dans l'option globale.
![Ajoutez l'option d'identification.](https://www.jenkins.io/doc/book/resources/node/add-credentials.png)
4. Remplissez les informations suivantes, comme indiqué dans l'exemple, en remplaçant vos informations au besoin : 
    * King : Nom d'utilisateur SSH avec clé privée ;
    * ID : Jenkins ;
    * Description : La clé Jenkins SSH ;
    * Nom d'utilisateur : jenkins ;
    * Clé privée : sélectionnez **_Enter directly _** (Entrer directement), puis**_Add_** (Ajouter) pour insérer le contenu de votre fichier de clé privée (`~/.ssh/jenkins_agent_key`).
    * Phrase de passe: Entrez la phrase de passe utilisée pour générer la paire de clés SSH (ou laissez-la vide si vous n'en avez pas utilisé à l'étape précédente). 
![Formulaire de configuration d'identification rempli.](https://www.jenkins.io/doc/book/resources/node/credentials-configuration.png)
5. Sélectionnez **_Create_** (Créer) pour terminer votre configuration d'identification.

### Créer votre agent Docker

#### Sur Linux

Ici, nous utiliserons l'image [docker-ssh-agent](https://github.com/jenkinsci/docker-ssh-agent) pour créer les conteneurs d'agent. 

1. Exécutez la commande pour démarrer votre premier agent : 

``` bash title="BASH"
docker run -d --rm --name=agent1 -p 22:22 \
-e "JENKINS_AGENT_SSH_PUBKEY=<your_public_key>" \
jenkins/ssh-agent:alpine-jdk21
```

!!! info
    * N'oubliez pas de remplacer la balise <your_public_key> pour votre propre clé publique SSH ; 
    * Dans cet exemple, vous pouvez trouver la valeur de votre clé publique en exécutant la commande suivante : `cat ~/.ssh/jenkins_agent_key.pub` sur la machine sur laquelle vous l'avez créée. N'ajoutez pas les crochets `[]` autour de la valeur de la clé ;
    * La valeur de [votre-clé-publique] DOIT inclure le contenu complet de votre fichier .pub, y compris le préfixe `ssh-XXXX` ; 
        * Ex : `ssh-rsa aaaab3nzac1yc2eaaaadaqabaaaaqqco9 + bpmryq / dl3ds2cyjxrf + j6ctbt3 / qp84 + kefhnii7nt7felilkusnxs30wavqcco2yu1orfgqr41mm70mb` ;
    * Si votre machine dispose déjà d'un serveur SSH fonctionnant sur le port `22` (si vous vous êtes connecté à cette machine grâce à la commande `ssh`, c'est le cas), vous devez utiliser un autre port pour la commande `docker`, telle que `-p 4444: 22`. 

2. Le conteneur `agent1` est désormais en cours d'exécution.
Astuce : la commande `docker ps ` permet de vérifier si le conteneur fonctionne comme prévu.

#### Sur Windows

Ici, nous utiliserons l'image [docker-ssh-agent](https://github.com/jenkinsci/docker-ssh-agent) pour créer les conteneurs d'agent. 

1. Exécutez la commande pour démarrer votre premier agent : 

``` bash title="BASH"
docker run -d --rm --name=agent1 --network jenkins -p 22:22 `
  -e "JENKINS_AGENT_SSH_PUBKEY=<your_public_key>" `
  jenkins/ssh-agent:jdk21
```

!!! info
    * N'oubliez pas de remplacer la balise <your_public_key> par votre propre clé publique SSH ;
    * Dans cet exemple, votre clé publique est : `Get-Content $Env:USERPROFILE\.ssh\jenkins_agent_key.pub`.

2. Le conteneur `agent1` est désormais en cours d'exécution.
Astuce : la commande `docker ps` permet de vérifier si le conteneur fonctionne comme prévu. De plus, la commande `docker container inspect agent1 | Select-String -Pattern “« IPAddress »: « \d+\.\d+\.\d+\.\d+ »”` permet de voir l'hôte à définir dans Jenkins pour l'agent.

### Configurez `agent1` sur Jenkins 

1. Depuis votre tableau de bord Jenkins, accédez à **_Manage Jenkins_** (gérer Jenkins). 
2. Sélectionnez **-Nodes-** (Nœuds). 
![Gérer le menu de nœud.](https://www.jenkins.io/doc/book/resources/node/manage-nodes.png)
3. Sélectionnez **_New Node_** (Nouveau Nœud) pour créer votre agent. 
4. Entrez le nom de votre Nœud, sélectionnez **_Permanent Agent option,_** (l'option Agent Permanent) et sélectionnez **_Create_** (Créer). 
5. Sur la page de création d'agent, remplissez les champs : 
    * Répertoire racine distant ;
    * Étiquettes ;
    * Usage ;
    * Méthode de lancement ;
        * Hôte ;
        * Informations d'identification ;
        * Hôte Stratégie de vérification des clés 
        ![Page de configuration de nœud remplie.](https://www.jenkins.io/doc/book/resources/node/node-configuration.png)
6. Sélectionnez **_Save_** (Enregistrer) et `agent1` sera enregistré, mais hors ligne pour le moment. Sélectionnez le nœud `agent1` pour afficher son statut.
7. La page d'état doit afficher le message : `This node is being launched` (ce nœud est lancé). Si ce n'est pas le cas, sélectionnez **_Relaunch agent_** (agent de relance) et attendez quelques secondes. 
8. Après avoir attendu, sélectionnez l'option `Log` (Journal) pour afficher les journaux. Au bas du journal, vous devriez recevoir le message : `Agent successfully connected and online.` (Agent connecté et en ligne).

Si votre contrôleur Jenkins ne démarre pas l'agent via SSH, veuillez vérifier le port que vous avez configuré sur votre agent. Copiez le numéro de port, puis sélectionnez **_Advanced_** (Avancé). Sous **_Advanced_**, vous pouvez coller le numéro de port dans le champ **_Port_**.

### Déléguer le premier emploi à `agent1`

1. À partir de votre tableau de bord Jenkins, sélectionnez **_New Item_** (Nouvel Article). 
2. Entrez un nom, par exemple ` First job on agent1.` (Première tâche sur agent1). 
3. Sélectionnez **_Freestyle project_** (projet Freestyle) et sélectionnez OK pour créer le travail. 
4. Sélectionnez l'option **_Restrict where this project can be run_** (Restreindre l'endroit où ce projet peut être exécuté).
5. Entrez l'étiquette du nœud (`agent1`) dans le champ **_Label Expression_** (Expression d'étiquette). 
![Champ d'expression d'étiquette avec la valeur agent1.](https://www.jenkins.io/doc/book/resources/node/label-expression.png)

    !!! info
        Soyez prudent avec les espaces blancs avant ou après l'étiquette. 
        
6. Sélectionnez l'option **_Execute shell_** (Exécuter Shell) dans la liste déroulante des étapes de construction.
7. Ajoutez la commande: `echo $NODE_NAME` dans le champ **_Command_** (ommande) et le nom de l'agent sera imprimé à l'intérieur du journal lorsque ce travail sera exécuté. 
![Entrer la commande pour l'étape de construction.](https://www.jenkins.io/doc/book/resources/node/build-step-execute.png)
8. Sélectionnez **_Save_** (Enregistrer) puis sélectionnez **_Build Now_** (Construire Maintenant). 
9. Attendez quelques secondes, puis accédez à la page **_Console Output_** (Sortie de la Console). Vous devriez recevoir une sortie similaire à : 
![Sortie de la console d'une construction réussie.](https://www.jenkins.io/doc/book/resources/node/console-output.png)

## Redémarrer un agent Jenkins

Cette vidéo fournit des instructions sur la façon de redémarrer un agent Jenkins en utilisant diverses méthodes.
<iframe width="800" height="420" src="https://www.youtube.com/embed/MTLgbp0GH8w" title="How to Restart a Jenkins Agent" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

