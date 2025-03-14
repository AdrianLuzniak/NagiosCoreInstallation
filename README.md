# Nagios Core Installation 
NagiosCoreInstallation is a script created with **Ansible** that automates the installation of Nagios Core on the host and client machines It checks system status, installs required packages, and configures Nagios for seamless infrastructure monitoring on both server and client environments.


## Key Features
- Automated installation of Nagios Core Server and Clients using Ansible.
- Configurable role variables for customization.
- Dynamic inventory management.
- SELinux exception handling for Nagios.
- Firewall configuration for required ports.
- Plugin installation and management.
---
## Prerequisites

Ensure Ansible is installed on your control node. Follow the steps below to set it up if not already installed.

##### Installing Ansible
For CentOS/RHEL-based systems, use the following commands:

```bash
sudo yum install epel-release -y
sudo yum install ansible -y
```
Verify the installation:

```bash
ansible --version
```
---

## Network Configuration

When configuring network settings for virtual machines, it's important to choose the right networking mode based on how you want the machines to behave in your network.

##### Bridge Adapter
If your virtual machines (VMs) are configured with a **Bridge Adapter**, they will behave like separate physical devices on your network. Each VM will receive its own **unique IP address** from your local network (just like any other computer connected to the network). This means that each machine will be identified as an independent device, and you can access them directly using their IP addresses, just as you would with physical machines.

##### NAT Network
If you're using **NAT Network** for your virtual machines, all the VMs will share the same **external IP address** as the physical computer (host machine) they are running on. This means that, from the perspective of the network, all your VMs will appear as the same device. To differentiate between VMs and allow individual access, you will need to use **different ports** for each virtual machine.

In this case, you'll also need to configure **port forwarding** in your virtualization software (e.g., VirtualBox). This allows you to specify which port on the host machine corresponds to each VM, enabling external access to the VMs through different ports. 
For example, you might forward port **8080** to **VM1** and port **8081** to **VM2**.

##### How to Configure Port Forwarding in VirtualBox:
1. Open the VirtualBox Manager and select the virtual machine.
2. Go to **Settings** > **Network**.
3. Under the **Adapter** tab, ensure the network is set to **NAT Network**.
4. Click on the **Advanced** dropdown, then click on **Port Forwarding**.
5. Add a rule that maps a specific port on the host (e.g., 8080) to the internal IP of the VM.

By doing this, each VM will be accessible through a different port on the host machine, even though they share the same external IP address.

---

## Role Description

##### system_setup
* Temporarily disables SELinux.
* Installs EPEL repository.
* Updates system packages.

##### install_nagios_dependencies
Installs all required packages for Nagios Core Server and Client installation.


##### install_nagios_core_server
* Installs Nagios server with all necessary code compilations.

* Allows setting login credentials via variables.

* Creates the nagios user and adds it to the nagcmd group.

* Assigns proper file permissions.

* Uses temporary directories for installation.

##### Variables:

**`force_install_nagios: false`** - Set to ```true``` to force the installation of Nagios Core Server, even if it is already installed. By default playbook checks if the key nagios folder exists, if so, the installation is skipped, in this case, you need to set the value of this variable to ```true```.

**`nagios_admin_username: "nagiosadmin"`** - The username for the Nagios administrator. 

**`nagios_admin_password: "password"`** - The password for the Nagios administrator.

##### configure_firewall
Opens necessary TCP ports for Nagios, defined in the ports variable.

##### Variables:
**`ports`**: A list of TCP ports to be opened in the firewall.

##### restart_nagios_and_check_service_status 
Restarts Nagios and verifies its status.


##### add_nagios_to_selinux_exception
Prepares a SELinux module `NagiosRules.te`, transfers it to the machine where the server is installed, loads the module into SELinux, and reloads SELinux to allow Nagios to run with SELinux enabled.

##### Variables:
- **`nagios_module_name`**: `"NagiosRules"`  
  (Name of the SELinux module.)
  
