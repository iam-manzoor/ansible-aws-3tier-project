## Project Overview

This repository contains an Ansible-driven reference implementation for deploying a secure, isolated 3-tier portfolio application on AWS. The playbooks and roles are organized to provision infrastructure (VPC, subnets, security groups, EC2), configure a small application fleet (load balancer, web servers, database), and perform a one-step teardown to stop billing.

**Key Components**
- **Provisioning**: `provision.yml` — creates VPC, subnets, security groups, and EC2 instances using the `amazon.aws` collection.
- **Configuration**: `deploy.yml` — applies roles to hosts: `common`, `webserver`, `database`, and `loadbalancer`.
- **Teardown**: `destroy.yml` — safely removes AWS resources created by `provision.yml`.
- **Inventory**: `production.aws_ec2.yml` — dynamic EC2 inventory via the `amazon.aws.aws_ec2` plugin.

## About Ansible

Ansible is an agentless automation platform for provisioning, configuration management, and application deployment. It uses simple, human-readable YAML playbooks to describe automation tasks and organizes reusable configuration into `roles`. Key Ansible concepts used in this project:

- **Playbooks**: ordered lists of automation steps (e.g., `provision.yml`, `deploy.yml`).
- **Roles**: modular collections of tasks, handlers, templates and defaults (e.g., `roles/webserver`).
- **Inventory**: defines the hosts to manage; this project uses the EC2 dynamic inventory plugin to discover instances by tag.
- **Vault**: secures sensitive variables like database passwords with encryption.
- **Collections**: packaged modules and plugins (this project requires `amazon.aws` and `community.mysql`).

This repository demonstrates how to combine these Ansible concepts to manage both infrastructure and application configuration in a single, reproducible workflow.

## Architecture Diagram

```mermaid
graph LR
	LB[Load Balancer (NGINX)] --> WS1[Webserver 1]
	LB --> WS2[Webserver 2]
	WS1 --> DB[Database (MariaDB)]
	WS2 --> DB
	subgraph VPC
		LB
		WS1
		WS2
		DB
	end
```

## Quickstart — Prerequisites

- Control node must have Ansible 2.12+ and the required collections installed:

```bash
pip install ansible
ansible-galaxy collection install amazon.aws community.mysql
```

- Ensure AWS credentials are available in the environment (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) or via a configured profile.
- Ensure the `key_name` defined in `group_vars/all/vars.yml` exists in your AWS account.
- The repository expects a vault password file referenced by `ansible.cfg` (`.vault_pass`) or provide `--ask-vault-pass` when running vault operations.

## Important Files & Variables

- **Global variables**: `group_vars/all/vars.yml` — region, instance type, app port, DB name and user.
- **Secrets**: `group_vars/all/vault.yml` — database passwords (encrypted with Ansible Vault).
- **Roles**: `roles/common`, `roles/webserver`, `roles/database`, `roles/loadbalancer`.
- **Inventory**: `production.aws_ec2.yml` — dynamic inventory using EC2 tags to group hosts.

## Common Commands

- Lint repository with `ansible-lint`:
```bash
ansible-lint .
```
- Check playbook syntax:
```bash
ansible-playbook --syntax-check provision.yml
ansible-playbook --syntax-check deploy.yml
ansible-playbook --syntax-check destroy.yml
```
- Provision infrastructure (runs locally against AWS):
```bash
ansible-playbook provision.yml
```
- Apply configuration to inventory-managed hosts (use dynamic inventory):
```bash
ansible-playbook -i production.aws_ec2.yml deploy.yml
```
- Tear down infrastructure:
```bash
ansible-playbook destroy.yml
```

## CI / Linting

This repository includes a GitHub Actions workflow (`.github/workflows/lint.yml`) configured to run `ansible-lint` and basic checks. If you need to ignore a specific rule (for example trailing-spaces in generated templates), consult `ansible-lint` documentation and update the workflow or add a `.ansible-lint` configuration file.

## Best Practices & Notes

- Keep the vault password file out of version control. Use your CI secrets store to provide vault access for automated runs.
- Install required Ansible collections on the control node or in your CI image.
- Validate dynamic inventory locally with `ansible-inventory -i production.aws_ec2.yml --list` before running playbooks.
- The `ansible.cfg` in this repo contains guidance for SSH proxy configuration — avoid placing Jinja inside `ansible.cfg`; instead set per-host `ansible_ssh_common_args` in inventory when necessary.


## License

This project is provided as-is for learning and demonstration purposes. Add a license file if you plan to publish or reuse code commercially.

---
Last updated: 2026-08-16