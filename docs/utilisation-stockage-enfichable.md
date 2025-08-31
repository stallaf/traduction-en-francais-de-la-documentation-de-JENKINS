# Stockage enfichable

L'approche actuelle consistant à tout stocker dans le système de fichiers est la principale raison pour laquelle Jenkins ne correspond pas au modèle « Cloud Native », avec des fonctionnalités telles que la haute disponibilité, l'absence de temps d'arrêt ou le paiement à l'utilisation. Bien qu'il existe de nombreux plugins qui mettent en œuvre certaines parties de cette vision, cela devient fastidieux à configurer et un cauchemar en termes d'ergonomie pour les utilisateurs, comme l'a souligné [JEP-300](https://github.com/jenkinsci/jep/blob/master/jep/300/README.adoc). En évoluant vers un modèle où les services cloud sont utilisés là où cela est pertinent, la complexité globale de l'exploitation de Jenkins dans un environnement cloud ou conteneurisé est considérablement réduite. D'autres projets connexes tireraient grandement profit du stockage cloud natif pour Jenkins.

Il existe plusieurs domaines évidents où des améliorations sont possibles. L'un des principaux points faibles est l'utilisation du disque local comme stockage polyvalent, ce qui pose des problèmes dans les environnements conteneurisés ou distribués, nécessite des systèmes de fichiers très performants et entraîne toutes les difficultés de configuration telles que le dimensionnement initial et le redimensionnement avec des temps d'arrêt. Nous pensons qu'en utilisant les services fournis par le cloud, il est possible d'améliorer la convivialité, les performances et l'évolutivité globales tout en offrant de nouvelles fonctionnalités très demandées.

Vous pouvez trouver plus d'informations sur le stockage et les priorités enfichables [dans ce blog](https://www.jenkins.io/blog/2018/07/30/introducing-cloud-native-sig/).

### Résumé de l'état

Vous trouverez ci-dessous un résumé des activités en cours et de leur statut actuel :

