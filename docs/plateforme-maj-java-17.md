# Mise à niveau vers Java 17

Lors de la mise à niveau du JVM utilisé pour exécuter Jenkins de Java 11 à Java 17, il y a des détails que vous devez connaître et des précautions que vous devez prendre.

_Mise à niveau de la version Jenkins Java de 11 à 17_.

<iframe width="800" height="420" src="https://www.youtube.com/embed/ZabUz6sl-8I" title="Upgrading Jenkins Java Version From 11 to 17" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Sauvegarder Jenkins

Comme pour toute mise à niveau, nous recommandons : 

1.  [Sauvegarde de JENKINS_HOME](./administration-sauvegarde.md#jenkins-home).
2.  Test de la mise à niveau avec votre sauvegarde.
3.  Une fois tous les tests requis réussis, procédez à la mise à niveau sur votre contrôleur de production.

## Mettre à niveau Jenkins

Si vous avez besoin de mettre à niveau Jenkins, ainsi que le JVM, nous vous recommandons : 

1.  [Sauvegarde de JENKINS_HOME](./administration-sauvegarde.md#jenkins-home).
2. Arrêtez le contrôleur Jenkins. 
3. Mettez à niveau le JVM sur lequel Jenkins fonctionne. 
    * Utilisez un gestionnaire de packages pour installer le nouveau JVM ;
    * Assurez-vous que le JVM par défaut est la version nouvellement installée ; 
        * Si ce n'est pas le cas, exécutez `systemctl edit jenkins` et définissez la variable d'environnement `JAVA_HOME ` ou la variable d'environnement `JENKINS_JAVA_CMD `. 
4. Mettez à niveau Jenkins vers la version la plus récente. 
    * La façon dont vous mettez à niveau Jenkins dépend de votre méthode d'installation d'origine Jenkins. 
!!! Success 
    Nous vous recommandons d'utiliser le gestionnaire de paquets de votre système (comme `apt` ou `yum`). 
5. Validez la mise à niveau pour confirmer que tous les plugins et travaux sont chargés. 
6. Mettez à niveau les plugins requis.

Lors de la mise à niveau de la version Java pour Jenkins et le JVM, il est important de mettre à niveau tous les plugins qui prennent en charge Java 17. Les mises à niveau du plugin assurent la compatibilité avec les versions les plus récentes de Jenkins. 

!!! info 
    Si vous découvrez un problème auparavant non déclaré, veuillez nous le faire savoir. Reportez-vous à notre [documentation sur le signalement des problèmes](https://www.jenkins.io/participate/report-issue/#issue-reporting) pour obtenir des conseils.

## Version JVM sur les agents

Tous les agents doivent fonctionner sur la même version de JVM que le contrôleur, en raison de la manière dont les contrôleurs et les agents communiquent. Si vous mettez à niveau votre contrôleur Jenkins pour qu'il fonctionne sous Java 17, vous devez mettre à niveau la JVM sur vos agents.

La validation de la version de chaque agent peut être effectuée à l'aide du plugin [Versions Node Monitors](https://plugins.jenkins.io/versioncolumn). Ce plugin fournit des informations sur la version JVM de chaque agent sur l'écran de gestion des nœuds de votre contrôleur Jenkins. Ce plugin peut également être configuré pour déconnecter automatiquement tout agent dont la version JVM est incorrecte.
