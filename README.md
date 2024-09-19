Here’s the updated **README** to include the modification of the `inventory.ini` file before running the playbooks.

---

# Project: Node Exporter Secure Deployment and Prometheus Update

This project is designed to securely deploy **Node Exporter** instances on client servers and update the **Prometheus** instance on the Linux-Man infrastructure to scrape metrics from those client servers.

For each client, you will clone the repository into the client folder, modify the configuration file based on the client environment, update the inventory file to reflect the client’s server details, and then run the Ansible playbooks to generate certificates, deploy the Node Exporter, and update Prometheus with the new target.

## Objectives:
1. **Securely deploy Node Exporter**: Generate and deploy TLS certificates to client servers and configure Node Exporter to expose metrics over HTTPS.
2. **Prometheus target update**: Add the new client servers as scrape targets in the **Prometheus** configuration on the Linux-Man infrastructure, using secure TLS settings.
3. **Client-specific project management**: For each client, clone the repository, update the configuration and inventory files, and execute the Ansible playbooks.

---

## Project Workflow for Each Client

### Step 1: Clone the Repository

1. **Navigate to the client folder**:
   Ensure you are in the correct directory for the client you are working with. For example:
   ```bash
   cd /path/to/client-folder
   ```

2. **Clone the repository**:
   Clone the project into the client directory:
   ```bash
   git clone git@gitlab.com:babidi34/monitoring-prometheus.git
   cd node-exporter-secure
   ```

### Step 2: Modify the `variables.yml`

1. Edit the `variables.yml` file to match the specific client details. 
   - Update values such as **client_name**, **node_exporter_version**, **prometheus_scrape_configs**, etc.

   Example:
   ```yaml
   ---
   client_name: "client-xyz"
   node_exporter_version: "1.8.2"
   node_exporter_web_listen_address: "0.0.0.0:9100"
   node_exporter_cert_dir: /etc/node_exporter/

   prometheus_scrape_configs:
     - job_name: "client-xyz-node-exporter"
       metrics_path: "/metrics"
       scheme: https
       tls_config:
         ca_file: "{{ ca_cert_path }}"
         cert_file: "{{ prometheus_tls_cert_path }}"
         key_file: "{{ prometheus_tls_key_path }}"
       static_configs:
         - targets:
             - "client-server.domain.com:9100"
           labels:
             job: "client-xyz"
             environment: "production"
             client: "{{ client_name }}"
   ```

### Step 3: Modify the `inventory.ini`

1. **Edit the `inventory.ini` file** to reflect the client’s infrastructure. Update the IP addresses, usernames, and paths to the SSH private keys as required.

   Example `inventory.ini`:
   ```ini
   [clients]
   client-server-1 ansible_host=192.168.56.101 ansible_user=admin
   client-server-2 ansible_host=192.168.56.102 ansible_user=admin

   [clients:vars]
   ansible_ssh_private_key_file=/path/to/private/key

   [prometheus]
   prometheus-server ansible_host=192.168.56.101 ansible_user=admin

   [prometheus:vars]
   ansible_ssh_private_key_file=/path/to/private/key
   ```

   This step ensures that Ansible knows how to connect to the correct servers for the client and for Prometheus.

### Step 4: Generate Certificates

1. Run the **generate-certs.yml** playbook to generate TLS certificates for the client’s Node Exporter instance:
   ```bash
   ansible-playbook playbooks/generate-certs.yml -e @variables.yml
   ```

### Step 5: Deploy Node Exporter

1. Run the **node-exporter.yml** playbook to install Node Exporter on the client's server and configure it with TLS.
   - Ensure that you have the correct SSH access and inventory setup for the client’s server:
   ```bash
   ansible-playbook playbooks/node-exporter.yml -e @variables.yml -i inventory.ini
   ```

### Step 6: Update Prometheus

1. Run the **update-prometheus.yml** playbook to add the client’s server as a scrape target for your Prometheus instance:
   ```bash
   ansible-playbook playbooks/update-prometheus.yml -e @variables.yml
   ```

2. Prometheus will be updated with the new scrape configuration, and it will start pulling metrics securely from the client's Node Exporter.

### Step 7: Monitor the Results

1. **Verify the Node Exporter**:
   - Navigate to `https://<client-server>:9100/metrics` to ensure that the Node Exporter is serving metrics over HTTPS.

2. **Verify Prometheus**:
   - Log in to your Prometheus instance, go to **Status -> Targets**, and ensure that the new client target is listed and reporting metrics.

---

## Playbooks Overview:

1. **generate-certs.yml**:
   - Generates TLS certificates for Node Exporter.
   
2. **node-exporter.yml**:
   - Deploys and configures Node Exporter on the client’s server, using the generated TLS certificates.

3. **update-prometheus.yml**:
   - Updates the scrape targets in the Prometheus configuration on Linux-Man's infrastructure with secure settings.

---

## Summary of Commands for Each Client:

```bash
# Go to the client folder
cd /path/to/client-folder

# Clone the repository
git clone git@gitlab.com:babidi34/monitoring-prometheus.git
cd node-exporter-secure

# Edit the variables.yml file for the client

# Edit the inventory.ini file to configure client and Prometheus servers

# Generate TLS certificates
ansible-playbook playbooks/generate-certs.yml -e @variables.yml

# Deploy Node Exporter
ansible-playbook playbooks/node-exporter.yml -e @variables.yml -i inventory.ini

# Update Prometheus scrape targets
ansible-playbook playbooks/update-prometheus.yml -e @variables.yml
```

---

This workflow ensures that each client has a secure Node Exporter instance and that your Prometheus server is updated to scrape those metrics with the proper configuration. Each project is fully isolated, allowing you to manage multiple clients easily without forking the repository.