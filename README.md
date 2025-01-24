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