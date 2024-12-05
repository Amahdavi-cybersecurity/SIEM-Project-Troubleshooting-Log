# SIEM-Project-Troubleshooting-Log

A detailed account of the challenges faced and the solutions implemented during the SIEM setup.

## 1. Elasticsearch Service Issues

Problem:

The Elasticsearch service failed to start after updating the configuration file (elasticsearch.yml).
Troubleshooting Steps:

Verified syntax in elasticsearch.yml:

Checked for indentation and typos in critical parameters like network.host and discovery.seed_hosts.

Ensured that network.host was set to the server's IP address (192.168.1.174).

Restarted the service:

sudo systemctl restart elasticsearch

Reviewed the status and logs:

sudo systemctl status elasticsearch

sudo journalctl -xeu elasticsearch.service

Identified errors related to incorrect configurations or missing dependencies.

Corrected the network.host setting to bind only to the required address instead of localhost.

**Outcome:** Restarted Elasticsearch successfully after fixing configuration issues.

## 2. Connectivity to Elasticsearch

Problem:

Unable to connect to Elasticsearch on port 9200.
Troubleshooting Steps:

Verified Elasticsearch was listening on the correct IP and port:

sudo netstat -tuln | grep 9200

Found that Elasticsearch was listening only on 127.0.0.1.

Updated the network.host setting to the correct IP (192.168.1.174) in elasticsearch.yml:

network.host: 192.168.1.174

discovery.seed_hosts: ["192.168.1.174"]

Restarted the Elasticsearch service:

sudo systemctl restart elasticsearch

Verified connectivity using:

curl -X GET "http://192.168.1.174:9200" -u elastic

**Outcome:** Confirmed Elasticsearch was accessible on the correct IP and port.

## 3. Filebeat Configuration

Problem:

Filebeat service failed to start due to misconfigured modules.

Troubleshooting Steps:

Reviewed the Filebeat service status:

sudo systemctl status filebeat

Identified errors related to missing or misconfigured modules (e.g., netflow).

Disabled problematic modules:

sudo filebeat modules disable netflow

Verified active modules:

sudo filebeat modules list

Re-enabled required modules (apache, mysql, system):

sudo filebeat modules enable apache mysql system

Tested Filebeat configuration:

sudo filebeat test config

Restarted the service:

sudo systemctl restart filebeat

**Outcome:** Resolved Filebeat issues and ensured logs were being forwarded to Elasticsearch.

## 4. Filebeat Connectivity to Elasticsearch
Problem:

Filebeat couldn’t send logs to Elasticsearch, showing connection errors.

Troubleshooting Steps:

Tested connectivity from Filebeat to Elasticsearch:


sudo filebeat test output

Encountered connection refused errors due to incorrect Elasticsearch host configuration.

Updated filebeat.yml:

output.elasticsearch:

  hosts: ["http://192.168.1.174:9200"]
  
  username: "elastic"
  
  password: "new_password"
  
Restarted Filebeat:

sudo systemctl restart filebeat

Verified log ingestion:

curl -X GET "http://192.168.1.174:9200/filebeat-*/_count" -u elastic

**Outcome:** Filebeat successfully connected to Elasticsearch and started shipping logs.

## 5. Kibana Configuration

Problem:

Kibana failed to connect to Elasticsearch due to authentication errors.

Troubleshooting Steps:

Verified Kibana service status:


sudo systemctl status kibana

Updated kibana.yml to use the correct Elasticsearch host and credentials:


elasticsearch.hosts: ["http://192.168.1.174:9200"]

elasticsearch.username: "elastic"

elasticsearch.password: "new_password"

Restarted Kibana:

sudo systemctl restart kibana

Accessed Kibana via the browser at http://192.168.1.174:5601.

**Outcome:** Successfully connected Kibana to Elasticsearch and accessed the interface.

## 6. General Troubleshooting Commands
   
Key commands used throughout the project:

Check service status:

sudo systemctl status <service_name
                        
Restart a service:

sudo systemctl restart <service_name>

View detailed logs:

sudo journalctl -xeu <service_name>

Test Elasticsearch connection:

curl -X GET "http://<elasticsearch_ip>:9200" -u <username>

List enabled Filebeat modules:

sudo filebeat modules list

Test Filebeat configuration:

sudo filebeat test config

## 7. Resetting the Elasticsearch Password

Problem:

Lost or forgotten Elasticsearch elastic user password.

Troubleshooting Steps:

Reset the password using the elasticsearch-reset-password tool:

sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic

Updated all configuration files (filebeat.yml, kibana.yml) with the new password.

**Outcome:** Successfully restored access to Elasticsearch.