- **`nagios_rules_file`**: `{{ nagios_module_name }}.te`  
  (File path of the SELinux module.)

- **`nagios_module_priority`**: `"300"`  
  (Priority of the SELinux module.)

- **`local_nagios_rules_path`**: `{{ role_path }}/files/{{ nagios_module_name }}.te`  
  (Path to the SELinux rules file in the role.)

- **`force_add_module`**: `true`  
  (If `true`, forces adding the module to SELinux, even if already exist)



##### remove_nagios_from_selinux_exception
Removes the previously added `NagiosRules.te` module from SELinux.

##### Variables:
- **`nagios_module_name`**: `"NagiosRules"`  
  (Name of the SELinux module.)
  
- **`nagios_rules_file`**: `{{ nagios_module_name }}.te`  
  (File path of the SELinux module.)

- **`nagios_module_priority`**: `"300"`  
  (Priority of the SELinux module.)

- **`local_nagios_rules_path`**: `{{ role_path }}/files/{{ nagios_module_name }}.te`  
  (Path to the SELinux rules file in the role.)

- **`force_add_module`**: true **#not used**

##### set_server_custom_name_and_config_paths
Creates the Nagios server configuration file using the `nagios.cfg.j2` template. It also checks whether the directories defined in the configuration exist and generates the `localhost.cfg` file based on the `localhost.cfg.j2` template. Services are dynamically added to `nrpe.cfg` based on variables in the playbook.


##### install_nagios_client
Installs the NRPE client on client machines along with the necessary plugins. The role dynamically fetches the IP of the Nagios server from the inventory.

### Variables:
- **`nagios_plugins_version`**: `"2.4.4"`  
  (Version of the Nagios plugins to be installed.)

- **`force_install_nrpe_and_plugins`**: `true`  
  (If `true`, forces installation of NRPE and plugins even if they are already installed.)

- **`add_allowed_hosts`**: `true`  
  (If `true`, adds allowed hosts to the NRPE client configuration.)

- **`add_nrpe_commands`**: `true`  
  (If `true`, adds NRPE command configurations to the client.)

- **`default_nrpe_directory`**: `"/usr/lib64/nagios/plugins"`  
  (Default directory for NRPE plugins.)

- **`plugins_directory`**: `"/usr/local/nagios/libexec"`  
  (Directory where the plugins will be placed on the client machine.)

- **`nrpe_commands`**: A list of NRPE commands for system monitoring, for example:
  - `{ name: "check_load", command: "{{ plugins_directory }}/check_load -w 5,4,3 -c 10,6,4" }`
  - `{ name: "check_disk", command: "{{ plugins_directory }}/check_disk -w 20% -c 10% -p /" }`

---
  ## Inventory configuration
```yml
  all:
  children:
    servers:
      hosts:
        AnsibleTarget1:
          ansible_host: <YOUR_SERVER_IP>
          ansible_ssh_user: <YOUR_SSH_USER>
          ansible_ssh_pass: <YOUR_SSH_PASSWORD>
          ansible_become: yes
          ansible_become_method: sudo
          ansible_become_user: root
          ansible_become_pass: <YOUR_ROOT_PASSWORD>

    clients:
      hosts:
        AnsibleTarget2:
          ansible_host: <YOUR_CLIENT_IP>
          ansible_ssh_user: <YOUR_SSH_USER>
          ansible_ssh_pass: <YOUR_SSH_PASSWORD>
          ansible_become: yes
          ansible_become_method: sudo
          ansible_become_user: root
          ansible_become_pass: <YOUR_ROOT_PASSWORD>
```


## Playbook configuration and execution

To install Nagios Server playbook `install_nagios_core_server` should be configured like this:
```yml
- name: Setup Nagios Monitoring Server
  hosts: AnsibleTarget1
  become: true
  roles:
    #- system_setup
    - install_nagios_dependencies
    - install_nagios_server
    - install_nrpe_on_nagios_server
    - configure_firewall
    - install_nagios_plugins
    - restart_nagios_check_service_status
    - add_nagios_to_selinux_exception
    # - remove_nagios_from_selinux_exception
    - set_server_custom_name_and_config_paths
```

