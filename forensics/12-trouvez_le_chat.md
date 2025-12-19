link location: https://www.coordonnees-gps.fr/

ce challenge m'a fatigue je me suis fais aider sinon j'aurai pas trouve donc comment j'ai pocede.

- Dans un premier temps, on récupère l’archive, on la décompresse et on cherche à connaitre le type de données :
```
gunzip ch9.gz

file ch9
```

- On peut ainsi voir qu’il s’agit d’un disque complet.  On va ensuite récupérer la table de partition du disque :
```
mmls ch9
```
ou
```
fdisk -l ch9
```
	       `Slot      Start        End          Length       Description   
	 000:  Meta      0000000000  0000000000   0000000001   Primary Table (#0)   
	 001:  -------  0000000000   0000002047   0000002048   Unallocated   
	 002:  000:000   0000002048   0000262143   0000260096   Win95 FAT32 (0x0b)`

- On se retrouve donc avec un offset de 2048 pour la partition contenant les données.
On peut ensuite utiliser la commande fls afin de lister tous les fichiers présents dans la partition :
```
fls -o 2048 -r ch9
```

- recherchons les fichiers supprimes
```
fls -o 2048 -r chall9 | grep \*
```
	`d/d 5:        Documentations   
	+ r/r 25:        tartes_flambee_a_volonte_francais_2013.pdf   
	+ r/r 28:        mangeur-de-cigogne (1).pdf   
	+ r/r * 32:        La rÃ©sistance Ã©lectronique.pdf      
	+ r/r 34:        Menu AC.pdf   
	+ r/r 54726:        brasserie_jo_dinner_menu.pdf   
	+ r/r 54729:        Courba13-01.pdf`

Nous allons pouvoir retrouver un fichier au nom interessant : revendications.odt. Grace à la commande fls, nous avons pu récuperer son numéro d’inode : 246775.

- extraction du fichier
```
icat -o 2048 -r chall9 246775 > revendications.odt
```

- A ce niveau on a juste a ouvrir le fichier .odt, enregistrer l'image qui y est contenu et faire un exiftool afin de voir les coordonnees de localisation.  

