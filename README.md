<div align="center">
  <img src="https://github.com/user-attachments/assets/92169d73-f0c5-41de-a56d-a6f783d30c02" 
       alt="Elastic SIEM Banner" 
       width="90%" 
       style="border-radius:10px;" />
</div>

---

# 🛡️ **Elastic SIEM Runbook** 
### Hector M. Reyes | SOC Analyst  
### TL;DR
- Deployed and configured the **Elastic Stack (Elasticsearch, Logstash, Kibana, Beats)** in a virtualized lab.  
- Ingested and correlated logs from **Windows, Linux, and attack simulations** (Kali, BeEF, PowerShell).  
- Visualized detections and validated **real-time alerts** in Kibana.  
- Documented **SOC workflows, blue-team detections, and automation processes**.

---

## 🧩 **Description & Blue Team Implementation**  
Following a simulated company-wide breach, I developed an **Elastic SIEM Lab** to replicate enterprise-level monitoring and detection workflows.  
This environment enables centralized log collection, attack simulation, and threat correlation — providing a realistic platform to train, test, and optimize blue-team operations.

The **Elastic SIEM** acts as the analytical core for defenders, continuously ingesting endpoint, network, and system data to identify anomalies, automate correlation, and generate actionable alerts.  
By simulating attacker activity through Kali Linux and BeEF, and collecting event data from Windows and Linux endpoints, this lab demonstrates how a modern SOC leverages Elastic to maintain real-time visibility and resilience.

**Objectives:**  
- Centralize and normalize event logs from multiple operating systems.  
- Detect and correlate malicious behaviors across environments.  
- Validate SIEM effectiveness through controlled attack simulations.  
- Build reusable detection pipelines and SOC playbooks for rapid response.

<div align="center">
  <img src="https://github.com/user-attachments/assets/c70da220-57ba-4bbc-9ef3-505184b9feb7" width="80%" />
</div>

---

## 🎯 **Overview**  
The **Elastic Stack (SIEM)** integrates data ingestion, analysis, and visualization to provide a unified threat detection platform.  
It transforms raw logs into contextual intelligence, enabling analysts to detect, investigate, and respond efficiently.

**Key Capabilities:**  
- **Early Threat Detection:** Identifies suspicious behaviors through rule-based and behavioral analytics.  
- **Correlated Insights:** Connects related events for deeper investigation.  
- **Custom Rules & Alerts:** Tailors detections to specific network conditions.  
- **Continuous Monitoring:** Ensures 24/7 visibility across distributed assets.  

<div align="center">
  <img src="https://github.com/user-attachments/assets/5af11229-f21c-467f-b618-84495b065757" width="60%" />
</div>

---

## ⚙️ **Tools & Architecture Overview**  

| **Tool / Component** | **Purpose / Function** | **Environment / Notes** |
|-----------------------|------------------------|--------------------------|
| **Elastic Stack** (Elasticsearch, Logstash, Kibana, SIEM App) | Core SIEM platform for log ingestion, detection, and visualization. | Deployed on Ubuntu/Debian VM; accessed via Kibana (port 5601). |
| **Beats & Elastic Agent** (Filebeat, Winlogbeat) | Collects endpoint logs and metrics from Windows and Linux systems. | Installed on endpoints; forwards to Logstash or Elasticsearch. |
| **Logstash** | Filters and enriches log data before indexing. | Pipelines configured in `/etc/logstash/conf.d/`. |
| **Kibana** (SIEM App) | Dashboards, visualization, and alert management. | Connects to Elasticsearch for correlation and analytics. |
| **Kali Linux** | Simulates attacker behavior for testing detections. | Generates traffic and exploits to test rule accuracy. |
| **BeEF** (Browser Exploitation Framework) | Conducts browser-based exploitation simulations. | Integrated with Elastic SIEM to test detection and correlation. |
| **PowerShell Scripts** | Simulate Windows log events and user actions. | Run on Windows endpoints to validate pipeline coverage. |
| **VMware / VirtualBox** | Virtualization platforms for multi-node deployments. | Hosts Elastic Stack, endpoints, and attacker VMs. |

---

## 🏗️ **Architecture Snapshot**
```text
                  ┌───────────────────────────────┐
                  │        Elastic Stack          │
                  │ (Elasticsearch + Kibana)      │
                  └──────────────┬────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
            ┌───────▼───────┐         ┌──────▼────────┐
            │   Logstash    │         │   Beats Agent  │
            │ (Pipelines)   │         │ (Filebeat, Winlogbeat) │
            └───────┬───────┘         └────────┬──────┘
                    │                          │
                    ▼                          ▼
         ┌─────────────────────┐    ┌─────────────────────┐
         │ Windows Endpoint(s) │    │ Kali Linux (Attacks) │
         └─────────────────────┘    └─────────────────────┘
```