To install **Nagios Core Server** run the following command:

```bash
ansible-playbook -I inventory.yml install_nagios_core_server.yml
```


If EPEL repository is missing on your system you can uncomment `system_setup`, but linux package conflicts must be resolved manually, you can find more info in `troubleshooting` section.

To install **Nagios Client** playbook `install_nagios_core_client` should be configured like this:

```yml
- name: Setup Nagios Monitoring Client
  hosts: clients
  become: true
  roles:
    - install_nagios_dependencies
    - install_nagios_client
  tasks:
    - name: Extract NRPE command names for Nagios Client
      set_fact:
        nrpe_commands: "{{ nrpe_commands }}"

- name: Ensure nrpe_commands is available from clients
  hosts: servers
  become: true
  tasks:
    - name: Retrieve nrpe_commands from a client host
      set_fact:
        nrpe_commands: "{{ hostvars[item].nrpe_commands }}"
      loop: "{{ groups['clients'] }}"
    
    - name: Add clients to Nagios Server
      include_role:
        name: add_nagios_clients_to_server
```

To install **Nagios Core Client** run the following command (**server must be installed first!**):

```bash
ansible-playbook -I inventory.yml install_nagios_core_client.yml
```

---
## SELinux Useful Information

## Checking SELinux Logs:
- **`sealert -a /var/log/audit/audit.log`**  
  Analyzes the SELinux audit log for alerts.

- **`ausearch -m avc -ts recent`**  
  Displays recent SELinux access vector cache (AVC) denials in the logs.

## Compiling SELinux Module:
- **`checkmodule -M -m -o NagiosRules.mod NagiosRules.te`**  
  Compiles a SELinux module from a `.te` policy file into a `.mod` file.

- **`semodule_package -o NagiosRules.pp -m NagiosRules.mod`**  
  Creates a SELinux package (`.pp`) from the compiled module `.mod`.

## Loading and Verifying SELinux Modules:
- **`semodule -X 300 -i NagiosRules.pp`**  
  Installs the SELinux module with priority 300.

- **`semodule -l | grep nagios_rules`**  
  Verifies if the SELinux module is loaded correctly.

- **`audit2allow -M NagiosRule < /var/log/audit/audit.log`**  
  Generates SELinux policy modules from audit logs to allow previously denied actions.

- **`semodule -i NagiosRule.pp`**  
  Installs a module even if the `.pp` file does not exist (use with caution).

## Displaying SELinux Modules:
- **`semodule --list-modules=full`**  
  Lists all SELinux modules, showing the full details, including priorities.

## Removing SELinux Modules:
- **`semodule -r NagiosRules`**  
  Attempts to remove a module, but may fail if the priority is too high.

- **`semodule -X 300 -r NagiosRules`**  
  Removes the SELinux module at priority 300.

### SELinux Module Priority Information:
- SELinux modules have an assigned priority number, which determines the order in which modules are loaded or applied.
- Modules with higher priorities are loaded later and can overwrite rules from modules with lower priorities.
- Higher priority modules take precedence in case of conflicts.

##### Example
If the module was added with **priority 300** then the command:

`semodule -r NagiosRules`

Will return an error:
```bash
libsemanage.semanage_direct_remove_key: Unable to remove module NagiosRules at priority 400. (No such file or directory).
semodule: Failed!
```
You should use

`semodule -X 300 -r NagiosRules`

---

## Nagios Useful Information

##### Configuration Files:
- **`/usr/local/nagios/etc/nagios.cfg`**  
  Main Nagios configuration file to point to the folders or files to monitor.

- **`/usr/local/nagios/etc/objects`**  
  Directory containing objects like services, hosts, etc.

