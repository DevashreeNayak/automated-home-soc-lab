# Enterprise Telemetry Aggregation & Adversary Simulation Lab

## 📌 Executive Summary
This project showcases the implementation of an isolated, multi-node Security Operations Center (SOC) home lab environment running entirely on a Windows 11 host using Oracle VirtualBox. The project establishes an enterprise-grade detection engineering pipeline by isolating target systems, installing low-level kernel monitoring via Microsoft Sysmon, aggregating security telemetry into a centralized open-source SIEM platform (Wazuh), and evaluating coverage by launching adversary simulation frameworks.

---

## 🏗️ Lab Architecture & Network Topology

The entire lab is contained within a private, software-defined VirtualBox NAT network (`SOC-Private-Net`) using the `10.0.2.0/24` subnet. This architecture ensures all attack vectors, command-and-control loops, and explosive payloads remain securely sandboxed away from the physical home network infrastructure.

```mermaid
graph TD
    %% Define Network Boundaries
    subgraph Isolated_Lab_Network [VirtualBox Isolated NAT Network: SOC-Private-Net 10.0.2.0/24]
        
        %% Attacker Zone
        subgraph Attack_Zone [Adversary Simulation]
            Kali["Kali Linux VM <br> (Attacker Context)"]
        end

        %% Target Zone
        subgraph Corporate_Network [Victim Subnet]
            Win10["Window10 VM <br> (Target Endpoint)"]
            Sysmon["Sysmon Driver <br> (SwiftOnSecurity Rules)"]
            WazuhAgent["Wazuh Agent <br> (Log Forwarder)"]
            
            Win10 ---> Sysmon
            Sysmon --->|Generates Event IDs| WazuhAgent
        end

        %% Defensive Zone
        subgraph Security_Operations_Center [Centralized SIEM Matrix]
            Ubuntu["ubs22045vm <br> (Ubuntu 22.04 LTS)"]
            WazuhManager["Wazuh Manager <br> (XDR Engine)"]
            Indexer["Wazuh Indexer <br> (Log Storage)"]
            Dashboard["Wazuh Dashboard <br> (SIEM Console)"]
            
            Ubuntu ---> WazuhManager
            WazuhManager ---> Indexer
            Indexer ---> Dashboard
        end

    end

    %% Host Access Layer
    subgraph Management_Layer [Windows 11 Host Laptop]
        Browser["Chrome/Edge Browser <br> (Security Analyst View)"]
    end

    %% Data Flow and Attack Vectors
    Kali --->|1. Simulates Attack Vector <br> e.g., Brute Force / Malware Execution| Win10
    WazuhAgent --->|2. Secure Logs via Port 1514/1515| WazuhManager
    Dashboard -.->|3. Access Dashboard UI via HTTPS| Browser

    %% Styling Elements
    style Kali fill:#721c24,stroke:#f5c6cb,stroke-width:2px,color:#fff
    style Win10 fill:#155724,stroke:#c3e6cb,stroke-width:2px,color:#fff
    style Ubuntu fill:#004085,stroke:#b8daff,stroke-width:2px,color:#fff
    style Browser fill:#333,stroke:#fff,stroke-width:1px,color:#fff
```

---

## 🛠️ System Inventory & Provisioning Profiles

| Component | Virtual Instance | Operating System | Operational Role |
| :--- | :--- | :--- | :--- |
| **SIEM Core** | `ubs22045vm` | Ubuntu 22.04 LTS | Log Indexing, XDR Rule Engine, Dashboard Interface |
| **Victim Node** | `Window10` | Windows 10 Enterprise | High-fidelity logging source running Sysmon & Wazuh Agent |
| **Adversary** | `Kali Linux` | Kali Linux | Attack platform executing scanners and payload delivery |

---

## 🚀 Step-by-Step Deployment Logs

### Phase 1: Virtual Network Isolation
1. Configured an internal VirtualBox network infrastructure (`SOC-Private-Net`).
2. Configured all 3 instances to interface over the private virtual network card (`Adapter 1` -> `NAT Network`).
3. Verified end-to-end routing using private ICMP echoing across subnets.

### Phase 2: Orchestrating the Wazuh Matrix
Deployed a production-ready single-node Wazuh cluster on the `ubs22045vm` node by pulling down the comprehensive automation package:
```bash
curl -sO https://wazuh.com && sudo bash wazuh-install.sh -a
```

### Phase 3: Hardening Target Telemetry (`Window10`)
1. Deployed Microsoft Sysmon to intercept deeper kernel actions via the command line.
2. Interfaced Sysmon with the industry-standard `SwiftOnSecurity` configuration template to capture core system actions (Process creation, raw network connections, alternate data streams).
3. Registered the remote Windows endpoint to the central SIEM Manager using secure endpoints over TCP ports `1514`/`1515`.

---

## 🛡️ Active Threat Analysis Write-up & Verification

### Test Case: Detecting MITRE ATT&CK T1110 (Brute Force Attack)
* **Objective:** Simulate a malicious, script-based authentication bypass attempt and verify metric detection coverage inside the SIEM dashboard.
* **Execution (Kali Linux Node):** Executed a targeted password spraying attack against the target `Window10` profile over open administrative network services.
* **Telemetry Analysis:** The SIEM system immediately registered multiple sequential high-severity anomalies mapping to Windows Security Event ID **4625** (An account failed to log on).
* **Defensive Correlation:** Custom alerting metrics successfully surfaced the rogue internal IP address, triggering automated triage metrics on the manager dashboard.

