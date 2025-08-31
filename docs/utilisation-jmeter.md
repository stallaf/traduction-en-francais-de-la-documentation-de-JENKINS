# Utilisation de JMeter avec Jenkins

Il y a plusieurs avantages à utiliser JMeter et Jenkins ensemble. L'intégration continue et l'automatisation des tests sont devenues des normes dans le monde DevOps, mais les niveaux de performance et la complexité du système augmentent constamment.

Avec Jenkins, vous pouvez intégrer tous les tests JMeter dans votre processus Pipeline et mieux comprendre les détails de vos applications.

Certains des principaux avantages de l'utilisation de JMeter avec Jenkins sont : 

* Exécution de test sans surveillance pour chaque système ;
* Construire des journaux d'échec et des étapes de récupération ;
* Accès sécurisé et facile aux rapports de test de chaque version ;
* Automatisation du travail de routine. 

!!! info
    Cette page décrit comment utiliser Apache JMeter avec Jenkins. Les instructions sont intentionnellement effectuées en exécutant Apache JMeter sur le contrôleur Jenkins. Apache JMeter dans un environnement de production Jenkins devrait être exécuté sur un agent Jenkins, pas sur le contrôleur Jenkins. Pour en savoir plus sur les agents de Jenkins, reportez-vous à la [page d'agents de Jenkins](./utilisation-agents-jenkins.md).

## Apache JMeter

[Apache JMeter](https://jmeter.apache.org/) peut être utilisé pour tester les performances des sites statiques, des sites dynamiques et des applications Web complètes. Il peut également être utilisé pour simuler une charge lourde sur un serveur, un groupe de serveurs, un réseau ou un objet, permettant des tests de résistance ou une analyse globale des performances sous différents types de charge.

## Installation de Jenkins

La documentation Jenkins a une page pour aider avec le [processus d'installation de Jenkins](./installation-presentation.md). Ce guide utilise l'installation .jar. Reportez-vous à la page [visite guidée](tutoriels-visite-guidee.md) si vous souhaitez également utiliser .jar. Les deux méthodes d'installation produisent les mêmes résultats.

## Installez le plugin Performance

Pour intégrer JMeter à Jenkins, nous utiliserons le [plugin Performance](https://plugins.jenkins.io/performance).

Suivez ces étapes pour l'installer : 

1. Depuis votre tableau de bord Jenkins, accédez à **_Manage Jenkins._** (Gérer Jenkins).
2. Accédez à la page **Plugins**.
3. Sélectionnez **_Available_** (Disponible), puis saisissez « performance » dans le champ de recherche.
4. Cochez la case d'installation, puis sélectionnez **_Install without restart._** (Installer sans redémarrer).
![Plugins disponibles filtrés pour les performances.](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-00.png)

Si tout réussit, vous lirez cet écran de confirmation :
![Installez la page de confirmation de succès.](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-01.png)

## Installation de JMETER

Pour installer JMeter, suivez ces étapes : 

1. Reportez-vous à la [page de téléchargement d'Apache JMeter](https://jmeter.apache.org/download_jmeter.cgi).
2. Sélectionnez l'option de téléchargement adaptée à votre système : .zip pour Windows ou .tgz pour Linux. Ce tutoriel est réalisé sous Linux, c'est pourquoi l'option .tgz s'affiche.
3. Extrayez le fichier téléchargé à l'emplacement de votre choix, par exemple `/usr/jmeter`.
4. Modifiez le fichier : `<VOTRE-CHEMIN-JMETER>>/bin/user.properties`. Par exemple, usr/jmeter/bin est le chemin d'accès au fichier utilisé ici.
5. Ajoutez cette commande à la dernière ligne du fichier : `jmeter.save.saveservice.output_format=xml`. Enregistrez et fermez le fichier pour que les modifications soient prises en compte.
![Commande user.properties](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-03.png)

Cette commande intègre la sortie de JMeter dans Jenkins. Créons maintenant notre plan de test JMeter.

### Premier plan de test JMeter en utilisant une interface graphique

JMeter utilise des plans de test pour organiser chaque test. Une fois configuré, Jenkins appelle tous les plans de test définis dans un pipeline, puis affiche les résultats dans les rapports de build. Cela signifie que tous les plans de test doivent d'abord être configurés sur JMeter. Une fois ceci terminé, entrez les informations dans Jenkins afin qu'il sache quels tests doivent être appelés.

Suivez ces étapes pour créer un plan de test : 

1. Exécutez le fichier : `<VOTRE-CHEMIN-JMETER>>/bin/jmeter.sh` pour ouvrir l'interface graphique JMeter. Par exemple /usr/jmeter/bin/jmeter.sh serait utilisé dans cet exemple. Dans une installation définitive, vous pouvez définir ces commandes dans votre système de chemin d'accès ou vos variables système.

    !!! info
        Pour les utilisateurs de Windows, le fichier sera `jmeter.bat`. 

2. À partir de l'interface graphique JMeter, allez dans **_File_**_** (Fichier), puis sélectionnez **_New_** (Nouveau). 
3. Entrez un nom pour votre plan de test. 
4. Sur le côté gauche de l'écran, en utilisant la sélection droite ou secondaire avec votre souris, sélectionnez votre plan de test. Suivez ce chemin : **_Add > Thread(Users) > Thread Group_** (Ajoutez> Thread (Utilisateurs)> Thread Groupe) et sélectionnez-le. 
![Ajout d'un groupe de threads au plan de test.](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-04.png)
5. Dans le Groupe de Threads, augmentez **_Number of Threads (users)_** (Nombre de Threads (utilisateurs)) à cinq et **_Loop Count_** (le nombre de boucles) à deux. 
6. Sur le côté gauche de l'écran, sélectionnez **_Thread Group_** (Groupe de Thread) avec votre souris, puis suivez ce chemin : **_Add > Sampler > HTTP Request_** (Ajouter > Échantillonneur > Requête HTTP), et sélectionnez l'option _HTTP Request_.
![Groupe de discussion](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-05.png)
7. Dans la demande HTTP, entrez le nom de votre test, le nom du serveur ou l'IP et le contexte de chemin. Par exemple, ici, nous utiliserions `Installing`, `www.jenkins.io`, et `/doc/book/installing/`.
![Échantillon de demande HTTP](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-06.png)
8. Répétez les étapes six et sept deux fois de plus vers différents contextes/pages. Par exemple, nous utiliserons www.jenkins.io/node. Maintenant, notre plan a trois choses à tester. 
9. Pour ajouter un rapport visuel, sélectionnez votre groupe de threads avec le bouton droit ou le bouton secondaire, puis suivez le chemin : **_Add > Listener > View results in table_** (Ajouter > Écouteur > Afficher les résultats dans un tableau). Sélectionnez l'option **_View Results in Table_** (Afficher les résultats dans un tableau).
![Enregistrer et exécuter le plan de test](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-07.png) 
10. Pour enregistrer le plan de test, sélectionnez l'icône Enregistrer (disque) en haut à gauche de l'écran ou accédez à **_File > Save,_** (Fichier > Enregistrer) et entrez un nom pour le plan de test avec une extension .jmx. Par exemple : `jenkins.io.jml`.

Exécutez le test et affichez les résultats du tableau.
![Résultats des tests](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-08.png)

### Premier plan de test JMeter à l'aide de commandes de terminal

Notre test fonctionne bien dans l'interface utilisateur graphique, mais pour l'intégrer à Jenkins, il doit être exécuté à partir de la ligne de commande.

Pour exécuter le plan de test à l'aide de la ligne de commande, suivez ces étapes : 

1. À partir du terminal, exécutez la commande suivante : 
``` bash title="BASH"
set OUT=jmeter.save.saveservice.output_format
set JMX=/usr/jmeter/bin/jenkins.io.jmx
set JTL=/usr/jmeter/reports/jenkins.io.report.jtl
/usr/jmeter/bin/jmeter -j %OUT%=xml -n -t %JMX% -l %JTL%
```
2. Si tout fonctionne correctement, le fichier de rapport est créé à l'emplacement indiqué par le paramètre `-l`. 
![Résultats du test de ligne de commande jMeter.](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-09.png)

## Jenkins et JMeter fonctionnant ensemble

Après avoir exécuté JMeter à partir de la ligne de commande, nous disposons désormais de tout le nécessaire pour exécuter JMeter à partir de Jenkins.

Pour exécuter Jmeter à partir de Jenkins, procédez comme suit :

1. Dans le tableau de bord Jenkins, sélectionnez **_New Item_** (Nouvel élément).
2. Entrez le nom de l'élément, par exemple `JmeterTest`, sélectionnez freestyle project (projet libre), puis sélectionnez **OK**.
3. Accédez à l'onglet **_Build Environment_** (Environnement de Construction), sélectionnez **_Add build step_** (Ajouter une étape de construction), puis sélectionnez l'option **_Execute Windows batch command_** (Exécuter la commande batch Windows).
4. Entrez le même code que celui utilisé pour exécuter JMeter dans la section précédente :
![Étape de construction jenkins jmeter](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-10.png)
5.  Accédez à l'onglet **_Post-build Action_** (Action Post-compilation), sélectionnez **_ Add post-build action_** (Ajouter une action post-compilation), puis sélectionnez **_Publish Performance test result report._** (Publier le rapport des résultats du test de performance).

    !!! info
        Cette option provient du plugin performance. Si elle n'est pas disponible, consultez la section précédente et assurez-vous que vous avez installé le plugin.

6. Remplissez la source de ces rapports :
![Source des rapports](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-11.png)
7. Enregistrez le projet, puis sélectionnez **_Build Now_** [Compiler Maintenant) dans la page JmeterTest.
8. Une fois la tâche terminée, accédez à la vue **_Console Output_** (Sortie de la Console) pour afficher les détails de l'exécution.
![Détails de l'exécution](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-12.png)
9. À partir de la vue **_Console Output_** (Sortie de la Console), vous pouvez accéder au rapport de performances et afficher les données du rapport JMeter.
![Détails de l'exécution du rapport](https://www.jenkins.io/doc/book/resources/jmeter/jmeter-13.png)

JMeter s'exécute désormais dans Jenkins et vous pouvez utiliser les données fournies.