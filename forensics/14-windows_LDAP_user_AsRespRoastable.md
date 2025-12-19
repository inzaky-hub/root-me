good blog: https://www.semperis.com/fr/blog/as-rep-roasting-explained/
help link:
 [7953376-effectuez-un-mouvement-lateral-avec-le-protocole-kerberos](https://openclassrooms.com/fr/courses/7723396-assurez-la-securite-de-votre-active-directory-et-de-vos-domaines-windows/7953376-effectuez-un-mouvement-lateral-avec-le-protocole-kerberos) (openclassrooms.com)
https://beta.hackndo.com/kerberos-asrep-roasting/) (beta.hackndo.com)

- fichier recu: ch32.json
- utilisation de ldap2jon pour l'identification des utilisateurs n'ayant pas l option pre-authentication active:
```
(tools) ┌──(isaak㉿init-1)-[~/Downloads]  
└─$ python ../Documents/ldap2json/analysis/analysis.py -f ch32.json  
[>] Loading ch32.json ... done.  
[]> help  
- searchbase                          Sets the LDAP search base.    
- object_by_property_name             Search for an object containing a property by name in LDAP.    
- object_by_property_value            Search for an object containing a property by value in LDAP.    
- object_by_dn                        Search for an object by its distinguishedName in LDAP.    
- search_for_kerberoastable_users     Search for users accounts linked to at least one service in LDAP.    
- search_for_asreproastable_users     Search for users with DONT_REQ_PREAUTH parameter set to True in LDAP.    
- help                                Displays this help message.    
- exit                                Exits the script.    
[]> search_for_asreproastable_users  
[CN=Fitzgerald,CN=Users,DC=ROOTME,DC=local] => userAccountControl  
- 4260352  
[]> object_by_dn CN=Fitzgerald,CN=Users,DC=ROOTME,DC=local
```
 voila donc le mail que dis je notre flag :)
	
![Texte alternatif de l'image](image/14.png)

- #### **other solutions**
- On parle d’utilisateur AS_REP Roastable lorsque la pré-authentification Kerberos n’est pas requise pour cette utilisateur. Nous pouvons alors demander un TGT (Ticket Granting Ticket) au KDC (Key Distribution Center) à son nom et cracker une partie de la réponse KRB_AS_REP, qui contient le TGT et une clé de session chiffré avec son hash NT. Un attaquant peut tenter de retrouver le password de ce compte de domaine via du bruteforce en offline.
```
curl -s http://challenge01.root-me.org/forensic/ch32/ch32.json | jq -r '."DC=local"."DC=ROOTME"."CN=Users" | to_entries[] | select(.key|startswith("CN=")) | .value | select( .userAccountControl > 4194304) | .mail'
```

On peut récupérer la valeur de l’attribut userAccountControl du compte de domaine fitzgerald.landry (avec la [console d’analyse de ldap2json](https://github.com/p0dalirius/ldap2json/tree/master/analysis)) et vérifier les flags de l’attribut avec l’outil [msFlagsDecoder](https://github.com/p0dalirius/msFlagsDecoder) développé par Podalirius :
```
- ❯ python3 analysis.py -f /tmp/ch32.json
- [>] Loading /tmp/msFlagsDecoder/ch32.json ... done.
- []> searchbase CN=Fitzgerald,CN=Users,DC=ROOTME,DC=local
- [DC=local,DC=ROOTME,CN=Users,CN=Fitzgerald]> object_by_property_name userAccountControl
- [CN=Fitzgerald,CN=Users,DC=ROOTME,DC=local] => userAccountControl
-  - 4260352
  
- ❯ ./msFlagsDecoder.py userAccountControl 4260352 --bits --colors
- [>] userAccountControl value       : .........1.....1......1.........
- [█] DONT_REQ_PREAUTH               : .........1......................
- [█] DONT_EXPIRE_PASSWORD           : ...............1................
- [█] NORMAL_ACCOUNT
```

- le flag `DONT_REQUIRE_PREAUTH` a bien une valeur de  
4194304 / 0x00400000 / 0b010000000000000000000000

Mais une recherche de valeur supérieure à ce nombre peut également retourner des valeurs avec d’autres flags tels que `ERROR_PASSWORD_EXPIRED` ou `PARTIAL_SECRETS_ACCOUNT`, comme l’indique [la documentation](https://learn.microsoft.com/fr-fr/troubleshoot/windows-server/identity/useraccountcontrol-manipulate-account-properties)

faisons donc un petit script en python pour parser le JSON et vérifier à chaque utilisateur si le bit du flag est bien set :
```
1. import json
2. f = open("ch32.json", "r")
3. j = json.load(f)
4. for username in j['DC=local']['DC=ROOTME']["CN=Users"]:
5.     user = j['DC=local']['DC=ROOTME']["CN=Users"][username]
6.     if not type(user) is dict or not "userAccountControl" in user:
7.             continue
8.     binary = bin(user["userAccountControl"])[::-1]
9.     if len(binary) >= 22 and binary[22] == '1':
10.             print(user["mail"])
```

- En regardant sur le net je suis tombé sur cet article :
[https://seuforia.wordpress.com/2018/09/19/do-not-require-kerberos-pre-authentication-for-users-create-by-ambari-on-ad/](https://seuforia.wordpress.com/2018/09/19/do-not-require-kerberos-pre-authentication-for-users-create-by-ambari-on-ad/)
celui-ci nous indique qu’un utilisateur ASRepRoastable à comme valeur "userAccountControl" : 4260352 .
```
sed 's/"cn":/\n"cn":/g' ch32.json | sed -En '/4260352/ s/^.* "mail": "([^"]+).*$/\1/p''
```
