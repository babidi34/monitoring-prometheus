# Project: Node Exporter Secure Deployment and Prometheus Update

This project is designed to securely deploy **Node Exporter** instances on client servers and update the **Prometheus** instance on the Linux-Man infrastructure to scrape metrics from those client servers.

For each client, you will clone the repository into the client folder, modify the configuration file based on the client environment, update the inventory file to reflect the client’s server details, and then run the Ansible playbooks to generate certificates, deploy the Node Exporter, and update Prometheus with the new target.

## Installing Ansible Builder and Ansible Runner

Before running this project, you need to install **Ansible Builder** and **Ansible Runner**. These tools allow you to create a containerized execution environment and run Ansible playbooks inside it.

### Install Ansible Builder

1. **Using `pip`**:
   ```bash
   pip install ansible-builder
   ```

### Install Ansible Runner

1. **Using `pip`**:
   ```bash
   pip install ansible-runner
   ```


## Building the Execution Environment with Ansible Builder

Once Ansible Builder is installed, you can use it to build the execution environment for the project.

1. **Navigate to the project directory**:
   ```bash
   cd /path/to/client-folder/node-exporter-secure
   ```

2. **Build the execution environment**:
   Ansible Builder will use the `execution-environment.yml` file to build a containerized environment that contains all the necessary dependencies (Ansible collections, roles, Python packages, etc.).

   ```bash
   ansible-builder build --tag monitoring-ee:latest
   ```

   This command creates a Docker image called `monitoring-ee:latest` with all the dependencies required for this project.

---

## Running the Playbooks with Ansible Runner

After building the execution environment, use Ansible Runner to execute the playbooks inside the containerized environment.

1. **Run the playbooks**:
   Replace `<playbook_name>` with the appropriate playbook (e.g., `generate-certs.yml`, `node-exporter.yml`, `update-prometheus.yml`), and provide the necessary variables and inventory file.

   ```bash
   ansible-runner run . --playbook <playbook_name> -e @variables.yml -i inventory.ini --container-image monitoring-ee:latest
   ```

   Example:
   ```bash
   ansible-runner run . --playbook node-exporter.yml -e @variables.yml -i inventory.ini --container-image monitoring-ee:latest
   ```

   This command runs the playbook in the containerized environment, ensuring all dependencies are properly handled.

---

## Project Workflow for Each Client

### Step 1: Clone the Repository

1. **Navigate to the client folder**:
   ```bash
   cd /path/to/client-folder
   ```

2. **Clone the repository**:
   ```bash
   git clone git@gitlab.com:babidi34/monitoring-prometheus.git
   cd monitoring-prometheus
   ```

### Step 2: Modify the `variables.yml`

Edit the `variables.yml` file to match the client-specific details (e.g., `client_name`, `node_exporter_version`, `prometheus_scrape_configs`, etc.).

### Step 3: Modify the `inventory.ini`

Edit the `inventory.ini` file to reflect the client’s infrastructure. Update the IP addresses, usernames, and paths to SSH keys.

### Step 4: Generate Certificates

1. **Run the `generate-certs.yml` playbook** to generate TLS certificates for the client’s Node Exporter instance:
   ```bash
   ansible-runner run . --playbook generate-certs.yml -e @variables.yml --container-image monitoring-ee:latest
   ```

### Step 5: Deploy Node Exporter

1. **Run the `node-exporter.yml` playbook** to install and configure Node Exporter on the client’s server with TLS:
   ```bash
   ansible-runner run . --playbook node-exporter.yml -e @variables.yml -i inventory.ini --container-image monitoring-ee:latest
   ```

### Step 6: Update Prometheus

1. **Run the `update-prometheus.yml` playbook** to add the client’s server to Prometheus' scrape targets:
   ```bash
   ansible-runner run . --playbook update-prometheus.yml -e @variables.yml --container-image monitoring-ee:latest
   ```

### Step 7: Monitor the Results

1. **Verify Node Exporter**: Navigate to `https://<client-server>:9100/metrics` to check if Node Exporter is serving metrics securely over HTTPS.

2. **Verify Prometheus**: Log into your Prometheus instance, go to **Status -> Targets**, and verify that the new client server is listed and reporting metrics.

---

## Summary of Commands for Each Client:

```bash
# Go to the client folder
cd /path/to/client-folder

# Clone the repository
git clone git@gitlab.com:babidi34/monitoring-prometheus.git
cd node-exporter-secure

# Build the Ansible execution environment
ansible-builder build --tag monitoring-ee:latest

# Generate TLS certificates
ansible-runner run . --playbook generate-certs.yml -e @variables.yml --container-image monitoring-ee:latest

# Deploy Node Exporter
ansible-runner run . --playbook node-exporter.yml -e @variables.yml -i inventory.ini --container-image monitoring-ee:latest

# Update Prometheus scrape targets
ansible-runner run . --playbook update-prometheus.yml -e @variables.yml --container-image monitoring-ee:latest
```

---

This workflow ensures that each client has a secure Node Exporter instance and that your Prometheus server is updated to scrape those metrics with the proper configuration. Each project is fully isolated, allowing you to manage multiple clients easily.
