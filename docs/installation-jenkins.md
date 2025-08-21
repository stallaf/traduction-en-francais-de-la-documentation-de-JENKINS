# Installer Jenkins

Les procédures de ce chapitre sont destinées aux nouvelles installations de Jenkins.

Jenkins est généralement exécuté comme une application autonome dans son propre processus. Le fichier WAR Jenkins intègre [Winstone](https://github.com/jenkinsci/winstone), un conteneur de servlets [Jetty](https://www.eclipse.org/jetty/), et peut être lancé sur n'importe quel système d'exploitation ou plateforme disposant d'une version de Java prise en charge par Jenkins.

Théoriquement, Jenkins peut également être géré comme un servlet dans un conteneur de servlet traditionnel comme [Apache Tomcat](https://tomcat.apache.org/) ou [Wildfly](https://www.wildfly.org/), mais en pratique, cela n'est pas testé et il y a de nombreuses mises en garde. En particulier, la prise en charge des agents WebSocket n'est implémentée que pour le conteneur de servlets Jetty. Pour plus d'informations, consultez la page [Politique de prise en charge des conteneurs Servlets](./plateforme-politique-de-support-de containeur-servlets).
<hr>