# Setup the Pi

1. run `$ passwd` to change the default password
2. run `$ sudo apt-get update`
3. run `sudo apt-get upgrade`
4. run `$sudo raspi-config` to launc the configuration editor
  1. exapnd the file system
  2. in interfacing enable SSH
5. REBOOT

adjust the volume on the pi:'sudo alsamixer'

install setup tools `sudo apt-get install python-setuptools`
install pip`sudo easy_install pip`

# Setup MOPIDY
https://www.mopidy.com/

1. add the archives GPG key
`wget -q -O - https://apt.mopidy.com/mopidy.gpg | sudo apt-key add -`

2. Add the APT repo to your package sources:
`sudo wget -q -O /etc/apt/sources.list.d/mopidy.list https://apt.mopidy.com/jessie.list`

3. Install Mopidy and all dependencies:
```
sudo apt-get update
sudo apt-get install mopidy
```

## install the google play music extension
https://github.com/mopidy/mopidy-gmusic

`sudo pip install mopidy-gmusic`

you may need to update the pyasn1 package
`sudo pip install --upgrade pyasn1`


## Edit the Configuration

configuration file will be located in `/etc/mopidy/mopidy.conf`

Base config file will look like this:
```
[core]
cache_dir = /var/cache/mopidy
config_dir = /etc/mopidy
data_dir = /var/lib/mopidy

[logging]
config_file = /etc/mopidy/logging.conf
debug_file = /var/log/mopidy/mopidy-debug.log

[http]
enabled = true
hostname = ::
port = 6680
zeroconf =

[local]
media_dir = /var/lib/mopidy/media

[m3u]
playlists_dir = /var/lib/mopidy/playlists
```
You will need to add section for google play music
```
[gmusic]
username = alice@gmail.com
password = secret
deviceid = ################

#These match the defaults used by the extension
radio_stations_in_browse = true
radio_stations_as_playlists = true
# Leaving this unset make it unlimited
radio_stations_count =
# Limit the number or tracks for each radio station
radio_tracks_count = 50
```

## test the installation

``` bash
$ sudo service mopidy start
$ sudo service mopidy stop
```

## add the service to systemd

to  add the mopidy service to systemd you will need to execute the following command: `sudo systemctl status mopidy`

once the service has been added it will automatically start running when the pi is turned on. to manage the service you can use the commands below:
```bash
$ sudo systemctl start mopidy
$ sudo systemctl stop mopidy
$ sudo systemctl restart mopidy
$ sudo systemctl status mopidy
```

## Internet archives

execute the following command to install the backend extension: `sudo pip2 install Mopidy-Internetarchive`

Basic Configuration File
```
[internetarchive]
enabled = true

collections =
    audio
    oldtimeradio
    etree
    librivoxaudio
    audio_bookspoetry
    audio_tech
    audio_music
    audio_news
    audio_foreign
    audio_podcast
    audio_religion
```

## installed front end
https://github.com/martijnboland/moped
https://github.com/tkem/mopidy-mobile

`sudo pip2 install Mopidy-Moped`
`sudo pip2 install Mopidy-Mopify`
`sudo pip2 install Mopidy-Mobile`


test by going to http://localhost:6680/moped
http://localhost:6680/Mopify

## Useful commands

* Insect the services configuration: `sudo mopidyctl config`
* scan the music library: `sudo mopidyctl local scan`

# Setup Home Assistant

run the all in one installer: https://home-assistant.io/getting-started/installation-raspberry-pi-all-in-one/

```bash
$ wget -Nnv https://raw.githubusercontent.com/home-assistant/fabric-home-assistant/master/hass_rpi_installer.sh && chown pi:pi hass_rpi_installer.sh && bash hass_rpi_installer.sh
```
## creating user and group
Because raspbian doesn't let you use SFTP with an account that has root access you will want to create an account that cant connect to your pi via SFTP do administer the files.

1. Added a new user `$ sudo useradd -M brian` the -M flag is used to create a user without a home directory. I am doing this since this user will just be used to administer the server.

2. execute the command to add a password to the user that you just created `$ sudo passwd brian`

3. Execute the command `$ sudo usermod -a -G homeassistant brian` to add the user that was just created to the homeassistant group.

4. modify the permissions of the /home/homeassistant/.homeassistant folder to allow the group to write to it. `$ sudo chmod g=rwx  /home/homeassistant/.homeassistant`



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
To start HASS you will need to complete two operations before it can be started.

1. switch the user to the homeassistant user
2. activate the virtual environment

``` bash
$ sudo su -s /bin/bash homeassistant
$ source /srv/homeassistant/homeassistant_venv/bin/activate
$ hass
```

## Auto start
1. Create the service file
```bash
$ sudo nano /lib/systemd/system/home-assistant@pi.service
```
2. add the following text to the file

```
[Unit]
Description=Home Assistant
After=network.target

[Service]
Type=simple
User=homeassistant
ExecStartPre=source /srv/homeassistant/homeassistant_venv/bin
ExecStart=/srv/homeassistant/homeassistant_venv/bin/hass -c "/home/homeassistant/.homeassistant"

[Install]
WantedBy=multi-user.target
```
3. Run the following commands:
```
sudo systemctl --system daemon-reload
sudo systemctl enable home-assistant@pi
sudo systemctl start home-assistant@pi
```
4. REBOOT

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

#installing music server
