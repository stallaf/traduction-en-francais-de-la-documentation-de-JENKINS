# Formatage des Balises

Jenkins permet aux utilisateurs disposant des autorisations appropriées de saisir des descriptions de divers objets, tels que des vues, des tâches, des builds, etc. Ces descriptions sont filtrées par des _formateurs de balisage_. Elles ont deux objectifs :

1. Permettre aux utilisateurs d'utiliser un formatage riche pour ces descriptions.
2. Protéger les autres utilisateurs contre les attaques de type [Cross-Site Scripting](https://en.wikipedia.org/wiki/Cross-site_scripting) (XSS).

## Configuration du formateur de balisage

Le formateur de balisage peut être configuré dans _Manage Jenkins » Security » Markup Formatter_.

Le formateur de balisage par défaut Plain text affiche toutes les descriptions telles qu'elles ont été saisies : les métacaractères HTML non sécurisés tels que `<` et `&` sont échappés, et les sauts de ligne sont affichés sous forme de balises HTML` <br/>`.

Un autre formatage couramment installé est le formatage HTML sécurisé, fourni par le [plugin OWASP Markup Formatter](https://plugins.jenkins.io/antisamy-markup-formatter). Il permet l'utilisation d'un sous-ensemble HTML basique et sécurisé.

## Considérations de sécurité

### Descriptions des profils utilisateur

Tout utilisateur disposant d'un compte et d'une autorisation globale/de lecture peut modifier son propre profil utilisateur. Cela inclut une description qui est affichée à l'aide du formatage configuré.

Il peut donc être dangereux de configurer un formateur de balisage autorisant du code HTML arbitraire, même en limitant les autorisations telles que _Job/Configure_ et _Build/Update_ à des utilisateurs entièrement fiables : toute personne disposant d'un compte pourra modifier sa propre description et tout autre utilisateur accédant à son profil pourra être victime d'une attaque XSS.

Cela est particulièrement risqué sur les instances Jenkins accessibles au public lorsque le domaine de sécurité est mis en œuvre à l'aide d'un service tel que GitHub, GitLab ou les comptes Google, ce qui signifie que n'importe qui peut potentiellement se connecter à Jenkins et modifier son propre profil.