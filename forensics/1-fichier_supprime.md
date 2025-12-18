- nous avons un ".gz" donc allons chercher a dezippe:
```
tar -xvf ch39.gz
```

- chercher a savoir a quoi l'on a affaire:
```
file usb.image
```

- on peut aussi lister les fichiers ou voir les partirions
```
fls usb.image
```

```
fdisk -l usb.image
```
on constae qu'il n y a pas d epartition.

- nous avons comme sortie avec une image et d'autre contenu. extrayons l'image a partir de son inode.
```
icat usb.image 5 > anonyme.png
```

- rien de marquer dessus, allons voir les meta donnee
```
exiftool anonyme.png
```

il y'a bien un nom qui est le flag:
```
javier_turcot
```
