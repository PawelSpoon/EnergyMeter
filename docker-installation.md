# content
this one describes the first installation as docker
it does not contain the final flow BUT it might contain helpfull parts for docker-setup 

# built following these links
https://www.youtube.com/watch?v=D1rHGaviECY

https://schroederdennis.de/tutorial-howto/shelly-tasmota-mqtt-node-red-influxdb-ins-grafana-dashboard-anleitung/

# ports
mosquitto: 1883
node-red: 1880
influx: 8086
grafana: 3000

# install docker

# install mosquitto
docker volume create mosquitto_data
docker volume create mosquitto_log
docker volume create mosquitto_conf
 
docker run -it -d \
    -p 1883:1883 \
    -p 9001:9001 \
    -v mosquitto_conf:/mosquitto/config \
    -v mosquitto_data:/mosquitto/data \
    -v mosquitto_log:/mosquitto/log \
    --restart=always \
    --name=mosquitto-server \
    eclipse-mosquitto:latest


# install node-red
docker volume create node_red_data

docker run -it -d \
    -p 1880:1880 \
    -v node_red_data:/data \
    --name=mynodered \
    --restart=always \
    nodered/node-red:latest

# install influx
rem: i do not want to have the data in a docker volume but in a mount ?

docker volume create influxdb
 
docker run -it -d \
    -p 8086:8086 \
    -v influxdb:/var/lib/influxdb \
    --name=influxdb \
    --restart=always \
    influxdb:1.8

# create db using influx cli
docker exec -it <ID> /bin/bash
 
influx 
create database <DATENBANKNAME>   
zb. tarija


# grafana
docker volume create grafana
 
docker run -it -d \
    -p 3000:3000 \
    --name=grafana \
    -v grafana:/var/lib/grafana \
    --name=grafana \
    --restart=always \
    grafana/grafana:latest

http://localhost:3000
login as admin/admin
change pwd to tarija



listener 1883

# use ip address from docker-container
go to bridge network in visual code, inspect it and check for

172.17.0.2

topic sample:
inverter/total/P_AC

install in nod-red/manage palleten/install tab:
node-red-contrib-influxdb

if (msg.topic.split("/")[4] !== undefined) {
    msg.key = msg.topic.split("/")[3] + "_" + msg.topic.split("/")[4]
    msg.payload = Number(msg.payload)
    return msg;
}





