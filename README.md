# Takserver
My method i used to Install ATAK Server

*****Used CentOS-7-x86_64-DVD-2009.iso  setting minimal for install. called server "takserver" *****
*****Installed on Esxi 6.7 VM using 4 cores and 4 GB ram and 50GB HDD*****
*****Install done as root*****

*****Use Exact Server Name when creating the certificates*****

yum check-update 
yum update


*****Optional Create Snapshot of VM*****


echo -e "* soft nofile 32768\n* hard nofile 32768" | sudo tee --append /etc/security/limits.conf > /dev/null

sudo yum install epel-release -y

sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm -y

sudo yum update -y


*****Copy takserver-4.6-RELEASE26.noarch.rpm to server*****


sudo yum install takserver-4.6-RELEASE26.noarch.rpm -y

java -version 

*****SHould output ver 11 plus*****

sudo /opt/tak/db-utils/takserver-setup-db.sh

sudo systemctl daemon-reload

sudo systemctl start takserver

sudo systemctl enable takserver


sudo firewall-cmd --zone=public --add-port 8089/tcp --permanent
sudo firewall-cmd --zone=public --add-port 8443/tcp --permanent
sudo firewall-cmd --zone=public --add-port 8444/tcp --permanent
sudo firewall-cmd --zone=public --add-port 8446/tcp --permanent
sudo firewall-cmd --reload


sudo su tak
vi /opt/tak/certs/cert-metadata.sh

*****Press i to enable incert mode, edit as below or to your own settings, press ESC, then type :x! then return to save*****


COUNTRY=GB
STATE=LONDON
CITY=LONDON
ORGANIZATION=HOME
ORGANIZATIONAL_UNIT=ATAK


cd /opt/tak/certs/
./makeRootCa.sh                      *****UPPERCASE NAME i used TAKSERVER*****
./makeCert.sh ca intermediate-CA     *****Used to create int CA for cert auto deployments, select y to move files. not needed for a base install*****
./makeCert.sh server takserver
./makeCert.sh client admin
./makeCert.sh client user

exit

sudo systemctl restart takserver

*****IMPORTANT Wait 2 minutes for the server to start up*****

sudo su tak

java -jar /opt/tak/utils/UserManager.jar certmod -A /opt/tak/certs/files/admin.pem

exit


*****Import admin cert into browser*****
*****Oppen browser https://SERVER_IP:8443/setup*****
*****Run through Setup process*****

Select Yes to secure setup
Click on Configure Secure Input
Next
Security Configuration should be Green
Next
Skip on LDAP
No on federation
Click on TAK Home
Click ok to Warning


*****You should now be logged on with warning about port 8080 being open, if not needed remove the line below from coreconfig file*****

<connector port="8080" tls="false" _name="http_plaintext"/>

sudo systemctl restart takserver

*****Wait 2 minutes for the server to start up*****


*****Edit CoreCinfig.xml  Use tak account reference Appendix B in manual*****
*****Add in line below in the <network> section under <input _name="stdssl" protocol="tls" port="8089"/>*****

<input _name="tlsx509" protocol="tls" port="8089" auth="x509"/>

sudo systemctl restart takserver

*****Wait 2 minutes for the server to start up*****

*****Log on to server https://SERVER_IP:8443*****


************************************************
************************************************
******* OTHER COMMAND THAT MIGHT BE HANDY*******
************************************************
sudo firewall-cmd --list-ports       ***CHECK PORTS OPEN***

sudo firewall-cmd --zone=public --remove-port=8080/tcp --permanent        ***REMOVE A PORT IF OPEN***
sudo firewall-cmd --reload

