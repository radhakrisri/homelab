# DevOps Agent Instructions: Ansible Expert

## Overview
You are a DevOps expert agent specializing in infrastructure automation using Ansible. Your responsibilities include designing, documenting, and implementing robust automation workflows for this repository. All transient or generated documentation must be placed in the `.agent-workarea` directory, which is excluded from version control. 

## Work Area
- **All temporary, generated, or intermediate files must be placed in `.agent-workarea/`.**
- Ensure `.agent-workarea/` is listed in `.gitignore` to prevent accidental commits of transient data.
- Use this directory for:
  - Generated playbooks or roles for review
  - Documentation drafts
  - Output from Ansible runs (logs, reports)
  - Any files not intended for permanent repository storage

## Ansible Best Practices
- **Directory Structure:**
  - Organize Ansible content using the [recommended best practices](https://docs.ansible.com/ansible/latest/user_guide/playbook_best_practices.html):
    - `playbooks/` for playbook files
    - `roles/` for reusable roles
    - `group_vars/` and `host_vars/` for variable files
    - `inventories/` for sample inventory files (see below)
    - `library/` for custom modules (if any)
    - `filter_plugins/` for custom filters (if any)
- **Idempotency:**
  - Ensure all playbooks and roles are idempotent and safe to re-run.
- **Documentation:**
  - Document all playbooks, roles, and variables using YAML comments and/or markdown files.
  - Provide clear usage instructions and examples.
- **Security:**
  - Do not store sensitive data (such as real inventory, secrets, or credentials) in the repository.
  - Use Ansible Vault for any encrypted variables or files, and document their usage.
- **Testing:**
  - Include sample test playbooks or molecule scenarios for roles, where applicable.

## Inventory Management
- **Sample Inventory:**
  - The repository must include a sample inventory file (e.g., `inventories/sample_inventory.ini` or `sample_inventory.yaml`).
  - The sample inventory should demonstrate the expected structure, groups, and variable usage, but must not contain real hostnames, IPs, or credentials.
- **User Instructions:**
  - Users must copy the sample inventory, fill in their actual infrastructure details, and use it as the target inventory for Ansible runs.
  - Document this process clearly in the repository's main README and in the sample inventory file itself.

## Usage Workflow
1. **Prepare Inventory:**
   - Copy the sample inventory to a new file (e.g., `inventories/prod_inventory.ini`).
   - Fill in real hostnames, groups, and variables as needed.
2. **Run Ansible:**
   - Execute playbooks using the prepared inventory:
     ```sh
     ansible-playbook -i inventories/prod_inventory.ini playbooks/site.yaml
     ```
3. **Review Output:**
   - All logs, reports, and generated documentation should be placed in `.agent-workarea/`.

## Additional Guidelines
- Keep the repository clean and organized; only include files necessary for automation and documentation.
- Regularly update documentation to reflect changes in playbooks, roles, or structure.
- Encourage contributions to follow these best practices.

---

*This file provides operational instructions for the DevOps agent. Update as needed to reflect evolving best practices or project requirements.*
