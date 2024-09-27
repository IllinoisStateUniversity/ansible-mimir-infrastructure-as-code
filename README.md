# Mimir - Infrastructure as Code

This repository contains an Ansible playbook designed to automate the deployment of Grafana Mimir, scalable time-series database, onto an OpenShift cluster using Infrastructure as Code (IaC) principles.

## Overview

The Ansible playbook in this repository automates the deployment of Grafana Mimir on an OpenShift cluster. It performs the following tasks:

1. **Retrieves Mimir Ceph credentials** from Keeper Secrets Manager.
2. **Creates a ConfigMap** for Mimir configuration.
3. **Deploys a Mimir instance** as a Kubernetes Deployment.
4. **Exposes the Mimir service** via a Kubernetes Service resource.

## Prerequisites

- **Ansible Automation Platform (AAP) or AWX**: The playbook is designed to run within AAP or AWX.
- **Access to an OpenShift Cluster**: Ensure you have the necessary permissions.
- **Keeper Secrets Manager**: For securely retrieving secrets like access keys.

## Configuration

### Variables

Variables are defined in the `vars/` directory for different environments (e.g., `prod.yml`, `test.yml`). These variables include:

- `openshift_namespace`: The OpenShift namespace where Mimir will be deployed.
- `s3_endpoint`: The S3 endpoint for the storage backend.
- `blocks_bucket_name`: The name of the S3 bucket for time-series data storage.
- `ruler_bucket_name`: The name of the S3 bucket for rule storage.

### Secrets Management

Secrets such as `access_key_id` and `secret_access_key` are retrieved securely from Keeper Secrets Manager. Ensure you have the necessary permissions and configurations set up for accessing Keeper.

## Playbook Structure

- **`site.yml`**: The main playbook file that orchestrates the deployment tasks.
- **`vars/`**: Contains environment-specific variable files.
  - `prod.yml`
  - `test.yml`
- **`templates/`**: Contains Jinja2 templates for Kubernetes resource definitions.
  - `mimir-config.yml.j2`: Template for the Mimir ConfigMap.
  - `mimir-deploy.yml.j2`: Template for the Mimir Deployment.
- **`collections/requirements.yml`**: Specifies the required Ansible collections.

## Tasks Breakdown

1. **Get Mimir Ceph Account from Keeper**
   - Uses the `keepersecurity.keeper_secrets_manager.keeper_get_record` module to retrieve the Ceph credentials securely.
   - Stores the credentials in the `keeper_mimir_ceph` variable.
   
2. **Create ConfigMap for Mimir Configuration**
   - Uses the `redhat.openshift.k8s` module to create or update a ConfigMap named `mimir-config`.
   - The ConfigMap includes the `config.yaml` file generated from the `mimir-config.yml.j2` template.

3. **Deploy Mimir Instance**
   - Deploys Mimir using a Kubernetes Deployment resource defined in the `mimir-deploy.yml.j2` template.
   - Ensures that the Mimir pod is running with the correct configuration and image.

4. **Expose the Mimir Port via Services**
   - Creates a Kubernetes Service named `mimir-service` to expose Mimir on port `9009`.
   - This allows external access to the Mimir instance within the cluster.