> **Goal:** Build a full detection pipeline from log generation to SIEM analysis — enabling threat simulation, alert validation, and SOC process refinement.

---

## 1️⃣ **Pre-Setup — System Preparation & Kali Linux Readiness**
Prepare all VMs before installing Elastic Stack. This step updates systems, sets hostnames, installs essentials, and configures Kali for attacker simulation so you can validate detection pipelines end-to-end.

### **Objective**
Get the Elastic SIEM server and Kali attacker VM ready:
- Patch and baseline Linux hosts.  
- Set clear hostnames for log correlation.  
- Install offensive tools on Kali for realistic telemetry.  
- Verify network connectivity between lab hosts.

### **System Update & Baseline** (Ubuntu / Debian hosts)
Update, upgrade, and install required packages on the Elastic/Kibana/Logstash nodes:

```bash
sudo apt update && sudo apt -y upgrade
sudo apt -y install curl jq apt-transport-https gnupg lsb-release net-tools
```

Set a clear hostname for each VM (helps correlate host logs in Kibana):
```bash
sudo hostnamectl set-hostname elastic-lab
# example: sudo hostnamectl set-hostname kibana-node
```
> 💡**Tip:** Use short, descriptive hostnames like `elastic-lab`, `kibana-node`, `win-endpoint`, `kali-attack01`.

### **Kali Linux Preparation** (Attacker VM)
Kali will act as the Red Team node to generate recon/exploit traffic for SIEM validation.
Update the system and remove unused packages:
```bash
sudo apt update && sudo apt -y full-upgrade
sudo apt autoremove -y
```

Install core offensive & network tools used for testing:
```bash
sudo apt -y install nmap metasploit-framework beef-xss curl netcat-traditional hping3
```
> These tools create realistic logs: network scans, web exploitation, SSH brute-force, HTTP payloads, and more.

#### **Diagnostics & Testing** (what to generate from Kali)
Use Kali to simulate attacker behavior and validate detection:
- Reconnaissance: nmap, hping3, HTTP probes.
- Exploitation: `msfconsole` (Metasploit), `beef-xss` for browser attacks.
- Lateral movement / command execution patterns via `netcat`, `ssh` tests.
- Generate Windows events with PowerShell on Windows VMs to validate endpoint coverage.

Ensure events flow through your pipeline:
`Beats` → `Logstash` → `Elasticsearch` → `Kibana`
This confirms ingestion, parsing, correlation, and alerting are functioning.

## **Network Verification**
Confirm connectivity between core lab hosts:
If you use firewalls, ensure ports are open (ex: 9200 for Elasticsearch, 5601 for Kibana, Beats ports as needed).

Checklist (pass/fail before install)
- [ ] All VMs patched & updated 
- [ ] Hostnames set and documented
- [ ] Kali has required tools installed
- [ ] Basic network connectivity verified
- [ ] Plan for test scenarios documented (recon, exploit, logs to generate)

---

## 2️⃣ **Installation**
### **Elastic SIEM Tools Installation**
Install Elasticsearch, Logstash, and Kibana on your Linux Machine:
- Elasticsearch is a distributed, RESTful search and analytics engine.
- Logstash ingests, transforms, and ships data.
- Kibana is the visualization/dashboard layer.

### **Two Ways: Basic install (Ubuntu/Debian repo) or Official Elastic 8.x Packages**
#### **Option A: Basic install** Install Elasticsearch, Logstash, and Kibana on your Linux or Ubuntu machine.
```bash
sudo apt update
sudo apt install -y elasticsearch logstash kibana
```

**Option B: Official Elastic Packages** Install official Elastic packages (official 8.x apt repo for newer builds.)
```bash
# Import Elastic’s GPG key and add APT repo
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg

echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | \
  sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update
sudo apt -y install elasticsearch logstash kibana

# Check versions
elasticsearch --version
logstash --version
kibana --version
```

---

## 3️⃣ **Configuration**
### **Elasticsearch Configuration**
- Configure Elasticsearch settings such as cluster name and node settings.
```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

### **Logstash Configuration**
- Configure Logstash pipelines to ingest, transform, and output data.
```bash
sudo nano /etc/logstash/conf.d/<pipeline_name>.conf
```

### **Kibana Configuration**
- Configure Kibana settings such as server host and port.
```bash
sudo nano /etc/kibana/kibana.yml
```
<img src="https://github.com/user-attachments/assets/754cf30d-709c-4600-82f2-db7bc9a3f10a" width="70%" />

---

## 4️⃣ **Starting Services**
### **Start Elasticsearch**
- Start the Elasticsearch service.
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now elasticsearch
sudo systemctl status elasticsearch --no-pager
```
<img src="https://github.com/user-attachments/assets/2d2e5dc8-b9e9-4af8-9c8d-5ef383ce804c" width="70%" />

