help:
https://www.youtube.com/watch?v=ct1ovt0hEAw
https://beta.hackndo.com/kerberoasting/

- apres plusieurs prompte, recherche et script echoue l'on sais rabattue sur le tools proposer sur la plateforme root me qui est beaucoup simple.
- telecharger l'outil et installation des dependances
```
git clone https://github.com/p0dalirius/ldap2json.git
```
```
pip install requirements.txt
```
faut savoir que l'outil a une option dump et une option d'analyse que nous allons utiliser.

- lancement de l'outil
```
python analysis/analysis.py -f ../ch31.json
```

- liste des comptes avec SPN
le manuel d'aide:
	> help
```
search_for_kerberoastable_users
```

- avec les infos du user affichons son email
```
object_by_dn CN=Alexandria,CN=Users,DC=ROOTME,DC=local
```
tout en bas voila notre mail
![[Pasted image 20251021031539 1.png]]

- **other solution 1:**
 Blog post Kerberoasting (Author : @pixis) : [https://beta.hackndo.com/kerberoasting/](https://beta.hackndo.com/kerberoasting/)  
 Blog post SPN (Author : @pixis) : [https://beta.hackndo.com/service-principal-name-spn/](https://beta.hackndo.com/service-principal-name-spn/)  
 GitBook page Kerberoast (Author : @Shutdown) : https://thehacker.recipes/ad/movement/kerberos/kerberoast  
 YouTube Video "Manually Parse Bloodhound Data with JQ to Create Lists of Potentially Vulnerable Users and Computers" (Author @ippsec) : [https://www.youtube.com/watch?v=o3W4H0UfDmQ](https://www.youtube.com/watch?v=o3W4H0UfDmQ)  
 Tool JSON processor named JQ (Author : @stedolan) : [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)  
 Tool to make JSON greppable named gron (Author : @tomnomnom) : [https://github.com/tomnomnom/gron](https://github.com/tomnomnom/gron)  
 ldap2json (Author : @p0dalirius) : [https://github.com/p0dalirius/ldap2json](https://github.com/p0dalirius/ldap2json)  
 LDAP offline analysis tool (Author @p0dalirius) : [https://github.com/p0dalirius/ldap2json/tree/master/analysis](https://github.com/p0dalirius/ldap2json/tree/master/analysis)  
 Impacket’s tool to exploit Kerberoasting (Author : @agsolino) : [https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py)  
 RFC Kerberos : [https://www.ietf.org/rfc/rfc4120.txt](https://www.ietf.org/rfc/rfc4120.txt)
