# Splunk-App-Auto-Blocker

Python tool + custom Splunk app for automatic IP blocking based on brute-force detection.

## What we built
- Custom Splunk app: threat_blocker
- Custom alert action: Threat Blocker
  - alert_actions.conf: is_custom = true, fields = src_ip
  - command = /usr/bin/python3 /opt/splunk/etc/apps/threat_blocker/bin/threat_blocker.py "$result.src_ip$"
- Alert: Brute Force Block
  - index = linux_security
  - SPL: index=linux_security "Failed password" | rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)" | stats count by src_ip | where count >=5
  - Cron: */5 * * * *
  - Trigger: >0 results → For each result
  - Action: threat_blocker.py $result.src_ip$
- Forwarder monitors /var/log/auth.log → index=linux_security (inputs.conf)
- Python script threat_blocker.py enriches IP (AbuseIPDB, IPinfo, Shodan) and blocks via iptables if score ≥80

## Process we did
1. Created app in Splunk → threat_blocker
2. sudo mkdir -p /opt/splunk/etc/apps/threat_blocker/{bin,local}
3. sudo cp threat_blocker.py config.py /opt/splunk/etc/apps/threat_blocker/bin/
4. sudo chmod +x /opt/splunk/etc/apps/threat_blocker/bin/threat_blocker.py
5. sudo nano /opt/splunk/etc/apps/threat_blocker/local/alert_actions.conf (added custom action)
6. sudo /opt/splunk/bin/splunk restart
7. Created alert in Search & Reporting → added Run script action with $result.src_ip$
8. Tested with Hydra on 192.168.50.20 → checked iptables & logs

## Install & Run
git clone https://github.com/exoshade-arch/Splunk-App-Auto-Blocker.git

cd Splunk-App-Auto-Blocker

python3 -m venv venv && source venv/bin/activate

pip install requests

chmod +x threat_blocker.py

## Create 
(config.py with API keys – never commit)

ABUSEIPDB_API_KEY = "xxx"

IPINFO_TOKEN = "xxx"

SHODAN_API_KEY = "xxx"

sudo ./threat_blocker.py <IP>

## License
MIT – see LICENSE
