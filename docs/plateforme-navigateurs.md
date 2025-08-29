# Compatibilité avec les Navigateurs

Cette page documente la politique de support du navigateur pour les contrôleurs Jenkins. REMARQUE : Le contenu ici ne s'applique pas au site Web de Jenkins ou à d'autres services organisés par le projet Jenkins.

## Modèle de support

La prise en charge du navigateur Web Jenkins se divise en trois niveaux :

1. Niveau 1: Viser à soutenir de manière proactive ces navigateurs et à fournir un UX égal à tous. 
2. Niveau 2 : Accepter les correctifs pour résoudre les problèmes et faire le meilleur effort pour vous assurer qu'il existe au moins une façon de faire une action. 
3. Niveau 3 : Aucune garantie. Nous accepterons les correctifs, mais seulement s'ils sont à faible risque. **Il s'agit de la valeur par défaut, sauf si un navigateur/version est répertorié ci-dessous.**

Nous ne garantissons aucune compatibilité avec les versions préliminaires (par exemple alpha, bêta ou canary) des navigateurs et n'acceptons pas les rapports de bogues ni les correctifs les concernant.

## Tableau de compatibilité des navigateurs

| Navigateur    | Niveau 1  | Niveau 2  | Niveau 3  |
| ------------------- | -------------- | --------------- | ------------- |
| Google Chrome | Dernière version régulière/correctif  | Version N-1, dernier correctif    | Autres versions |
| Mozilla Firefox   | Dernière version régulière/correctif  | Version N-1, dernier correctif ; Dernière version [ESR](https://www.mozilla.org/en-US/firefox/organizations/). | Autres versions
| Microsoft Edge    | Dernière version régulière/correctif | Version N-1, dernier correctif | Autres versions 
| Apple Safari  | Dernière version régulière/correctif | Version N-1, dernier correctif | Autres versions|

La prise en charge des navigateurs mobiles (par exemple iOS Safari) n'a pas encore été déterminée.

## Historique des modifications

* 16/06/2025 - Déplacer Firefox ESR du niveau 1 au niveau 2 [(discussion dans la liste de diffusion des développeurs](https://groups.google.com/g/jenkinsci-dev/c/jbhWs4JZ-j4/m/yBXEBW3DAAAJ)) ;
* 01/02/2022 - Supprimer la prise en charge de Internet Explorer, Ajouter Edge ([discussion dans la liste de diffusion des développeurs](https://groups.google.com/g/jenkinsci-dev/c/piANoeohdik)) ;
* 19/11/2019 - Mise à jour des politiques ([discussion dans la liste de diffusion des développeurs](https://groups.google.com/forum/#!topic/jenkinsci-dev/TV_pLEah9B4)) ;
* 03/09/2014 - Politique originale pour Jenkins 1.579 ([notes de réunion de gouvernance](http://meetings.jenkins-ci.org/jenkins/2014/jenkins.2014-09-03-18.01.html)).
