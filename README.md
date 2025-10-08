<div align="center">
  <img src="https://github.com/user-attachments/assets/daa6f862-cd17-44da-a19a-618b004a6c5b" width="640" />
</div>

# Runbook: Elastic SIEM 
#### Hector M. Reyes | SOC Analyst

---

<div align="center">
  <img src="https://github.com/reyestech/Elastic-SIEM-Lab-Runbook/assets/153461962/f7d69934-f43a-4762-93f5-8e7374bb31bc" width="65%" />
</div>

# **Capstone Introduction**
**Description**
A client sought our security services after experiencing a company-wide security breach. We faced a significant challenge because our teams are located in different parts of the US. To provide the best protection for our clients' businesses, we operate across multiple time zones and promptly monitor various systems.

To address this challenge, I created an Elastic SIEM Lab. This lab environment enables me to remotely connect, monitor, test, and analyze various operating systems across multiple network systems. By utilizing these tools, we can ensure prompt and effective protection for our clients' businesses.

| **Tools Used**                              | **Environments / Purpose**                      |
|---------------------------------------------|-----------------------------------------------|
| Elastic Cloud / Elastic Stack (Elasticsearch, Logstash, Kibana, SIEM app) | Log processing, storage, visualization, detection |
| Beats (Filebeat, Winlogbeat) & Elastic Agent | Log and metric collection from endpoints     |
| VMware • VirtualBox                        | Virtualization platforms                     |
| Kali Linux                                 | Offensive / testing tools, network simulation |
| BeEF (Browser Exploitation Framework)       | Simulate browser-based attacks / pivoting     |
| PowerShell                                 | Windows scripting & log generation           |


----

<div align="center">
  <img src="https://github.com/user-attachments/assets/c70da220-57ba-4bbc-9ef3-505184b9feb7" width="90%" />
</div>

# **Implementing Elastic SIEM**  
## **Blue Team Solution**  
The Blue Team has implemented a Security Information and Event Management (SIEM) system to enhance network security. This system collects logs from various sources, including servers and network devices, to detect unusual activities through correlation and alerts. It also automates incident response and integrates threat intelligence for real-time updates on known threats.

For Blue Teams, the SIEM system is essential in identifying threats early, thereby preventing escalation. It provides contextual insights through log correlation, which is vital for incident response. Additionally, the customizable rules enable tailored configurations that enhance both efficiency and effectiveness. Operating continuously, the SIEM ensures 24/7 monitoring for rapid detection of threats.

**Key Features:**  
- **Early Threat Detection:** Identifies potential threats and suspicious activities promptly.  
- **Insights:** Correlation of logs provides the necessary context for understanding incidents.  
- **Customizable Rules:** Allows adaptation to specific environments and threat landscapes.  
- **Remote Monitoring:** Facilitates round-the-clock monitoring for detecting threats across client networks.

---

## 1. **Purpose**  
SIEM systems analyze log data from various sources to enable swift detection and response to security incidents. Utilizing advanced analytics, they deliver a centralized platform for managing security events, offering a comprehensive view of the organization's security posture.

<img src="https://github.com/user-attachments/assets/5af11229-f21c-467f-b618-84495b065757" width="60%" />

## 2. **Installation**
### **Elastic SIEM Tools Installation:**
Install Elasticsearch, Logstash, and Kibana on your Linux Machine:
- Elasticsearch is a distributed, RESTful search and analytics engine.
- Logstash ingests, transforms, and ships data.
- Kibana is the visualization/dashboard layer.

### **Option A:** Install Elasticsearch, Logstash, and Kibana on your Linux machine.
```bash
sudo apt update
sudo apt install -y elasticsearch logstash kibana
```

### **Option B:** Install official Elastic packages (official 8.x apt repo for newer builds.)
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

## 3. **Configuration**
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

## 4. **Starting Services**
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

## 5. **Basic Usage**
### **Access Kibana Interface**
- Open a web browser and navigate to the Kibana web interface:
```text
http://<your-host-or-ip>:5601
```

### **Create Index Patterns**
- In Kibana: Stack Management → Data Views → Create data view (e.g., logs-*, filebeat-*).

### **Explore Data**
- Use the Discover tab in Kibana to explore and search through log data.

## 6. **Beginner Commands**
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

