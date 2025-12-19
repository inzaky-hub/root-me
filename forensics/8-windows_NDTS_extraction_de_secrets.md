link: https://github.com/SecureAuthCorp/impacket/

- extraction de fichier ch34.zip
Contexte : Sauvegarde offline d’un contrôleur de domaine (fichiers fournis : `Active Directory/ntds.dit`, `Active Directory/ntds.jfm`, `registry/SYSTEM`, `registry/SAM`, `registry/SECURITY`).
Objectif : Extraire la clé Kerberos `aes256-cts-hmac-sha1-96` associée au compte `krbtgt` (flag du CTF).

- Installation (si nécessaire)
```
python3 -m pip install --user impacket
```

- Extraction basique des hashes NTLM
s'assurer d'etre dans le bon repertoire
```
secretsdump.py -ntds "Active Directory/ntds.dit" -system "registry/SYSTEM" LOCAL
```

`secretsdump.py` utilise la ruche `SYSTEM` pour extraire la bootKey, déchiffre la PEK dans `ntds.dit` et déchiffre les hash stockés (LM/NT).

- On a procede a une redirection de la sortie afin de facilite la recherche du compte **krbtgt** (utilisation de la recherche rapide "CTRL + F")
```
secretsdump.py -ntds "Active Directory/ntds.dit" -system "registry/SYSTEM" LOCAL > note.txt
```

voila donc notre flag: 85c422e6d4f4e340b445c6a3f16d8d7b25bfdf290d956134bc0d5b6ab272b475

