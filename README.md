ansible-aws-3tier-portfolio/
├── .github/
│   └── workflows/
│       └── lint.yml          # GitHub Actions for automated code quality checks
├── group_vars/
│   ├── all/
│   │   ├── vars.yml          # Public variables (Ports, DB names, AWS Region)
│   │   └── vault.yml         # Encrypted variables (Passwords, Secret keys)
│   ├── webservers.yml        # Configurations specific to application nodes
│   └── loadbalancers.yml     # Configurations specific to the Nginx node
├── roles/
│   ├── common/               # Applies to ALL servers (Security, Updates, Utilities)
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── handlers/
│   │       └── main.yml
│   ├── database/             # Installs & secures PostgreSQL/MySQL
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   └── templates/
│   │       └── pg_hba.conf.j2
│   ├── webserver/            # Deploys backend app, passes Environment Variables
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   └── app.service.j2
│   │   └── handlers/
│   │       └── main.yml
│   └── loadbalancer/         # Configures Nginx with Jinja2 templates & Handlers
│       ├── tasks/
│       │   └── main.yml
│       ├── templates/
│       │   └── nginx.conf.j2
│       └── handlers/
│           └── main.yml
├── ansible.cfg               # Ansible settings (Default inventory path, vault password file)
├── inventory_aws.yml         # Dynamic inventory configuration file for AWS EC2
├── provision.yml             # Playbook 1: Boots up AWS infrastructure (VPC, SG, EC2)
├── deploy.yml                # Playbook 2: Master configuration playbook that triggers roles
├── destroy.yml               # Playbook 3: 1-Click teardown script to keep AWS costs at $0
├── .gitignore                # Protects your local vault password files from being pushed
└── README.md    