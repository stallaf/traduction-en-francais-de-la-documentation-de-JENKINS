# Reverse proxy - Iptables

<div class="couleur-introduction">
L'installation par défaut de Jenkins s'exécute sur les ports 8080 et 8443. En général, les serveurs HTTP/HTTPS s'exécutent respectivement sur les ports 80 et 443. Mais ces ports sont considérés comme privilégiés sur les systèmes Unix/Linux, et le processus qui les utilise doit appartenir à root. Il n'est pas recommandé d'exécuter Jenkins en tant que root ; il doit être exécuté en tant qu'utilisateur distinct. Une solution consiste à placer Jenkins devant un serveur web tel qu'Apache et à lui laisser le soin de proxyfier les requêtes vers Jenkins, mais cela nécessite également la maintenance de l'installation Apache. Si vous souhaitez exécuter Jenkins sur le port 80 ou 443 (c'est-à-dire HTTP/HTTPS), mais que vous ne voulez pas configurer de serveur proxy, vous pouvez utiliser iptables sous Linux pour transférer le trafic.
</div>

## Installations Ubuntu

Suivez les [instructions d'installation Ubuntu](./installation-linux.md#debianubuntu) pour installer et configurer l'installation initiale de Jenkins sur une version prise en charge d'Ubuntu. Ces instructions ne fonctionnent pas sur les versions d'Ubuntu qui ne sont plus prises en charge par le projet Ubuntu.

## Prérequis

Afin de transférer le trafic de 80/443 vers 8080/8443, vous devez d'abord vous assurer qu'iptables a autorisé le trafic sur ces 4 ports. Utilisez la commande suivante pour afficher la configuration actuelle d'iptables :

``` shell
iptables -L -n
```

Vous devriez voir apparaître les entrées 80, 443, 8080 et 8443. Voici un exemple de résultat à titre de comparaison.

``` console
ain INPUT (policy ACCEPT)target     prot opt source               destination
target     prot opt source               destination
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:443
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:80
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:8080
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:8443
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
target     prot opt source
```

Si vous ne voyez pas d'entrées pour ces ports, vous devez alors exécuter des commandes (en tant qu'administrateur ou avec sudo) pour ajouter ces ports. Par exemple, si vous ne voyez aucune de ces entrées et que vous devez toutes les ajouter, vous devrez exécuter les commandes suivantes :

``` shell
sudo iptables -I INPUT 1 -p tcp --dport 8443 -j ACCEPT
sudo iptables -I INPUT 1 -p tcp --dport 8080 -j ACCEPT
sudo iptables -I INPUT 1 -p tcp --dport 443 -j ACCEPT
sudo iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT
```

**REMARQUE**

J'ai utilisé -I INPUT 1. Dans de nombreux exemples/documentations iptables, vous verrez -A INPUT. La différence est que -A ajoute à la liste des règles, tandis que -I INPUT 1 insère avant la première entrée. Généralement, lorsque vous ajoutez de nouveaux ports acceptés à la configuration iptables, vous souhaitez les placer au début du jeu de règles, et non à la fin. Exécutez à nouveau iptables -L -n et vous devriez maintenant voir les entrées pour ces 4 ports.

## Transfert

Une fois que le trafic sur les ports requis est autorisé, vous pouvez exécuter la commande pour transférer le trafic du port 80 vers le port 8080 et le trafic du port 443 vers le port 8443. Les commandes se présentent comme suit :

``` shell
sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8443
```

Vous pouvez vérifier les règles de transfert à l'aide de la commande ci-dessous.

``` shell
[root@xyz~]# iptables -L -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:http redir ports 8080
REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:https redir ports 8443

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
```

+

Une fois ces règles définies et confirmées avec iptables -L -n, et une fois que votre contrôleur Jenkins est opérationnel sur le port 8080, essayez d'accéder à votre contrôleur Jenkins sur le port 80 au lieu du port 8080. Cela devrait fonctionner et votre URL devrait rester sur le port 80. En d'autres termes, elle ne devrait pas être redirigée vers le port 8080. Le fait que le transfert de 80 vers 8080 (ou de 443 vers 8443) doit rester caché au client.

### Enregistrement de la configuration iptables

L'utilisation de la commande iptables pour modifier la configuration des ports et les règles de routage ne modifie que la configuration actuelle en mémoire. Elle ne persiste pas entre les redémarrages du service iptables. Vous devez donc vous assurer d'enregistrer la configuration pour que les modifications soient permanentes.

La sauvegarde de la configuration est légèrement différente entre les systèmes basés sur Red Hat rpm et ceux basés sur Debian. Sur les systèmes basés sur Red Hat (Red Hat Enterprise Linux, Fedora, Alma Linux, Rocky Linux, Oracle Linux, CentOS, etc.), exécutez la commande suivante :

``` shell
sudo iptables-save > /etc/sysconfig/iptables
```

Sur un système basé sur Debian (Debian, Ubuntu, Mint, etc.), exécutez la commande suivante :

``` shell
sudo sh -c « iptables-save > /etc/iptables.rules »
```

La commande iptables-restore devra être exécutée manuellement, ou votre système devra être configuré pour l'exécuter automatiquement au démarrage, par rapport au fichier /etc/iptables.rules que vous avez créé, afin que votre configuration iptables soit conservée après les redémarrages. Sur Ubuntu, le moyen le plus rapide consiste à installer `iptables-persistent` après avoir configuré iptables. Il créera automatiquement les fichiers requis à partir de la configuration actuelle et les chargera au démarrage.

``` shell
sudo apt-get install iptables-persistent
```

Consultez [https://help.ubuntu.com/community/IptablesHowTo](https://help.ubuntu.com/community/IptablesHowTo) pour connaître les autres options Ubuntu. Il existe de nombreuses autres ressources décrivant cette procédure ; veuillez consulter la documentation de votre système ou effectuer une recherche sur Internet pour obtenir des informations spécifiques à votre version de Linux.

Si vous n'êtes pas sûr du type de système dont vous disposez, consultez la documentation de ce système pour savoir comment mettre à jour la configuration d'iptables.

## Utilisation de firewalld

Certaines distributions Linux (Red Hat Enterprise Linux, Rocky Linux, Alma Linux, Oracle Linux, CentOS, etc.) sont livrées avec firewalld, qui sert d'interface pour iptables. La configuration via firewalld s'effectue à l'aide de la commande firewall-cmd. Au lieu d'utiliser l'une des commandes iptables mentionnées ci-dessus, il vous suffit d'exécuter une commande telle que :

``` shell
# allow incoming connections on port 80.
# You can also use --add-service=http instead of adding a port number
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --permanent \
                  --add-forward-port=port=80:proto=tcp:toaddr=127.0.0.1:toport=8080

# allow incoming connections on port 443.
# You can also use --add-service=https instead of adding a port number
sudo firewall-cmd --add-port=443/tcp --permanen
t
sudo firewall-cmd --permanent \
                  --add-forward-port=port=443:proto=tcp:toaddr=127.0.0.1:toport=8443
sudo firewall-cmd --reload
```

Avec les commandes ci-dessus, Jenkins peut être configuré pour s'exécuter sur localhost:8080 et/ou localhost:8443 (selon que vous avez besoin ou souhaitez utiliser SSL ou non).

firewalld créera ensuite les règles iptables requises afin que les connexions entrantes sur le port 80 soient redirigées vers Jenkins sur le port 8080 (et celles sur le port 443 vers le port 8443)
