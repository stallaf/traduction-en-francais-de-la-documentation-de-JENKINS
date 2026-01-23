# FIPS - 140

Il est possible d'exécuter Jenkins de manière conforme à la norme FIPS-140 lorsque l'indicateur de conformité est activé et que le conteneur de servlets, la JVM et l'OS hôte sont tous correctement configurés. La configuration du conteneur de servlets, de la JVM et de l'hôte ne relève pas du projet communautaire Jenkins, car il s'agit d'un domaine complexe comportant de nombreux pièges et difficultés. Certaines fonctionnalités de Jenkins peuvent ne pas fonctionner ou être désactivées.
	
!!! info
    La communauté Jenkins ne vérifie pas activement la conformité de Jenkins ou des plugins à la norme [FIPS-140](https://csrc.nist.gov/pubs/fips/140-2/upd2/final).

## Plugins

Les plugins peuvent ou non respecter une demande d'exécution en mode de conformité [FIPS-140](https://csrc.nist.gov/pubs/fips/140-2/upd2/final). Avant d'installer ou de mettre à niveau un plugin, vous devez vérifier son code pour vous assurer qu'il respecte la norme FIPS-140.

## Fonctionnement du mode FIPS-140

Si vous activez le mode FIPS-140, cela indique à Jenkins et à tous les plugins qui ont opté pour cette option qu'ils doivent privilégier les algorithmes cryptographiques **susceptibles** [^1] d'être approuvés par la norme FIPS-140. Cela peut signifier que certaines fonctionnalités sont entièrement désactivées ou qu'elles utilisent une forme de cryptographie moins sécurisée (mais conforme) que la normale.
[^1]:
    Les algorithmes ne sont pas approuvés, mais plutôt une implémentation spécifique d'un algorithme spécifique. Cependant, l'implémentation utilisée lors de l'exécution dépend de la JVM, de la configuration de la JVM et du système d'exploitation hôte. Comme cela dépasse le cadre du projet Jenkins, les algorithmes ciblés sont disponibles auprès d'au moins un [fournisseur conforme à la norme FIPS-140](https://csrc.nist.gov/projects/cryptographic-module-validation-program/validated-modules/search), à savoir la [bibliothèque BouncyCastle FIPS](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/3514). 

## Ce que le mode FIPS-140 ne fait pas

Si un code provenant de la JVM, du conteneur de servlets, de Jenkins ou d'un plugin demande un algorithme non conforme, cela restera le cas et la demande pourra être honorée. Par exemple, ce mode ne permet pas de configurer la JVM, de sorte que les connexions TLS vers des sites web sécurisés externes peuvent toujours utiliser un cryptage non conforme. De plus, Jenkins ne peut pas garantir que les plugins utiliseront un cryptage, le cas échéant. En fin de compte, le fait que Jenkins et les plugins fonctionnent lorsque le mode FIPS-140 est activé ne signifie pas pour autant qu'ils respectent la norme du gouvernement américain.

## Comment exécuter un Jenkins entièrement conforme à la norme FIPS-140

Comme mentionné précédemment, l'hôte, la JVM et le conteneur de servlets doivent tous être configurés de manière appropriée pour garantir la conformité de Jenkins à la norme FIPS-140. Il convient d'être extrêmement prudent lors de l'installation ou de la mise à niveau des plugins, car ceux-ci peuvent être ou non conformes à la norme FIPS-140, et ils peuvent introduire du code non conforme ou modifier la configuration de la JVM de manière à compromettre la conformité.

La communauté Jenkins ne prend pas en charge le mode Jenkins FIPS-140 et, en raison de la nature complexe de la configuration de la JVM et du servlet qui peut changer d'une version à l'autre, ne fournit pas de documentation sur la configuration complète requise pour exécuter Jenkins de manière entièrement conforme à la norme FIPS-140. Si vous devez exécuter Jenkins de manière à ce qu'il soit conforme à la norme FIPS-140, il est recommandé de faire appel à un fournisseur commercial. La communauté Jenkins peut être en mesure de résoudre les problèmes liés à la conformité FIPS-140 ; ceux-ci seront traités comme n'importe quel autre rapport de bogue ou demande de fonctionnalité.
