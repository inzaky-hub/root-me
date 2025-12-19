help link: https://repository.root-me.org/Forensic/EN%20-%20Volatility%20cheatsheet%20v2.4.pdf?_gl=1*1dd3rh7*_ga*MTkyOTY3NTQyMi4xNzU5NzAwODA4*_ga_SRYSKX09J7*czE3NjExNDY2OTMkbzMzJGcxJHQxNzYxMTQ2NzY2JGo2MCRsMCRoMA..
vol3: https://volatility3.readthedocs.io/en/latest/getting-started-windows-tutorial.html

- extraction de fichier
```
tar -xvjf ch2.tbz2
```

- recherche d'info sur l image 
```
vol2 -f ch2.dmp imageinfo
```

- utilisation des profiles afin de trouver le bon
```
vol2 -f ch2.dmp  --profile="Win7SP1x86_23418" hashdump
```

- faisons une redirection de la commande precedente dans un .txt ou copions juste la sortie dans  un .txt

- craquons le pass des users avec hashcat ou john 
```
hashcat -m 1000 -a 0 pass.txt /usr/share/wordlists/rockyou.txt
```

```
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt pass.txt
```

	voila donc notre flag: passw0rd

- other solution:
https://www.passware.com/kit-forensic/
outil sympathique qui permet de retrouver et cracker les mots de passe d’un bon nombre de format fichier.  
Nous allons aller dans **Memory Analysis**
