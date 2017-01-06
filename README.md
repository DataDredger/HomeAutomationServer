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
6. restart the samba service by typing sudo service smbd restart



## Start HASS

``` bash
$ source /srv/homeassistant/homeassistant_venv/bin/activate
$ HASS
```

## useful information
if you need to stop and start the server from the command line

```
$ sudo systemctl stop home-assistant@pi
$ sudo systemctl start home-assistant@pi
```
if there are proplems with the YAML configuration run
`$ hass --script check_config` to check the configuration
