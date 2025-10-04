# Securiser Jenkins

Jenkins est utilisé partout, des postes de travail sur les intranets d'entreprise aux serveurs haute performance connectés à l'Internet public. Afin de prendre en charge en toute sécurité cette large gamme de profils de sécurité et de menaces, Jenkins offre de nombreuses options de configuration permettant d'activer, de personnaliser ou de désactiver diverses fonctionnalités de sécurité.

La plupart des options de sécurité sont activées par défaut lors du passage de l'assistant de configuration interactif afin de garantir la sécurité de Jenkins. D'autres impliquent une configuration et des compromis spécifiques à l'environnement et dépendent de cas d'utilisation spécifiques pris en charge dans des instances Jenkins individuelles.

Ce chapitre présente les différentes options de sécurité disponibles pour les administrateurs et les utilisateurs de Jenkins, en expliquant les protections offertes et les compromis liés à la désactivation de certaines d'entre elles.

##Configuration de base

[**Isolation du contrôleur**](./securite-isolation-du-controleur.md)<br>
Les builds ne doivent pas être exécutés sur le nœud intégré, mais ce n'est qu'un début : cette section traite des autres mesures qui peuvent être prises pour protéger le contrôleur contre l'impact de l'exécution des builds.
**Cela doit être configuré en fonction des besoins de votre environnement.**

[**Contrôle d'accès**](./securite-controle-acces.md)<br>
Par défaut, Jenkins n'autorise pas l'accès anonyme et un seul utilisateur administrateur existe. Ce chapitre traite du niveau d'accès fourni par les autorisations et de la manière d'accorder en toute sécurité l'accès à davantage d'utilisateurs.
_Ceci est configuré de manière sécurisée par l'assistant de configuration. Si l'assistant de configuration est désactivé lors du premier lancement, il se peut que cette configuration ne soit pas sécurisée par défaut._

## Comportement de la construction

[**Contrôle d'accès pour les constructions**](./securite-autoriser-les compilations.md)<br>
Découvrez comment restreindre les actions que les constructions individuelles peuvent effectuer dans Jenkins une fois qu'elles sont en cours d'exécution.<br>
**Cela doit être configuré en fonction des besoins de votre environnement.**

[**Sécurisation des constructions**](./securite-securisation-des-compilations.md)<br>
Découvrez comment les constructions peuvent interférer entre elles et avec votre infrastructure, et comment y remédier.<br>
**Cela doit être configuré en fonction des besoins de votre environnement.**

[**Gestion des variables d'environnement**](./securite-variables-environnement.md)
Des scripts de build mal écrits peuvent être amenés à se comporter différemment de ce qui était prévu en raison de noms ou de valeurs de variables d'environnement spéciaux injectés en tant que paramètres de build. Cette section explique comment protéger vos builds.<br>
**Cela doit être configuré en fonction des besoins de votre environnement.**

[**Sécurisation des informations d'identification du dossier d'organisation et du Pipeline multibranches**](./securite-securisation-pipeline-multibranches.md)<br>
Découvrez comment les informations d'identification du dossier d'organisation et du pipeline multibranches peuvent être accessibles à des utilisateurs non privilégiés d'une manière que vous n'aviez pas prévue, et comment y remédier.<br>
**Ceci doit être configuré en fonction des besoins de votre environnement.**

## Interface utilisateur

[**Protection CSRF**](./securite-protection-csrf.md)<br>
Jenkins protège par défaut contre les attaques de type « cross-site request forgery » (CSRF). Ce chapitre explique comment contourner les problèmes que cela peut causer.<br>
_Cette protection est configurée de manière sécurisée par défaut._

[**Formatage du balisage**](./securite-formatage-du-balisage.md)<br>
Le formatage par défaut affiche le texte tel qu'il a été saisi (c'est-à-dire en échappant les métacaractères HTML). Ce chapitre explique comment passer à un autre formatage et ce que les administrateurs doivent savoir.<br>
_Cette fonctionnalité est configurée de manière sécurisée par défaut._

[**Affichage du contenu utilisateur**](./securite-contenu-utilisateur.md)<br>
Par défaut, Jenkins limite strictement les fonctionnalités utilisables dans le contenu utilisateur (fichiers provenant d'espaces de travail, artefacts archivés, etc.) qu'il fournit. Ce chapitre explique comment personnaliser cette fonctionnalité et rendre les rapports HTML et autres contenus similaires à la fois fonctionnels et sûrs à consulter.<br>
_Cette fonctionnalité est configurée de manière sécurisée par défaut._


