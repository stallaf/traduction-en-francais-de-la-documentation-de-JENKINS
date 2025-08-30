# Référencer un autre projet par son nom

Dans de nombreux endroits à travers Jenkins, vous pouvez vous référer à un autre projet/travail par leur nom. Par exemple, dans un script Pipeline, vous voudrez peut-être [copier des artefacts](https://plugins.jenkins.io/copyartifact/) à partir d'un autre projet :

    copyArtifacts projectName: 'myproject'

C'est tout ce que vous devez faire si le nom de votre projet cible est simplement alphanumérique et est un projet simple sans sous-projets, et a un nom unique dans tout votre contrôleur Jenkins. Lisez la suite pour des scénarios plus complexes…

## Différencier plusieurs projets portant le même nom

Si vous utilisez le [plugin Folders](https://plugins.jenkins.io/cloudbees-folder) et que vous avez plusieurs projets portant le même nom dans des dossiers différents, vous pouvez les différencier à l'aide d'un chemin d'accès, similaire à un chemin d'accès au système de fichiers Unix. Il existe deux types de chemins d'accès :

### Chemins absolus

Les chemins absolus commencent par une barre oblique et font référence à un projet en décrivant le chemin complet permettant d'accéder au projet à partir de la page d'accueil de votre contrôleur Jenkins. Par exemple, pour référencer un projet à la racine de votre contrôleur Jenkins :

    /myproject

Ou, pour référencer un projet dans un sous-dossier :

    /myfolder/myproject

### Chemins relatifs

Les chemins relatifs commencent par un caractère autre que la barre oblique et font référence à un autre projet par rapport au projet actuel. Par exemple, supposons que vous ayez des projets avec les chemins absolus suivants :

    /thatproject
    /folder/someproject
    /folder/subfolder/myproject
    /folder/subfolder/anotherproject

Dans un script Pipeline pour ` /folder/subfolder/myproject`, vous pouvez vous référer à `/folder/subfolder/anotherproject` en utilisant ce chemin relatif :

    anotherproject

Et vous pouvez faire référence à `/folder/someproject` en utilisant ce chemin relatif, où `. .` signifie chercher dans le dossier parent :

    ../someproject

Et vous pouvez faire référence à `/thatproject` en utilisant ce chemin relatif :

    ../../thatproject

## Référence à des composants à l'intérieur des projets

Certains types de projets, tels que les projets Maven, Matrix et Multibranch, comportent des sous-composants. Vous pouvez faire référence à ces sous-composants comme suit :

### Projets maven

Vous pouvez faire référence à un projet Maven entier :

    mymavenproject

Ou à un groupe au sein d'un projet Maven :

    mymavenproject/my.group

Ou à un module particulier :

    mymavenproject/my.group$MyModule

### Projets Matrix

Vous pouvez faire référence à toutes les configurations d'un projet Matrix :

    mymatrixproject

Ou à une configuration particulière, restreinte par un axe :

    mymatrixproject/someaxis=somevalue

Ou restreinte par plusieurs axes :

    mymatrixproject/someaxis=somevalue,anotheraxis=anothervalue

### Pipelines multibranches

Vous pouvez faire référence à une branche particulière :

    mymultibranchproject/mybranch

## Encodage des noms

Les caractères spéciaux dans les chemins d'accès doivent être encodés en URL. Par exemple, si votre pipeline multibranches comporte une branche contenant une barre oblique `(feature/myfeature`), remplacez la barre oblique par `%2F` :

    mymultibranchproject/feature%2Fmyfeature

## Pour les développeurs de Jenkins et des plugins Jenkins

Consultez la fonction [Jenkins::getItem()](https://javadoc.jenkins.io/jenkins/model/Jenkins.html#getItem-java.lang.String-hudson.model.ItemGroup-).
