# Modifier le Fuseau Horaire Système

<div class="couleur-introduction">
La configuration du fuseau horaire du système correspond au fuseau horaire par défaut affiché par Jenkins. La page « Manage Jenkins » ⇒ « System Information » (Gérer Jenkins ⇒ Informations système) affiche la valeur des propriétés système qui définissent le fuseau horaire du contrôleur Jenkins.
</div>

Consultez la vidéo suivante pour obtenir des conseils sur la modification du fuseau horaire.

_Modification du fuseau horaire dans Jenkins_

<iframe width="800" height="420" src="https://www.youtube.com/embed/4UmY4dDAlo0" title="How to Change the Time Zone in Jenkins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Fuseau horaire défini par l'utilisateur

Un fuseau horaire défini par l'utilisateur pour le compte peut être configuré à partir de l'option **Account** (Compte) dans les paramètres utilisateur.

![Modifier le fuseau horaire de l'utilisateur via les paramètres du compte utilisateur](https://www.jenkins.io/doc/book/resources/managing/change-system-timezone-user-defined-timezone.png)

Il est automatiquement défini sur la valeur par défaut du système, mais peut être modifié pour correspondre à n'importe quel pays ou fuseau horaire, selon vos préférences.

![Fuseau horaire de l'utilisateur basé sur la valeur par défaut du système](https://www.jenkins.io/doc/book/resources/managing/user-defined-timezone-default.png)

## Propriétés du fuseau horaire du système

Si vous ne pouvez pas modifier le fuseau horaire de votre serveur, vous pouvez forcer jelly à utiliser un fuseau horaire donné pour formater les horodatages.

Vous devez démarrer Jenkins avec la propriété système java suivante :

``` java
java -Dorg.apache.commons.jelly.tags.fmt.timeZone=TZ ...
```

où TZ est un identifiant java.util.TimeZone (par exemple « Europe/Paris »).

_Notez que `user.timezone=Europe/Paris` fonctionnera également, mais cela peut interférer avec d'autres contextes_.

Si vous exécutez Jenkins via un paquet Linux, vous pouvez y parvenir en exécutant `systemctl edit jenkins` et en ajoutant ce qui suit :

``` cfg
[Service]
Environment="JAVA_OPTS=-Dorg.apache.commons.jelly.tags.fmt.timeZone=America/New_York"
```

ou, si cela ne fonctionne pas :

``` cfg
[Service]
Environment=« JAVA_OPTS=-Duser.timezone=America/New_York »
```

Sous FreeBSD, le fichier à modifier est /etc/rc.conf, et l'option à utiliser est :

``` cfg
jenkins_java_opts=« -Dorg.apache.commons.jelly.tags.fmt.timeZone=America/Denver »
```

Sous Windows, modifiez `%INSTALL_PATH%/jenkins/jenkins.xml`. Placez `-Dargs` avant `-jar` :

``` cfg
<arguments>-Duser.timezone=« Europe/Minsk » -jar « %BASE%\jenkins.war »</arguments>
```

Vous pouvez également le définir à partir de la [console de script](./gestion-cli.md) Jenkins sur un système en direct sans avoir à redémarrer. Cela peut également être inclus dans un [script post-initialisation](./gestion-scripts-groovy-hook.md) pour le rendre permanent.

``` groovy
System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'America/New_York')
```
