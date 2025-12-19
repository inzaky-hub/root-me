link: https://github.com/ANSSI-FR/bmc-tools

- extraction et identification des donnees
```
unzip ch16.zip
```

**fichier E01** (`image_forensic.e01`), c’est-à-dire une **image disque forensique** au format **EnCase (EWF : Expert Witness Format)**.  la commande qui suis nous donne assez d'info sur le fichier notemmant une mention RDP.
```
ewfinfo image_forensic.e01
```

- On convertit proprement l’E01 en image brute (`.raw`) pour pouvoir utiliser tous les outils.
```
ewfexport -t raw image_forensic.e01
```

- on a maintenant un .raw qu on va chercher a identifier un peu plus avec file, mmls, fls et extraire le contenu.
```
mkdir extracted
tar -xvf raw.raw -C extracted
```

- on a un nouveau fihier .bmc bien evidemment on essai binwalk file et autre mais apres quelque clic google on a une idee du fichier et on tombe sur un git. 
```
git clone https://github.com/ANSSI-FR/bmc-tools.git
```
```
python3 bmc-tools/bmc-tools.py -s extracted/bcache24.bmc -d mnt -v
```

apres utilisation de l'outil dont le manuel se trouve sur  le git on a une sortie de plein d image dans notre repertoire de mnt. 
avec l'explorateur de fichier on peut constater des images avec des mots suspects et en regardant tout en bas on a notre flag au complet.

![Texte alternatif de l'image](image/22.png)

- #### **other solutions**
1. Tout d’abord, je recherche quel est le type de ce fichier **.e01**:
```
file image_forensic.e01
EWF/Expert Witness/EnCase image file format
```

on pouvais monter le fichier et faire extraire celui qu il ya a l interieur et utiliser le git etc.
```
ewfmount image_forensic.e01 tmpmount/

file tmpmount/ewf1

tar -xvf ewf1

git clone https://github.com/ANSSI-FR/bmc-tools.git
```

2. utilisation de XMOUNT
**xmount** est un petit binaire qui va nous permettre de monter et convertir tout un tas de choses, sauf qu’il se base entre autre sur la **libewf**, qui lui permet de monter des fichiers **EnCase**, **e01**, **e02**...
link: [https://www.pinguin.lu/xmount](https://www.pinguin.lu/xmount)

3. utilisation de dff (digital forensics framework)
`root@kali:#dff -g`  
j’ouvre mon image : Open Evidence >> EWF Format >> image_forensic.e01

clic droit dessus >> Extract, puis je récupère une archive de type `POSIX tar archive (GNU)`
On l’ouvre:
`tar -xvf Root-me.tar`
On récupère un fichier `bcache24.bmc`

bcache24.bmc dans DFF cette fois en tant que RAW data
J’ouvre l’éditeur hexa dans l’onglet "Pixel" , à gauche dans les options je mets "Résolution" au minimum et "Scale factor" au maximum.
En scrollant, on tombe sur l’image à l’envers.

4. Lecture du fichier .bmp
L’outil BMC Viewer est notamment disponible à cette adresse : [https://turbolab.it/scarica/9](https://turbolab.it/scarica/9)


