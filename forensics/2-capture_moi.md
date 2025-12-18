- fichier en .zip cherchons deja à dezipper
```
unzip ch42.zip
```

- la decompression donne une image et un ".kdbx" qui correspond a une base de donnee de mot de passe de keepass. il faut donc acceder à cette BD.
- apres recherche l'image est soumis a une la faille acropalyse. Pour la reconstituer on devra utiliser sois le site sois le git.
```
https://acropalypse.app/
```

```
https://github.com/frankthetank-music/Acropalypse-Multi-Tool
```

- Utilisation du GIT/
apres installation des dependances, on execute.
```
pip install requirement.txt
```
```
python gui.py 
```

- importation de notre image a reconstituer et on genere l'image reconstituer puis on telecharge.
- le mot de passe de keepass y est assigner, on peut donc telecharger keepass. 

dans mon cas je l'ai fais sur windows j'ai importer  la BD et essayer d'editer le mot de passe afin de l'afficher en claire.
