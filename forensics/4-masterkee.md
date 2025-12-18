- extraction des fichiers 
-  methode d'approche:
on peut constater la presente de deux fichier un '.DMP' qui est dump memoire des crash windows et un '.kdbx' qui est une base de donne keepass.
on pourait penser a prendre volatility afin d'extraire des infos du dump comme je l'ai fais mais apres plusieurs erreur, echec et recherche il se trouve que l image n'est pas complete c'est un mini dump. pour lire dans ce type de dump qui n'est pas un dump memoire complet il ya d'autre outil comme 'WinDbg'.

-  installer windbg et ouvrir le dump
https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/
![[Pasted image 20251019202523 1.png]]

![Texte alternatif de l'image](image/im4.png)

on peut constater la presence de KeePass : Cela fait référence au logiciel KeePass
+0x304fde : Il s'agit d'un décalage mémoire (offset) dans le programme KeePass.
(00000000 003e4fde): C'est une autre adresse mémoire, probablement extraite du dump ou de la trace de pile, indiquant où se situe cette adresse spécifique dans la mémoire.

- apres recherche l'on a trouver des outils pour l'extraction des mots de passe keepass d'un dump. utilisons [keepass-password-dumper](https://github.com/vdohney/keepass-password-dumper) sur le dump memoire.
NB: installer NET8.0 SDK si besoin ou version ulterieur.
```
dotnet run "Z:\ch45\MasterKee.DMP"
```
![[Pasted image 20251019203638 1.png]]
voila donc notre mot de passe. faut savoir aussi que le debut du mot de passe n'est souvent pas perceptible.

- ouvrir la base de donnee de keepass avec le mot de passe en question
```
Here_Is_My_V3ry_S3cr3t_P4ssw0rd2024!
```

voila notre flag: RM{Upd4T3_KeEPas5_t0_2.54}

- other solution:
Un petit utilitaire nommé `KeePwn` peut être utilisé pour résoudre le challenge.
Nous pouvons donc l’utiliser pour casser la base de données.
	KeePwn parse_dump -d MasterKee.DMP

