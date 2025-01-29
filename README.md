# NagiosCoreInstallation
NagiosCoreInstallation is a script that automates the installation of Nagios Core on the host and client machines, if needed. It checks system status, installs required packages, and configures Nagios for seamless infrastructure monitoring on both server and client environments.

v1. Nagios installed properly, need to turn off selinux (I) or manually trigger check of one monitor (audit.log must have errors to create selinux exceptions) then execute commands to add exceptions to selinux (II):
Check if selinux is on: getenforce
I.
Turn off selinux using command: setenforce 0

Then 
sudo systemctl daemon-reload
sudo systemctl restart nagios


Or trigger manual check of any monitor, then on root execute commands (one by one)

sudo du root

For cmd.cgi
ausearch -c 'cmd.cgi' --raw | audit2allow -M my-cmdcgi
semodule -X 300 -i my-cmdcgi.pp

For status.cgi:
ausearch -c 'status.cgi' --raw | audit2allow -M my-statuscgi
semodule -X 300 -i my-statuscgi.pp

system_setup – SELinux setup, EPEL installation and system updates
install_nagios_dependencies – Nagios dependency installation
install_nagios_server – Nagios installation and configuration
install_nagios_plugins_dependencies – Nagios plugin dependency installation
install_nagios_plugins – Nagios plugin installation and configuration

Szukanie desc monitora w nagios:
grep -r "check_command.*check_http" /usr/local/nagios/etc/
grep -A 10 "define service" /usr/local/nagios/etc/objects/localhost.cfg


- name: Debug selinux_enabled rc
  debug:
    msg: "SELinux status: {{ ansible_facts.selinux.status }}, Selinux mode: {{ ansible_facts.selinux.mode }}"


To check, why: 

TASK [add_nagios_to_selinux_exception : Allow Nagios cmd.cgi access through SELinux if SELinux is enabled and errors are found] *******************************************************************************
skipping: [AnsibleTarget1]

TASK [add_nagios_to_selinux_exception : Allow Nagios status.cgi access through SELinux if SELinux is enabled and errors are found] ****************************************************************************
skipping: [AnsibleTarget1]

TASK [add_nagios_to_selinux_exception : Reload systemd daemon if SELinux is enabled and errors are found] *****************************************************************************************************
skipping: [AnsibleTarget1]

TASK [add_nagios_to_selinux_exception : Restart Nagios service if SELinux is enabled and errors are found] ****************************************************************************************************
skipping: [AnsibleTarget1]