### **Start Logstash**
- Start the Logstash service.
```bash
sudo systemctl enable --now logstash
sudo systemctl status logstash --no-pager
```

### **Start Kibana**
- Start the Kibana service.
```bash
sudo systemctl enable --now kibana
sudo systemctl status kibana --no-pager
```

---

## 5️⃣**Basic Usage**
### **Access Kibana Interface**
- Open a web browser and navigate to the Kibana web interface:
```text
http://<your-host-or-ip>:5601
```

### **Create Index Patterns**
- In Kibana: Stack Management → Data Views → Create data view (e.g., logs-*, filebeat-*).

### **Explore Data**
- Use the Discover tab in Kibana to explore and search through log data.

---

## 6️⃣ **Beginner Commands**
**Index Management**
- Create an index in Elasticsearch.
```bash
curl -X PUT "http://localhost:9200/lab-index-1?pretty"
```

Document Indexing
- Index a document in Elasticsearch.
```bash
curl -X POST "http://localhost:9200/lab-index-1/_doc?pretty" \
  -H "Content-Type: application/json" \
  -d '{"event":"hello","severity":"info","ts":"'"$(date -Iseconds)"'"}'
```

---

## 7️⃣ **Intermediate Commands**
### **Logstash Configuration**
- Verify Logstash configuration syntax.
```bash
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
```

### **Pipeline Debugging**
- Debug Logstash pipelines by printing output to the console.
```bash
sudo systemctl stop logstash
sudo -u logstash /usr/share/logstash/bin/logstash \
  --path.settings /etc/logstash \
  -f /etc/logstash/conf.d/<pipeline_name>.conf --log.level debug
```
<img src="https://github.com/user-attachments/assets/b35f4068-9161-460c-a460-e07a19550436" width="50%" />

---

## 8️⃣ **Advanced Commands**
### **Elasticsearch Query DSL**
- Perform advanced queries using Elasticsearch Query DSL.
```bash
curl -s -X POST "http://localhost:9200/logs-*/_search" \
  -H 'Content-Type: application/json' -d @- <<'JSON'
{
  "size": 10,
  "query": { "match": { "message": "failed" } },
  "aggs": { "by_host": { "terms": { "field": "host.name.keyword", "size": 5 } } }
}
JSON
```

### **Logstash Plugins**
- Install additional Logstash plugins for extended functionality.
```bash
sudo /usr/share/logstash/bin/logstash-plugin list
sudo /usr/share/logstash/bin/logstash-plugin install logstash-filter-translate
```
---

## 9️⃣ **SOP (Standard Operating Procedures)**
### **Incident Response**
- Develop SOPs for incident response, including steps for detecting, analyzing, and mitigating security incidents using SIEM tools.

### **Regular Maintenance**
- Establish SOPs for regular maintenance tasks such as index management, log rotation, and performance optimization.
<img src="https://github.com/user-attachments/assets/855a1756-d498-481f-be45-25ec41f299e8" width="80%" />

---

## 🔟 **Documentation and Reporting**
### **Doument Configuration**
- Maintain detailed documentation of SIEM configuration settings, including Elasticsearch indices, Logstash pipelines, and Kibana visualizations.

### **Generate Reports**
- Use Kibana dashboards and visualizations to generate reports on security incidents, log trends, and system performance.
```bash
# Export saved objects (dashboards, visualizations, data views, etc.)
curl -X POST "http://localhost:5601/api/saved_objects/_export" \
  -H "kbn-xsrf: true" -H "Content-Type: application/json" \
  -d '{"type":["dashboard","visualization","index-pattern"]}' > kibana-objects.ndjson
```
<img src="https://github.com/user-attachments/assets/14c23d92-7df6-4f08-81f7-61fbfadaf367" width="60%" />

---

## 1️⃣1️⃣ **Compliance and Regulatory Considerations**
### **Compliance Frameworks**
- Ensure SIEM configurations comply with relevant industry regulations, standards, and frameworks such as PCI DSS, HIPAA, and GDPR.

### **Auditing and Monitoring**
- Implement auditing and monitoring mechanisms to track changes to SIEM configurations and detect unauthorized access or tampering.

---

## 1️⃣2️⃣ **Continuous Learning**
### **Training and Education**
- Invest in continuous training and education to stay updated on the latest SIEM technologies, best practices, and emerging threats.

---

