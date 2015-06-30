# Le serveur de transfert de fichiers {#ftp}

Le protocole FTP permet, comme son nom l’indique, le transfert de fichiers d’un
client vers un serveur (dans les deux sens).

Ce protocole est généralement le seul disponible pour les offres mutualisées des
hébergeurs, mais pour un usage personnel, de par mon expérience, je m’en sert
très peut : il est plus rapide de faire un `wget` via SSH ou encore
d’utiliser `vi` pour éditer les fichiers. Par contre si vous souhaitez partager
votre serveur c’est une bonne idée de proposer ce service, et au vu de la
simplicité d’installation, il serais dommage de s’en priver.

Ayant installer SSH dans le chapitre précédent, vous pouvez disposez déjà d’un
serveur FTP sécurisé, SFTP. Voila pour la partie installation…

Concernant la partie configuration, l’authentification est basée sur le système
de clés SSH, donc encore une fois pas grand chose à ajouter. Pour tester, vous
pouvez utiliser le programme `sftp` en ligne de commande (similaire à la
commande `ftp`) ou un client graphique comme
[filezilla](https://fr.wikipedia.org/wiki/FileZilla).

Un petit problème cependant : la configuration par défaut permet à l’utilisateur
de se balader sur l’ensemble du disque[^1], c’est pourquoi il plus courant de
bloquer l’utilisateur dans son répertoire personnel (on parle d’ environnement
*chrooter*).

C’est là que les choses se corsent légèrement, si vous êtes le seul utilisateur
de votre serveur je vous conseille de vous arrêter là.

Pour les autres, ou les curieux, depuis la version 4.9 de OpenSSH, il existe
l’option `ChrootDirectory` qui permet de définir un répertoire de base. Il faut
pour cela utiliser le serveur FTP interne à `ssh`. Éditez le fichier
`/etc/ssh/sshd_config` et remplacez la ligne `Subsystem` par :

```
Subsystem sftp internal-sftp
```

Nous allons limiter les droits pour les utilisateurs du groupe `sftp` :

```
Match Group sftp
    ChrootDirectory %h
    ForceCommand internal-sftp
    AllowTcpForwarding no
    PermitTunnel no
    X11Forwarding no
```

Redémarrons SSH pour que la nouvelle configuration soit prise en compte.

> ***Warning*** Vous ferrez forcement l’erreur une fois, mais lorsque vous
> redémarrer le serveur SSH suite à la mise à jour de la configuration, ayez
> toujours un autre moyen d’accéder à votre serveur en cas d’erreur.

```
# systemctl restart sshd.service
```

Créons le groupe d’utilisateur correspondant :

```
# addgroup sftp
```

Et ajoutons notre invité à ce groupe :

```
# adduser guest sftp
```

Pour que cela fonctionne, il faut que le propriétaire du répertoire de base soit
`root`, par conséquent, l’utilisateur ne pourra pas écrire dans le répertoire de
base (c’est à dire créer de nouveaux fichiers), rien de dramatique, il suffit de
lui créer les répertoires de bases (`.ssh`, `cgi-bin` et `public_html` par
exemple).

Si vous ne souhaitez pas le faire manuellement pour chaque nouvel utilisateur,
vous pouvez compléter le squelette utilisé par la commande `adduser` :

```
# mkdir /etc/skel/{.ssh,cgi-bin,public_html}
```

Si, suite à ces manipulations, vous ne parvenez plus à vous connecter, vous
pouvez afficher le journal des authentifications qui devrait vous aider :

```
# tail -f /var/log/auth.log
Jul  1 12:34:58 debian-jessie sshd[856]: pam_unix(sshd:session): session opened for user guest by (uid=0)
Jul  1 12:34:58 debian-jessie sshd[858]: fatal: bad ownership or modes for chroot directory "/home/guest"
Jul  1 12:34:58 debian-jessie sshd[856]: pam_unix(sshd:session): session closed for user guest
```

Au final, une installation extrêmement simple, une configuration tout aussi
simple pour un serveur personnel et une sécurité accrue. Par contre, si l’on
souhaite profiter du *chroot*, comme pour un FTP classique, les choses se
compliquent. Pour l’utilisation que j’ai de mon serveur, le choix est vite
fait : encore un mot de passe en moins à retenir !

[^1] Rassurez vous, il existe un système de droits pour éviter qu’un utilisateur
simple puisse modifier ou lire les fichiers sensible, cependant il subsiste des
moyens (volontaire ou non) de nuire : remplir le dossier `/tmp` ou accéder à la
liste complète des utilisateurs, …