- **`/usr/local/nagios/etc/host`**  
  Defines the names of the servers to be monitored.

- **`/usr/local/nagios/etc/services`**  
  Defines details of the services to monitor.

- **`/usr/local/nagios/etc/windows`**  
  Configurations for Windows servers to monitor both host and services.

- **`/usr/local/nagios/etc/windows/windowsservers.cfg`**  
  Configuration for monitoring Windows servers. Each server should have its own file.

- **`/usr/local/nagios/etc/vmware`**  
  Directory for VMware vCenter and ESX host configuration files.

- **`/usr/local/nagios/etc/object/command.cfg`**  
  Contains commands used for sending notifications, such as email configurations.

##### Email Configuration:
1. **Install `mailx`**:
   - **`dnf install mailx`**  
     Installs the `mailx` package for email functionality.

2. **Test Email Command**:
   - **`echo "Mail Body - Test Message from Nagios Digital Avenue" | mailx -vvv -s "Subject is Mail Sending from The Agile Ops" ibrahim@gmail.com`**  
     Sends a test email to verify email functionality in Nagios.

##### Log Files (Important):
- **`/usr/local/nagios/etc/objects/commands.cfg`**  
  File responsible for sending notifications for emails.

- **`/usr/local/nagios/etc/objects/contacts.cfg`**  
  Create contacts for Out of Hours (OOH) or distribution lists for OOH notifications.

- **`/usr/local/nagios/etc/resource.cfg`**  
  Resource file containing configuration details.

- **`/etc/postfix/main.cf`**  
  Postfix configuration file for email notifications (e.g., Gmail setup).

- **Log Files**:
  - **`/var/log/maillog`**  
    Logs related to email activities.
  
  - **`/var/log/messages`**  
    General system logs.

  - **Nagios Logs**:
    - **`/usr/local/nagios/var/nagios.log`**  
      Logs related to Nagios events.

##### Service Management:
- **`systemctl restart postfix `**  
  Restarts the Postfix service for email notifications.

- **`systemctl restart nagios`**  
  Restarts the Nagios service.

##### Config Validation:
- **`/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg`**  
  Validates the Nagios configuration file.

##### NRPE Plugin Commands and Locations:
- **Plugin Paths:**
  - **`/usr/local/nagios/libexec/check_nrpe -H localhost -c check_load`**
  - **`/usr/lib64/nagios/plugins/check_nrpe -H localhost -c check_load`**

- **Configuration Path**:
  - **`/etc/nagios/nrpe.cfg`**  
    Configuration file for NRPE.

- **Check Load Command**:
  - **`/usr/local/nagios/libexec/check_load -w 5,4,3 -c 10,6,4`**  
    Defines load thresholds for monitoring.

##### Searching for Monitor Definitions:
- **Search for Monitor in Nagios:**
  - **`grep -r "check_command.*check_http" /usr/local/nagios/etc/`**
  - **`grep -A 10 "define service" /usr/local/nagios/etc/objects/localhost.cfg`**

##### Manually Triggering a Monitor Check:
- **`echo "[`date +%s`] PROCESS_SERVICE_CHECK_RESULT;localhost;HTTP;0;OK - Manual check triggered" > /usr/local/nagios/var/rw/nagios.cmd`**  
  Manually triggers a service check in Nagios.

##### Checking Nagios Logs:
- **`tail -f /usr/local/nagios/var/nagios.log`**  
  Displays the latest logs from Nagios in real-time.

## TODO List
##### install_nagios_core_server
* Add variables to select Nagios version in the download link.

* Add a variable to store the installation location.

##### remove_nagios_from_selinux_exception
* Expose role variables from add_nagios_to_selinux_exception.

##### install_nagios_client
* Adjust monitoring thresholds.
* Fix configurations for monitors that return errors.

##### add_nagios_clients_to_server
* Separate host group definitions into a separate file to prevent config duplication.

##### install_nagios_plugins_dependencies
* To delete not used

