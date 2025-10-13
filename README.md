# 3SI2-OTA-PA
1. Project Objective

The project aims to set up an Over-The-Air (OTA) update system for a Raspberry Pi equipped with a Caliope2 module. The main goal is to ensure the secure and reliable distribution of software updates and protect both the device and infrastructure from attacks.

Specific Objectives

Deploy software updates in an automated and secure manner.

Ensure the integrity and authenticity of OTA files.

Secure the LTE communication channel against MITM or interception attacks.

Implement secure rollback mechanisms in case of update failure. (To be done if time allows)

Set up monitoring and anomaly detection for each OTA device. (ELK stack incoming)

2. Cybersecurity Challenges

The OTA nature exposes the device to several risks:

MITM (Man-in-the-Middle): interception and modification of OTA packages.

Malicious package injection: attempts to compromise the Raspberry Pi.

Unauthorized rollback: attempts to force installation of old vulnerable versions.

LTE exposure: attacks through the mobile network if communication is unsecured.

Alteration of logs or OTA process: preventing traceability and attack detection.

The project therefore focuses on:

The confidentiality, integrity, and authentication of updates.

The resilience of the device against OTA failures or attacks.

Traceability and proactive anomaly detection.

3. System Architecture
Main Components

OTA Server

Hardened Linux server (minimal Debian/Ubuntu)

Encrypted storage for OTA images

Internal PKI for package signing

Raspberry Pi + LTE Module

Verification of OTA package signatures and hashes

Secure and timestamped update logging

Network Infrastructure

Isolated LTE for OTA communication

Firewall, ELK for IDS/IPS

VPN

4. Secure OTA Flow

Image Preparation

Compression and encryption of the OTA image

Digital signing on the server side

Calculation of a SHA-512 or BLAKE3 hash

Device-Side Verification

Mutual authentication via mTLS

Verification of signature and hash

Version validation and rollback policy

Isolated update within a container or temporary partition

Update Application

Atomic update to prevent corruption

Automatic rollback in case of failure

Complete and secure logging

Monitoring and Alerts

Logs sent to the OTA server

Detection of network anomalies or corrupted packages

Audit of installed versions and integrity

Simulation and attack testing to verify resilience

6. Attack Demos and Proof-of-Concept Scenarios (for evaluation)
6.1 Attack 1 — MITM on the OTA Channel (Packet Modification)

Goal: demonstrate that an OTA package not properly verified can be modified in transit and redirected.

6.2 Attack 2 — Repacking/Tampering of a Signed Package (Repackaging)

Goal: show the necessity of using signatures that cover the entire package and metadata.

Preparation:

Create a signed OTA package.

Attempt to decompress, modify a binary, then recompress without resigning.

If the signature does not cover all metadata, the attacker can modify unprotected elements.

Mitigation: sign the entire package + metadata; include timestamping.

6.3 Attack 3 — Theft/Exfiltration of Private Keys (Server or Supply Chain Compromise)

Goal: demonstrate the impact of a compromised OTA key.

Once the key is compromised, the attacker can sign authorized images.

7. ELK (Elastic Stack) Integration for Monitoring / Detection
7.1 ELK Objectives

Centralize all OTA logs (server + devices) in an Elasticsearch index.

Build Kibana dashboards to visualize: version status, failure rates, suspicious connection attempts, geographic anomalies (IMSI/CellID), replay/rollback counts.

Detect and alert in real time on critical events (rejected package, invalid signature, spike in failures, device under rogue cell influence).

7.2 Proposed ELK Architecture

Filebeat on devices (if possible): sends OTA logs to Logstash/Elasticsearch.

If devices are constrained, set up a local collector (edge gateway) to receive all device logs and forward them.

Logstash: parsing (grok/json), enrichment (geoip, lookup), signature/consistency checks.

Elasticsearch: strict mapping for fields device_id, version, hash, signature_ok, result, cellid, imsi, ip, geoip.

Kibana: dashboards + alerting (Watcher or built-in Alerting).
