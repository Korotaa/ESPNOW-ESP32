## I. Introduction sur le protocole ESP_NOW

ESPNOW est un protocole de communication sans fil pour des réponses rapides et un contrôle à faible consommation dévéloppé par espressif pour les puces ESP32, ESP32-S, ESP32-C et ESP8266. Il est dédié aux réseaux locaux et est basé sur l'interface wifi. Il permet aux cartes ESP d'écahnger directement des données sans pour autant avoir besoin d'un routeur ou d'un autre équipement réseau quelconque. Ce qui fait de lui un protocole de communication très rapide(Latence faible). Il peut fonctionner avec WiFi ou Bluetooth LE. 


### 1. Caractéristiques du protocoles


* Bande passante: espnow utilise la bande passante de 2.4GHz. Il repose sur le même principe de fonctionnement que la souris sans fil.
* Débit : il a débit par défaut de **1Mbps**.
* Porté : espnow permet d'ateindre une portée d'environ 100m.  Maintenant avec le mode longue portée on est capable d'atteindre les 480m.
* Taille du réseau : Pour les pairs cryptés on peut avoir 10 pairs maximum. Pour des pairs non cryptés on peut aller jusqu'à 20.
* Taille du payload : On ne peut envoyer que des paquets de tailles inférieures ou égale à 250octets.

### 2. Les limites du prtocoles espnow.

* Prend en charge uniquement la communication unicast. Envoie de données vers un destinataire bien précis.
* Impossible d'envoyer des paquets depassant 250octets.
* Nombre de pairs cryptés très limités.

### 3. Les types de communication supporté par espnow

