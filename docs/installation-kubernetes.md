# Kubernetes

Kubernetes (K8S) est un système open source pour l'automatisation du déploiement, de la mise à l'échelle et de la gestion des applications conteneurisées.

Un cluster Kubernetes ajoute une nouvelle couche d'automatisation à Jenkins. Kubernetes s'assure que les ressources sont utilisées efficacement et que vos serveurs et votre infrastructure sous-jacente ne sont pas surchargés. La capacité de Kubernetes à orchestrer le déploiement des conteneurs garantit que Jenkins a toujours la bonne quantité de ressources disponibles.

L'hébergement de Jenkins sur un cluster Kubernetes est bénéfique pour les déploiements basés sur Kubernetes et les agents Jenkins évolutifs basés sur un conteneur dynamique.

Plusieurs stratégies pour une telle configuration et maintenance sont explorées ci-dessous, notamment : 

* Utilisation directe des fichiers `kubectl` et YAML pour configurer certains aspects du déploiement. 
* Utilisation de l'outil `helm` et des fichiers Helm Chart pour gérer un écosystème complet. Par exemple, le contrôleur et une population d'agents où le travail réel se produit. 
* Utilisation supplémentaire de l'opérateur Jenkins pour gérer les opérations de Jenkins sur Kubernetes dans le cluster.

<hr>

Ici, nous voyons un processus étape par étape pour configurer Jenkins sur un cluster Kubernetes.

## Configuration de jenkins sur Kubernetes

Pour configurer un cluster Jenkins sur Kubernetes, nous ferons ce qui suit : 

