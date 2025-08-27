# Autres conteneurs de servlets

Jenkins est généralement exécuté en tant qu'application autonome dans son propre processus. Le fichier WAR Jenkins intègre [Winstone](https://github.com/jenkinsci/winstone), un conteneur de servlets [Jetty](https://www.eclipse.org/jetty/), et peut être lancé sur n'importe quel système d'exploitation ou plateforme disposant d'une version de Java prise en charge par Jenkins. Il s'agit de la méthode recommandée pour déployer Jenkins, qui est entièrement prise en charge.

En théorie, Jenkins peut également être exécuté en tant que servlet dans un conteneur de servlets traditionnel tel [qu'Apache Tomcat](https://tomcat.apache.org/) ou [WildFly](https://www.wildfly.org/), mais dans la pratique, cela n'a pas été largement testé et il existe de nombreuses mises en garde. En particulier, la prise en charge des agents WebSocket n'est implémentée que pour le conteneur de servlets Jetty. Pour plus d'informations, consultez la page [Politique de prise en charge des conteneurs de servlets](./plateforme-servlet.md).

!!! warning

    La prise en charge des conteneurs de servlets traditionnels pourrait être interrompue à l'avenir.


## Prérequis

* **Jenkins 2.492.3+ nécessite un servlet API 5.0 (Jakarta EE 9)** ; 
* Conteneurs compatibles ;
* **Tomcat 10.1.x** (testé avec 10.1.40 - dernière version) ;
* La version Wildfly a besoin de vérification (minimum 30+ recommandée) ;
* Nécessite JDK 17+ pour une compatibilité complète.

## Tomcat 10

### Déploiement

``` bash title="BASH"
# Download Tomcat 10.1.x
wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.40/bin/apache-tomcat-10.1.40.tar.gz

# Extract and deploy
tar xzf apache-tomcat-10.1.40.tar.gz
export CATALINA_HOME=$(pwd)/apache-tomcat-10.1.40
cp jenkins.war $CATALINA_HOME/webapps/
```

### Configuration

**Définir l'emplacement de Jenkins :**

``` bash title="BASH"
mkdir /var/lib/jenkins
echo "export CATALINA_OPTS=-DJENKINS_HOME=/var/lib/jenkins" > $CATALINA_HOME/bin/setenv.sh
chmod +x $CATALINA_HOME/bin/setenv.sh
```

!!! info

    L'exécution de plusieurs contrôleurs Jenkins dans un seul processus Java n'est pas supporté.

La sélection du schéma dans les URL de redirection est déléguée au conteneur de servlets, et Jetty gère par défaut les en-têtes `X-Forwarded-For, X-Forwarded-By` et `X-Forwarded-Proto`. Avec Tomcat, il faut ajouter une [Valve IP distante](https://tomcat.apache.org/tomcat-10.0-doc/config/valve.html#Remote_IP_Valve) pour exposer ces en-têtes à Jenkins via l'API Servlet. Ajoutez ce qui suit à `${CATALINA_HOME}/conf/server.xml`dans la section `<Host>` :

``` xml title="XML"
<Valve className="org.apache.catalina.valves.RemoteIpValve"
       remoteIpHeader="X-Forwarded-For"
       proxiesHeader="X-Forwarded-By"
       protocolHeader="X-Forwarded-Proto" />
```

## WildFly

### Prérequis 

* **WildFly 30+**  requis pour Jenkins 2.492.3+ avec JDK 17 ;
* Utilise l'API Servlet 5.0 (Jakarta EE 9) avec l'espace de noms `jakarta.servlet` ;
* **Incompatible avec WildFly 26 et versions antérieures** (API Servlet 4.0).

### Déploiement

``` bash title="BASH"
cp jenkins.war $JBOSS_HOME/standalone/deployments/
```

### Configuration

``` bash title="BASH"
# Add this to $JBOSS_HOME/bin/standalone.conf
JAVA_OPTS="$JAVA_OPTS -DJENKINS_HOME=/var/lib/jenkins"
```

!!! info

    L'exécution de plusieurs contrôleurs Jenkins dans un seul processus Java n'est pas supporté.
