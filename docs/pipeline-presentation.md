# Pipeline

Ce chapitre couvre tous les aspects recommandés de la fonctionnalité du Pipeline Jenkins, y compris comment : 

Commencez avec le pipeline, qui couvre comment définir un pipeline Jenkins (votre pipeline) à travers Blue Ocean, à travers l'interface utilisateur classique ou dans SCM. 

Créez et utilisez un Jenkinsfile, qui couvre les scénarios d'utilisation sur la façon de créer et de construire votre Jenkinsfile. 

Travaillez avec les succursales et les demandes de traction. 

Utilisez Docker avec Pipeline, couvrant comment Jenkins peut invoquer des conteneurs Docker sur les agents / nœuds (à partir d'un JenkinsFile) pour construire vos projets de pipeline. 

Étendre le pipeline avec des bibliothèques partagées. 

Utilisez différents outils de développement pour faciliter la création de votre pipeline. 

Travaillez avec la syntaxe des pipelines, qui fournit une référence complète de toutes les syntaxes déclaratives du pipeline.

Pour un aperçu du contenu dans le manuel de l'utilisateur Jenkins, reportez-vous à la vue d'ensemble du manuel de l'utilisateur.
Qu'est-ce que Jenkins Pipeline?

Jenkins Pipeline (ou simplement "pipeline" avec un "P" en capital) est une suite de plugins qui prend en charge la mise en œuvre et l'intégration de pipelines de livraison continue dans Jenkins.

Un pipeline de livraison continu (CD) est une expression automatisée de votre processus pour obtenir des logiciels de la commande de version jusqu'à vos utilisateurs et clients. Chaque modification de votre logiciel (engagé dans le contrôle des sources) passe par un processus complexe en route pour être publié. Ce processus consiste à construire le logiciel de manière fiable et reproductible, ainsi qu'à progresser le logiciel construit (appelé "build") à travers plusieurs étapes de test et de déploiement.

Le pipeline fournit un ensemble extensible d'outils pour modéliser les pipelines de livraison simples à complex "comme code" via la syntaxe du langage spécifique au domaine du pipeline (DSL). [1]

La définition d'un pipeline Jenkins est écrite dans un fichier texte (appelé JenkinsFile) qui à son tour peut être engagé dans le référentiel de contrôle source d'un projet. [2] Ceci est le fondement de "pipeline en tant que code"; Traiter le pipeline CD comme faisant partie de l'application à verser et examiné comme tout autre code.

La création d'un JenkinsFile et la commettre à la commande des sources offre un certain nombre d'avantages immédiats: 

Crée automatiquement un processus de construction de pipeline pour toutes les branches et les demandes de traction. 

Examen / itération de code sur le pipeline (avec le code source restant). 

AUDIT TRACH pour le pipeline. 

Source unique de vérité [3] pour le pipeline, qui peut être consultée et édité par plusieurs membres du projet.

Bien que la syntaxe de définir un pipeline, soit dans l'interface utilisateur Web, soit avec un Jenkinsfile, est la même, il est généralement considéré comme la meilleure pratique de définir le pipeline dans un JenkinsFile et de vérifier cela au contrôle de la source.
Syntaxe de pipeline déclaratif versus scénarisé

Un JenkinsFile peut être écrit à l'aide de deux types de syntaxe - déclaratifs et scriptés.

Les pipelines déclaratifs et scriptés sont construits fondamentalement différemment. Le pipeline déclaratif est conçu pour faciliter l'écriture et la lecture du code du pipeline, et fournit des fonctionnalités syntaxiques plus riches par rapport à la syntaxe des pipelines scriptée.

Beaucoup de composants syntaxiques individuels (ou "étapes") écrits dans un JenkinsFile, cependant, sont communs au pipeline déclaratif et scripté. En savoir plus sur la façon dont ces deux types de syntaxe diffèrent dans les concepts de pipeline et la vue d'ensemble de la syntaxe des pipelines ci-dessous.
Pourquoi le pipeline?

Jenkins est, fondamentalement, un moteur d'automatisation qui prend en charge un certain nombre de modèles d'automatisation. Pipeline ajoute un ensemble puissant d'outils d'automatisation sur Jenkins, prenant en charge les cas d'utilisation qui s'étendent d'une simple intégration continue aux pipelines CD complets. En modélisant une série de tâches connexes, les utilisateurs peuvent profiter des nombreuses fonctionnalités du pipeline: 

Code: Les pipelines sont implémentés dans le code et généralement vérifiés dans le contrôle de la source, donnant aux équipes la possibilité de modifier, d'examiner et d'itérer sur leur pipeline de livraison. 

Durable: les pipelines peuvent survivre à la fois des redémarrages planifiés et imprévus du contrôleur Jenkins. 

Pausable: les pipelines peuvent éventuellement s'arrêter et attendre l'entrée ou l'approbation humaine avant de poursuivre l'exécution du pipeline. 

Polvalent: les pipelines prennent en charge les exigences complexes du CD du monde réel, y compris la possibilité de fourche / jointure, boucle et effectuent des travaux en parallèle. 

Extensible: le plugin Pipeline prend en charge les extensions personnalisées vers son DSL [1] et plusieurs options d'intégration avec d'autres plugins.

Bien que Jenkins ait toujours permis aux formes rudimentaires de chouchage de travail de freestyle ensemble pour effectuer des tâches séquentielles, [4] Pipeline fait de ce concept un citoyen de première classe à Jenkins.
Quelle est la différence entre le freestyle et le pipeline à Jenkins

S'appuyant sur la valeur de noyau jenkins de l'extensibilité, le pipeline est également extensible à la fois par les utilisateurs avec des bibliothèques partagées de pipeline et par des développeurs de plugin. [5]

L'organigramme ci-dessous est un exemple d'un scénario de CD facilement modélisé dans le pipeline Jenkins:

Flux de pipeline
Concepts de pipeline