## Troubleshooting
#### SSL Certificate Issue During Package Installation
##### 1. Check System Date and Time  
Check the date and time on the system:

```bash
date
```
If the date or time is incorrect, correct it. You can do this manually or using **NTP (Network Time Protocol)**. To synchronize the time with an NTP server, use the following command (**it usually fixes the problem**):
```bash
sudo timedatectl set-ntp true
sudo shutdown -r
```
##### 2. Update Repositories
Try updating the repositories:
```bash
sudo dnf upgrade --refresh -y
```

##### 3. Install missing CA certificates
```bash
sudo dnf reinstall -y ca-certificates
sudo update-ca-trust
```

##### 4. Check SSL Certificate
Try to manually connect to the repository using curl and check if the problem persists:
```bash
curl -v https://mirrors.centos.org/metalink?repo=centos-baseos-9-stream&arch=x86_64&protocol=https,http
```
This will allow you to see details of the SSL certificate error.

#### Resolving Package Conflicts During Package Installation / System Upgrade
##### 1. Identify Conflicting Packages
If there is a conflict between two packages, you need to remove the already installed package and reattempt the package update. First, check which packages are causing the conflict:

```bash
rpm -q tuned-ppd power-profiles-daemon
```
##### 2. Remove Problematic Packages
Remove the conflicting packages (only those that were installed):
```bash
sudo dnf remove tuned-ppd -y
sudo dnf remove power-profiles-daemon -y
```

If `dnf` **does not work**, remove them using rpm:
```bash
sudo rpm -e --nodeps tuned-ppd
sudo rpm -e --nodeps power-profiles-daemon
```
##### 3. Clean Cache
Clean the dnf cache to ensure no stale metadata is causing issues:
```bash
sudo dnf clean all
```

##### 4. Retry Upgrade
Finally, retry the system upgrade:
```bash
sudo dnf update -y
```

## The Prosperity Public License 3.0.0

**Contributor**: Adrian Łuźniak  
**Source Code**: [GitHub Repository](https://github.com/AdrianLuzniak/NagiosCoreInstallation)

##### Purpose
This license allows you to use and share the software for non-commercial purposes for free, and to try it for commercial purposes for up to thirty days.

##### Agreement
By receiving this license, you agree to follow its rules, which are both obligations and conditions for using the software. Do not use the software in any way that would violate these rules.

##### Notices
Ensure that anyone who receives a copy of the software, with or without modifications, also receives the text of this license along with the contributor and source code details.

##### Commercial Trial
You may use the software commercially for up to thirty days. If your company uses it for work, the trial period applies to the entire company, not individually per person.

##### Contributions Back
Contributions made back to the project, such as feedback, changes, or additions, do not count as commercial use if provided under a standardized public software license like the [Blue Oak Model License 1.0.0](https://blueoakcouncil.org/license/1.0.0), [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0.html), [MIT License](https://spdx.org/licenses/MIT.html), or [BSD-2-Clause License](https://spdx.org/licenses/BSD-2-Clause.html).

##### Personal Uses
Non-commercial uses, such as personal research, testing, or hobby projects, do not count as commercial use.

##### Noncommercial Organizations
Charitable organizations, educational institutions, public research organizations, public safety or health organizations, environmental protection groups, or government entities may use the software without it counting as commercial use.

##### Defense
You may not make any legal claims accusing others of patent infringement regarding this software, whether modified or unmodified, on its own or combined with other technologies.

##### Copyright
The contributor grants you permission to use the software without infringing on their copyright.

##### Patent
The contributor also grants you a license to use the software without infringing on any patents they can license.

##### Reliability
The contributor cannot revoke this license.

##### Excuse
If you unknowingly violate the "Notices" section, you are excused as long as you take steps to comply within thirty days of discovering the violation.

##### No Liability
**As far as the law allows, this software is provided "as is" without any warranty or condition. The contributor is not liable for any damages related to the software or license under any legal claims.**


