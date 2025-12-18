cheat volatility: [https://repository.root-me.org/Forensic/EN - Volatility cheatsheet v2.4.pdf?_gl=1*1ftbqeh*_ga*MTkyOTY3NTQyMi4xNzU5NzAwODA4*_ga_SRYSKX09J7*czE3NjA4MjkzMjEkbzEyJGcxJHQxNzYwODI5ODM5JGo0OCRsMCRoMA](https://repository.root-me.org/Forensic/EN%20-%20Volatility%20cheatsheet%20v2.4.pdf?_gl=1*1ftbqeh*_ga*MTkyOTY3NTQyMi4xNzU5NzAwODA4*_ga_SRYSKX09J7*czE3NjA4MjkzMjEkbzEyJGcxJHQxNzYwODI5ODM5JGo0OCRsMCRoMA)..

- extraction du fichier **ch2.tbz2**

```
tar -xvf **ch2.tbz2**
```

on a bien notre dump memoire (**ch2.dmp**) comme le signifie l’enonce.

- trouver des infos sur le dump notamment le profile

```
vol2 -f **ch2.dmp** imageinfo
```

- Listons **toutes les clés du registre en mémoire**

```
vol2 -f **ch2.dmp** --profile=Win7SP1x86_23418 hivelist
```

- nous allons nous concentrer sur l’offset ou adresse virtuel de “\REGISTRY\MACHINE\SYSTEM”

```
vol2 -f **ch2.dmp** --profile=Win7SP1x86_23418 printkey -o 0x8b21c008
```

- nous avons des sous cles notamment “ControlSet001”

```
vol2 -f **ch2.dmp** --profile=Win7SP1x86_23418 printkey -o 0x8b21c008 -K "ControlSet001"
```

- apres recherche sur google on a trouve les sous cles qui contiennent le nom de la machine:

```
vol2 -f **ch2.dmp** --profile=Win7SP1x86_23418 printkey -o 0x8b21c008 -K "ControlSet001\Control\ComputerName\ComputerName"
```
