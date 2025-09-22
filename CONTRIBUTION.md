# Voici les étapes pour ajouter de la documentation. 

1. Vérifiez la documentation d'origine de [JENKINS](https://www.jenkins.io/doc/).
2. Vérifiez que cette version française ne la contient pas déjà ou qu'une version connexe peut être améliorée. 
2. Déterminer le placement approprié. Dans le dépôt, accédez au dossier utilisateur (ou tout autre dossier spécifique) et créez une nouvelle branche. 
3. Créez un nouveau fichier et ajoutez le.
4. Assurez-vous d'insérer le nom et l'extension du fichier. 
5. Commentez vos modifications et ajoutez un court message pour les décrire.
7. Testez les modifications localement afin de les vérifier. 
8. Soumettez une _PR_ en incluant un titre clair et une description des modifications proposées.


# Mettre à jour la documentation existante.

Si vous voyez une page qui doit être corrigée, mise à jour ou qui peut être améliorée, suivez ces étapes :

1. Identifiez la page de cette version française qui doit être mise à jour. 
2. Cliquez sur le bouton «Améliorer cette page sur GitHub» dans le coin supérieur droit. 
3. Modifiez le fichier.
4. Soumettez vos modifications. Nommez votre branche et cliquez sur le bouton «Proposer les modifications». 
5. Testez les modifications localement afin de les vérifier.
6. Soumettez une _PR_ en incluant un titre clair et une description des modifications ajoutées

Merci pour votre contribution. Stallaf ou l'équipe examinera la demande et approuvera les modifications nécessaires.

# Création d'un environnement local 

Vous pouvez vérifier comment le site de documentation reflétera vos modifications. Suivez les étapes ci-dessous pour apprendre à construire votre environnement local et à vérifier toutes vos modifications avant d'envoyer une demande.

## Installer les dépendances (Linux)

* Assurez-vous que des versions récentes de Python et pip soient présentes sur votre système,
sinon installer-les.

```console
1  python3 --version
2  Python 3.12.3
3  pip --version
4  pip 24.0 from /usr/lib/python3/dist-packages/pip (python 3.12)
```

Material pour MkDocs est fourni sous forme de paquet Python et peut être installé avec pip, 
idéalement en utilisant un [environnement virtuel](https://realpython.com/what-is-pip/#using-pip-in-a-python-virtual-environment).

* Installez le package mkdocs-material via pip :

```console
1  pip install mkdocs-material
```

La commande mkdocs doit maintenant être installée sur votre système. 
Exécutez mkdocs --version pour vérifier que tout a bien fonctionné.

```console
1  mkdocs --version
2  mkdocs, version 1.5.3 from /usr/lib/python3/dist-packages/mkdocs (Python 3.12)
```

* Clonez le dépôt https://github.com/stallaf/traduction-en-francais-de-la-documentation-de-GRAV-1.7/tree/main


## Exécuter le serveur d'application

Vous êtes prêt à démarrer votre site de documentation local.

* Placez-vous à la racine de votre dossier ;
* Lancez le serveur Jekyll :

```console
1  mkdocs serve
```

Puis ouvrez 127.0.0.1:8000 dans votre navigateur.

