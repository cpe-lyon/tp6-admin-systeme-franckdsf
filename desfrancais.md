# TP 6 - Gestion des disques / Tâches d’administration 

## Exercice 1. Disques et partitions

**1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM**

J'ai fait clic droit configuration -> stockage -> créer un nouveau disque dur virtuel

**2. Vérifiez que ce nouveau disque dur est bien détecté par le système**

il est bien detecté : fdisk -l

**3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)**

premiere partition : <br>
- fdisk /dev/sdb<br>
- appuyer sur `n`<br>
- laisser a default le début de partition<br>
- entrer +2g pour la fin de partition<br>
- appuyer sur `n`<br>
- laisser a default le début de partition<br>
- entrer +3 pour la fin de partition (ou laisser le faire automatiquement). <br>
- appuyer sur `t`<br>
- choisir la deuxieme partition (2)<br>
- appuyer sur 7 (format NTFS)<br>
- appuyer sur `p` pour afficher les partitions.<br>
- appuyer sur `w` pour quitter et sync.

**4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)**

J'ai fait la commande `mkfs.ext4 /dev/sdb1` pour la première partition J'ai fait la commande `mkfs.ntfs /dev/sdb2` pour la deuxième partition

**5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?**

La commande `df -T` n'affiche pas le disque sdb car il n'est pas encore monté

**6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)**

`nano /etc/fstab`
on ajoute :
```
#device        mountpoint             fstype    options    dump   fsck
/dev/sdb1      /data                   ext4     defaults     0       0
/dev/sdb2      /win                    ntfs     defaults     0       0
```

**7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration**

Pas besoin de monter car les dd sont montés automatiquement au redemarrage. Une fois redemarrée, la VM affiche bien les partitions montées.

**8. Montez votre clé USB dans la VM**



**9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox**



## Exercice 2. Partitionnement LVM

**1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes du fichier /etc/fstab**

`umount /data`<br>
`umount /win`

**2. Supprimez les deux partitions du disque, et créez une partition unique de type LVM**

```
La création d’une partition LVM n’est pas indispensable, mais vivement recommandée quand
on utilise LVM sur un disque entier. En effet, elle permet d’indiquer à d’autres OS ou logiciels de
gestion de disques (qui ne reconnaissent pas forcément le format LVM) qu’il y a des données sur
ce disque.
```

supprimer une partition : 

`fdisk /dev/sdb`<br>
appuyer sur `d` pour supprimer les partitions.
enregistrer avec `w`

**3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay.**

```
Toutes les commandes concernant les volumes physiques commencent par pv. Celles concernant
les groupes de volumes commencent par vg, et celles concernant les volumes logiques par lv.
```

- faire `fdisk /dev/sdb`<br>
- appuyer sur `n`<br>
- laisser a default le début de partition<br>
- laisser a default pour la fin de partition<br>
- appuyer sur `t` et choisir `8E` (Linux LVM)<br>

`pvcreate /dev/sdb1`

**4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay.**

```
Par convention, on nomme généralement les groupes de volumes vgxx (où xx représente l’indice du groupe de volume, en commençant par 00, puis 01...)
```

`vgcreate volume1 /dev/sdb1`

**5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.**

```
On peut renseigner la taille d’un volume logique soit de manière absolue avec l’option -L (par exemple -L 10G pour créer un volume de 10 Gio), soit de manière relative avec l’option -l : -l 60%VG pour utiliser 60% de l’espace total du groupe de volumes, ou encore -l 100%FREE pour utiliser la totalité de l’espace libre.
```

`lvcreate -l 100%FREE -n 1vData volume1`

**6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.**

```
A ce stade, l’utilité de LVM peut paraître limitée. Il trouve tout son intérêt quand on veut par exemple agrandir une partition à l’aide d’un nouveau disque.
```

`fdisk /dev/mapper/volume1/1vData`<br>
on créer une partition de 2gb, puis on la formate.<br>
`mkfs.ext4 /dev/mapper/volume1-1vData`<br>

`nano /etc/fstab`<br>

on ajoute :
```
#device                         mountpoint      fstype    options    dump   fsck
/dev/mapper/volume1-1vData      /data            ext4     defaults     0       0
```


**7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.**

Fait.

**8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes**

j'ai fait la commande  `vgextend volume1 /dev/sdc1` 

**9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.**
```Il est possible d’aller beaucoup plus loin avec LVM, par exemple en créant des volumes par
bandes (l’équivalent du RAID 0) ou du mirroring (RAID 1). Le but de cet exercice n’était que de
présenter les fonctionnalités de base.
```

commande :  `lvextend -l 100%FREE /dev/volume1/1vData` puis `resize2fs /dev/volume1/1vData` 
## Exercice 3. Exécution de commandes en différé : at et cron

**1. Programmez une tâche qui affiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez
entre temps que la tâche est bien programmée.**

**2. Est-ce que le message s’est affiché ? Si la réponse est non, essayez de trouver la cause du problème (par
exemple en vous aidant des logs, du manuel...)**

**3. Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple,
l’affichage de “Il faut réviser pour l’examen !”, toutes les 3 minutes.**

**4. Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure**

**5. Programmez l’exécution d’une commande toutes les cinq minutes à partir de 2 (2, 7, 12, etc.) à 18
heures les 1er et 15 du mois :**

**6. Programmez l’exécution d’une commande du lundi au vendredi à 17 heures**

**7. Modifiez votre crontab pour que les messages ne soient plus envoyés par mail, mais redirigés dans un
fichier de log situé dans votre dossier personnel**

**8. Videz votre crontab**
