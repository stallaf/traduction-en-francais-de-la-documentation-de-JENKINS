# Politique d'Assistance des Conteneurs Servlet

Cette page documente la politique de support des conteneurs Servlet pour le contrôleur Jenkins.

## Pourquoi ?

Jenkins fonctionne généralement comme une application autonome dans son propre processus. Le fichier WAR de Jenkins regroupe [Winstone](https://github.com/jenkinsci/winstone), un emballage de conteneur de servlet [Jetty](https://www.eclipse.org/jetty/), et peut être démarré sur n'importe quel système ou plate-forme d'exploitation avec une version de Java prise en charge par Jenkins. Il s'agit du moyen préféré de déployer Jenkins et est entièrement pris en charge.

Théoriquement, Jenkins peut également être géré comme un servlet dans un conteneur de servlet traditionnel comme [Apache Tomcat](https://tomcat.apache.org/) ou [Wildfly](https://www.wildfly.org/). Cependant, dans la pratique, cela est largement non testé, et il y a beaucoup de mises en garde. En particulier, la prise en charge des agents WebSocket n'est implémentée que pour le conteneur servlet Jetty. 

!!! warning "Attention"

    Le soutien aux conteneurs de servlets traditionnels peut être interrompu à l'avenir.

## Niveau de support

Nous définissons plusieurs niveaux de support pour les conteneurs de servlets.

|  Niveau de support | Description | Conteneur de Servlet |
| ----------------------------- | ------------------ | --------------------------------- |
| Niveau 1 : Supporté | Nous effectuons des tests automatisés pour ces conteneurs de servlets, et nous avons l'intention de résoudre les problèmes signalés en temps opportun. | Les versions de Winstone et Jetty se sont regroupées dans le [fichier WAR](./installation-war.md) de Jenkins.
| Niveau 2 : Correctifs envisagés | Le support peut avoir des limitations et des exigences supplémentaires. Nous ne testons pas régulièrement la compatibilité et nous pouvons abandonner le soutien à tout moment. Nous prenons en considération les correctifs qui ne compromettent pas le support de niveau 1 et qui n'entraînent pas de coût de maintenance supplémentaires. | * Tomcat 9, basé sur l'API Servlet 4.0 (Jakarta EE 8) avec importations `javax.servlet`. (**Weekly 2.474, LTS 2.462.3 et versions antérieures**) ;<br> * WildFly 26, basé sur l'API Servlet 4.0 (Jakarta EE 8) avec importations `javax.servlet`. (**Weekly 2.474, LTS 2.462.3 et versions antérieures**) ;<br> * Autres conteneurs de servlets basés sur l'API Servlet 4.0 (Jakarta EE 8) avec importations `javax.servlet`. (**Weekly 2.474, LTS 2.462.3 et versions antérieures**) ;<br> * Jetty 11 ou version ultérieure, basé sur l'API Servlet 5.0 (Jakarta EE 9) ou version ultérieure avec importations `jakarta.servlet`. (**Weekly 2.475, LTS 2.479.1 et versions ultérieures**) ;<br> * Tomcat 10 ou version ultérieure, basé sur l'API Servlet 5.0 (Jakarta EE 9) ou version ultérieure avec importations `jakarta.servlet`. (**Weekly 2.475, LTS 2.479.1 et versions ultérieures**) ;<br> * WildFly 27 ou version ultérieure, basé sur Servlet API 5.0 (Jakarta EE 9) ou version ultérieure avec importations `jakarta.servlet`. (**Weekly 2.475, LTS 2.479.1 et versions ultérieures**).
| Niveau 3 : Non pris en charge | Ces versions sont connues pour être incompatibles ou avoir des limitations graves. Nous ne prenons pas en charge les conteneurs de servlet répertoriés. | * Jetty 11 ou version ultérieure, basée sur le servlet API 5.0 (Jakarta EE 9) ou ultérieur avec les importations de `jakarta.servlet`. (**Weekly 2.474, LTS 2.462.3 et ultérieur**) ; <br>* Tomcat 10 ou version ultérieure, basé sur Servlet API 5.0 (Jakarta EE 9) ou ultérieur avec les importations de `jakarta.servlet`. (**Weekly 2.474, LTS 2.462.3 et ultérieur**) ;<br> * Wildfly 27 ou version ultérieure, basée sur le servlet API 5.0 (Jakarta EE 9) ou ultérieur avec les importations de `jakarta.servlet`. (**Weekly 2.474, LTS 2.462.3 et ultérieur**) ;<br> * Autres conteneurs de servlets basés sur le servlet API 5.0 (Jakarta EE 9) ou ultérieur avec les importations de `jakarta.servlet`. (**Weekly 2.474, LTS 2.462.3 et ultérieur**) |

!!! Warning "Attention"

    Le soutien à Jakarta EE 8 devrait se terminer avec la sortie du LTS d'octobre.

## Références 

* [Instructions d'installation](./installation-autres-conteneurs.md) ;
* [Versions de Jetty](https://www.eclipse.org/jetty/) ; 
* [Versions de Tomcat](https://tomcat.apache.org/whichversion.html) ; 
* [Annonce de versions de Wildfly 27](https://www.wildfly.org/news/2022/11/09/WildFly27-Final-Released/).

## Contribution

N'hésitez pas à proposer des _PRs_ (Pull Requests) qui ajoutent la prise en charge ou la documentation d'autres conteneurs de servlets ou à partager vos commentaires ; nous apprécions vos contributions ! La prise en charge des conteneurs de servlets dans Jenkins relève du groupe d'intérêt spécial Platform, qui dispose d'un chat, d'un forum et organise des réunions régulières. Vous êtes les bienvenus sur ces canaux.