![Texte alternatif de l'image](image/121.png)

- utiliser le site de geolocalisation pour trouver le npm de la ville qui est notre flag:

![Texte alternatif de l'image](image/122.png)

- **othersolutions 1:** etape du fichier .odt
Les fichiers ODT étant des ZIP de fichiers XML, nous pouvons le décompresser afin d’accéder à son contenu :
	`$unzip revendications.odt   
	Archive:  revendications.odt   
	extracting: mimetype                  
	 extracting: Thumbnails/thumbnail.png      
	 inflating: Pictures/1000000000000CC000000990038D2A62.jpg      
	 inflating: content.xml                
	 inflating: styles.xml                  
	 inflating: settings.xml             

On y retrouve un image : 1000000000000CC000000990038D2A62.jpg, celle d’un chat.  
Nous allons pouvoir aller regarder les données Exifs de cette dernière afin de récupérer les coordonnées GPS :
```
exif Pictures/1000000000000CC000000990038D2A62.jpg
```

- **other solution 2:** 
On récupère l’archive et on la décompresse. Un petit test pour voir à quoi on a affaire :
```
$file chall9   
chall9: x86 boot sector; partition 1: ID=0xb, starthead 32, startsector 2048, 260096 sectors, extended partition table (last)\011, code offset 0x0
```
`
- dans un premier temps et utilisé foremost pour voir si il y avait de la donnée a récupérer:
```
foremost -T -i chall9 -o ./output -t all
```

On fouille un peu et on tombe sur deux choses intéressantes :
- un GIF avec une revendication sur la libération de l’Alsace avec une photo de Chat
- un zip avec la même jolie photo de chat
On test les données EXIF du jpg :
```
exiftags 1000000000000CC000000990038D2A62.jpg
```

- #### **other solution 3**
Bulk_extractor est un outil de forensic qui permet de scanner des infos intéressantes (email, exif, zip...) à partir d’une image disque ou d’une partition ou de fichiers. Il est capable d’analyser et detecter une chaine ou une structure dans ces données.

Un de ses scanners est dédié aux coordonnées GPS...

Le lien du manuel et de l’install :  
[http://digitalcorpora.org/downloads/bulk_extractor/BEUsersManual.pdf](http://digitalcorpora.org/downloads/bulk_extractor/BEUsersManual.pdf)

La commande pour scanner toutes les infos :  
```
bulk_extractor chall9 -o test
```

La commande (plus rapide) pour utiliser uniquement le scanner gps :  
```
bulk_extractor chall9 -x all -e gps -o test
```

Les cordonnées de la ville se trouvent effectivement dans test/gps.txt.

- **other solution 4**
file et fdisk pour avoir les info sur l image.
Pour monter cette partition en read only, il faut faire : (La partition commence à 2048x512 soit 1048576 voir le fdisk )
```
sudo mount -t vfat -o loop,offset=1048576,ro,noexec  chall9 /media/thanatos/
```

On peut ensuite fouiller mais on ne trouve rien de glorieux.
Et si il y avait des fichiers effacés ? Regardons, j’utilise TESTDISK, mais il y a pléthore d’outils.  
[http://www.cgsecurity.org/wiki/TestDisk_Step_By_Step](http://www.cgsecurity.org/wiki/TestDisk_Step_By_Step)

- On choisis Disk chall9 > Intel > Analyse > Quick > P (list files) et on fouille un peut.. on trouve de la doc sur les tags Exifs et surtout :
```
`TestDisk 6.13, Data Recovery Utility, November 2011   
Christophe GRENIER <grenier@cgsecurity.org>   
http://www.cgsecurity.org     * FAT16 >32M               0   1  1    16 254 63     273042   Directory /Files   
Copy done!   
drwxr-xr-x     0     0         0 23-Jul-2013 02:28 .   
drwxr-xr-x     0     0         0 23-Jul-2013 02:28 ..   
>-rwxr-xr-x     0     0   2341273 23-Jul-2013 02:28 revendications.odt   
>-rwxr-xr-x     0     0   1197056 23-Jul-2013 02:28 421_20080208011.doc   
>-rwxr-xr-x     0     0    142336 23-Jul-2013 02:28 Coker.doc`
```

- On trouve un juteux revendication.odt Si on l’ouvre il contient une photo (je passe sous silence les revendications..) avec Libre office on sauve l’image (bouton droit).. et si on scrute les tags exifs on a tous les infos nécessaires...
- Ne reste plus qu’un coup de google map pour trouver le flag.

- **other solution 5**
La première partie pour déterminer ce que c’est est identique, (file/fdisk).

Le dump contient une partition ; il est essentiel de l’extraire avant de pouvoir la monter sur un loop device. On prend la valeur de début de la partition pour renseigner l’option "skip" de dd :
```
dd if=ch9 of=part1 skip=2048
```

Bien, on a donc un fichier part1 correspondant à la première partition, que l’on peut monter :
```
`root@lab32:~/challs/root-me/chat# mkdir bla   root@lab32:~/challs/root-me/chat# mount -t vfat ./part1 bla/`
```

Bon, en parcourant les répertoires. On essaye donc un tool de forensic, dispo sur debian : testdisk.
Nous allons donc extraire à la main, et s’intéresser plus particulièrement à l’image :
```
`root@lab32:~/challs/root-me/chat# cp revendications.odt  revendications.zip   root@lab32:~/challs/root-me/chat# unzip revendications.zip   
Archive:  revendications.zip   
extracting: mimetype   
extracting: Thumbnails/thumbnail.png    
inflating: Pictures/1000000000000CC000000990038D2A62.jpg    
inflating: content.xml    inflating: styles.xml`
```

- **other solution**
utilisation de photorec: *`photorec ch9`*
exif online: http://metapicz.com/#landing

- utilisationde binwalk: Il est possible d’en extraire des fichiers avec la commande **_binwalk -e chall9_** .  
Le contenu sera extrait dans le dossier **__chall9.extracted_**

- Il s’agit d’une partition. Il suffit alors d’utiliser ==DFF (Digital Forensic Framework)== en mode graphique par exemple pour analyser le fichier.: `dff -g`
https://github.com/arxsys/dff

- **Récupérer l’image de la clé USB et exporter son contenu dans votre machine**  
Voici une publication qui explique comment faire ==(FTK est un logiciel libre disponible uniquement sur Windows)== :  
[https://www.linkedin.com/pulse/practical-usb-forensics-test-case-tal-eliyahu](https://www.linkedin.com/pulse/practical-usb-forensics-test-case-tal-eliyahu)
