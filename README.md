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



Wysyłanie żądania - manuallny trigger monitora:
echo "[`date +%s`] PROCESS_SERVICE_CHECK_RESULT;localhost;HTTP;0;OK - Manual check triggered" > /usr/local/nagios/var/rw/nagios.cmd

Spradzanie logów nagios 
tail -f /usr/local/nagios/var/nagios.log

grep "host_name" /usr/local/nagios/etc/objects/*.cfg
cat /usr/local/nagios/etc/objects/localhost.cfg

Szukanie desc monitora w nagios:
grep -r "check_command.*check_http" /usr/local/nagios/etc/
grep -A 10 "define service" /usr/local/nagios/etc/objects/localhost.cfg


______________________________________________________________________________________
Logi selinux /var/log/audit/audit.log

Dodawanie modułu nagiosa jako root po instalacji:

Opróżnić plik audit log
> /var/log/audit/audit.log


Wkonać sudo su root

setenforce 0

Po instalacji nagiosa kliknąć dowolny monitor 	Re-schedule the next check of this service 

pojawi się error Error: Could not stat() command file '/usr/local/nagios/var/rw/nagios.cmd'!



_____________
Tworzenie modułu selinux
Sprawdzenie logów selinux:
sealert -a /var/log/audit/audit.log

Sprawdzenie ostatnich logów selinux:
ausearch -m avc -ts recent


pwd:
/home/osboxes

Tworzmy plik NagiosRules.te i wklejamy treść:
nano NagiosRules.te

module nagios_rules 1.0;

require {
    type usr_t;
    type httpd_sys_script_t;
    type httpd_t;
    type nagios_t;
    type var_log_t;
    class fifo_file { getattr open write };
    class file { read write execute };
    class dir { read execute };
}

#============= httpd_sys_script_t ==============

# Allow access to fifo_file for httpd_sys_script_t
allow httpd_sys_script_t usr_t:fifo_file { getattr open write };

#============= httpd_t ==============

# Allow to execute script cmd.cgi by httpd_t
allow httpd_t var_log_t:file { read write execute };
allow httpd_t var_log_t:dir { read execute };

#============= nagios_t ==============

# Allow to execute script nagios.cmd by nagios_t
allow nagios_t var_log_t:file { read write execute };
allow nagios_t var_log_t:dir { read execute };



Następnie kompilujemy moduł do formatu .mod za pomocą komendy
checkmodule -M -m -o NagiosRules.mod NagiosRules.te


Tworzymy pakiet modułu .pp za pomocą komendy:
semodule_package -o NagiosRules.pp -m NagiosRules.mod

Ładujemy moduł do selinux za pomocą:
semodule -X 300 -i NagiosRules.pp

Sprawdzamy czy moduł został załadowany:
semodule -l | grep nagios_rules

audit2allow -M NagiosRule < /var/log/audit/audit.log

Nawet jeśli nie ma pliku .pp to poniższa komenda nie zwraca błędu!!!
semodule -i NagiosRule.pp




__________________________________________________________________________
Usuwanie konfliktów z pakietami:
Jeżeli wystąpi konflikt pomiędzy dwoma pakietami, to trzeba usunąć zainstalowany już pakiet i ponowić aktualizację paczek

Sprawdzenie które powodują konflikt
rpm -q tuned-ppd power-profiles-daemon

usuwanie problematycznych pakietów (tylko tych co były zainstalowane)
sudo dnf remove tuned-ppd -y
sudo dnf remove power-profiles-daemon -y

Jeśli dnf nie działa to usuwamy za pomocą rpm
sudo rpm -e --nodeps tuned-ppd
sudo rpm -e --nodeps power-profiles-daemon

Czyścimy cache 
sudo dnf clean all

Ponowny upgrade:
sudo dnf update -y