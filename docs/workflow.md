## Workflow of the project : 

---
### Part 1 : Large view and logical Architecture 
**Server (OTA)**
- Image management through an API
- Stock signed images
- Distribution of images through a secure canal (Not HTTPS) we are going to create our own secure protocol.)
- Log Service to track anomalies and alerts (Instance ELK)

**CLient (Raspberry PI + Sequans Modem)**
- Agent OTA that will check for any updates -> Performs handshake -> Download -> Verify signature -> Install -> Rollback
- Network module : Handle the modem connexion + IPSEC Tunnel to OTA Server.
- Internal PKI (Need to focus on that point)
- Filebeat on the client to send logs

---
### Part 2 : Packets format and sign tool 
For the server side, we need to implements the metadata, the process of signing packages and transfer format

--- 
### Part 3 : Secure Communications Canal between Raspi and Server 
For this part we need to create a algo for handshake, check signed certificates, ways to communicated a approve the communication between raspi and srv. 
Recommanded algo : ECDH P-256 + HKDF(SHA256) + AES-GCM-256 + ECDSA P-256 signatures.

---
### Part 4 : Tunnel between Sequans Modem and OTA Srv 
If we got the time and if it's not too complicated.. We can implpement our own vpn tunnel. 
chatGPT POV of this : Risques : tu réimplémentes un VPN — bugpotentiel élevé (replay, fragmentation, route leak). Pour un projet étudiant c’est excellent d’un point de vue pédagogique.

---
### Part 5 : Stack ELK on docker for less cpu/ram consuption
Creating a full and complete ELK instance (Filebeat, Elasticsearch, Kibana)

---
### Part 6 : POC and Tests
