# Politique d'Assistance Windows

Cette page documente la politique de support Windows pour le contrôleur et les agents Jenkins.

## Portée

Les plugins Jenkins peuvent définir des exigences supplémentaires sur les versions Windows sur les contrôleurs et/ou les agents. Cette page ne documente pas de telles exigences. Veuillez vous référer à la documentation du plugin.

## Pourquoi ?

Théoriquement, Jenkins peut fonctionner partout où vous pouvez exécuter une version Java prise en charge, mais il y a certaines limites dans la pratique. Jenkins Core et certains plugins incluent le code natif ou dépendent de l'API et des sous-systèmes Windows, et donc ils s'appuient sur des plates-formes et versions Windows spécifiques. Dans Windows Services, nous utilisons également [Windows Service Wrapper (WINSW)](https://github.com/winsw/winsw), qui nécessite .NET Framework.

## Niveaux de support

Nous définissons plusieurs niveaux de support pour les plates-formes Windows.

| Niveau de support    | Description   | Plateforme |
| ------------------------------- | ------------------- | ------------------ |
| **Niveau 1** - Support complet    | Nous effectuons des tests automatisés pour ces plateformes et nous avons l'intention de résoudre<br> les problèmes signalés en temps opportun. | * Versions 64 bits (AMD-64) Windows Server, avec le dernier pack de mise à jour GA ;<br> * Versions Windows utilisées dans les images officielles Docker.
| **Niveau 2** - Supporté   | Nous ne testons pas activement ces plateformes,<br> mais nous avons l'intention de maintenir la<br> compatibilité. Nous sommes heureux d'accepter<br> les correctifs. | * Versions 64 bits (AMD-64) Windows Server généralement prises en charge par Microsoft ;<br> * Versions 64 bits (AMD-64) Windows 10 et 11 généralement prises en charge par Microsoft. |
| **Niveau 3** - Correctifs envisagés   | Le support peut avoir des limitations et des<br> exigences supplémentaires. Nous prendrons en considération les correctifs s'ils ne<br> compromettent pas la prise en charge des<br> niveaux 1 ou 2 et n'entraînent pas de frais de maintenance supplémentaires. | * x86 et autres architectures non AMD64 ;<br> * Versions non courantes, par exemple, Windows intégrés ;<br> * Versions de prévisualisation ;<br> * Moteurs d'émulation API Windows, par exemple, vin ou reactos. |
| **Niveau 4** - Non pris en charge | Ces versions sont connues pour être<br> incompatibles ou avoir des limitations graves. Nous ne prenons pas en charge les plateformes répertoriées et nous n'accepterons pas les correctifs. | * Les versions Windows ne sont plus prises en charge par Microsoft ;<br> * Windows XP antérieur à SP3 ;<br> * Windows Phone ;<br> * Autres plates-formes Windows publiées avant 2008.|

## Exigences .NET  

* À partir de `Jenkins 2.238`, .NET Framework 4.0 ou supérieur est requis pour toutes les installations de service Windows et la logique de gestion des services Windows intégrée ; 
* Avant `Jenkins 2.238`, .NET Framework 2.0 a été pris en charge ;
* Pour les plates-formes qui ne prennent pas en charge ces versions, envisagez d'utiliser des exécutables natifs fournis par le projet [Windows Service Wrapper](https://github.com/winsw/winsw).

## Références 

* [Politique du cycle de vie de Microsoft](https://docs.microsoft.com/en-us/lifecycle/ ; 
* Recherche de cycle de vie des produits Microsoft](https://support.microsoft.com/en-us/lifecycle/search).

## Contributions

Si vous souhaitez ajouter la prise en charge d'autres plateformes Windows ou partager vos commentaires, nous apprécions sincèrement vos contributions ! La prise en charge de Windows dans Jenkins est assurée par le groupe d'intérêt spécial [Platform Special Interest Group](https://www.jenkins.io/sigs/platform/), qui dispose d'un chat, d'une liste de diffusion et organise des réunions régulières. Vous êtes les bienvenus sur ces canaux.

## Historique de versions

* 03 juin 2020 - Première version ([discussion dans la liste de diffusion](https://groups.google.com/forum/#!msg/jenkinsci-dev/oK8pBCzPPpo/1Ue1DI4TAQAJ), [notes de réunion de gouvernance](https://docs.google.com/document/d/11Nr8QpqYgBiZjORplL_3Zkwys2qK1vEvK-NYyYa4rzg/edit#heading=h.ele42cjexh55)).
