# Processus de Génération à partir de la Compilation

<div class="couleur-introduction">
Il est possible de lancer un processus à partir d'une compilation et de faire en sorte que ce processus dure plus longtemps que la compilation elle-même. Par exemple, la compilation peut lancer un nouveau serveur d'applications avec le résultat de la compilation. Dans les versions antérieures, la compilation ne se terminait souvent pas. Au lieu de cela, l'étape spécifique (telle que le script shell, Ant ou Maven) se terminait, mais la compilation elle-même ne se terminait pas.
</div>

Jenkins détecte cette situation et, au lieu de bloquer indéfiniment, affiche un avertissement et termine la build.

## Pourquoi ?

Cela se produit en raison de la manière dont les descripteurs de fichiers sont utilisés entre les processus dans une build. Jenkins et le processus enfant sont connectés par trois pipes (`stdin`, `stdout` et `stderr`). Cela permet à Jenkins de capturer la sortie du processus enfant. Le processus enfant peut écrire beaucoup de données dans le canal et se fermer immédiatement après, donc Jenkins attend la fin du fichier (EOF) pour s'assurer qu'il a vidé les canaux avant de terminer la compilation.

Chaque fois qu'un processus se termine, le système d'exploitation ferme tous les descripteurs de fichiers qu'il possédait. Ainsi, même si le processus n'a pas fermé `stdout` et `stderr`, Jenkins obtient la fin du fichier (EOF).

La complication survient lorsque ces descripteurs de fichiers sont hérités par d'autres processus. Supposons que le processus enfant crée un autre processus en arrière-plan. Le processus en arrière-plan (qui est en fait un démon) hérite de tous les descripteurs de fichiers du processus parent, y compris la partie écriture des pipes `stdout` et `stderr` qui relient le processus enfant et Jenkins. Si le démon oublie de les fermer, Jenkins ne reçoit pas de EOF pour les pipes même lorsque le processus enfant se termine, car le démon a toujours ces descripteurs ouverts. C'est ainsi que ce problème se produit.

Un démon doit fermer tous les descripteurs de fichiers pour éviter ce genre de problèmes, mais certains démons ne respectent pas cette règle. Vous pouvez atténuer ce problème à l'aide de différentes solutions de contournement.

## Solutions de contournement

Sous Unix, vous pouvez utiliser un wrapper comme celui-ci pour que le démon se comporte correctement. Par exemple :

``` cfg title="DAEMON"
daemonize -E BUILD_ID=dontKillMe /path/to/your/command
```

Dans un pipeline Jenkins, utilisez `JENKINS_NODE_COOKIE` à la place de `BUILD_ID`.

Notez que cela définira la variable d'environnement BUILD_ID pour le processus en cours de création sur une valeur différente de la valeur BUILD_ID actuelle. Vous pouvez également démarrer Jenkins avec `-Dhudson.util.ProcessTree.disable=true`.

Sous Windows, utilisez la [commande « at »](http://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/ntcmds.mspx?mfr=true) pour lancer un processus en arrière-plan. Par exemple :

``` cfg
<scriptdef name="get-next-minute" language="beanshell">
  <attribute name="property" />

  date = new java.text.SimpleDateFormat(« HH:mm »)
    .format(new Date(System.currentTimeMillis() + 60000));
  project.setProperty(attributes.get(« property »), date);
</scriptdef>

<get-next-minute property="next-minute" />
<exec executable="at">
  <arg value="${next-minute}" />
  <arg value="/interactive" />
  <arg value="${jboss.home}\bin\run.bat" />
</exec>
```

Une autre solution similaire sous Windows consiste à utiliser un script wrapper et à lancer votre programme via celui-ci :

``` cfg
// antRunAsync.js - Wrapper script to run an executable detached in the
// background from Ant's <exec> task.  This works by running the executable
// using the Windows Scripting Host WshShell.Run method which doesn't copy
// the standard filehandles stdin, stdout and stderr. Ant finds them closed
// and doesn't wait for the program to exit.
//
// requirements:
//   Windows Scripting Host 1.0 or better.  This is included with Windows
//   98/Me/2000/XP.  Users of Windows 95 or Windows NT 4.0 need to download
//   and install WSH support from
//   http://msdn.microsoft.com/scripting/.
//
// usage:
// <exec executable="cscript.exe">
//   <env key="ANTRUN_TITLE" value="Title for Window" />  <!-- optional -->
//   <env key="ANTRUN_OUTPUT" value="output.log" />  <!-- optional -->
//   <arg value="//NoLogo" />
//   <arg value="antRunAsync.js" />  <!-- this script -->
//   <arg value="real executable" />
// </exec>


var WshShell = WScript.CreateObject("WScript.Shell");
var exeStr = "%comspec% /c";
var arg = "";
var windowStyle = 1;
var WshProcessEnv = WshShell.Environment("PROCESS");
var windowTitle = WshProcessEnv("ANTRUN_TITLE");
var outputFile = WshProcessEnv("ANTRUN_OUTPUT");
var OS = WshProcessEnv("OS");
var isWindowsNT = (OS == "Windows_NT");

// On Windows NT/2000/XP, specify a title for the window.  If the environment
// variable ANTRUN_TITLE is specified, that will be used instead of a default.
if (isWindowsNT) {
  if (windowTitle == "")
     windowTitle = "Ant - " + WScript.Arguments(i);
  exeStr += "title " + windowTitle + " &&";
}

// Loop through arguments quoting ones with spaces
for (var i = 0; i < WScript.Arguments.count(); i++) {
  arg = WScript.Arguments(i);
  if (arg.indexOf(' ') > 0)
    exeStr += " \"" + arg + "\"";
  else
    exeStr += " " + arg;
}

// If the environment variable ANTRUN_OUTPUT was specified, redirect
// output to that file.
if (outputFile != "") {
  windowStyle = 7;  // new window is minimized
  exeStr += " > \"" + outputFile + "\"";
  if (isWindowsNT)
    exeStr += " 2>&1";
}

// WScript.Echo(exeStr);
// WshShell.Run(exeStr);
WshShell.Run(exeStr, windowStyle, false);
```

Ce script wrapper fourni par l'utilisateur peut être appelé depuis Ant, par exemple :

```cfg
<exec executable="cscript.exe">
   <env key="ANTRUN_TITLE" value="Title for Window" />  <!-- facultatif -->
   <env key="ANTRUN_OUTPUT" value="output.log" />  <!-- facultatif -->
   <arg value="//NoLogo" />
   <arg value="antRunAsync.js" />  <!-- ce script -->
   <arg value="exécutable réel" />
</exec>
```

Une autre solution pour Windows consiste à planifier une tâche permanente et à forcer son exécution à partir du script Ant. Par exemple, exécutez la commande :

```cfg
C:\>SCHTASKS /Create /RU SYSTEM /SC ONSTART /TN Tomcat /TR
« C:\Program Files\Apache Software Foundation\Tomcat 6.0\bin\startup.bat »
```

Notez que `ONSTART` peut être remplacé par `ONCE` si vous ne souhaitez pas que Tomcat reste en cours d'exécution. Ajoutez le code suivant à votre script Ant :

``` cfg
<exec executable="SCHTASKS">
    <arg value="/Run"/>
    <arg value="/TN"/>
    <arg value="Tomcat"/>
```


