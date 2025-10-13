# 3SI2-OTA-PA
# Secure OTA Project for Raspberry Pi + Caliope2 LTE Module

## 1. Project Objective

The project consists of setting up an **Over-The-Air (OTA) update system** for a Raspberry Pi equipped with a Caliope2 module. The primary objective is to ensure the **secure and reliable distribution of software updates**, and to protect the device and the infrastructure from attacks.

### Specific Objectives

* Deploy software updates in an **automated and secure** manner.  
* Ensure the **integrity and authenticity** of OTA files.  
* Secure the **LTE communication channel** against MITM or interception attacks.  
* Implement **secure rollback mechanisms** in case of update failure. (To be done if time allows)  
* Set up **monitoring and anomaly detection** for each OTA device. (ELK stack incoming)

---

## 2. Cybersecurity Challenges

The OTA nature exposes the device to several risks:

* **MITM (Man-in-the-Middle)**: interception and modification of OTA packages.  
* **Malicious package injection**: attempts to compromise the Raspberry Pi.  
* **Unauthorized rollback**: attempts to force the application of old vulnerable versions.  
* **LTE exposure**: attack via the mobile network if communication is not secured.  
* **Alteration of logs or the OTA process**: preventing traceability and attack detection.

The project therefore focuses on:

* **Confidentiality**, **integrity**, and **authentication** of updates.  
* The **resilience** of the device against OTA failures or attacks.  
* **Traceability** and proactive anomaly detection.

---

## 3. System Architecture

### Main Components

1. **OTA Server**
   * Hardened Linux server (minimal Debian/Ubuntu)  
   * Encrypted storage for OTA images  
   * Internal PKI for package signing  

2. **Raspberry Pi + LTE Module**
   * Verification of OTA package signatures and hashes  
   * Secure and timestamped update logging  

3. **Network Infrastructure**
   * Isolated LTE for OTA communication  
   * Firewall, ELK for IDS/IPS  
   * VPN  

---

## 4. Secure OTA Flow

1. **Image Preparation**
   * Compression and encryption of the OTA image  
   * Digital signing on the server side  
   * Calculation of a SHA-512 or BLAKE3 hash  

2. **Device-Side Verification**
   * Mutual authentication via mTLS  
   * Verification of the signature and hash  
   * Version validation and rollback policy  
   * Isolated update within a container or temporary partition  

3. **Update Application**
   * Atomic update to prevent corruption  
   * Automatic rollback in case of failure  
   * Complete and secure logging  

4. **Monitoring and Alerts**
   * Logs sent to the OTA server  
   * Detection of network anomalies or corrupted packages  
   * Audit of installed versions and integrity  

---

* Simulation and attack testing to verify resilience  

---

## 6. Attack Demos and Proof-of-Concept Scenarios (for evaluation)

### 6.1 Attack 1 — MITM on the OTA Channel (Packet Modification)

**Goal**: demonstrate that an OTA packet not properly verified can be modified in transit and redirected.

---

### 6.2 Attack 2 — Repacking / Tampering of a Signed Package (Repackaging)

**Goal**: demonstrate the need to use signatures that cover the entire package and metadata.

**Preparation**:
* Create a signed OTA package.  
* Attempt to decompress, modify a binary, then recompress without resigning.  
* If the signature does not cover all metadata, the attacker can modify unprotected elements.

**Mitigation**: sign the entire package + metadata; include timestamping.

---

### 6.3 Attack 3 — Theft/Exfiltration of Private Keys (Server or Supply Chain Compromise)

**Goal**: demonstrate the impact of a compromised OTA key.

* Once the key is compromised, the attacker can sign authorized images.

---

## 7. ELK (Elastic Stack) Integration for Monitoring / Detection

### 7.1 ELK Objectives

* Centralize all OTA logs (server + devices) in an Elasticsearch index.  
* Build Kibana dashboards to visualize: version status, failure rates, suspicious connection attempts, geographic anomalies (IMSI/CellID), replay/rollback counts.  
* Detect and alert in real time on critical events (rejected package, invalid signature, spike in failures, device under rogue cell influence).

### 7.2 Proposed ELK Architecture

* **Filebeat** on devices (if possible): sends OTA logs to Logstash/Elasticsearch.  
* If devices are constrained, set up a **local collector** (edge gateway) that receives all device logs and forwards them.  
* **Logstash**: parsing (grok/json), enrichment (geoip, lookup), signature/consistency checks.  
* **Elasticsearch**: strict mapping for fields `device_id`, `version`, `hash`, `signature_ok`, `result`, `cellid`, `imsi`, `ip`, `geoip`.  
* **Kibana**: dashboards + alerting (Watcher or built-in Alerting).

