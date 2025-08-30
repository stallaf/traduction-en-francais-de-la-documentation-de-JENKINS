# Utilisation des Identifiants

Il existe de nombreux sites et applications tiers qui peuvent interagir avec Jenkins, par exemple, les référentiels d'artefacts, les systèmes et services de stockage basés sur le cloud, etc.

Un administrateur systèmes d'une telle application peut configurer des informations d'identification dans l'application pour une utilisation dédiée à Jenkins. Cela serait généralement fait pour «verrouiller» les zones de la fonctionnalité de l'application disponibles pour Jenkins, généralement en appliquant des contrôles d'accès à ces informations d'identification. Une fois qu'un gestionnaire de Jenkins (c'est-à-dire un utilisateur de Jenkins qui administre un site Jenkins) ajoute/configure ces informations d'identification dans Jenkins, les informations d'identification peuvent être utilisées par des projets Pipeline pour interagir avec ces applications tiertes.

**REMARQUE :** La fonctionnalité d'informations d'identification Jenkins décrite sur cette page et les pages associées est fournie par le plugin Credentials Binding.

_La bonne façon de gérer les informations d'identification dans Jenkins_
<iframe width="800" height="420" src="https://www.youtube.com/embed/yfjtMIDgmfs" title="The Correct Way to Handle Credentials in a Jenkins Pipeline" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Les informations d'identification stockées dans Jenkins peuvent être utilisées : 

