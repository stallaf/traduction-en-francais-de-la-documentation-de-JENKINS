# Politique d'Assistance Linux

Cette page documente la politique de support Linux pour le contrôleur et les agents Jenkins.

## Portée

Les plugins Jenkins individuels peuvent définir des exigences supplémentaires pour les versions Linux sur les contrôleurs et/ou les agents. Cette page ne documente pas de telles exigences. Reportez-vous à la [documentation du plugin](https://plugins.jenkins.io/) pour des exigences supplémentaires.

## Pourquoi ?

Théoriquement, Jenkins peut fonctionner partout où vous pouvez exécuter une version Java prise en charge, mais, en pratique, il y a certaines limites. Jenkins Core et certains plugins incluent le code natif ou dépendent de l'API et des sous-systèmes Linux, ce qui les rend dépendants de versions Linux spécifiques. Les packages d'installation spécifiques à la plate-forme Jenkins reposent sur des versions Linux spécifiques.

## Niveaux de support

Nous définissons plusieurs niveaux de support pour les plates-formes Linux.

| Niveau de support | Description   | Plateforme |
| ---------------------------- | -------------------- | ------------------ |
| **Niveau 1**  - Soutenu   | Nous effectuons des tests automatisés<br> d'installation du gestionnaire de paquets<br> pour ces plateformes et nous avons<br> l'intention de corriger les problèmes signalés<br> dans les meilleurs délais. Nous recommandons,<br> soit des installations basées sur un<br> gestionnaire de paquets, soit des<br> installations basées sur des conteneurs<br> pour Linux. Les installations peuvent<br> également utiliser `jenkins.war` sans<br> gestionnaire de paquets, bien que nos tests automatisés se concentrent sur les<br> installations avec gestionnaire de paquets et conteneurs.   | * Versions Linux 64 bits (amd64) utilisant le format de paquet Debian, [testées sur ci.jenkins.io](https://ci.jenkins.io/job/Packaging/job/packaging/job/master/) ;<br> * Versions Linux 64 bits (amd64) utilisant le format de paquet Red Hat rpm, [testées sur ci.jenkins.io](https://ci.jenkins.io/job/Packaging/job/packaging/job/master/) ;<br> * Versions Linux 64 bits (amd64) utilisant le format de paquet OpenSUSE rpm, [testées sur ci.jenkins.io](https://ci.jenkins.io/job/Packaging/job/packaging/job/master/) ;<br> * Versions Linux 64 bits (arm64, s390x) utilisant le format de paquet Debian, [testées sur ci.jenkins.io](https://ci.jenkins.io/job/Infra/job/acceptance-tests/) ;<br> * Versions Linux 64 bits (arm64, s390x) utilisant le format de paquetage rpm, [testées sur ci.jenkins.io](https://ci.jenkins.io/job/Infra/job/acceptance-tests/) ;<br> * Images de conteneurs Linux (amd64, arm64, s390x) publiées pour le [contrôleur](https://hub.docker.com/r/jenkins/jenkins) et divers agents.    |
| **Niveau 2** - Correctifs envisagés   | L'assistance peut être soumise à certaines<br> restrictions et exigences supplémentaires.<br> Nous ne testons pas la compatibilité et<br> pouvons interrompre l'assistance à tout<br> moment. Nous prenons en considération les correctifs qui ne compromettent pas<br> l'assistance de niveau 1 et n'entraînent<br> pas de coût de maintenance supplémentaires.  | * Versions Linux 32 bits (x86, ARM) ;<br> * RISC-V et autres architectures non incluses dans le support de niveau 1 ;<br> * Versions de prévisualisation. |
| **Niveau 3** - Non pris en charge | Ces versions sont connues pour être<br> incompatibles ou avoir des limitations graves. Nous ne prenons pas en charge les plateformes répertoriées et nous n'acceptons pas les correctifs. |  * Les versions Linux ne sont plus prises en charge par les fournisseurs de systèmes d'exploitation |

## Références 

* [Support de Debian à long terme](https://wiki.debian.org/LTS) ;
* [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata) ;
* [Le cycle de vie du support des produits OpenSUSUSE](https://en.opensuse.org/Lifetime) ;
* [Cycle de vie Ubuntu et cadence des versions](https://ubuntu.com/about/release-cycle).

## Contributions

N'hésitez pas à proposer des _PR_ (Pull Request) qui ajoutent la prise en charge d'autres plateformes Linux ou à partager vos commentaires ; nous apprécions sincèrement vos contributions ! La prise en charge de Linux dans Jenkins est assurée par le [Platform Special Interest Group](https://www.jenkins.io/sigs/platform/), qui dispose d'un chat, d'un forum et organise des réunions régulières. Vous êtes les bienvenus sur ces canaux.

## Historique des versions 

* Mars 2022 - Première version ([discussion dans la liste de diffusion](https://groups.google.com/g/jenkinsci-dev/c/cYi4GyG7Il8/m/oQ2m0C3UAgAJ), [notes de réunion de gouvernance et enregistrement](https://community.jenkins.io/t/governance-meeting-jan-26-2022/1348)).
