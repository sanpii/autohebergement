# Accès à distance

Afin d’avoir un accès distant à notre machine, nous avons besoin d’installer un
serveur (non plus la machine mais un logiciel) pour *SSH* qui est un
protocole de connexion sécurisée. Ce protocole permet d'intéragir avec votre
serveur comme s'il sagissez de votre ordinateur de bureau. Autant vous dire que
c'est ici que vous passerai le plus clair de votre temps lorsque vous aurez
besoin de faire des modifications sur votre serveur.

## Installation {#installation}

Sous Debian le paquet se nomme `sshd` pour *ssh daemon* dont
l'installation s'effectue comme nous avons déjà dans le chapitre précédent :

```
# apt-get install sshd
```

En plus du serveur, vous devez avoir la client `ssh` sur notre ordinateur
de bureau :

```
# apt-get install ssh
```

Vous pouvez déjà tester que tout fonctionne bien en lançant la commande suivante
sur votre ordinateur :

```
$ ssh login@ip.de.votre.serveur
```

Le login et le mot de passe sont les mêmes que ceux utilisés sur votre serveur.

Vous devriez vous retrouver avec un nouveau prompt qui porte le nom de votre
serveur, par exemple :

```
sanpi@handy:~$ ssh 192.0.2.4

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Jun 21 18:18:40 2015 from 192.0.2.2
sanpi@debian:~$
```

Vous pouvez dés à présent travailler sur votre serveur comme si vous
étiez devant, sauf qu'il se trouve à l'autre bout de la pièce.

## Configuration {#configuration}

Nous allons devoir modifier la configuration de SSH afin de le rendre plus
sécurisé (c'est le point d'entrée sur votre serveur le plus recherché par des
personnes mal attentionnée puisqu'elles peuvent avoir un accès complet au
ressources par ce biais). Vous allons donc utiliser uniquement
l'authentification par clés.

### Authentification par clés ssh {#configuration-cles}

Commencez par générer vos clés sur votre PC :

```
$ ssh-keygen -t rsa
```

Vous pouvez laisser le mot de passe vide, ce qui vous évitera de devoir le
taper à chaque fois que vous vous connectez, à la condition que vous aillez
confiance en votre ordinateur (s'il est chez vous par exemple), mais il est
fortement déconseillé de ne pas mettre de mot de passe sur votre clés si votre
ordinateur se trouve dans un lieu public.

Ensuite, transférez la clés sur votre serveur :

```
$ ssh-copy-id -i ~/.ssh/id_dsa.pub sanpi@serveur
```

> **todo** ssh-agent ?!

### Interdire l’accès par mot de passe {#configuration-password}

Maintenant que l’accès par clés fonctionne, autant désactiver l’accès classique
par mot de passe.

Pour cela, éditer le fichier `/etc/ssh/sshd_config` et modifier l’option
`PasswordAuthentication` à non :

```
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no
```

Comme souvent, lorsque vous modifiez la configuration d'un serveur, il est
nécessaire de le redémarrer pour que celle-ci soit prise en compte :

```
# systemctl restart ssh.service
```
