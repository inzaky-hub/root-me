quelque site pour l'analyse de log:
https://cloudvyzor.com/
https://github.com/danielefavi/log-explorer
https://www.logviewplus.com/
https://goaccess.io/download
https://www.ibm.com/docs/en/taddm/7.3.0?topic=taddm-log-analyzer
https://www.screamingfrog.co.uk/log-file-analyser/

help link: https://baotdvi.wordpress.com/2019/02/25/root-me-forensic/

- Contexte : Analyse des logs du serveur web (ch13.txt) dans le cadre d'un défi CTF pour identifier les données exfiltrées par un attaquant.

- Description de l'Attaque

L'attaquant a exploité une vulnérabilité dans l'endpoint /admin/?action=membres du serveur web, en injectant des payloads SQL malveillants via le paramètre order. Cette vulnérabilité permettait d'exécuter des requêtes SQL arbitraires dans la clause ORDER BY, sans sanitisation appropriée des entrées.

L'attaque utilisée est une **injection SQL aveugle basée sur le temps** (time-based blind SQL injection), où l'attaquant infère les données en mesurant les délais de réponse du serveur. Les payloads étaient encodés en Base64 et URL-encodés pour être inclus dans le paramètre order.

- Analyse des Logs

Les logs dans ch13.txt contiennent des requêtes GET avec des timestamps et des payloads SQL encodés. Chaque payload cible un caractère spécifique d'un champ dans la table membres pour l'utilisateur avec id=1. Voici le processus de l'attaquant :

**Structure des payloads** :
    Chaque requête injecte une sous-requête SQL via order, par exemple :
```
ASC,(SELECT CASE WHEN (FIELD(substring(bin(ascii(substring(flag,1,1))),1,2),'00','01','10','11')=1) THEN 0 ELSE SLEEP(2) END FROM membres WHERE id=1)
```
Le payload extrait un caractère à une position donnée (par exemple, substring(flag,1,1)), convertit en ASCII (ascii(...)), puis en binaire (bin(...)), et teste des bits spécifiques avec FIELD.

**Mécanisme d'exfiltration**
L'attaquant extrait 7 bits par caractère en 4 requêtes :
    - 3 requêtes testent 2 bits chacune (00, 01, 10, 11) avec des délais de 0s, 2s, 4s, ou 6s (via SLEEP).
    - 1 requête teste 1 bit (0 ou 1) avec des délais de 2s ou 4s.
Les délais indiquent la valeur correcte :
    - 0s → FIELD=1 (par exemple, 00 pour les bits 1-2).
    - 2s → FIELD=2 (par exemple, 01).
    - 4s → FIELD=3 (par exemple, 10).
    - 6s → FIELD=4 (par exemple, 11).
 Les 7 bits sont complétés par un 0 initial pour former 8 bits, convertis en un caractère ASCII.

- utilisation de script python pour la reconstruction
$$
import sys
from datetime import datetime

f = open("ch13.txt", "r")
timeList = []

char = ''
flag = ""

for line in f:
    timeList += [line[30:38]]
f.close()

for i in range(len(timeList)-1):
    timeleft = datetime.strptime(timeList[i+1], '%H:%M:%S') - datetime.strptime(timeList[i], '%H:%M:%S')
    if i % 4 in [0, 1, 2]:
        if str(timeleft) == '0:00:00':
            char += '00'
        if str(timeleft) == '0:00:02':
            char += '01'
        if str(timeleft) == '0:00:04':
            char += '10'
        if str(timeleft) == '0:00:06':
            char += '11'
    if i % 4 == 3:
        if str(timeleft) == '0:00:02':
            char += '0'
        if str(timeleft) == '0:00:04':
            char += '1'
        flag += chr(int(char, 2))
        char = ''

print("Flag exfiltré :", flag)
$$

voila donc notre flag: g9UWD8EZgBhBpc4nTSAS

