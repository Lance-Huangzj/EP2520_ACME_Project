# EP2520_ACME_Project
## VM1
This VM is used as a VPN access server. And there is an intrusion detection for interface enp0s3.<br>
For IDS, you can just copy the etc folder in your VM and use the command:<br>

* sudo snort -A console -i eth0 -u snort -g snort -c /etc/snort/snort.conf

to start. You need to change eth0 to the interface you want to listen to. And you need to change “ipvar HOME_NET server_public_ip/32” in the /etc/snort/snort.conf file to the network addresses you are protecting. You can add more rules by editing the /etc/snort/rules/local.rules file. The log will show in the terminal. For more information, you can read the reference from the reference of Documentation
For VPN, you can configure your own one by following the guide from the reference of Documentation which shows each detail in how to use OpenVPN to set up a VPN access server and how to get an ovpn file for clients. But before that, you need to build a CA and produce a private key and self-signature certification for the server. And to make your configuration work, you need to set static routes and port forwarding.
 
## VM2

The Web server and the FreeRadius server are set up on VM2.
To set up the Web server, Apache2, MySQL, and PHP should be installed first. The file /etc/apache2/sites-available/fosslinuxowncloud.com.conf is the new virtual host file, which needs to be enabled to replace the default file:
sudo a2dissite 000-default.conf
sudo a2ensite fosslinuxowncloud.com.conf
sudo systemctl restart apache2
As for the MySQL setting, you should create a new database and a new user.
create database your_database_name
create user 'your_user_name'@'localhost' identified BY 'QB35JaFV6A9=BJRiT90'
grant all privileges on your_database_name.* to your_user_name@localhost
Having configured Apache2 and MySQL correctly, open the browser and visit the IP address of localhost to register an admin account.
For more details, please refer to https://www.fosslinux.com/8797/how-to-install-and-configure-owncloud-on-ubuntu-18-04-lts.htm 

The main config files of the FreeRadius server are already contained. We mainly modify or create three files:
/etc/freeradius/3.0/mods-available/eap
/etc/freeradius/3.0/sites-enabled/mynetwork
/etc/freeradius/3.0/clients.cnf
Remember to configure the router which needs to be the FreeRadius client, modifying the encryption method to “WPA(2)-EAP”.
We also have created our own certificates files in

/etc/freeradius/3.0/certs/

To generate new user certificates, just modify the client.cnf in this directory according to the actual situation of users.

[ req ]
…...
default_bits		= 4096
distinguished_name	= client
input_password		= user_password
output_password		= user_password
…...

[client]
countryName		= ...
stateOrProvinceName	= ...
localityName		= ...
organizationName	= ...
emailAddress		= ...
commonName		= ...

After modifying the config, run the following command to generate client certificates. Distribute the second certificate to Android mobile phone users.
make client.pem
make client_android.p12
To start the FreeRadius server, run:
freeradius -X
If everything works correctly, “Ready to process requests” will be shown. Then connect Wifi using the Android mobile phone and attaching the user certificate, and the connection can be created.
For more details, please refer to
https://www.ossramblings.com/RADIUS-3.X-Server-on-Ubuntu-14.04-for-WIFI-Auth 

VM3

VM3 will be used to serve as a VPN client and a gateway. To run this client, we have already created this client file named LondonClient.conf. We have also enabled IP forwarding by modifying the /etc/sysctl.conf file. You can simply copy the etc folder and replace it in your VM. Then reboot the VM. 
For configuration details, please refer to:
https://openvpn.net/vpn-server-resources/site-to-site-routing-explained-in-detail/ 