| Type/Status | Commentaire | Implémentation(s) |
| ------------------ | ---------------------- | ---------------------------- |
| Artefacts<br> (Disponible) | Entièrement livré, avec le support pour le téléchargement des artefacts directement des agents. JEPs connexes : [JEP-202](https://github.com/jenkinsci/jep/blob/master/jep/202/README.adoc). <br><br> [Point d'extension](https://www.jenkins.io/doc/developer/extensions/jenkins-core/#artifactmanagerfactory) | [Artefact Manager sur S3](https://plugins.jenkins.io/artifact-manager-s3), [Azure Artefact Manager](https://plugins.jenkins.io/azure-artifact-manager),  [Plus d'implémentations](https://www.jenkins.io/doc/developer/extensions/jenkins-core/#artifactmanagerfactory) |
| Informations d'identification (Disponible) | Achevé avant l'introduction du processus JEP. <br><br>[Point d'extension](https://www.jenkins.io/doc/developer/extensions/credentials/#credentialsprovider) | [Fournisseur d'identification Kubernetes](https://plugins.jenkins.io/kubernetes-credentials-provider), [Fournisseur d'identification AWS Secrets Manager](https://plugins.jenkins.io/aws-secrets-manager-credentials-provider), [Plus d'implémentations](https://www.jenkins.io/doc/developer/extensions/credentials/#credentialsprovider) |
| Construire des journaux<br> (Aperçu/Pause) |L'API de stockage de journaux Pipeline et les implémentations de référence sont disponibles pour l'aperçu, seuls les types de travaux Pipeline Jenkins sont pris en charge. JEP connexe : [JEP-210](https://github.com/jenkinsci/jep/blob/master/jep/210/README.adoc).<br><br>Les API de base Jenkins et les implémentations de référence n'ont pas encore été fusionnées/publiées, mais des prototypes sont disponibles pour l'évaluation. JEPS connexes : [JEP-207](https://github.com/jenkinsci/jep/blob/master/jep/207/README.adoc), [JEP-212](https://github.com/jenkinsci/jep/blob/master/jep/212/README.adoc) | Journalisation du Pipeline :<br>* [AWS CloudWatch](https://github.com/jenkinsci/pipeline-cloudwatch-logs-plugin) ;<br>* [Elasticsearch](https://github.com/SAP/elasticsearch-logs-plugin).<br><br><br><br>Noyau Jenkins :<br>  * [API de journalisation externe](https://github.com/jenkinsci/external-logging-api-plugin) ;<br>*  [Elasticsearch](https://github.com/jenkinsci/external-logging-elasticsearch-plugin). |
| Journaux système (Disponible) | Jenkins prend en charge les appendeurs de journaux personnalisés à l'aide des [options de configuration](https://jenkov.com/tutorials/java-logging/configuration.html) standard `java.util.logging`. | [Enregistreur Syslog](https://plugins.jenkins.io/syslog-logger), implémentation non Jenkins |
| Journaux de tâches (Non démarré)| Stockage des journaux système et de diverses tâches (par exemple, connexion de l'agent ou requête SCM) | N/A |
| Configurations (En pause) | Largement remplacé par le plugin [Configuration as Code](https://plugins.jenkins.io/configuration-as-code), qui permet de stocker les configurations Jenkins dans SCM ou d'autres emplacements non liés à une base de données.<br><br>JEP associés : [JEP-213](https://github.com/jenkinsci/jep/blob/master/jep/213/README.adoc) | N/A |
|Résultats des tests (Disponible) | L'API est disponible dans le [Plugin JUnit](https://plugins.jenkins.io/junit). | [Junit SQL Storage](https://plugins.jenkins.io/junit-sql-storage) |
| Résultats de couverture de code (Non démarré) | Prévu uniquement pour les plugins basés sur l'API de couverture de code, qui unifie l'implémentation du stockage. Voir Exécutions pour les autres types de rapports de couverture. | N/A |
| Builds/Exécutions (Non démarré) | Stockage des enregistrements de build complets dans une base de données externe. Comprend le stockage des données de build non répertoriées ailleurs (telles que les journaux ou les résultats de test). | N/A |
| Tâches (Non démarré) | Stockage des configurations de tâches et des métadonnées spécifiques aux tâches dans une base de données externe. Les plugins existants tels que Jenkins Pipeline et JobDSL traitent partiellement ce cas en conservant les configurations dans SCM. | N/A |
| Empreintes digitales (Aperçu) | Jenkins 2.252+ inclut l'API de stockage externe d'empreintes digitales qui peut être utilisée par les plugins. Plus d'informations : [Page du projet GSoC](https://www.jenkins.io/projects/gsoc/2020/projects/external-fingerprint-storage/)<br><br>JEP connexes : [JEP-226](https://github.com/jenkinsci/jep/blob/master/jep/226/README.adoc) | [Redis](https://plugins.jenkins.io/redis-fingerprint-storage), [PostgreSQL (non publié)](https://github.com/jenkinsci/postgresql-fingerprint-storage-plugin). | 
| Espaces de travail (Non démarré) | Proposé comme projet GSoC 2019 : [Fonctionnalités cloud pour le plugin External Workspace Manager](https://www.jenkins.io/projects/gsoc/2019/project-ideas/ext-workspace-manager-cloud-features/) | N/A |

La liste ci-dessus n'est pas complète. D'autres types de stockage peuvent être pris en compte en fonction des commentaires. Vous pouvez trouver plus d'informations sur le stockage et les priorités enfichables [dans ce blog](https://www.jenkins.io/blog/2018/07/30/introducing-cloud-native-sig/).

### Stockage d'artefact

Il existe de nombreux plugins existants permettant de télécharger des artefacts à partir de stockage externe (par exemple S3, Artifactory, Publish sur SFTP, etc., etc.), mais il n'y a pas de plugins qui peuvent le faire de manière transparente sans utiliser de nouvelles étapes. Dans de nombreux cas, les artefacts sont également téléchargés via le contrôleur Jenkins, et il augmente la charge sur le système. Ce serait formidable s'il y avait une couche qui permettrait de stocker des artefacts à l'extérieur lors de l'utilisation d'étapes communes comme des artefacts d'archives.

Jenkins 2.118+ propose une API `jenkins.util.VirtualFile` qui permet de mettre en œuvre des gestionnaires d'artefacts externes à l'aide du point d'extension [ArtifactManagerFactory](https://www.jenkins.io/doc/developer/extensions/jenkins-core/#artifactmanagerfactory).

Exemple(s) d'implémentation : 

* [Artefact Manager pour S3](https://plugins.jenkins.io/artifact-manager-s3).

JEPs connexes : 

* [JEP-202: stockage d'artefact externe](https://github.com/jenkinsci/jep/blob/master/jep/202/README.adoc).

### Créer un stockage de journaux

L'utilisation du disque pour les journaux entraîne les mêmes problèmes que ceux mentionnés précédemment pour les artefacts. De plus, un stockage de journaux externe tel que [AWS CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) permet de bénéficier de fonctionnalités très demandées telles que la gestion centralisée des journaux, les politiques de conservation des journaux, les requêtes avancées, etc. Il existe déjà des options permettant d'envoyer les journaux vers un backend externe, ou des plugins qui le font, comme [aws-cloudwatch-logs-publisher-plugin](https://github.com/jenkinsci/aws-cloudwatch-logs-publisher-plugin), mais il n'existe pas de moyen intégré pour à la fois envoyer et afficher les journaux à partir d'un stockage externe. Le travail sur le stockage externe des journaux est suivi sous la référence [JENKINS-38313](https://issues.jenkins.io/browse/JENKINS-38313).

Implémentatio (s) de référence : 

* [Pipeline Logging sur Fluentd + CloudWatch Plugin](https://github.com/jenkinsci/pipeline-log-fluentd-cloudwatch-plugin) ;
* [Journalisation Externe pour Elasticsearch](https://github.com/jenkinsci/external-logging-elasticsearch-plugin).

JEPs connexes : 

* [JEP-207: Support de journalisation de construction externe dans le noyau Jenkins](https://github.com/jenkinsci/jep/blob/master/jep/207/README.adoc) ;
* [JEP-210: stockage de journaux externes pour le pipeline](https://github.com/jenkinsci/jep/blob/master/jep/210/README.adoc) ;
* [JEP-212: plugin API de journalisation externe](https://github.com/jenkinsci/jep/blob/master/jep/212/README.adoc) ;
* [JEP-206: Utilisez UTF-8 pour les journaux de construction de pipelines](https://github.com/jenkinsci/jep/blob/master/jep/206/README.adoc).

### Stockage de configuration

Bien que les configurations ne soient pas grandes, les externaliser est une tâche critique pour obtenir des contrôleurs Jenkins hautement disponibles ou jetables. Il existe de nombreuses façons de stocker des configurations dans Jenkins, mais 95% des cas sont couverts par la couche `XmlFile` qui sérialise les objets au disque et les lit en utilisant la bibliothèque Xstream. L'extériorisation des `XmlFiles` serait un excellent pas en avant.

Il existe plusieurs prototypes pour les configurations d'externalisation, par exemple en DotCI. Il existe également d'autres implémentations qui pourraient être intégrées au noyau Jenkins.

JEPS connexes : 

* [JEP-213: API de stockage de configuration dans le noyau Jenkins](https://github.com/jenkinsci/jep/blob/master/jep/213/README.adoc).

### Informations d'identification

Dans [Credentials Plugin](https://plugins.jenkins.io/credentials) 1.15+, il existe un point d'extension [CredentialsProvider](https://www.jenkins.io/doc/developer/extensions/credentials/#credentialsprovider) qui permet de référencer et de résoudre des informations d'identification externes. Ce moteur permet de mettre en œuvre des informations d'identification externes pour les plugins implémentant l'API Credentials.

Exempl (s) d'implémentation : 

* [Fournisseur d'identification Kubernetes](https://plugins.jenkins.io/kubernetes-credentials-provider).

D'autres API d'identification dans Jenkins (comme Jenkinsdoc: Hudson.util.secret) ne sont pas prises en charge.

### Résultats des tests

Dans les cas d'utilisation CI/CD courants, une grande partie de l'espace est consommée par les rapports de test. Ces données sont stockées dans `JENKINS_HOME`, et le format de stockage actuel nécessite d'énormes coûts lors de la récupération des statistiques et, en particulier, des tendances. Pour afficher les tendances, chaque rapport doit être chargé puis traité en mémoire.

L'objectif principal de l'externalisation des résultats des tests est d'optimiser la logique de Jenkins en interrogeant les données souhaitées à partir de stockages externes spécialisés, par exemple à partir de bases de données basées sur des documents comme Elasticsearch. Selon le plan actuel, le [Plugin JUnit](https://plugins.jenkins.io/junit) sera étendu afin de prendre en charge un tel stockage externe dans ses API largement utilisés par les plugins de rapport de test.

Statut : 

* Une implémentation SQL est disponible dans le [plugin Junit SQL Storage](https://plugins.jenkins.io/junit-sql-storage/).

Veuillez l'essayer, signalez les problèmes à [GitHub](https://github.com/jenkinsci/junit-plugin/issues) et les commentaires généraux à [GitHub#142](https://github.com/jenkinsci/junit-plugin/issues/142).

### Empreintes digitales

Les empreintes digitales sont stockées dans `JENKINS_HOME ` dans une base de données locale au format XML. L'externalisation des empreintes digitales réduit la dépendance de Jenkins au stockage physique sur disque du contrôleur et permet de configurer des stockages cloud qui peuvent être moins coûteux et plus fiables. Un autre avantage est qu'elle permettrait de tracer les empreintes digitales sur les contrôleurs Jenkins et l'ensemble du flux CI/CD.

Statut : 

* En cours  ;
* JEP connexe:  [JEP-226 : Stockage d'empreintes digitales externe](https://github.com/jenkinsci/jep/blob/master/jep/226/README.adoc) ;
* [API prototype](https://github.com/jenkinsci/jenkins/pull/4731) ; 
* Implémentation de référence:  [Plugin de stockage d'empreintes digitales Redis](https://github.com/jenkinsci/redis-fingerprint-storage-plugin).
