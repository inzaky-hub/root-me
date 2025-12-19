- extraction du fichier:
```
tar -xvjf ch2.tbz2
```

- dump memoire, volatility pour lister les processus et avoir un chemin detaille des repertoires de chaque process.
```
vol3 -f ch2.dmp windows.pslist.PsList
```
```
vol3 -f ch2.dmp windows.pstree.PsTree
```

- pour ce challenge j'ai splitter mon terminal afin de regarder et les pid, ppid et les path. j'ai un peu tourner en rond en pensant au depart de le malware etait le processus: 

**** 3144 3152    winpmem-1.3.1.  0x87cbfd40      1       23      1       False   2013-01-12 16:59:17.000000 UTC  N/A     \Device\HarddiskVolume1\Users\JOHNDO~1\AppData\Local\Temp\imagedump\winpmem-1.3.1.exe   winpmem-1.3.1.exe  ram.dmpC:\Users\JOHNDO~1\AppData\Local\Temp\imagedump\winpmem-1.3.1.exe**

car il s'executait dans un repertoire temporaire et avait ete lance avec cmd mais je me trompais. je me suis donc interesse au cmd, j'en ai lister deux qui avait pour parent **explorer.exe** et **iexplorer.exe**.

- verification des path d'execution des parents. on peut donc constater que **iexplorer.exe** a demarrer depuis un repertoire temporaire.
path: C:\Users\John Doe\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\iexplore.exe

on a donc notre flag:
MD5: 49979149632639432397b3a1df8cb43d

- #### **other solution**
1. Une autre solution consiste à utiliser l’outil redline de Mandiant (uniquement sous windows et gratuit) :  
[http://www.mandiant.com/resources/download/redline](http://www.mandiant.com/resources/download/redline)

- cliquez sur "Analyze data" -> "From a saved memory file"
- sélectionnez ch2.dmp
- une fois le dump scanné, rendez-vous dans l’onglet "Process" à gauche et la liste de process apparait

Redline va automatiquement marquer d’une icône rouge les process suspects et donner le path du process.  
On trouve ainsi tout de suite  
`C:\Users\John Doe\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\iexplore.exe`

