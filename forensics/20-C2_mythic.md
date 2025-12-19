- objectif
recuperer les infos ayant ete exfiltres.
- identification des fichiers 
```
> unzip ch43.zip    
Archive:  ch43.zip  
  creating: C2_Mythic/  
 inflating: C2_Mythic/Mythic_C2.pcap     
 inflating: C2_Mythic/medusa.py
```

- Désobfuscation du Code Agent
$$
# decode_xor.py -- exécutez dans une VM isolée / labo

import base64

from itertools import cycle

  

# remplacez par votre chaîne complète b64

b64_fragment = b"******bloc a deofusquer*******"

# clé telle qu'utilisée dans votre snippet (octets ASCII)

key = b'66ec80c110f42039d7d0dcb3db5f43b7'

  

# assure padding

while len(b64_fragment) % 4 != 0:

b64_fragment += b'='

  

decoded = base64.b64decode(b64_fragment)

plain = bytes([c ^ k for c,k in zip(decoded, cycle(key))])

  

# écrire dans un fichier texte pour examen (ne pas exécuter ce fichier)

with open('deobfuscated_output.txt', 'wb') as f:

f.write(plain)

  

print("Déchiffrage terminé — résultat dans deobfuscated_output.txt")
$$

on a une sortie detaille du code et en regardant tout en bas de celui-ci on s'appercois d'information importante comme cette fonction qui contient les cle.
$$
def __init__(self):
        self.socks_open = {}
        self.socks_in = queue.Queue()
        self.socks_out = queue.Queue()
        self.taskings = []
        self._meta_cache = {}
        self.moduleRepo = {}
        self.current_directory = os.getcwd()
        self.agent_config = {
            "Server": "http://10.0.3.221",
            "Port": "80",
            "PostURI": "/data",
            "PayloadUUID": "d4c81c06-eef1-42e3-8c7a-dfcdf4c6fc88",
            "UUID": "",
            "Headers": {"User-Agent": "Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko"},
            "Sleep": 3,
            "Jitter": 23,
            "KillDate": "2024-06-18",
            "enc_key": {"dec_key": "DmVWKchtzpH0jA5/EXN2N5ORGwhYBGFdUBm7iel2r/0=", "enc_key": "DmVWKchtzpH0jA5/EXN2N5ORGwhYBGFdUBm7iel2r/0=", "value": "aes256_hmac"},
            "ExchChk": "True",
            "GetURI": "/index",
            "GetParam": "q",
            "ProxyHost": "",
            "ProxyUser": "",
            "ProxyPass": "",
            "ProxyPort": "",
        }
$$

- Déchiffrement des Communications
on va chercher a rendre les flux http comprehensible parce qu'avec tous ces fichiers chiffres tu sais pas qu est ce qui passe mais tu sais que ya quelque chose.
bon !!!!!
exportons les objets HTTP du pcap: FILE > EXPORT OBJECT > HTTP puis sauvegarder tout.
on constatera la presence de plusieurs fichiers contenant les communications chiffres qui sont plus chiffres pour nous puisse qu'on a les cles haahahaha :) je ris car ca m'a pris du temp pour comprendre cela. Au depart j'etais trop concentre sur le pcap et quand je dis concentrer je l'etait. 

$$
import os

import base64

import json

from Crypto.Cipher import AES

from Crypto.Util.Padding import unpad

import hmac

from hashlib import sha256

  

def decrypt_medusa_payload(encrypted_b64, key_b64):

try:

key = base64.b64decode(key_b64)

encrypted_data = base64.b64decode(encrypted_b64)

# Structure: UUID (36 bytes) + IV (16) + ciphertext + HMAC (32)

uuid = encrypted_data[:36].decode('utf-8', errors='ignore')

iv = encrypted_data[36:52]

ciphertext = encrypted_data[52:-32]

received_hmac = encrypted_data[-32:]

# Vérification HMAC

calculated_hmac = hmac.new(key, iv + ciphertext, sha256).digest()

if not hmac.compare_digest(calculated_hmac, received_hmac):

return None, "HMAC verification failed"

cipher = AES.new(key, AES.MODE_CBC, iv)

decrypted = unpad(cipher.decrypt(ciphertext), AES.block_size)

return uuid, json.loads(decrypted.decode('utf-8', errors='ignore'))

except Exception as e:

return None, f"Erreur: {e}"

  

# Clé de déchiffrement du code désobfusqué

key_b64 = "DmVWKchtzpH0jA5/EXN2N5ORGwhYBGFdUBm7iel2r/0="

  

# Dossier contenant les exports HTTP

EXPORT_DIR = "/home/isaak/Downloads/C2Mythic/C2_Mythic/export" \

"" # modifie selon ton dossier

  

print("🔓 DÉCHIFFREMENT DES PAYLOADS MEDUSA...")

print("=" * 50)

  

# Parcourir tous les fichiers du dossier

for filename in os.listdir(EXPORT_DIR):

filepath = os.path.join(EXPORT_DIR, filename)

if not os.path.isfile(filepath):

continue

  

print(f"\n🔎 Traitement du fichier : {filename}")

  

with open(filepath, "rb") as f:

data = f.read()

  

# Chercher tous les blobs base64

base64_blobs = []

for line in data.splitlines():

try:

decoded = base64.b64decode(line, validate=True)

base64_blobs.append(line)

except Exception:

continue

  

print(f" {len(base64_blobs)} blobs base64 détectés.")

  

# Déchiffrer chaque payload

for i, blob_b64 in enumerate(base64_blobs, 1):

uuid, decrypted_data = decrypt_medusa_payload(blob_b64, key_b64)

if uuid:

print(f"\n📦 Payload {i}:")

print(f"✅ UUID: {uuid}")

print(f"📄 Données déchiffrées:")

print(json.dumps(decrypted_data, indent=2, ensure_ascii=False))

  

# Recherche automatique des flags

data_str = json.dumps(decrypted_data)

flags = []

start = 0

while True:

start = data_str.find("FLAG{", start)

if start == -1:

break

end = data_str.find("}", start) + 1

flags.append(data_str[start:end])

start = end

if flags:

for flag in flags:

print(f"\n🎯 FLAG TROUVÉ: {flag}")

else:

print(f"❌ Blob {i} non déchiffrable : {decrypted_data}")

  

print("\n" + "=" * 50)

print("✅ Analyse terminée!")
$$

apres un peu de lecture voila notre flag:
```
  
📦 Payload 1:  
✅ UUID: 925ad889-7966-468b-90fd-725a6f693e24  
📄 Données déchiffrées:  
{  
 "action": "post_response",  
 "responses": [  
   {  
     "task_id": "aae8337c-fc3a-4a96-a5d5-aae3c4691158",  
     "user_output": "HACKDAY{Myth1c_C2_15_FuN}\n",  
     "completed": true  
   }  
 ]  
}
```

NB: remerciement a GPT et DEEPSEEK.
je mets ca c'est qu'il on jouer un role incroyable.