## 1️⃣3️⃣ **Troubleshooting**
### **Logstash Debugging**
- Troubleshoot Logstash configuration errors by examining Logstash logs for errors and warnings.
```bash
sudo journalctl -u logstash -e --no-pager
tail -n 200 /var/log/logstash/logstash-plain.log
```

### **Elasticsearch Health Check**
- Check the health status of Elasticsearch to identify any issues.
```bash
sudo journalctl -u logstash -e --no-pager
tail -n 200 /var/log/logstash/logstash-plain.log
```

---

## 1️⃣4️⃣ **Integration with Other Tools**
### **Integration with IDS/IPS**
- Integrate SIEM with Intrusion Detection/Prevention Systems to correlate security events and alerts.

### **Integration with Vulnerability Scanners**
- Integrate SIEM with vulnerability scanners to identify security weaknesses and prioritize remediation efforts.
```bash
# /etc/logstash/conf.d/vulnscan.conf (CSV example)
input {
  file {
    path => "/opt/scans/*.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
filter {
  csv { separator => "," autodetect_column_names => true }
}
output {
  elasticsearch { hosts => ["http://localhost:9200"] index => "vulnscan-%{+YYYY.MM.dd}" }
}
```
<img src="https://github.com/user-attachments/assets/24c1bfbf-7a9d-451e-86ce-483a8da25dd2" width="60%" />

---

## 1️⃣5️⃣ **Advanced Techniques**
### **Custom Dashboards**
- Develop custom dashboards and visualizations in Kibana to monitor specific security metrics and KPIs.

### **Machine Learning and Threat Intelligence**
- Leverage machine learning algorithms and threat intelligence feeds to enhance threat detection and automate incident response.
<img src="https://github.com/user-attachments/assets/acce705c-9e05-4e47-a7bb-50a3eafd0b0b" width="70%" />

---

## 1️⃣6️⃣ **Automation**
### **Automated Alerting**
- Configure automated alerting mechanisms in Kibana to notify security teams of potential incidents or abnormal behavior.
```bash
# Example: list rules via Kibana Alerting API (authentication required)
curl -X GET "http://localhost:5601/api/alerting/rule/_find" -H "kbn-xsrf: true"
```

### **Automated Remediation**
- Implement automated response actions to mitigate incidents (block IPs, quarantine hosts, deactivate accounts) via webhooks/SOAR.
<img src="https://github.com/user-attachments/assets/d82a73ff-524b-4074-a162-5ac3de692120" width="50%" />

---

## 1️⃣7️⃣ **Scalability and High Availability**
### **Cluster Deployment**
- Deploy Elasticsearch/Logstash/Kibana to achieve scalability and high availability.

### **Load Balancing**
Implement load balancing to distribute incoming log data across cluster nodes.

---

## 1️⃣8️⃣ **Disaster Recovery and Backup**
### **Backup and Restore**
- Establish backup and restore procedures for SIEM data to ensure integrity and facilitate recovery.
```bash
# Register snapshot repository (filesystem example)
sudo mkdir -p /var/backups/es-snapshots
sudo chown elasticsearch:elasticsearch /var/backups/es-snapshots

curl -X PUT "http://localhost:9200/_snapshot/lab_snapshots" \
  -H "Content-Type: application/json" -d '{
    "type": "fs"," settings":{"location":"/var/backups/es-snapshots"}
  }'

# Create a snapshot of all indices
curl -X PUT "http://localhost:9200/_snapshot/lab_snapshots/snap-$(date +%Y%m%d%H%M%S)?wait_for_completion=true"

# Restore example
curl -X POST "http://localhost:9200/_snapshot/lab_snapshots/<SNAP_NAME>/_restore" \
  -H "Content-Type: application/json" -d '{"indices ": "logs- *" ,"include_global_state ":false}'
```

### **Disaster Recovery Planning**
- Develop DR plans to minimize downtime and restore SIEM functionality after catastrophic events or security breaches.
<img src="https://github.com/user-attachments/assets/cbc704c7-6be5-4198-ae3f-9b946100779d" width="50%" />

---

# 🏁 Conclusion
In today's world, cybersecurity is crucial to any organization. That's why SIEMs are an essential tool in any cybersecurity arsenal. SIEMs offer remote visibility into security events, allowing us to stay one step ahead of cyber threats by proactively detecting anomalies in our networks. Following security best practices, maintaining proper documentation, and keeping staff up to date on security standards are crucial to maximizing the benefits of SIEM and ensuring a safe and secure work environment. Doing so can enhance your organization's security posture, mitigate the risk of cyberattacks, and safeguard your valuable assets.

<div align="center">
  <img src="https://github.com/user-attachments/assets/fc9bbfc4-e5c1-4ef8-b60a-3465b5c83d75" width="70%" alt="screenshot-security-network-overview-8-2" />
</div>