## 7. **Intermediate Commands**
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

## 8. **Advanced Commands**
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


## 9. **SOP (Standard Operating Procedures)**
### **Incident Response**
- Develop SOPs for incident response, including steps for detecting, analyzing, and mitigating security incidents using SIEM tools.

### **Regular Maintenance**
- Establish SOPs for regular maintenance tasks such as index management, log rotation, and performance optimization.
<img src="https://github.com/user-attachments/assets/855a1756-d498-481f-be45-25ec41f299e8" width="80%" />

## 10. **Documentation and Reporting**
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

## 11. **Compliance and Regulatory Considerations**
### **Compliance Frameworks**
- Ensure SIEM configurations comply with relevant industry regulations, standards, and frameworks such as PCI DSS, HIPAA, and GDPR.

### **Auditing and Monitoring**
- Implement auditing and monitoring mechanisms to track changes to SIEM configurations and detect unauthorized access or tampering.

## 12. **Continuous Learning**
### **Training and Education**
- Invest in continuous training and education to stay updated on the latest SIEM technologies, best practices, and emerging threats.

## 13. **Troubleshooting**
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

## 14. **Integration with Other Tools**
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

## 15. **Advanced Techniques**
### **Custom Dashboards**
- Develop custom dashboards and visualizations in Kibana to monitor specific security metrics and KPIs.

### **Machine Learning and Threat Intelligence**
- Leverage machine learning algorithms and threat intelligence feeds to enhance threat detection and automate incident response.
<img src="https://github.com/user-attachments/assets/acce705c-9e05-4e47-a7bb-50a3eafd0b0b" width="70%" />

## 16. **Automation**
### **Automated Alerting**
- Configure automated alerting mechanisms in Kibana to notify security teams of potential incidents or abnormal behavior.
```bash
# Example: list rules via Kibana Alerting API (authentication required)
curl -X GET "http://localhost:5601/api/alerting/rule/_find" -H "kbn-xsrf: true"
```

### **Automated Remediation**
- Implement automated response actions to mitigate incidents (block IPs, quarantine hosts, disable accounts) via webhooks/SOAR.
<img src="https://github.com/user-attachments/assets/d82a73ff-524b-4074-a162-5ac3de692120" width="50%" />

## 17. **Scalability and High Availability**
### **Cluster Deployment**
- Deploy Elasticsearch/Logstash/Kibana to achieve scalability and high availability.

### **Load Balancing**
Implement load balancing to distribute incoming log data across cluster nodes.

## 18. **Disaster Recovery and Backup**
### **Backup and Restore**
- Establish backup and restore procedures for SIEM data to ensure integrity and facilitate recovery.
```bash
# Register snapshot repository (filesystem example)
sudo mkdir -p /var/backups/es-snapshots
sudo chown elasticsearch:elasticsearch /var/backups/es-snapshots

curl -X PUT "http://localhost:9200/_snapshot/lab_snapshots" \
  -H "Content-Type: application/json" -d '{
    "type":"fs","settings":{"location":"/var/backups/es-snapshots"}
  }'

# Create a snapshot of all indices
curl -X PUT "http://localhost:9200/_snapshot/lab_snapshots/snap-$(date +%Y%m%d%H%M%S)?wait_for_completion=true"

# Restore example
curl -X POST "http://localhost:9200/_snapshot/lab_snapshots/<SNAP_NAME>/_restore" \
  -H "Content-Type: application/json" -d '{"indices":"logs-*","include_global_state":false}'
```

### **Disaster Recovery Planning**
- Develop DR plans to minimize downtime and restore SIEM functionality after catastrophic events or security breaches.
<img src="https://github.com/user-attachments/assets/cbc704c7-6be5-4198-ae3f-9b946100779d" width="50%" />

---

# Conclusion
In today's world, cybersecurity is crucial to any organization. That's why SIEMs are an essential tool in any cybersecurity arsenal. SIEMs offer remote visibility into security events, allowing us to stay one step ahead of cyber threats by proactively detecting anomalies in our networks. Following security best practices, maintaining proper documentation, and keeping staff up to date on security standards are crucial to maximizing the benefits of SIEM and ensuring a safe and secure work environment. Doing so can enhance your organization's security posture, mitigate the risk of cyberattacks, and safeguard your valuable assets.
<br />

