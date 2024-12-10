# THM-Writeup-Logging
Writeup for TryHackMe "Logs" Lab - Log Monitoring using rsyslog, logrotate, and commands such as grep, awk, and sed.

Developed by Ramyar Daneshgar 

### **Introduction**
This lab focused on understanding log collection, management, and analysis, leveraging tools such as `rsyslog`, `logrotate`, and command-line utilities (`grep`, `awk`, `sed`, etc.) to process system and application logs effectively. The tasks included identifying failed login attempts, extracting IP addresses from configurations, and automating log rotation. These techniques are essential for security monitoring, compliance, and incident response.

---

### **Objective**
1. Configure log collection for specific sources (e.g., SSH and cron logs).
2. Analyze logs to extract critical information such as usernames and IP addresses.
3. Automate log retention and rotation while ensuring log integrity.
4. Parse and normalize logs for consistent analysis across different formats.

---

### **Tools and Commands**
#### **Tools**
- **`rsyslog`:** Collect and route logs from various services.
- **`logrotate`:** Manage log retention and automate rotation and compression.
- **`grep`, `awk`, `sed`:** Process logs to extract and normalize data.

#### **Key Commands**
- Check `rsyslog` status:
  ```bash
  sudo systemctl status rsyslog
  ```
- Create custom logging configurations:
  ```bash
  sudo nano /etc/rsyslog.d/98-websrv-02-sshd.conf
  ```
- Restart `rsyslog` to apply changes:
  ```bash
  sudo systemctl restart rsyslog
  ```
- Trigger SSH login attempts:
  ```bash
  ssh localhost
  ```
- Analyze logs for failed login attempts:
  ```bash
  sudo cat /var/log/websrv-02/rsyslog_sshd.log | grep "Failed password"
  ```
- Extract IP addresses from configuration files:
  ```bash
  sudo grep '@@' /etc/rsyslog.d/99-websrv-02-cron.conf
  ```
- Configure and force log rotation:
  ```bash
  sudo nano /etc/logrotate.d/98-websrv-02_sshd.conf
  sudo logrotate -f /etc/logrotate.d/98-websrv-02_sshd.conf
  ```

---

### **Walkthrough**

#### **Log Collection with rsyslog**
1. **Verify rsyslog Installation and Status**
   - Checked the status of `rsyslog` to confirm it was active and running:
     ```bash
     sudo systemctl status rsyslog
     ```

2. **Configure Logging for SSHD**
   - Created a configuration file `/etc/rsyslog.d/98-websrv-02-sshd.conf` to direct `sshd` logs to a specific file:
     ```
     $FileCreateMode 0644
     :programname, isequal, "sshd" /var/log/websrv-02/rsyslog_sshd.log
     ```
   - Restarted `rsyslog` to apply the configuration:
     ```bash
     sudo systemctl restart rsyslog
     ```

3. **Simulate SSH Login Attempts**
   - Triggered multiple failed SSH login attempts using:
     ```bash
     ssh localhost
     ```
   - Incorrect passwords were intentionally entered to generate failed login logs.

4. **Analyze SSH Logs**
   - Extracted and filtered failed login attempts from the log file:
     ```bash
     sudo cat /var/log/websrv-02/rsyslog_sshd.log | grep "Failed password"
     ```
   - Identified usernames using:
     ```bash
     sudo cat /var/log/websrv-02/rsyslog_sshd.log | awk '{print $9}' | sort | uniq -c
     ```
   - This provided the count of failed attempts per username, indicating potential brute-force attacks.

#### **Log Forwarding and SIEM**
1. **Extract IP Address of SIEM-02**
   - Analyzed the cron log configuration in `/etc/rsyslog.d/99-websrv-02-cron.conf` to locate the forwarding destination:
     ```bash
     sudo grep '@@' /etc/rsyslog.d/99-websrv-02-cron.conf
     ```
   - The line `*.* @@10.10.10.101:514` indicated the IP address of the SIEM server (10.10.10.101).

#### **Log Rotation with logrotate**
1. **Configure Log Rotation**
   - Created a logrotate configuration file for SSH logs:
     ```bash
     sudo nano /etc/logrotate.d/98-websrv-02_sshd.conf
     ```
     Configuration:
     ```
     /var/log/websrv-02/rsyslog_sshd.log {
         daily
         rotate 24
         compress
         delaycompress
         missingok
         notifempty
         lastaction
             systemctl restart rsyslog
         endscript
     }
     ```
   - This ensured daily rotation, compression of old logs, and retention of the last 24 versions.

2. **Force Log Rotation**
   - Tested the configuration:
     ```bash
     sudo logrotate -f /etc/logrotate.d/98-websrv-02_sshd.conf
     ```
   - Verified rotated logs and their integrity:
     ```bash
     ls /var/log/websrv-02/
     sha256sum /var/log/websrv-02/rsyslog_sshd.log.*
     ```

#### **Parsing and Consolidation**
1. **Normalize Logs**
   - Used `awk` and `sed` to extract and reformat logs for consistent analysis:
     ```bash
     awk -F'[][]' '{print "[" $2 "]", $0}' /var/log/nginx/access.log
     sed "s/ +0000//g" /var/log/nginx/access.log
     ```

2. **Sort and Deduplicate Logs**
   - Sorted logs by timestamp and removed duplicates:
     ```bash
     sort /tmp/parsed_logs.log | uniq > /tmp/normalized_logs.log
     ```

---

### **Challenges and Resolutions**
1. **SSH Logs Not Captured**
   - **Issue:** SSH attempts weren’t logged initially.
   - **Resolution:** Verified `rsyslog` was running, corrected configuration, and restarted the service.

2. **Parsing Inconsistent Log Formats**
   - **Issue:** Logs from Nginx, syslog, and JSON sources were inconsistent.
   - **Resolution:** Used `awk` and `sed` to normalize log formats for unified analysis.

3. **Automating Log Retention**
   - **Issue:** Manual log rotation was inefficient.
   - **Resolution:** Configured `logrotate` for automatic rotation and compression while ensuring compliance with retention policies.

4. **Identifying SIEM Configuration**
   - **Issue:** The IP address for SIEM-02 wasn’t readily apparent.
   - **Resolution:** Used `grep` to extract relevant lines from the cron configuration file.

---

### **Conclusion**  

Using `rsyslog`, logs were collected and routed efficiently, ensuring precise categorization and accessibility. Automated log retention and compression were implemented with `logrotate`, maintaining storage efficiency while adhering to retention policies. Advanced parsing and normalization techniques using `awk`, `sed`, and `grep` ensured uniformity across diverse log formats, this workflow can be used in incident response, compliance enforcement, and large-scale log management environments.