1. [Créer un espace de noms](#espace-de-noms) ; 
2. [Créez un compte de service](#compte-de-services) avec les autorisations d'administration Kubernetes ;
3. [Créez un volume persistant local](#volume-local-persistant) pour les données persistantes de Jenkins sur les redémarrages POD ; 
4. [Créez un déploiement YAML](#deploiement-yaml) et déployez-le ; 
5. [Créez un service YAML](#service-yaml) et déployez-le. 

!!! info

    Ce guide n'utilise pas de volume persistant local car il s'agit d'un guide générique. Pour utiliser un volume persistant pour vos données Jenkins, vous devez créer des volumes de cloud ou de centre de données sur prématrices et les configurer.

### Fichiers de manifeste Jenkins Kubernetes

Tous les fichiers manifestes Jenkins Kubernetes utilisés ici sont hébergés sur GitHub. Veuillez cloner le référentiel si vous avez du mal à copier le manifeste du document.

``` bash title="BASH"
git clone https://github.com/scriptcamp/kubernetes-jenkins
```

Utilisez les fichiers GitHub pour référence et suivez les étapes des sections suivantes.

### Déploiement de Kubernetes Jenkins

Commençons par le déploiement de Jenkins sur Kubernetes.

**Étape 1 :** Créez un espace de noms pour Jenkins. Il est bon de classer tous les outils DevOps en tant qu'espace de noms distinct des autres applications.

``` bash title="BASH"
kubectl create namespace devops-tools
```

** Étape 2 :** Créez un fichier «jenkins-01-serviceAccount.yaml» et copiez le manifeste du compte admin service suivant.

``` yaml title="YAML"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-admin
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["*"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin
  namespace: devops-tools
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins-admin
subjects:
- kind: ServiceAccount
  name: jenkins-admin
  namespace: devops-tools
```

Le fichier «jenkins-01-serviceAccount.yaml» crée un rôle de cluster « jenkins-admin », un compte de service « jenkins-admin » et lie le rôle de cluster au compte de service.

Le rôle de cluster «jenkins-admin» a toutes les autorisations pour gérer les composants du cluster. Vous pouvez également restreindre l'accès en spécifiant les actions de ressources individuelles.

Créez maintenant le compte de service à l'aide de kubectl.

``` bash title="BASH"
kubectl apply -f jenkins-01-serviceAccount.yaml
```

**Étape 3 :** Créez «jenkins-02-volume.yaml» et copiez le manifeste du volume persistant suivant :

``` yaml title="YAML"
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv-volume
  labels:
    type: local
spec:
  storageClassName: local-storage
  claimRef:
    name: jenkins-pv-claim
    namespace: devops-tools
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  local:
    path: /mnt
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - worker-node01
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pv-claim
  namespace: devops-tools
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

**Remarque importante :** Remplacez «worker-node01» par l'un des noms d'hôte de vos nœuds de travail du cluster.

Vous pouvez obtenir le nom d'hôte du nœud de travail en utilisant le kubectl.

``` bash title="BASH"
kubectl get nodes
```

Pour le volume, nous utilisons la classe de stockage «locale» à des fins de démonstration. Ce qui signifie qu'il crée un volume «_PersistentVolume_» dans un nœud spécifique sous l'emplacement «/mnt».

Comme la classe de stockage «locale» nécessite le sélecteur de nœud, vous devez spécifier correctement le nom du nœud de travail pour que le pod Jenkins soit planifié dans le nœud spécifique.

Si le pod est supprimé ou redémarré, les données seront persistées dans le volume du nœud. Cependant, si le nœud est supprimé, vous perdrez toutes les données.

Idéalement, vous devez utiliser un volume persistant en utilisant la classe de stockage disponible avec le fournisseur de cloud, ou celle fournie par l'administrateur de cluster pour persister des données sur les défaillances du nœud.

Créons le volume à l'aide de kubectl

``` bash title="BASH"
kubectl create -f jenkins-02-volume.yaml
```

**Étape 4 :** Créez un fichier de déploiement nommé «jenkins-03-deployment.yam» et copiez le manifeste de déploiement suivant :

``` yaml title="YAML"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: devops-tools
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins-server
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: jenkins-server
    spec:
      securityContext:
            # Note: fsGroup may be customized for a bit of better
            # filesystem security on the shared host
            fsGroup: 1000
            runAsUser: 1000
            ### runAsGroup: 1000
      serviceAccountName: jenkins-admin
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts
          # OPTIONAL: check for new floating-tag LTS releases whenever the pod is restarted:
          imagePullPolicy: Always
          resources:
            limits:
              memory: "2Gi"
              cpu: "1000m"
            requests:
              memory: "500Mi"
              cpu: "500m"
          ports:
            - name: httpport
              containerPort: 8080
            - name: jnlpport
              containerPort: 50000
          livenessProbe:
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 90
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 5
          readinessProbe:
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          volumeMounts:
            - name: jenkins-data
              mountPath: /var/jenkins_home
      volumes:
        - name: jenkins-data
          persistentVolumeClaim:
              claimName: jenkins-pv-claim
```

Dans ce déploiement de Jenkins Kubernetes, nous avons utilisé ce qui suit : 

1. La version Jenkins Container est référencée dans l'exemple YAML ci-dessus par une balise flottante `lts`, de sorte que la ligne en option  `imagePullPolicy: Always`  est ajoutée pour tirer et déployer une nouvelle version d'image LTS (si elle existe) chaque fois que le pod est redémarré. Sinon, la version actuellement déployée est utilisée après le redémarrage (et il n'y a aucun autre moyen de dire à Kubernetes de tirer une image plus récente, à moins de localiser et de supprimer l'ancienne du cache du nœud). 
    * Notez que "chaque fois que le pod est redémarré" comprend des événements tels qu'un redémarrage de serveur ou le redémarrage demandé dans Web-UI pour appliquer les mises à jour du plugin téléchargées. 
    * Kubernetes utilise automatiquement cette stratégie par défaut si l'image  `: latest` est spécifiée, ou aucune n'est spécifiée (alors l'image  `: latest` l'est automatiquement par défaut). 
    * Pour les mises à jour prévisibles et déterministes, utilisez plutôt les balises d'image basées sur des versions. Recherchez [hub.docker.com/r/jenkins/jenkins/tags](https://hub.docker.com/r/jenkins/jenkins/tags) pour ceux actuellement servis. 
    * Considérez une balise d'image comme `lts-jdk21` pour éviter les surprises lorsque le noyau Jenkins passe à l'utilisation d'un JDK plus récent (ils sont généralement rétrocompatibles, mais…) ou restez avec `lts` pour une approche de maintenance sans intervention indéfinie.
2. Le `spec:/strategy:/type: Recreate ` nécessite que Kubernetes arrête d'abord l'ancienne instance de conteneur, puis lance une nouvelle quelques secondes plus tard, chaque fois qu'un redémarrage est demandé, comme tirer une nouvelle version de Jenkins. 
3. «SecurityContext» pour que Jenkins Pod puisse écrire au volume persistant local. 
    * Notez que `fsGroup` spécifie l'ID de groupe utilisé pour l'accès au système de fichiers monté, tandis que `runAsUser` et `runAsGroup` spécifient les ID de compte sous lesquels tous les processus s'exécutent (tels qu'ils sont vus par le système d'exploitation hôte et l'environnement d'exploitation invité du conteneur). Il n'existe pas d'équivalent à `fsUser`.
    * Une mise en garde toutefois : le conteneur Jenkins a l'ID 1000 défini dans ses fichiers `/etc/passwd` et `/etc/group`, et le programme `git`, par exemple, exige que l'identifiant de l'utilisateur sous lequel il s'exécute soit résolvable. Bien qu'il soit trivial de définir les comptes souhaités dans un conteneur dérivé (créé `FROM jenkins:lts` par exemple) et de les exécuter en tant que comptes selon les préférences de votre site, cela reste peu pratique dans la configuration par défaut.
4. Sonde de vivacité et de disponibilité pour surveiller l'état de santé du pod Jenkins.
5. Volume persistant local basé sur une classe de stockage locale qui contient le chemin de données Jenkins '/var/jenkins_home'. Vous devrez peut-être le préparer avec : 
    ``` bash title="BASH"
    runAsUser=1000
    fsGroup=1000   # Or custom ID, per above
    mkdir -p /var/jenkins_home
    chown -R $runAsUser:$fsGroup /var/jenkins_home
    chmod -R g+rwX /var/jenkins_home
    ```

!!! info
    
    Le fichier de déploiement utilise le volume persistant de la classe de stockage locale pour les données Jenkins. Pour les cas d'utilisation de production, vous devez ajouter un volume persistant de la classe de stockage spécifique au cloud pour vos données Jenkins.

Si vous ne voulez pas que le volume persistant de stockage local, vous pouvez remplacer la définition de volume dans le déploiement par le répertoire hôte comme indiqué ci-dessous.

``` yaml title="YAML"
volumes:
- name: jenkins-data
  emptyDir: \{}
```

Créez le déploiement à l'aide de kubectl.

``` bash title="BASH"
kubectl apply -f jenkins-03-deployment.yaml
```

Vérifiez l'état de déploiement.

``` bash title="BASH"
kubectl get deployments -n devops-tools
```

Maintenant, vous pouvez obtenir les détails de déploiement en utilisant la commande suivante :

``` bash title="BASH"
kubectl describe deployments --namespace=devops-tools
```

### Accéder à Jenkins à l'aide du service Kubernetes

Nous avons maintenant créé un déploiement. Cependant, il n'est pas accessible depuis l'extérieur. Pour accéder au déploiement Jenkins depuis l'extérieur, nous devons créer un service et le mapper au déploiement.

Créez «jenkins-04-service.yaml» et copiez le manifeste de service suivant :

``` yaml title="YAML"
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  namespace: devops-tools
  annotations:
      prometheus.io/scrape: 'true'
      prometheus.io/path:   /
      prometheus.io/port:   '8080'
spec:
  selector:
    app: jenkins-server
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32000
```

!!! info

    Ici, nous utilisons le type « NodePort », qui exposera Jenkins sur toutes les adresses IP des nœuds Kubernetes sur le port 32000. Si vous disposez d'une configuration d'entrée, vous pouvez créer une règle d'entrée pour accéder à Jenkins. Vous pouvez également exposer le service Jenkins en tant que Loadbalancer si vous exécutez le cluster sur le cloud AWS, Google ou Azure.

Créez le service Jenkins à l'aide de kubectl.

``` bash title="BASH"
kubectl apply -f jenkins-04-service.yaml
```

Maintenant, lorsque vous parcourez l'une des IP de nœud sur le port 32000, vous pourrez accéder au tableau de bord Jenkins.

``` cfg
http: // <node-ip>: 32000
```

Jenkins demandera le mot de passe d'administration initial lorsque vous accédez au tableau de bord pour la première fois.

Vous pouvez obtenir ces informations à partir des journaux des pods, soit depuis le tableau de bord Kubernetes, soit depuis l'interface CLI. Vous pouvez obtenir les détails des pods à l'aide de la commande CLI suivante :

``` bash title="BASH"
kubectl get pods --namespace=devops-tools
```

Avec le nom du pod, vous pouvez obtenir les journaux comme indiqué ci-dessous. Remplacez le nom de pod par votre nom de pod.

``` bash title="BASH"
kubectl logs jenkins-deployment-2539456353-j00w5 --namespace=devops-tools
```

Le mot de passe peut être trouvé à la fin du journal.

!!! info

    Vous pouvez regarder les journaux de serveur Jenkins (publiés sur `stdout` ou `stderr` du JVM et collectés par Kubernetes) avec la boucle suivante pour gérer les redémarrages de Jenkins :
    ``` bash
    while sleep 0.1 ; do kubectl logs -f -l app=jenkins-server -n devops-tools ; done &
    ```
    (Le label `jenkins-server` est défini dans `jenkins-03-deployment.yaml`)
    Alternativement, l'outil [stern](https://github.com/stern/stern) peut être utilisé pour regarder les journaux de plusieurs pods.

Vous pouvez aussi exécuter la commande exec pour obtenir le mot de passe directement à partir de l'emplacement comme indiqué ci-dessous. 

* En utilisant la première instance (normalement uniquement) du pod d'application : 
    ``` bash title="BASH"
    kubectl exec -it "deployment.apps/jenkins" cat /var/jenkins_home/secrets/initialAdminPassword -n devops-tools
    ```
*   … ou, en utilisant une instance de conteneur spécifique :
    ``` bash title="BASH"
    kubectl exec -it jenkins-559d8cd85c-cfcgk cat /var/jenkins_home/secrets/initialAdminPassword -n devops-tools
    ```

Une fois le mot de passe saisi, procédez à l'installation des plugins suggérés et créez un utilisateur administrateur. Toutes ces étapes sont clairement expliquées dans le tableau de bord Jenkins.

Ci-dessous, nous explorerons d'autres stratégies de déploiement.

## Installez Jenkins avec Helm V3

Un déploiement Jenkins typique se compose d'un nœud contrôleur et, éventuellement, d'un ou plusieurs agents. Pour simplifier le déploiement de Jenkins, nous utiliserons [Helm](https://helm.sh/). Helm est un gestionnaire de paquets pour Kubernetes et son format de paquet est appelé « chart ». De nombreux charts développés par la communauté sont disponibles sur [GitHub](https://github.com/helm/charts).

Les charts Helm permettent de déployer et de supprimer des applications d'un simple clic, ce qui facilite l'adoption et le développement d'applications Kubernetes pour ceux qui ont peu d'expérience en matière de conteneurs ou de microservices.

### Prérequis

**Interface de ligne de commande Helm**

Si vous n'avez pas installé et configuré l'interface de ligne de commande Helm localement, consultez les sections ci-dessous pour [installer](#installer-h,elm) et [configurer Helm](#configurer-helm).

### Installer Helm

Pour installer l'interface CLI Helm, suivez les instructions de la page [Installation de Helm](https://helm.sh/docs/intro/install/).

### Configurer Helm

Une fois Helm installé et correctement configuré, ajoutez le référentiel Jenkins comme suit :

``` bash title="BASH"
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
```

Les charts Helm dans le référentiel Jenkins peuvent être répertoriés à l'aide de la commande :

``` bash title="BASH"
helm search repo jenkinsci
```

### Créer un volume persistant

Nous voulons créer un [volume persistant](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) pour notre pod de contrôleur Jenkins. Cela nous évitera de perdre toute la configuration du contrôleur Jenkins et nos tâches lorsque nous redémarrons notre minikube. Ce [document officiel minikube](https://minikube.sigs.k8s.io/docs/handbook/persistent_volumes/) explique quels répertoires nous pouvons utiliser pour monter nos données. Dans un cluster Kubernetes multi-nœuds, vous aurez besoin d'une solution comme NFS pour rendre le répertoire mount disponible dans l'ensemble du cluster. Mais parce que nous utilisons minikube qui est un cluster à un nœud, nous n'avons pas à nous en soucier.

Nous choisissons d'utiliser le répertoire `/data`. Ce répertoire contiendra notre configuration de contrôleur Jenkins.

**Nous allons créer un volume appelé Jenkins-PV :**

1. Collez le contenu de [https://raw.githubusercontent.com/installing-jenkins-on-kubernetes/jenkins-01-volmume.yaml](https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkins-01-volume.yaml) dans un fichier formaté YAML appelé `jenkins-01-volume.yaml`. 
2. Exécutez la commande suivante pour appliquer la spécification : 

    ``` bash title="BASH"
    kubectl apply -f jenkins-01-volume.yaml
    ```

!!! info

    Il convient de noter que, dans la spécification ci-dessus, hostPath utilise le /data/ jenkins-volume/ de votre nœud pour imiter le stockage attaché au réseau. Cette approche n'est adaptée qu'à des fins de développement et de test. Pour la production, vous devez fournir une ressource réseau telle qu'un disque persistant Google Compute Engine ou un volume Amazon Elastic Block Store.

!!! info 

    Minikube configuré pour hostPath définit les autorisations sur /data uniquement pour le compte root. Une fois le volume créé, vous devrez modifier manuellement les autorisations pour permettre au compte jenkins d'écrire ses données.

``` bash title="BASH"
minikube ssh
sudo chown -R 1000:1000 /data/jenkins-volume
```

### Créer un compte de service

Dans Kubernetes, les comptes de services sont utilisés pour fournir une identité pour les pods. Les pods qui souhaitent interagir avec le serveur API s'authentifieront avec un compte de service particulier. Par défaut, les applications s'authentifieront comme le compte de service `default` dans l'espace de noms dans lequel ils s'exécutent. Cela signifie, par exemple, qu'une application exécutée dans l'espace de noms `test` utilisera le compte de service par défaut de l'espace de noms `test`.

**Nous créerons un compte de service appelé Jenkins :**

Un ClusterRole est un ensemble d'autorisations qui peuvent être attribuées à des ressources au sein d'un cluster donné. Les API Kubernetes sont classées en groupes d'API, en fonction des objets API auxquels elles se rapportent. Lors de la création d'un ClusterRole, vous pouvez spécifier les opérations qui peuvent être effectuées par le ClusterRole sur un ou plusieurs objets API dans un ou plusieurs groupes d'API, comme nous l'avons fait ci-dessus. Les ClusterRoles ont plusieurs utilisations. Vous pouvez utiliser un ClusterRole pour :

* définir les autorisations sur les ressources avec espace de noms et les accorder au sein d'espaces de noms individuels ;
* définir les autorisations sur les ressources avec espace de noms et les accorder à tous les espaces de noms ;
* définir les autorisations sur les ressources au niveau du cluster.

Si vous souhaitez définir un rôle à l'échelle du cluster, utilisez un ClusterRole ; si vous souhaitez définir un rôle dans un espace de noms, utilisez un Rôle.

Un rôle de liaison accorde les autorisations définies dans un rôle à un utilisateur ou à un ensemble d'utilisateurs. Il détient une liste de sujets (utilisateurs, groupes ou comptes de services) et une référence au rôle accordé.

Une création de Rôle peut faire référence à tout Rôle dans le même espace de noms. Alternativement, une création de Rôle peut faire référence à un ClusterRole et lier ce ClusterRole à l'espace de noms de la création de Rôle. Pour lier une ClusterRole à toutes les espaces de noms de notre cluster, nous utilisons un ClusterRoleBinding. 

1. Collez le contenu de [https://raw.githubusercontent.com/installing-jenkins-on-kubernetes/jenkins-02-sa.yaml](https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkins-02-sa.yaml) dans un fichier formaté YAML appelé `jenkins-02-sa.yaml`. 

2. Exécutez la commande suivante pour appliquer la spécification : 

    ``` bash title="BASH"
    kubectl apply -f jenkins-02-sa.yaml
    ```

### Installer Jenkins

Nous déploierons Jenkins, y compris le plugin Jenkins Kubernetes. Voir le [tableau officiel](https://github.com/jenkinsci/helm-charts/tree/main/charts/jenkins) pour plus de détails. 

1. Pour activer la persistance, nous allons créer un fichier de remplacement et le transmettre en tant qu'argument à l'interface CLI Helm. Collez le contenu de raw.githubusercontent.com/jenkinsci/helm-charts/main/charts/jenkins/values.yaml dans un fichier au format YAML appelé `jenkins-values.yaml`.
Le  jenkins-values.yaml est utilisé comme modèle pour fournir des valeurs nécessaires à la configuration. 

2. Ouvrez le fichier `jenkins-values.yaml` dans votre éditeur de texte préféré et modifiez ce qui suit : 
    * nodePort: Parce que nous utilisons minikube, nous devons utiliser NodePort comme type de service. Seuls les fournisseurs de cloud proposent des équilibreurs de charge. Nous définissons le port 32000 comme port. 
    * storageClass: 
    ``` yaml title="YAML"
    storageClass: jenkins-pv
    ```
    * serviceAccount: La section serviceAccount du fichier `jenkins-values.yaml` devrait ressembler à ceci :
    ``` yaml
    serviceAccount:
    create: false
    # Service account name is autogenerated by default
    name: jenkins
    annotations: {}
    ```
    Où « name: jenkins » fait référence au serviceAccount créé pour jenkins.
    * Nous pouvons également définir les plugins que nous souhaitons installer sur nos Jenkins. Nous utilisons certains plugins par défaut comme git et le plugin pipeline.

3. Vous pouvez désormais installer Jenkins en exécutant la commande `helm install` et en lui passant les arguments suivants : 
    * Le nom de la version `jenkins` ; 
    * Le drapeau `-f` avec le fichier YAML qui rnemplace `jenkins-values.yaml` ; 
    * Le nom du chart `jenkinsci/jenkins` ; 
    * Le drapeau `-n` avec le nom de votre espace de noms `jenkins` 
    ``` bash title="BASH"
    chart=jenkinsci/jenkins
    helm install jenkins -n jenkins -f jenkins-values.yaml $chart
    ```
    Cela produit quelque chose de similaire à ce qui suit :
    ``` bash title="BASH"
    NAME: jenkins
    LAST DEPLOYED: Wed Sep 16 11:13:10 2020
    NAMESPACE: jenkins
    STATUS: deployed
    REVISION: 1
    ```

4. Obtenez votre mot de passe utilisateur «administrateur» en exécutant :

    ``` bash title="BASH"
    jsonpath="{.data.jenkins-admin-password}"
    secret=$(kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath)
    echo $(echo $secret | base64 --decode)
    ```

5. Obtenez l'URL Jenkins à visiter en exécutant ces commandes dans le même shell :

    ``` bash title="BASH"
    jsonpath="{.spec.ports[0].nodePort}"
    NODE_PORT=$(kubectl get -n jenkins -o jsonpath=$jsonpath services jenkins)
    jsonpath="{.items[0].status.addresses[0].address}"
    NODE_IP=$(kubectl get nodes -n jenkins -o jsonpath=$jsonpath)
    echo http://$NODE_IP:$NODE_PORT/login
    ```

6. Connectez-vous avec le mot de passe de l'étape 1 et du nom d'utilisateur : admin 

7. Utilisez la configuration de Jenkins comme code en spécifiant les configurations dans votre adaptation du fichier values.yaml. Voir la [configuration comme documentation de code](https://plugins.jenkins.io/configuration-as-code) et [exemples](https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos).

Visitez la [page Jenkins on Kubernetes Solutions](https://cloud.google.com/solutions/jenkins-on-container-engine) pour plus d'informations sur l'exécution de Jenkins sur Kubernetes. Visitez la [configuration Jenkins en tant que projet de code](https://www.jenkins.io/projects/jcasc/) pour plus d'informations sur la configuration en tant que code. Selon votre environnement, le démarrage de Jenkins peut prendre un certain temps. Entrez la commande suivante pour vérifier l'état de votre Pod :

``` bash title="BASG"
kubectl get pods -n jenkins
```

Une fois Jenkins installé, l'état doit être défini sur _Running_ comme dans la sortie suivante :

``` bash title="BASH"
kubectl get pods -n jenkins
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-645fbf58d6-6xfvj   1/1     Running   0          2m
```

1. Pour accéder à votre serveur Jenkins, vous devez récupérer le mot de passe. Vous pouvez récupérer votre mot de passe à l'aide des deux options ci-dessous. 

    **Option 1**

    Exécutez la commande suivante : 

    ``` bash title="BASH"
    jsonpath="{.data.jenkins-admin-password}"
    secret=$(kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath)
    echo $(echo $secret | base64 --decode)
    ```
    
    La sortie doit ressembler à ceci :

    ``` console title="CONSOLE"
    Um1kJLOWQY
    ```

    !!! info 

        👆🏻Veuillez noter que votre mot de passe sera différent.


    **Option 2**

    Exécutez la commande suivante :

    ```bash title="BASH"
    jsonpath="{.data.jenkins-admin-password}"
    kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath
    ```

    La sortie doit être une **chaîne codée Base64** comme ceci :

    ``` console title="CONSOLE"
    WkIwRkdnbDZYZg==
    ```

    Décodez la chaîne Base64 et vous avez votre mot de passe. Vous pouvez utiliser [ce site Web](https://www.base64decode.org/) pour décoder votre sortie.

2. Obtenez le nom du Pod qui exécute Jenkins à l'aide de la commande suivante :

    ``` bash title="BASH"
    kubectl get gods -n jenkins
    ```

3. Utilisez la commande kubectl pour configurer le transfert de port:

    ``` bash title="BASH"
    kubectl -n jenkins port-forward <pod_name> 8080:8080
    Forwarding from 127.0.0.1:8080 -> 8080
    Forwarding from [::1]:8080 -> 8080
    ```

Visitez [127.0.0.1:8080/](http://127.0.0.1:8080/) et connectez-vous en tant que `admin` comme nom d'utilisateur et le mot de passe que vous avez récupéré plus tôt.

## Installez Jenkins avec des fichiers YAML

Cette section décrit comment utiliser un ensemble de fichiers YAML (_Yet Another Markup Language)_) pour installer Jenkins sur un cluster Kubernetes. Les fichiers YAML sont facilement suivis, édités et peuvent être réutilisés indéfiniment.

### Créer un fichier de déploiement Jenkins

Copiez le contenu [ici](https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkinsHelm-03-deployment.yaml) dans votre éditeur de texte préféré et créez un fichier jenkinsHelm-03-deployment.yaml dans l'espace de noms «jenkins» que nous avons créé dans la section ci-dessus. 

* Ce [fichier définit un déploiement](https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkinsHelm-03-deployment.yaml) comme indiqué par le champ `king` ;
* Le déploiement spécifie une seule réplique. Cela garantit qu'une seule et une seule instance seulement sera maintenue par le contrôleur de réplication en cas d'échec ;
* Le nom de l'image du conteneur est `jenkins` et la version est une balise flottante `lts-jdk21 ` (pour le déterminisme, vous voudrez peut-être une balise de version spécifique comme `2.32.2` à la place - mais vous devrez alors la mettre à jour et la réappliquer avec les itérations de ce fichier) ; Notes que vous devrez peut-être configurer ` imagePullPolicy: Always` pour tirer de nouvelles images en fonction des modifications de la balise flottante (par exemple lors de la sortie d'une nouvelle version LTS) ;
* La liste des ports indiquée dans la spécification est une liste de ports à exposer à partir du conteneur sur l'adresse IP Pods ; 
    * Jenkins s'exécute sur le port (http) 8080 ;
    * Le Pod expose le port 8080 du conteneur Jenkins. 
* La section volumeMounts du fichier crée un volume persistant. Ce volume est monté dans le conteneur au niveau du chemin /var/ jenkins_home et donc des modifications des données dans /var/ jenkins_home sont écrites dans le volume. Le rôle d'un volume persistant est de stocker les données de base des Jenkins et de les préserver au-delà de la durée de vie d'un pod.

Sortez et enregistrez les modifications une fois que vous avez ajouté le contenu au fichier de déploiement Jenkins.

### Déployer Jenkins

Pour créer le déploiement, exécutez :

``` bash title="BASH"
kubectl create -f jenkinsHelm-03-deployment.yaml -n jenkins
```

La commande ordonne également au système d'installer Jenkins dans l'espace de noms jenkins.

Pour valider que la création du déploiement a réussi, vous pouvez invoquer :

``` bash title="BASH"
kubectl get deployments -n jenkins
```

### Accorder l'accès au service Jenkins

Nous avons un contrôleur Jenkins déployé mais il n'est toujours pas accessible. Le Pod Jenkins a reçu une adresse IP qui est interne au cluster Kubernetes. Il est possible de se connecter au nœud Kubernetes et d'accéder à Jenkins à partir de là, mais ce n'est pas un moyen très utile d'accéder au service.

Pour rendre Jenkins accessible en dehors du cluster Kubernetes, le Pod doit être exposé en tant que service. Un service est une abstraction qui expose Jenkins au réseau plus large. Il nous permet de maintenir une connexion persistante avec le Pod quels que soient les changements dans le cluster. Avec un déploiement local, cela signifie créer un type de service NodePort. Un type de service NodePort expose un service sur un port sur chaque nœud du cluster. Le service est accessible via l'adresse IP du nœud et le service nodePort. Un service simple est [défini ici](https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkinsHelm-04-service.yaml) : 

* Ce [fichier](https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkinsHelm-04-service.yaml) définit un service comme indiqué par le champ `kind` ; 
* Le Service est de type NodePort. Les autres options sont ClusterIP (uniquement accessible dans le cluster) et LoadBalancer (adresse IP attribuée par un fournisseur de cloud, par exemple AWS Elastic IP) ;
* La liste des ports spécifiées dans la spécification est une liste des ports exposés par ce service. 
    * Le port est celui qui sera exposé par le service. 
    * Le port cible est celui pour accéder aux Pods ciblés par ce service. Un nom de port peut également être spécifié. 
* Le sélecteur spécifie les critères de sélection des pods ciblés par ce service.

Pour créer le service éxécutez :

``` bash title="BASH"
kubectl create -f jenkinsHelm-04-service.yaml -n jenkins
```

Pour valider que la création du service a réussi, vous pouvez exécuter :

``` bash title="BASH"
kubectl get services -n jenkins
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP    PORT(S)           AGE
jenkins    NodePort    10.103.31.217    <none>         8080:32664/TCP    59s
```

### Accès au tableau de bord Jenkins

Alors maintenant, nous avons créé un déploiement et un service, comment pouvons-nous accéder à Jenkins ?

À partir de la sortie ci-dessus, nous pouvons voir que le service a été exposé sur le port 32664. Nous savons également que parce que le service est de type NodeType, le service mettra en route les demandes faites à n'importe quel nœud de ce port vers le pod Jenkins. Tout ce qui nous reste est de déterminer l'adresse IP de la machine virtuelle minikube. Minikube a rendu cela très simple en incluant une commande spécifique qui publie l'adresse IP du cluster en cours d'exécution :

``` bash title="BASH"
minikube ip
192.168.99.100
```
Nous pouvons maintenant accéder au contrôleur Jenkins au [192.168.99.100:32664/](http://192.168.99.100:32664/).

Pour accéder à Jenkins, vous devez initialement saisir vos informations d'identification. Le nom d'utilisateur par défaut pour les nouvelles installations est admin. Le mot de passe peut être obtenu de plusieurs manières. Cet exemple utilise le nom de la pod de déploiement de Jenkins.

Pour trouver le nom du pod, entrez la commande suivante :

``` bash title="BASH"
kubectl get pods -n jenkins
```

Une fois que vous avez localisé le nom du pod, utilisez-le pour accéder aux journaux du pod.

``` bash title="BASH"
kubectl logs <pod_name> -n jenkins
```

Le mot de passe est à la fin du journal formaté comme une longue chaîne alphanumérique :

``` log title="LOG"
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required.
An admin user has been created and a password generated.
Please use the following password to proceed to installation:

94b73ef6578c4b4692a157f768b2cfef

This may also be found at:
/var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
```

Vous avez installé avec succès Jenkins sur votre cluster Kubernetes et pouvez l'utiliser pour créer de nouveaux pipelines de développement efficaces.

## Installer Jenkins avec l'opérateur de Jenkins

[L'opérateur Jenkins](https://jenkinsci.github.io/kubernetes-operator/docs/) est un opérateur natif de Kubernetes qui gère les opérations de Jenkins sur Kubernetes.

Il a été construit avec l'immuabilité et la configuration déclarative comme code à l'esprit, pour automatiser de nombreuses tâches manuelles nécessaires pour déployer et exécuter Jenkins sur Kubernetes.

L'opérateur de Jenkins est facile à installer en appliquant seulement quelques manifestes YAML ou avec l'utilisation de Helm.

Pour des instructions sur l'installation de l'opérateur Jenkins sur votre cluster Kubernetes et le déploiement et la configuration de Jenkins, voir la [documentation officielle de l'opérateur Jenkins](https://jenkinsci.github.io/kubernetes-operator/docs/getting-started/latest/).

## Assistant de configuration post-installation

Après avoir téléchargé, installé et exécuté Jenkins en utilisant l'une des procédures ci-dessus (à l'exception de l'installation avec l'opérateur Jenkins), l'assistant de configuration post-installation commence.

Cet assistant de configuration vous guide à travers quelques étapes "uniques" rapides pour déverrouiller Jenkins, le personnaliser avec des plugins et créer le premier utilisateur administrateur à travers lequel vous pouvez continuer à accéder à Jenkins.

### Déverrouiller Jenkins

Lorsque vous accédez à un nouveau contrôleur Jenkins pour la première fois, il vous est demandé de le déverrouiller à l'aide d'un mot de passe généré automatiquement. 

1. Parcourez `http: // localhost: 8080` (ou le port que vous avez configuré pour Jenkins lors de l'installation) et attendez que la page de **déverrouillage de Jenkins** apparaisse. 

![Déverrouiller la page Jenkins ](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-01-unlock-jenkins-page.jpg)

2. À partir de la sortie du journal de la console Jenkins, copiez le mot de passe alphanumérique généré automatiquement (entre les 2 ensembles d'astérisques). 

![Copie de mot de passe d'administration initial](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-02-copying-initial-admin-password.png)

**Note :**

* La commande: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` imprimera le mot de passe à la console. 
* Si vous exécutez Jenkins dans Docker à l'aide de l'image officielle `enkins/jenkins `, vous pouvez utiliser `sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword` pour imprimer le mot de passe dans la console sans avoir à exécuter dans le conteneur. 

3. Sur la page **_Unlock Jenkins_**, collez ce mot de passe dans le champ de **_Administrator password_** et cliquez sur **Continuer**. 

**Note :**

Le journal de la console Jenkins indique l'emplacement (dans le répertoire domestique Jenkins) où ce mot de passe peut également être obtenu. Ce mot de passe doit être entré dans l'assistant d'installation sur les nouvelles installations de Jenkins avant de pouvoir accéder à l'interface utilisateur principale de Jenkins. Ce mot de passe sert également de mot de passe du compte administrateur par défaut (avec le nom d'utilisateur "admin") s'il vous arrive de sauter l'étape de création utilisateur ultérieure dans l'assistant de configuration.

### Personnalisation de jenkins avec des plugins

Après avoir [déverrouillé Jenkins](#déverrouiller-jenkins), la page **_Customize Jenkins_** apparaît. Ici, vous pouvez installer n'importe quel nombre de plugins utiles dans le cadre de votre configuration initiale.

Cliquez sur l'une des deux options indiquées : 

* **Installez les plugins suggérés** - pour installer l'ensemble recommandé de plugins qui sont basés sur les cas d'utilisation les plus courants. 
* **Sélectionnez des plugins à installer** - pour choisir le jeu de plugins à installer initialement. Lorsque vous accédez d'abord à la page de sélection des plugins, les plugins suggérés sont sélectionnés par défaut. 

    !!! info

        Si vous ne savez pas de quel plugins vous avez besoin, choisissez **Installer les plugins suggérés**. Vous pouvez installer (ou supprimer) des plugins Jenkins supplémentaires à un moment ultérieur via la page [Manage Jenkins](./gestion-de-jenkins.md)> [Plugins](./gestion-de-jenkins.md#plugins) dans Jenkins.

L'assistant de configuration montre la progression de Jenkins en cours de configuration et votre ensemble choisi de plugins Jenkins en cours d'installation. Ce processus peut prendre quelques minutes.

### Création du premier utilisateur administrateur

Enfin, après avoir [personnalisé Jenkins avec des plugins](#personnalisation-de-jenkins-avec-des-plugins), Jenkins vous demande de créer votre premier utilisateur administrateur. 

1. Lorsque la page **Créer le Premier Utilisateur Administrateur** s'affiche, indiquez les informations relatives à votre utilisateur administrateur dans les champs correspondants, puis cliquez sur **Enregistrer et Terminer**.
2. Lorsque la page **Jenkins est prête** apparaît, cliquez sur **Démarrer à l'aide de Jenkins**. 

    **Notes :** 

    * Cette page peut indiquer que **Jenkins est presque prêt !** Au lieu de cela et si oui, cliquez sur **Redémarrer**. 
    * Si la page ne se rafraîchit pas automatiquement après une minute, utilisez votre navigateur Web pour actualiser la page manuellement. 

3. Si nécessaire, connectez-vous à Jenkins avec les informations d'identification de l'utilisateur que vous venez de créer et vous êtes prêt à commencer à utiliser Jenkins !

## Conclusion

Lorsque vous hébergez Jenkins sur Kubernetes pour des charges de travail de production, vous devez envisager de configurer un volume persistant hautement disponible afin d'éviter toute perte de données lors de la suppression d'un pod ou d'un nœud.

La suppression d'un pod ou d'un nœud peut se produire à tout moment dans les environnements Kubernetes. Il peut s'agir d'une activité de correction ou de réduction.

Nous espérons que ce guide étape par étape vous aidera à découvrir et à comprendre les composants impliqués dans la configuration d'un serveur Jenkins sur un cluster Kubernetes.