* **Communication unidirectionnelle** : Une puce maitre envoie les données vers une autre carte qui est considérée comme l'exclave. Cette configuration est utilisée pour par exemple envoyé des commandes ON/OFF ou acquérir des données et les envoyées à la deuxième carte.
![Communication unidirectionnelles](/images/ESP_NOW_U.webp)  
[source](https://randomnerdtutorials.com/esp-now-two-way-communication-esp32/)  
On peut ajouter plusieurs cartes expéditeurs(masters) et avoir une configuration où plusieurs noeuds envoient des données vers un noued récepteur(slave).
![Plusieurs noeuds maitres vers un noeud esclave](/images/MULTIPLE-MASTERS.webp).  
Il est également possible de mettre en place une configuration où un seul noeud expéditeur appelé maitre envoie des données à plusieurs autres noeuds récepteurs exclaves. Cette configuration est utile pour envoyer des commandes à pluseurs équipements. Le noeud central agit comme une télécommande. ![maitre vers plusieurs exclaves](/images/ONE-MASTER.webp)
* **Communication bidirectionnelle** : Dans cette configuration chaque carte peut être une emetteur et un récepteur à la fois. ![Communicationdidirectionnele entre deux noeuds](/images/B.webp).  
En rajoutant plus de carte on peut avoir une configuration d'un réseau maillée ou plusieurs cartes peuvent échanger des données. ![Réseau maillé ](/images/maille.webp)


### 4. Sécurité 

* Chiffrer un message : Rendre un message secret 
* Déchiffrer un message : Rendre le message chiffrer lisible et compréhensible c'est à dire retrouver le message d'origine.
* Crypter un message : Altérer un message de sorte qu'il ne soit plus lisible.
* Décrypter un message : Retrouver un message chiffré lorsqu'on ne connait pas la clé de déchiffrement.  
Le principe de la cryptographie consiste dans un premier temps à trouver un algorithme de chiffrement afin de chiffrer le message et dans un deuxième temps partager cet algorithme à votre interlocuteur pour qu'il puisse déchiffrer le message.  
La cryptographie symétrique connue aussi sous le nom cryptographie à clé est le fait d'utiliser le même algorithme, la même clé pour le chiffrement et le déchiffrement du message. L'inconvénient ici est que la clé doit être protégée et envoyer de façon sécrète à votre correspondant. Aucun autre correspondant ne doit avoir la clé. Parmi les algorithes de cryptogophies les plus connus et performants ont peut reténir :  
a. **DES :** Data Encryption standard de IBM  
b. **AES** : Advanced Encryption Standard(le plus utilisé de nos jours).  
Pour résoudre le problème de criticité de l'unicité de la clé de chiffrément symétrique, on préconise l'utilisation de deux clés de chifferement. Cette méthode est connu sous le nom de cryptogophie asymétrique. On distingue donc :   
a. **Une clé publique :** dont le seul rôle est de chiffrer le message. il peut donc être distribuer publiquement.   
b. **Une clé privée:** qui permettra de déchiffrer le message. comme son nom l'indique cette clé est sécrète. Chaque machine possède sa propre clé privée.   
Lorsque deux machines désirent effectuer un échange sécurisé, la machine1 va utiliser la clé publique de la machine2 pour chiffrer le message. Seule la machine2 pourra dechiffrer le mssage avec sa clé privée vu que c'est sa clé publique qui a été utilisée pour chiffrer le message. Le même processus s'effectue lorsque la machine2 désire envoyer un message à la machine1.
a première machine envoie demande le certificat de la deuxième machine. Il s'agit bien là de la manière la plus simple. Sinon machine1 demande le certificat de machine2 avant tout le processus de cryptage et d'envoie. Ce certificat contient la clé publique, la signature de machine2, l'algorithme de chiffrement utilisé... La machine 1 va donc utiliser ce certificat comme moyen d'authentification. Avec ce document machine1 a aussi accès à la clé publique de ce dernier pour crypter les messages.  
  
ESP-NOW utilise un chiffrement symétrique c'est-à-dire que l'expéditeur et le récepteur doivent se mettre d'accord sur la clé à utiliser avant tout échange de données.  
Pour une communication chiffée chaque ESP dispose d'une clé principale **PMK(Primary Master Key)** et d'une ou plusieurs clés ***locales LMK(Local Master Key)**. Chaque clé possède une longuer de 16octets.
- PMK : C'est une clé globale utilisé par l'ESP pour chiffrer le LMK. C'est l'utiliseur qui spécifie le PMK autrement un LMK par défaut est  utiisé.  
```
esp_now_set_pmk((uint8_t *) cles) 
// Méthode utilisée pour définir la clé PMK
```  
- LMK : C'est une clé spécifique à chaque Node. Cela signie que chaque Node ESP  couplé possède sa propre clé locale. Le LMK est utilisée pour chiffrer les trames échangés entre pairs. Chaque pair est capable de maintenir jusqu'à maximum 6 LMK. Si le LMK du pair n'est pas définie, les trames transmis ne seront pas chiffrés.  
#### Exemples de configuration d'un échange sécurisé entre deux cartes ESP32.

**A. Configurations à effecturer sur l'ESP considérer comme le premier node**
1. On commence par déclarer les deux clés PMK et LMK. Il faut s'assurer que les deux clés ne soient pas identique. Ce sont ces clés qui chiffreront les données échangées entre les deux cartes.  

``` 
//Chaque clé possède une longeur de 16octets donc 16caractères
static const char* PMK_KEY = "ESTIA_RECHERCHE&";  
static const char* LMK_KEY = "ENSAM_CCPS_CASA#";
```  
  
2. Ajouter de la sécurité sur l'interface WiFi en définissant la clé PMK qu'on vient de déclarer.  
```
esp_now_set_pmk((uint8_t *)PMK_KEY);
```   
Notez ici qu'il est important d'inialiser d'abord le protocol espnow avant de faire appel à cette méthode. ***esp_now_set_pmk((uint8_t\*)PKM_KEY)*** a pour valeur de retour:  
- **ESP_OK** si tout se passe bien.
- **ESP_ERR_ESPNOW_NOT_INIT** si le protocole ESPNOW n'est pas initialisé sur la carte.
- **ESP_ERR_ESPNOW_INV** si l'argument fourni à la fonction n'est pas valide.  
3. Définir le LMK du noeud pair en ajouant le LMK déclaré précédemment dans une structure contenant les informations du noeud pair comme suit :  
```
esp_now_peer_info_t slaveNodeInfo;
for(int i = 0; i < 16; i++){
   slaveNodeInfo.lmk[i] = LMK_KEY[i];
    }
```  
4. Il faut finalement activé la propriété de chiffrement de SlaveNodeInfo comme suit:
```
slaveNodeInfo.encrypt = true;
```
**B. Configurations sur le noeud pair considéré comme le deuxième noeud.**
En ce qui concerne le deuxième noeud il faut effectuer exactement les mêmes configuration qu'on a eu à faire sur le premier noeud. Le PMK et le LMK définies sur ce noeud doivent obligatoirement être identiques aux PMK et LMK définies sur le premier noeud.

### II. Configuration d'une communication bidirectionnelle entre deux noeuds.

### III. Le protocole WHART(Wireless HART)

WHART est un protocole de communication sans fil base consommationn très adapté aux capteurs industriels sans fils. Sa portée nominale est d'environ 200m et le débit va de 20 à 250Kbps. WHART utilise la bande de fréquence de 2.4GHz sur une topologie maillée.

### IV. Le protocole Z-Wave 

### V. Le protocole EnOcean 

C'est un protocole sans fil très basse consoommation ayant une portée nominale de 30m et un débit d'environ 2Mbps. Bande de fréquence 800-900MHz et 2.4GHz. Il supporte le wifi et le ZigBee.

### VI Etude de l'ESP32
### Le bootloader


### CONSOMMATION DE L'ESP32 

- En Fonctionnement normal sans aucune sollicitation
- En fonctionnement normal avec **sollicitation**
- En mode deep-sleep 

Mesurer le courant avec un multimètre à chaque mode.
Idée sur la mesure de la Consommation réelle du wifi : Mesurer le courant en fonctionnement normal de la carte ensuite mesurer le courant lorsque la carte effectue une communication en wifi.

Même étape à suivre pour le Bluetooth ou pour l'ESP-NOW.

La consommation de l'esp32 est aussi lié à la vitesse de fonctionnement du CPU:
- 240MHz : 62.5mA
- 160MHz : 42.5mA
- 80 MHz : 29mA
- 40 MHz : 16mA
- 20 MHz : 10mA


### SEMINAIRE SUR L'INDUSTRIE 5.0

#### Présentation 3

Piliers de l'industrie 5.0 

- Fabrication centrée sur l'homme
- Les cobots
- IA et ML
- IoT et connectivité 
- Les CPS
- Les jumeaux numériques
- 

### LES CPS

Les CPS joue un rôle vital de nos jours dans la smartfactory. Il sont au coeur de l'industrie 4.0.  Le CPS est une association de systèmes embarqués interdépendants avec des périphériques physique en temps réel. Ils permettent de surveiller les process physiques et offre la possibilité aussi de e créer un modèle numérique ou virtuel de ce process. Les CPS sont donc à la base aujourdh'ui des jumeaux numériques.Ce modèle virtuel permet de controler totalement le modèle physique. En fonction des capteurs installés, le CPS établie une rélation collaborative entre les différents systèmes, intègre des composants hétérogènes... L'une des complexité de la mise en place d'un CPS réside au niveau de la nécessité d'un controle temps réel du process.






