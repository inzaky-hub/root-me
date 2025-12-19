link: hackmd.io/@TuX-/BymMpKd0s#Registry

- Objectif
Identifier l'adresse IP du serveur interne ciblé par les cybercriminels et le port utilisé.

- **Identification du processus malveillant**
```
vol2 -f ch2.dmp --profile=Win7SP1x86_23418 pslist
vol2 -f ch2.dmp --profile=Win7SP1x86_23418 pstree
```
**Résultat :** Identification d'**iexplore.exe (PID 2772)** comme processus suspect :
Lancé depuis un chemin inhabituel : `C:\Users\John Doe\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\` ,  A spawné un processus **cmd.exe (PID 1616)**, comportement anormal pour un navigateur web

- Analyse des connexions réseau
```
vol2 -f ch2.dmp --profile=Win7SP1x86_23418 netscan | grep 2772
```
**Observations :**
Connexion locale suspecte : `127.0.0.1:49178 → 127.0.0.1:12080` (ESTABLISHED)
Cette connexion via localhost suggère l'utilisation d'un proxy ou d'un relai TCP

- **Analyse de la console et historique des commandes**
faut savoir qu'avant de le faire j'ai d'abord fais un memdump du process et stringer jusqu'a etre fatigue. le truc c'est que je savais pas quoi chercher en faisant les strings.
```
vol2 -f ch2.dmp --profile=Win7SP1x86_23418 consoles
```
**Découverte critique :**
```
ConsoleProcess: conhost.exe Pid: 2168
AttachedProcess: cmd.exe Pid: 1616

CommandHistory:
- Application: tcprelay.exe
- Application: whoami.exe
- Application: cmd.exe
```
L'exécution de **tcprelay.exe** confirme l'utilisation d'un outil de pivoting réseau, typique des attaques APT.

- **Extraction de l'adresse IP cible**
```
strings ch2.dmp | grep -i tcprelay -A 5 -B 5
```
**Résultat décisif :** Découverte de la cible dans la mémoire brute :
**Adresse IP : 192.168.0.22**
**Port : 3389** (RDP - Remote Desktop Protocol)
Cette commande a permis d'extraire le contexte autour de l'exécution de tcprelay, révélant les paramètres de connexion vers le serveur interne ciblé.

- Indicateurs de compromission (IoC)

| Type                  | Valeur                          | Description                      |
| --------------------- | ------------------------------- | -------------------------------- |
| Processus malveillant | iexplore.exe (PID 2772)         | Faux navigateur utilisé comme C2 |
| Processus enfant      | cmd.exe (PID 1616)              | Shell de commande spawné         |
| Outil d'attaque       | tcprelay.exe                    | Relai TCP pour pivoting réseau   |
| IP cible              | 192.168.0.22                    | Serveur interne visé             |
| Port cible            | 3389                            | RDP (Remote Desktop Protocol)    |
| Chemin suspect        | AppData\Roaming...\iexplore.exe | Emplacement non standard         |
- other solutions
_Ps : Yarascan permet d’effectuer des recherches d’octets/ strings dans les espaces d’adressage physiques ou virtuels_
```
volatility -f ch2.dmp --profile=Win7SP1x86 yarascan -Y 'iexplore'
```
