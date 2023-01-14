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
- now setup wireless, again sudo raspi-config
 - system settings
 - wireless lan
 - type in ssid and password, store reboot done

now it is time to follow the link: (as some of the raspi parts were outdated..)

https://simonhearne.com/2020/pi-influx-grafana/




