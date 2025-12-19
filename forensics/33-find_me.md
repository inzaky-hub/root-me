- INFORMATIONS GÉNÉRALES
**Challenge** : Forensique - Analyse de dump mémoire Windows  
**Fichier** : `dump` (537 MB)  
**Système** : Windows 7 SP1 x86 (Profile: Win7SP1x86_23418)  
**Date du dump** : 2016-09-15 10:12:31 UTC+0000  
**Résultat** : ✅ **RÉUSSI**  
**Flag final** : `K33p4ss_its_a_gR3at_T00l_4_P@sSw0rD!`

- Énoncé
Votre fiston est un geek et veut vous prouver qu'il a des capacités à vous cacher 
de l'information. Dans un instant de faiblesse, vous avez laissé ouvert votre 
session et il a eu le temps de vous faire une blague. Il semblerait qu'il ait 
même réussi à trouver votre mot de passe de session ! Trouvez le mot de passe 
de validation dans le dump.

- Identification du dump mémoire
```
vol2 -f dump imageinfo
```
Résultat : Win7SP1x86_23418 (Windows 7 SP1 32-bit)

- Extraction du clipboard (Information critique #1)
```
vol2 -f dump --profile=Win7SP1x86_23418 clipboard
```
resultat:
```
Session    WindowStation Format           Handle   Object       Data
---------- ------------- ---------------- -------- ------------ ------------------
1          WinSta0       CF_UNICODETEXT   0xd02d1  0xffbb3fb0   R3sqdl3Fuuz2ZdbdYsf56opFFLe9sAsx
```

- Inventaire des processus suspects
```
vol2 -f dump --profile=Win7SP1x86_23418 pslist
```

Processus suspects identifiés :
- **TrueCrypt.exe** (PID 3224) - 10:11:20 → **Volume chiffré**
- **mspaint.exe** (PID 2644) - 10:11:13 → Ouverture d'image
- **notepad.exe** (PID 3716) - 10:11:59 → Notes/instructions
- **firefox.exe** (PID 2720) - 10:11:15 → Recherches web

TrueCrypt est immédiatement suspect car c'est un outil de chiffrement. La chronologie montre une séquence d'actions organisée par le "fils".

- Recherche du conteneur TrueCrypt
```
vol2 -f dump --profile=Win7SP1x86_23418 filescan | grep -i "desktop"
```
decouverte: `0x000000001ee20110   3   0 R--rwd \Device\HarddiskVolume2\Users\info\Desktop\findme`

Les conteneurs TrueCrypt sont souvent stockés sur le bureau ou dans des dossiers évidents. Le fichier "findme" correspond parfaitement au thème du challenge.

- Extraction du conteneur TrueCrypt
```
vol2 -f dump --profile=Win7SP1x86_23418 dumpfiles -Q 0x000000001ee20110 --dump-dir dos
```
Fichier de 3.1 MB extrait : `file.None.0x84e13338.dat`

- Déchiffrement du volume TrueCrypt
```
echo "R3sqdl3Fuuz2ZdbdYsf56opFFLe9sAsx" | sudo cryptsetup open --type tcrypt dos/file.None.0x84e13338.dat findme
```
```
sudo mount /dev/mapper/findme /mnt/findme
```
Volume déchiffré avec succès contenant 3 fichiers :
- `flag.png` (12 KB)
- `readme.odt` (1.8 MB)
- `readme.txt` (72 bytes)
Le mot de passe du clipboard était logiquement celui du volume TrueCrypt. Utilisation de `cryptsetup` car il supporte TrueCrypt.

`/dev/mapper/findme_decrypted` est un **device mapper** (périphérique virtuel) créé par `cryptsetup` dans `/dev/mapper/` - c'est le comportement normal de Linux pour les volumes chiffrés. C'est comme un disque virtuel déchiffré. (c'est la qu est mis un volume dechiffré.)

- Analyse des leurres et Découverte du fichier caché (Information critique #2)
Contenu de readme.txt :
Image dessinée avec texte "The flag is not here"
Ces deux fichiers sont des leurres pour distraire l'attaquant. Vérification systématique de tous les fichiers, même évidents, pour identifier les leurres et se concentrer sur les vraies pistes.
les fichiers '.odt' etant des fichiers zippé on va le decompressé et constater ce qu'on aura.
```
unzip -l readme.odt
```
decouverte: `1749806  2016-09-05 13:47   data/my_safety_box`

- Extraction et identification du fichier caché
```
unzip readme.odt data/my_safety_box
file data/my_safety_box
```
resultat: ``data/my_safety_box: Keepass password database 2.x KDBX

- Tentative d'ouverture de KeePass et Recherche d'indices supplémentaires
**Première tentative :** Utilisation du mot de passe TrueCrypt → **ÉCHEC**
**Problème :** Le mot de passe KeePass est différent du mot de passe TrueCrypt.
**Choix méthodologique :** Essayer d'abord le mot de passe connu est logique, mais échec indique un mot de passe différent.

```
strings 3716.dmp | grep -i "pass"
```
decouverte: `https://crackstation.net/CrackStation - Online Password Hash Cracking`

Le fils a visité CrackStation pour cracker des hash ! Indice vers l'utilisation de hash Windows.
**Choix méthodologique :** Analyse des dump mémoire de processus suspects (Notepad) pour trouver des traces d'activité.

- Extraction des hash Windows (Information critique #3) & Craquage du hash NTLM
```
vol2 -f dump --profile=Win7SP1x86_23418 hashdump
```
resultat: `info:1002:aad3b435b51404eeaad3b435b51404ee:dc3817f29d2199446639538113064277:::`

Hash NTLM de l'utilisateur 'info' : `dc3817f29d2199446639538113064277`
**Choix méthodologique :** La découverte de CrackStation indique que le fils a cracké des hash. Les hash Windows sont une source classique de mots de passe.

- Crackons le hash
Méthode 1 : CrackStation (en ligne - rapide)
```
# Va sur https://crackstation.net/
# Colle le hash : dc3817f29d2199446639538113064277
```

Méthode 2 : John The Ripper (local)

Méthode 3 : Hashcat


voila ainsi le mot de passe: `#1Godfather`

- Accédons maintenant à keepass avec notre mot de passe trouver
```
keepass2 data/my_safety_box
```
on constate plusieurs mot de passe mais aucun n est le flag c'est pas faute d'avoir essayer mais bon c'est comme ca.
deja exportons les mots de passe, dans mon cas je l ai fais en '.csv' et lorsqu'on scrolle on tombe sur une chaine differente des autres qui apparemment est en base 64.

- Decodons la chaine de caractere
```
import base64

encoded = "TON_STRING_ICI"
current = encoded

for i in range(10):  # Change 10 par le nombre voulu
    try:
        current = base64.b64decode(current).decode('utf-8')
        print(f"Itération {i+1}: {current}")
    except:
        break

print(f"\nFINAL: {current}")
```
j'ai eu le flag a 21 iterations mais j'ai mis 100 lorque j'ai exécuté. l'automatisation y'a pas mieux :)

Flag trouvé : `K33p4ss_its_a_gR3at_T00l_4_P@sSw0rD!`

- **POINTS CLÉS**
**Flag validé** : `K33p4ss_its_a_gR3at_T00l_4_P@sSw0rD!`
**4 couches de protection** : TrueCrypt → ODT → KeePass → Base64
**Erreur critique** : Réutilisation du mot de passe Windows
**Méthode gagnante** : Analyse clipboard → Hash Windows → CrackStation
**Outils utilisés** : 8 (Volatility, cryptsetup, mount, KeePass, Python, etc.)
**Fausses pistes** : 3 (image, documents texte, mot de passe TrueCrypt)