* n'importe où applicable dans tout Jenkins (Global Creasatings) ;
* par un projet/élément Pipeline spécifique (pour en savoir plus, consultez la section Gestion des [informations d'identification](./pipeline-jenkinsfile.md#gestion-des-informations-identification) de la section Utilisation d'un fichier [Jenkinsfile](./pipeline-jenkinsfile)) ;
* par un utilisateur spécifique de Jenkins (comme c'est le cas pour les [projets de Pipeline créés dans Blue Ocean](./blueocean-creation-pipeline)).

Jenkins peut stocker les types d'identification suivants : 

* **_Secret text_** (Texte secret) : un jeton tel qu'un jeton API ou un jeton d'accès personnel GitHub ;
* **_Username and password_** (Nom d'utilisateur et mot de passe) : ils peuvent être traités comme des composants distincts ou comme une chaîne séparée par des deux-points au format nom `username:password ` (pour en savoir plus, consultez la section [Gestion des informations d'identification](./pipeline-jenkinsfile.md#gestion-informations-identification)) ;
* **_ecret file_** (Fichier secret] : il s'agit essentiellement du contenu secret d'un fichier ;
* **_SSH Username with private key _** (Nom d'utilisateur SSH avec clé privée) : une [paire de clés publique/privée SSH](http://www.snailbook.com/protocols.html) ;
* **_Certificate_** (Certificat) : un [fichier de certificat PKCS#12](https://tools.ietf.org/html/rfc7292) et un mot de passe facultatif, ou
* **_Docker Host Certificate Authentication_** Informations d'(identification d'authentification du certificat hôte Docker).

## Garantie d'identification

Pour maximiser la sécurité, les informations d'identification configurées dans Jenkins sont stockées sous une forme chiffrée sur le contrôleur de Jenkins (chiffré par l'ID de contrôleur Jenkins) et ne sont gérés que dans des projets Pipeline via leurs identifiants.

Cela minimise les chances d'exposer les informations d'identification réelles elles-mêmes aux utilisateurs de Jenkins et entrave la possibilité de copier des informations d'identification fonctionnelles d'un contrôleur Jenkins à une autre.

## Configuration des informations d'identification

Cette section décrit les procédures de configuration des informations d'identification dans Jenkins.

Les informations d'identification peuvent être ajoutées à Jenkins par tout utilisateur Jenkins disposant de l'autorisation Informations **d'identification > Créer** (définie via la **sécurité basée sur Matrix**). Ces autorisations peuvent être configurées par un utilisateur Jenkins disposant de l'autorisation **Administre**r. Pour en savoir plus, consultez la section [Autorisation](./securite-gestion-de-la-securite.md#autorisation) de la page [Gestion de la sécurité](./securite-gestion-de-la-securite).

Sinon, tout utilisateur de Jenkins peut ajouter et configurer des informations d'identification si les paramètres **d'autorisation** de la page Paramètres de **sécurité** de votre contrôleur Jenkins sont définis sur les **utilisateurs connectés par défaut peuvent faire n'importe quoi** ou **n'importe qui peut faire n'importe quoi**.

### Ajout de nouvelles informations d'identification globales

Pour ajouter de nouvelles informations d'identification globales à votre contrôleur Jenkins:  

1. Si nécessaire, assurez-vous que vous êtes connecté à Jenkins (en tant qu'utilisateur avec les informations **d'identification> Créer une autorisation**) ;
2. À partir du tableau de bord Jenkins, naviguez jusqu'à  **_Manage Jenkins > Credentials_**. 
![image1](https://www.jenkins.io/images/using/manage_credentials.png) 
3. Sous **_Stores scoped to Jenkins_**, sélectionnez **_System_**.
![image2](https://www.jenkins.io/images/using/Stores_scoped_to_Jenkins.png) 
4. Sous **_System_**, sélectionnez le lien _**Global credentials (unrestricted)**_ (Identifiants globaux (sans restriction)) pour accéder à ce domaine par défaut.
![image3](https://www.jenkins.io/images/using/global_credentials.png)
5. Sélectionnez **_Add Credentials _** (Ajouter des informations d'identification) à gauche. 
    **REMARQUE :** S'il n'y a pas d'identification dans ce domaine par défaut, vous pouvez également sélectionner le lien **_add some credentials_** (ajouter des informations d'identification) (qui est la même que la sélection du lien **_Add Credentials_** (Ajouter des informations d'identification)). 
6. Dans le **_King_** (champ)  type, choisissez le type d'identification à ajouter. 
7. Dans le champ **_Scope_** (Portée), choisissez soit : 
    * **_Global_** (Globale) - Si les informations d'identification à ajouter sont/concernent un projet/élément Pipeline. Le choix de cette option applique l'étendue des informations d'identification au Pipeline Project/Item "Object" et tous ses objets descendants ;
    * **_System_** (Système) - Si l'/les information(s) d'identification à ajouter est/sont  destinée(s) au contrôleur Jenkins lui-même pour interagir avec les fonctions d'administration du système, telles que l'authentification par e-mail, la connexion de l'agent, etc. Le choix de cette option applique la portée des informations d'identification à un seul objet uniquement.
8. Ajoutez les informations d'identification elles-mêmes dans les champs appropriés pour votre type d'identification choisi : 
    * **_Secret text_** (Texte secret) : copiez le texte secret et collez-le dans le champ **Secret** ;
    * **_Username and password_** (Nom d'utilisateur et mot de passe) : indiquez le **nom d'utilisateur** et le **mot de passe** dans les champs correspondants ;
    * **_Secret file_** (Fichier secret] : sélectionnez l'option **_Choose file_** (Choisir un fichier) à côté du champ **_File_** (Fichier] pour sélectionner le fichier secret à télécharger vers Jenkins ;
    * **_SSH Username with private key_** (Nom d'utilisateur SSH avec clé privée] : indiquez le **_Username_** (nom d'utilisateur), **_Private Key_** la (clé privée) et, éventuellement, la **_Passphrase_** (phrase secrète) dans les champs correspondants ;
    **REMARQUE** : en sélectionnant **_Enter directly_** (Entrer directement), vous pouvez copier le texte de la clé privée et le coller dans la zone de texte **_Key_** (Clé) qui s'affiche ;
    * **_Certificate_** 'Certificat) : indiquez le **certificat** et le **_Password_** (mot de passe) facultatif. En sélectionnant **_Upload_** (Télécharger) le certificat PKCS#12, vous pouvez télécharger le certificat sous forme de fichier via le bouton **_Upload certificate_** (Télécharger le certificat) qui s'affiche ;
    * **_Docker Host Certificate Authentication_** (Authentification du certificat hôte Docker) : copiez et collez les informations appropriées dans les champs **_Client Key_** (Clé client), **_Client Certificate_** (Certificat client) et **_Server CA Certificate_** Certificat CA serveur.
9. Dans le champ ID, indiquez une valeur d'identifiant significative, par exemple `jenkins-user-for-xyz-artifact-repository`. Le fournisseur d'identifiants intégré (par défaut) peut utiliser des lettres majuscules ou minuscules pour l'ID d'identifiant, ainsi que n'importe quel caractère de séparation valide. D'autres fournisseurs d'identifiants peuvent appliquer des restrictions supplémentaires sur les caractères ou les longueurs autorisés. Cependant, dans l'intérêt de tous les utilisateurs de votre contrôleur Jenkins, il est préférable d'utiliser une convention unique et cohérente pour spécifier les ID d'identifiant.<br>
**REMARQUE :**  ce champ est facultatif. Si vous ne spécifiez pas sa valeur, Jenkins attribue une valeur d'identifiant unique global (GUID) à l'identifiant d'informations d'identification. N'oubliez pas qu'une fois l'identifiant d'informations d'identification défini, il ne peut plus être modifié.
10.  Spécifiez une **Description** facultative pour les informations d'identification.
11.  Sélectionnez **OK** pour enregistrer les informations d'identification.

