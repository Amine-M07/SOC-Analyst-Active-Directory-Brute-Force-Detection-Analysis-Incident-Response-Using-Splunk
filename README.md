# 📊 SOC Analyst Case Study: Brute-Force Attack Response

This report documents the creation of a home lab environment, the simulation of a cyber attack (brute-force), and the subsequent Detect, Analyze, and Respond phases, citing all necessary evidence files.

---

## 1. 🗺️ Home Lab Setup and Security Infrastructure

The project is built on an isolated network using VirtualBox to simulate a corporate environment. The network is configured as a private Host-Only Network.

### 1.1. Network Topology and Machines

(<img width="599" height="464" alt="diagram" src="https://github.com/user-attachments/assets/9c3d1701-3f9f-48df-ae96-24d768265dee" />
)

The network is a private subnet **192.168.10.0/24**. All machines were configured to use the same network adapter setting 

<img width="1366" height="768" alt="network adapter" src="https://github.com/user-attachments/assets/9ad36b81-2e38-4c0a-a541-277d733348eb" />


[](https://app.notion.com)

<img width="1366" height="768" alt="network confg  the same for all machines" src="https://github.com/user-attachments/assets/e1ed6150-0499-42c3-8c00-8208de4a9f9b" />


| Role | VM Name | IP Address | Description |
| --- | --- | --- | --- |
| **Attacker** | Kali Linux | 192.168.10.250 | Used to launch the brute-force attack. |
| **Target (Active Directory)** | ADC01 (Windows Server) | 192.168.10.7 | Domain Controller (DC) managing user authentication. |
| **SIEM Server** | Ubuntu Server | 192.168.10.10 | Central log collection and analysis point (Splunk Enterprise). |
| **Target (Client)** | Windows 10 | DHCP | A typical client workstation on the domain. |

The full network layout is documented in **diag.pdf** and a view of the running machines in the VM Manager is in **machines** 

<img width="1366" height="725" alt="home labs" src="https://github.com/user-attachments/assets/f314abdc-5a84-4ebc-aa38-5d2a98a55ca6" />


### 1.2. Logging and Monitoring Deployment

To enable analysis, the following tools were deployed and configured to forward logs to the Splunk Server (running via  **splunk in ubunto** 

<img width="1270" height="702" alt="starting splunk in ubunto" src="https://github.com/user-attachments/assets/fc163d95-37e0-4b23-9ea5-6415f4cf602d" />


- **Active Directory Domain Services (AD DS)**: Installed on ADC01

<img width="911" height="648" alt="creat new role" src="https://github.com/user-attachments/assets/338a5bb3-291a-4845-84b3-aaeee5c1a380" />


[](https://app.notion.com)

<img width="1366" height="708" alt="image" src="https://github.com/user-attachments/assets/7d2b8160-0453-4598-b1bb-fd96f6c00fda" />


- **Domain Promotion:** The server was promoted to a Domain Controller by selecting the **"Add a new forest"** deployment configuration, establishing the new domain environment.

     *Caption: The deployment wizard, showing the selection of **"Add a new forest"** to create the root           domain.* 

<img width="1366" height="768" alt="dns 3" src="https://github.com/user-attachments/assets/c15de8f1-54ec-4b3a-b93b-83f29b72196b" />


- **Splunk Universal Forwarder (UF)**: Installed on the Windows machines (ADC01 and Win10) to collect Windows Event Logs and Sysmon logs (**install splunk universal**).

<img width="918" height="743" alt="install splunk universal" src="https://github.com/user-attachments/assets/8269fc68-b4dc-4fe6-8120-121ee1572d35" />


- **Sysmon**: Installed on the targets to capture advanced, low-level endpoint activity (**install sysmon.png**).

<img width="1366" height="738" alt="install sysmon" src="https://github.com/user-attachments/assets/b1da4d4d-ec57-400f-9fb6-dc9bf68b6c98" />


---

## 2. 🎭 Attack Simulation: Brute-Force

The scenario involved an external attacker attempting to compromise a domain user account. A test user named **amir** or **Red** was created for this purpose 

<img width="1366" height="768" alt="creat new user" src="https://github.com/user-attachments/assets/0709ba81-82e6-46e9-8db5-1f107b8bbf79" />


### Attack Execution Timeline

| Step | Action | Result |
| --- | --- | --- |
| **Attack Prep** | The attacker uses Kali Linux to stage a dictionary file. | Confirmed dictionary file setup. |
| **Attack Execution** | The Hydra tool is used to launch the brute-force attack against the ADC01 server's Kerberos or SMB service. | SUCCESS: Hydra reports that the valid credential was found. |

<img width="1308" height="699" alt="43" src="https://github.com/user-attachments/assets/fc122a75-f70b-4216-9ffa-b4e215d8133d" />


<img width="1295" height="665" alt="Screenshot 2025-11-17 085757" src="https://github.com/user-attachments/assets/d9a0c145-a5a6-4c8e-9826-bdf9c8667b9b" />


---

## 3. 🔍 Detection and Analysis (The SOC Analyst Role)

The analyst monitors Splunk for high-volume authentication anomalies originating from external or unusual IP addresses.

### 3.1. Detection: Failed Logons

The first sign of an attack is a large number of failed attempts:

**SPL Query:**

```
index=endpoint EventCode=4625 Account_Name=amir

```

**Finding:** The query revealed **216 failed logon events** targeting the user **amir** from the attacker's source IP (**192.168.10.250**). This signature confirms a brute-force attempt.

<img width="1366" height="738" alt="splunk log failed" src="https://github.com/user-attachments/assets/042f1019-e8a0-460c-a16b-03a7baaf20db" />


<img width="1366" height="738" alt="splunk user amir" src="https://github.com/user-attachments/assets/2ceebdce-3216-4ed6-8588-59dd9cd2e7a2" />


### 3.2. Analysis: Successful Compromise

The analyst pivots to check for the attacker's success:

**SPL Query:**

```
index=endpoint EventCode=4624 Account_Name=amir

```

**Finding:** The search confirmed **3 successful logon events** for the user **amir** immediately following the failed attempts, indicating the attacker gained access. This is escalated to a **Critical Incident**.

**Evidence:** **logged om successfully .png**

<img width="1366" height="717" alt="logged on successfully" src="https://github.com/user-attachments/assets/45e9fb45-5071-4166-aecd-7b91ede83423" />


---

## 4. ✅ Incident Response, Eradication, and Recovery

A multi-step response plan was executed to contain the threat and prevent reoccurrence.

### 4.1. Containment (Immediate Action)

The primary goal is to isolate the threat source.

**Action:** A firewall rule was created on the target Windows system (ADC01) to block all traffic inbound from the attacker's IP (**192.168.10.250**).

<img width="1366" height="725" alt="respond  block the ip of the attacker" src="https://github.com/user-attachments/assets/b978822b-b11a-43c0-9985-c2df94259426" />

 The Kali attacker machine was shown trying to ping the ADC01 server and failing (100% packet loss), confirming containment 

<img width="1366" height="725" alt="respond  block the ip of the attacker" src="https://github.com/user-attachments/assets/a49d35eb-14e8-4f70-969d-eb6c3d9cb2a1" />


### 4.2. Eradication & Remediation

The compromised account was secured, and the system was checked for further compromise.

**Action (Eradication):** The password for the compromised user account (**amir**) was immediately changed.

**Evidence (Password Change):** **respond = change the password .png**

<img width="1009" height="262" alt="respond = change the password" src="https://github.com/user-attachments/assets/7a14e44f-5ddf-46b2-ba4a-e129669eb0b5" />


**Action (Remediation):** A system scan (`sfc /scannow`) was run on the target to ensure no system files were corrupted or replaced by the attacker.

**Evidence (System Scan):** **run a scan .png**

<img width="1366" height="282" alt="run a scan" src="https://github.com/user-attachments/assets/982f6d04-d036-4a24-8899-a965fe2ea869" />


4.3. Hardening (Recovery)

To prevent future incidents of this nature, security policies were adjusted.

**Action:** The **Account Lockout Policy** was enabled in Active Directory.

**Setting:** The Account Lockout Threshold was set to a low number (e.g., 8 attempts) to automatically lock out users after a few failed attempts.

<img width="1366" height="768" alt="respond to this incidence by lockout policies" src="https://github.com/user-attachments/assets/6b79f181-04f5-476b-a98f-b08341332de2" />


## 📝 Note on Eradication Strategy

The `sfc /scannow` command, combined with the administrative steps (password change, lockout policy), satisfies the **Eradication and Integrity** requirement for the **network scanning** threat simulated in this lab.

**However, in a real-world scenario involving actual malware or persistence, eradication would require deeper steps, including:**

- **Deep Endpoint Scans** using EDR/Antivirus to find and remove malicious files.
- **Persistence Sweeping** to manually check and delete malicious **Registry Run keys** or **Scheduled Tasks**.
- **Forensic Log Hunting** in Splunk for any process or file creation events that occurred before containment.

This approach validates the system's integrity while acknowledging the full, advanced forensic procedures required in a professional environment

---

## 5. 📊 Summary and Key Findings

### Attack Metrics

- **Total Failed Attempts:** 216
- **Successful Logons :** 3
- **Attack Source IP:** 192.168.10.250
- **Target Account:** amir
- **Attack Method:** Brute-force using Hydra

### Detection Effectiveness

- **Detection Method:** Splunk SIEM monitoring EventCode 4625 and 4624
- **Detection Time:** Immediate identification of anomalous authentication patterns
- **False Positive Rate:** 0% (confirmed malicious activity)

### Response Actions Taken

1. ✅ Network-level containment (firewall block)
2. ✅ Credential reset (password change)
3. ✅ System integrity verification (SFC scan)
4. ✅ Policy hardening (account lockout policy)

---

## 6. 🎓 Lessons Learned

### What Went Well

- Comprehensive logging infrastructure captured all attack phases
- SIEM correlation quickly identified brute-force pattern
- Rapid containment prevented further damage
- Complete evidence chain for incident documentation

### Recommendations for Improvement

1. **Preventive Controls:** Implement account lockout policies before attacks occur
2. **Multi-Factor Authentication:** Deploy MFA for all domain accounts
3. **Network Segmentation:** Isolate domain controllers on restricted VLANs
4. **Automated Alerting:** Configure real-time alerts for authentication anomalies
5. **Password Policy:** Enforce strong password complexity requirements

---

## 7. 🔧 Technical Skills Demonstrated

This project showcases proficiency in:

- **Infrastructure:** VirtualBox, network design, Windows Server, Active Directory
- **Security Monitoring:** Splunk Enterprise, Splunk Universal Forwarder, Sysmon
- **Log Analysis:** SPL queries, event correlation, Windows Event Logs
- **Incident Response:** NIST framework, containment, eradication, recovery
- **Offensive Security:** Kali Linux, Hydra, attack simulation
- **System Hardening:** Group Policy, account lockout policies, firewall rule

---

## 9. 🎯 Conclusion

This comprehensive home lab project successfully demonstrates the complete lifecycle of a cybersecurity incident from attack simulation through final recovery. The project showcases practical SOC analyst skills including SIEM deployment, threat detection, incident analysis, and structured incident response procedures.

**Key Outcomes:**

- ✅ Successful detection of brute-force attack (216 failed attempts)
- ✅ Confirmed account compromise identification (3 successful logons)
- ✅ Complete threat containment (network isolation verified)
- ✅ Effective eradication (credential reset, integrity verification)
- ✅ Security hardening (account lockout policy implementation)

This project demonstrates readiness for SOC analyst, security engineer, or incident response roles in enterprise environments.

---

*Project completed as part of cybersecurity skill development - November 2025*
