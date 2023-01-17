# EnergyMeter
config files for ahoy-dtu, shelley-3em, grafana dashboards

# Raspberry
my model: raspberry pie 3b

## Prepare raspi
- get an sd-card, i use 16 gb but bigger will not hurt
- download raspbian lite from: https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2022-09-26/2022-09-22-raspios-bullseye-arm64-lite.img.xz

- download balena-etcher from here: https://www.balena.io/etcher/
- make .appImage executable, e.g. via properties in 'files'
- burn image
- remount
- cd into boot (in root did not work, it worked in boot partition)
- create empty ssh file e.g. via cmd line : sudo touch ssh
- since 2021 you need to create the user yourself if you do not use rasp-installer:https://www.raspberrypi.com/news/raspberry-pi-bullseye-update-april-2022/
- do again in boot partition: touch userconf
- open file and store in first line: user:password
  where password was encoded using: echo 'mypassword' | openssl passwd -6 -stdi
- insert sd-card into raspi, connect raspi with ether net and power it up
you should see it now in your home network as raspberrypi
- get the ip adress and connect to it using ssh:
  ssh user@<ip>
  pwd is: mypassword
- you see the warning regarding localization: sudo raspi-config
 - L4
 - Wlan country
 - select yours and store
- on next ssh connection the issue is gone
- now setup wireless as per https://raspberrytips.com/raspberry-pi-wifi-setup/
 - again sudo raspi-config
 - system settings
 - wireless lan
 - type in ssid and password, store reboot done

now it is time to follow the link: (as some of the raspi parts were outdated..)
https://simonhearne.com/2020/pi-influx-grafana/

sudo apt update
sudo apt upgrade -y

# Influx

wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/os-release
echo "deb https://repos.influxdata.com/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

sudo apt update && sudo apt install -y influxdb

sudo systemctl unmask influxdb.service
sudo systemctl start influxdb
sudo systemctl enable influxdb.service

run influx client:
influx 

and there then:

create database home
use home

CREATE RETENTION POLICY "two_years" ON "home" DURATION 104w REPLICATION 1

create user grafana with password '<passwordhere>' with all privileges
grant all privileges on home to grafana

show users
 should return something like this:

user admin
---- -----
grafana true

# Grafana

wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt update && sudo apt install -y grafana

sudo systemctl unmask grafana-server.service
sudo systemctl start grafana-server
sudo systemctl enable grafana-server.service

# Mosquitto
https://randomnerdtutorials.com/how-to-install-mosquitto-broker-on-raspberry-pi/

sudo apt install -y mosquitto mosquitto-clients

sudo systemctl enable mosquitto.service

test
mosquitto -v

sudo nano /etc/mosquitto/mosquitto.conf

and add this to end of file for anonymous access:
listener 1883
allow_anonymous true

for user/pwd:
sudo mosquitto_passwd -c /etc/mosquitto/passwd <user>
then type-in the password

sudo nano /etc/mosquitto/mosquitto.conf
at the top of file !
per_listener_settings true

and at the bottom:
allow_anonymous false 
listener 1883  
password_file /etc/mosquitto/passwd

then restart
sudo systemctl restart mosquitto
check status
sudo systemctl status mosquitto

message looked good
then i did try to connect with mqtt-explorer and did succeed

# Node-red
https://www.raspberry-pi-geek.de/ausgaben/rpg/2019/02/node-red-auf-dem-raspberry-pi/
https://nodered.org/docs/getting-started/raspberrypi
download and execute script:

first install the build essentials 1

  bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)

and i did immediately configure it with user, pwd and all

autostart on boot:
sudo systemctl enable nodered.service

create database <DATENBANKNAME>   
zb. tarija

show databases
use tarija
show measurements


