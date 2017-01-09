## Setup the Pi

1. ran `$ passwd` to change the default password
2. ran `$ sudo apt-get update`
3. run `sudo apt-get upgrade`
4.  ran `$sudo raspi-config` to launc the configuration editor
  1. exapnded the file system
  2. in advnced I enabled SSH
5. REBOOT

ran the all in one installer: https://home-assistant.io/getting-started/installation-raspberry-pi-all-in-one/

```bash
$ wget -Nnv https://raw.githubusercontent.com/home-assistant/fabric-home-assistant/master/hass_rpi_installer.sh && chown pi:pi hass_rpi_installer.sh && bash hass_rpi_installer.sh
```
## creating user and group
Because raspbian doesn't let you use SFTP with an account that has root access you will want to create an account that cant connect to your pi via SFTP do administer the files.

1. Added a new user `$ sudo useradd -M brian` the -M flag is used to create a user without a home directory. I am doing this since this user will just be used to administer the server.

2. execute the command to add a password to the user that you just created `$ sudo passwd brian`

3. Execute the command `$ sudo usermod -a -G homeassistant brian` to add the user that was just created to the homeassistant group.



## Setup Samba

1. run sudo apt-get install samba
2. run sudo nano /etc/samba/smb.conf​
3. replace the existing configuration file with the file below

```
[global]
netbios name = raspberrypi
server string = The Pi File Center
workgroup = WORKGROUP
hosts allow =
remote announce =
remote browse sync =

[HomeAssistant]
path = /home/homeassistant/.homeassistant
comment = No comment
browsable = yes
read only = no
valid users =
writable = yes
guest ok = yes
public = yes
create mask = 0777
directory mask = 0777
force user = root
force create mode = 0777
force directory mode = 0777
hosts allow =
```
4. type sudo smbpasswd -a pi
5. type a new password went prompted
6. restart the samba service by typing `$ sudo service smbd restart`



## Start HASS

``` bash
$ sudo su -s /bin/bash hass
$ source /srv/homeassistant/homeassistant_venv/bin/activate
$ hass
```

sudo -u hass -H /srv/homeassistant/homeassistant_venv/bin/activate

## useful information
if you need to stop and start the server from the command line

```
$ sudo service hass-daemon restart
```
if there are problems with the YAML configuration run
`$ hass --script check_config` to check the configuration

Sometimes the yaml config file will get set to the incorrect location when this
happnes you should run the command `$ hass --config /home/homeassistant/.homeassistant` to reset the
location
