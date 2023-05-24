# LoraWan
Deploiement d'un réseau IoT lorawan Multitenant sur le technopole ElGhazala
# Project Description
Deployment of a Lorawan Multitenant IoT network on the ElGhazala technopole
This involves installing 3 gateways connected to the ChirpStack network server.
For a user to exploit the network, simply he should ask the admin to create a tenant on Chirpstack
and assign it as admin to that tenant. The admin adds shared gateways to all tenants.
Now the user can create applications and can add the end-devices associated to his application.
This project targets all students, PhD students and teachers of Sup'Com as well as professionals in partnership.
 

# Tutorial using _Chirpstack v3_
// project description

## Prerequisites
-   A single machine running a Linux based operating system. 
-   A single machine running a Windows based operating system for monitoring / administration.
-   Docker version 23.0.5 and Docker Compose version 2.17.3 is used in this tutorial. Detailed installation instructions can be found [here](https://docs.docker.com/install/).
-   Arduino IDE. Detailed installation instructions can be found [here](https://support.arduino.cc/hc/en-us/articles/360019833020-Download-and-install-Arduino-IDE).
-   Postman. Detailed installation instructions can be found [here](https://www.postman.com/downloads/).

-   Git

```bash
sudo apt-get install git
```

### _Chirpstack v3_ setup

-   To install _Chirpstack v3_ with docker, follow the next steps.

```bash
git clone https://github.com/chirpstack/chirpstack-docker.git
cd chirpstack-docker
git checkout v3
docker compose up
```

-   To check whether the installation was successful go to the server IP on port 8080.
In the following tutorial my server IP address is 192.168.33.69.

### Hardware

-   A seeeduino lorawana gateway kit. For further details on the kit visit this [wiki page](https://wiki.seeedstudio.com/LoRa_LoRaWan_Gateway_Kit/)
-   A DHT11 temperatur and humidity sensor.
<img src="http://www.senith.lk/media/2016/10/humidity%20and%20temperature%20dht11%20module.png" width="150">

## Architecture

The tutorial allows the deployment of the following system, comprising a basic FIWARE IoT stack:

![Architecture](https://github.com/Atos-Research-and-Innovation/IoTagent-LoRaWAN/blob/master/docs/img/stm32_ttn_tutorial/stm32_ttn_tutorial_architecture.png)

-   **A LoRaWAN end-node** based on an Seeeduino LoRaWAN development board. In our application, the device will readtemperatur and humidity values from the DHT11 sensor, encode the information using [CayenneLpp data model](https://developers.mydevices.com/cayenne/docs/lora/#lora-cayenne-low-power-payload) and forward the result to **Chirpstack** LoRaWAN stack through the _gateways_.
-   **The LoRaWAN gateway** plays the role of a concentrator which forwards the messages to the _LoRaWAN network server_, included in _Chirpstack v3_ stack.
-   **Chirpstack v3 stack** implements the functionalities of the _LoRaWAN network server_ and _LoRaWAN application server_. 
-   **FIWARE IoT Agent** enables the ingestion of data from _LoRaWAN application servers_ in _NGSI context brokers_, subscribing to appropriate communication channels (i.e., MQTT topics), decoding payloads and translating them to NGSI data model. It relies on a _MongoDB database_ to persist information.
-   **FIWARE Context Broker** manages large-scale context information abstracting the type of data source and the underlying communication technologies. It relies on a _MongoDB database_ to persist information.
-   **FIWARE Quantum Leap** subscribes to notifications of new data ingested by _FIWARE Context Brokers_ and stores the historical information in time-series format. It relies on a _CrateDB database_ to persist information.
-   **Grafana** provides an easy and intuitive mechanism to visualize and explore data by means of fashionable
    dashboards.

## Clone the GitHub repository

All the code and files needed to follow this tutorial are included in 
[FiwareAtSupCom\\LoraWan GitHub repository](https://github.com/FiwareAtSupCom/LoraWan).
To clone the repository:

```bash
git clone https://github.com/FiwareAtSupCom/LoraWan.git
```

## Gateway configuration

To configure the gateway for forwarding lorawan packets to the network server, you need to follow the next steps.
-   Hardware connection.
-   Open gateway terminal (over Serial or SSH). You can follow the instructions in the [Seeeduino getting started section](https://wiki.seeedstudio.com/LoRa_LoRaWan_Gateway_Kit/#getting-started).
-   Configure the gateway to forward the lorawan packets to chirpstack server:

```bash
nano /home/rxhf/risinghf/pktfwd/local_conf.json
```

Change the server_address to your chirpstack server IP address, change serv_port_up and server_port_down to 1700, and keep the gateway_ID, we will need it later.

```json
{
    "gateway_conf": {
        "gateway_ID": "b827ebfffe0e1174",
        "autoquit_threshold": 5,
        "server_address": "SERVER_ADDRESS",
        "serv_port_up": 1700,
        "serv_port_down": 1700
    }
}
```
-   To apply the changes reboot the gateway:

```bash
sudo reboot
```

## Adding the gateway to Chirpstack
-   Go to the web UI of chirpstack following this link: `http://192.168.33.69:8080` and login using "admin" as a username and a password.

-   Add a network server:

![image](https://raw.githubusercontent.com/FiwareAtSupCom/LoraWan/main/images/network_server_add.png)

-   Add a new service profile.

-   Add a new gateway:

![image](https://raw.githubusercontent.com/FiwareAtSupCom/LoraWan/main/images/gateway_1_add.png)

-   Verify that the gateway is active in the dashboard. 

![image](https://raw.githubusercontent.com/FiwareAtSupCom/LoraWan/main/images/gateway_1_active.png)


## Adding Application and device
-   Add a new device profile and make sure to check OTAA mode support and CayenneLPP as a codec:

![image](https://raw.githubusercontent.com/FiwareAtSupCom/LoraWan/main/images/device_profile_add.png)

-   Add an application. 

-   Add a device to the created application using the device profile. For that you will need to generate an Appkey (that you need to keep for the next step)and the DEV EUI which you can get following [Seeeduino AT communication section](https://wiki.seeedstudio.com/LoRa_LoRaWan_Gateway_Kit/#step-3-use-rhf2s001-integrated-lorawan-server).

![image](https://raw.githubusercontent.com/FiwareAtSupCom/LoraWan/main/images/device_add.png)

## End node setup
-   Open the end node device sketch located in `seeeduino_lorawana_end_device/end_node_cayenneLPP_payload_OTAA`.
-   Install the required libraries and seeeduino lorawan board. You can follow this [tutorial](https://wiki.seeedstudio.com/Seeeduino_LoRAWAN/). 
-   Change the appkey generated in the previous step.
-   Upload the code to the board and make the wiring as shown in the next figure.

<img src="https://raw.githubusercontent.com/FiwareAtSupCom/LoraWan/main/images/dht11-schema.png" width="350">

## Deploy FIWARE Stack

-   We will create the FIWARE stack containers using the following command.

```bash
docker compose -f docker/fiware/docker-compose.yml
```

-   For the containers administration and visualisation we will use Portainer web UI  installed using the previous cmd.(You will need to create an account.)

-   You can open the UI interface using this link: `https://192.168.33.69:9443` 

![image](https://raw.githubusercontent.com/FiwareAtSupCom/LoraWan/main/images/portainer_UI_containers.png)

-   Import the Postman workspace located in "postman/fiware1.postman_collection.json" which containes the needed HTTP requests.

### 1.  Check Orion version

-   GET request to:

```bash
http://192.168.33.69:1026/version
```

-   The output should be like this:

```json
{
    "orion": {
        "version": "3.8.1",
        "uptime": "0 d, 21 h, 19 m, 10 s",
        "git_hash": "30622caf67bb4384b73747abb17733e5ad9989a8",
        "compile_time": "Fri Feb 17 10:20:06 UTC 2023",
        "compiled_by": "root",
        "compiled_in": "02a18b9e9dcd",
        "release_date": "Fri Feb 17 10:20:06 UTC 2023",
        "machine": "x86_64",
        "doc": "https://fiware-orion.rtfd.io/en/3.8.1/",
        "libversions": {
            "boost": "1_74",
            "libcurl": "libcurl/7.74.0 OpenSSL/1.1.1n zlib/1.2.12 brotli/1.0.9 libidn2/2.3.0 libpsl/0.21.0 (+libidn2/2.3.0) libssh2/1.9.0 nghttp2/1.43.0 librtmp/2.3",
            "libmosquitto": "2.0.15",
            "libmicrohttpd": "0.9.73",
            "openssl": "1.1",
            "rapidjson": "1.1.0",
            "mongoc": "1.23.1",
            "bson": "1.23.1"
        }
    }
}
```

### 2.  Check IOT Agent about:

-   GET request to:

```bash
http://192.168.33.69:4041/iot/about
```
-   The output should be like this:

```json
{
    "libVersion": "2.21.0",
    "port": 4041,
    "baseRoot": "/",
    "version": "1.2.5"
}
```

### 3.  Add device to IOT Agent

-   POST request to:

```bash
http://192.168.33.69:4041/iot/devices
```
-   Add these headers:

```bash
fiware-service: smartroom
fiware-servicepath: /rooms
```

-   And this body(you should change the fields of: dev_eui, app_eui, application_id, application_key. You can get them from the chirpstack application.)


```json
{
  "devices": [
    {
      "device_id": "dht_001",
      "entity_name": "DHT_001",
      "entity_type": "DHT",
      "timezone": "Europe/Berlin",
      "attributes": [
        {
          "object_id": "rh1",
          "name": "relative_humidity_1",
          "type": "Number"
        },
        {
          "object_id": "t1",
          "name": "temperature_1",
          "type": "Number"
        }
      ],
      "internal_attributes": {
        "lorawan": {
          "application_server": {
            "host": "192.168.33.69",
            "username": "admin",
            "password": "admin",
            "provider": "chirpstack"
          },
          "dev_eui": "8cf957200004c487",
          "app_eui": "4569343567897875",
          "application_id": "1",
          "application_key": "75B7EBC63DA30B7C9E3BAE8DFFAE94B3",
          "data_model": "cayennelpp"
        }
      }
    }
  ]
}
```

-   The output should be like this with 201 status code


```json
{}
```


### 4.  Show the added device

-   GET request to:

```bash
http://192.168.33.69:4041/iot/devices
```

-   Add these headers:

```bash
fiware-service: smartroom
fiware-servicepath: /rooms
```

-   The output should be:

```json
{
    "count": 1,
    "devices": [
        {
            "device_id": "dht_001",
            "service": "smartroom",
            "service_path": "/rooms",
            "entity_name": "DHT_001",
            "entity_type": "DHT",
            "attributes": [
                {
                    "object_id": "rh1",
                    "name": "relative_humidity_1",
                    "type": "Number"
                },
                {
                    "object_id": "t1",
                    "name": "temperature_1",
                    "type": "Number"
                }
            ],
            "lazy": [],
            "commands": [],
            "static_attributes": [],
            "internal_attributes": {
                "lorawan": {
                    "application_server": {
                        "host": "192.168.33.69",
                        "username": "admin",
                        "password": "admin",
                        "provider": "chirpstack"
                    },
                    "dev_eui": "8cf957200004c487",
                    "app_eui": "4569343567897875",
                    "application_id": "1",
                    "application_key": "75B7EBC63DA30B7C9E3BAE8DFFAE94B3",
                    "data_model": "cayennelpp"
                }
            }
        }
    ]
}
```

### 5.  Show the DHT11 entity from Orion context broker

-   GET request to:

```bash
http://192.168.33.69:1026/v2/entities/DHT_001
```

-   Add these headers:

```bash
fiware-service: smartroom
fiware-servicepath: /rooms
```

-   The output should be:

```json
{
    "id": "DHT_001",
    "type": "DHT",
    "TimeInstant": {
        "type": "DateTime",
        "value": "2023-05-24T14:49:26.718Z",
        "metadata": {}
    },
    "relative_humidity_1": {
        "type": "Number",
        "value": 62,
        "metadata": {
            "TimeInstant": {
                "type": "DateTime",
                "value": "2023-05-24T14:49:26.718Z"
            }
        }
    },
    "temperature_1": {
        "type": "Number",
        "value": 26,
        "metadata": {
            "TimeInstant": {
                "type": "DateTime",
                "value": "2023-05-24T14:49:26.718Z"
            }
        }
    }
}
```

### 6.  Check Quantumleap version

-   GET request to:

```bash
http://192.168.33.69:8668/version
```

-   The output should be like this:

```json
{
    "version": "0.8.x"
}
```

### 7.  Add subscription to Orion context broker

-   POST request to:

```bash
http://192.168.33.69:1026/v2/subscriptions
```

-   Add these headers:

```bash
fiware-service: smartroom
fiware-servicepath: /rooms
```

-   And this body:

```json
{
  "description": "Notify QuantumLeap of all DHT Sensor changes",
  "subject": {
    "entities": [
      {
        "idPattern": "DHT*"
      }
    ],
    "condition": {
      "attrs": [
        "relative_humidity_1",
        "temperature_1"
      ]
    }
  },
  "notification": {
    "http": {
      "url": "http://quantumleap:8668/v2/notify"
    },
    "attrs": [
        "relative_humidity_1",
        "temperature_1"
    ],
    "metadata": ["dateCreated", "dateModified"]
  }
}
```

-   This should return an empty response body with a 201 status code 


### 8.  Show the created subscription

-   GET request to:

```bash
http://192.168.33.69:1026/v2/subscriptions
```

-   Add these headers:

```bash
fiware-service: smartroom
fiware-servicepath: /rooms
```

-   The output should be like this:

```json
[
    {
        "id": "646e24594282b5b0b902b9ce",
        "description": "Notify QuantumLeap of all DHT Sensor changes",
        "status": "active",
        "subject": {
            "entities": [
                {
                    "idPattern": "DHT*"
                }
            ],
            "condition": {
                "attrs": [
                    "relative_humidity_1",
                    "temperature_1"
                ]
            }
        },
        "notification": {
            "timesSent": 1,
            "lastNotification": "2023-05-24T14:51:06.000Z",
            "attrs": [
                "relative_humidity_1",
                "temperature_1"
            ],
            "onlyChangedAttrs": false,
            "attrsFormat": "normalized",
            "http": {
                "url": "http://quantumleap:8668/v2/notify"
            },
            "metadata": [
                "dateCreated",
                "dateModified"
            ],
            "lastSuccess": "2023-05-24T14:51:06.000Z",
            "lastSuccessCode": 200,
            "covered": false
        }
    }
]
```

### 9.  Show records history from quantumleap

-   GET request to:

```bash
http://192.168.33.69:8668/v2/entities/DHT_001
```

-   Add these headers:

```bash
fiware-service: smartroom
fiware-servicepath: /rooms
```

-   The output should be like this:

```json
{
    "attributes": [
        {
            "attrName": "TimeInstant",
            "values": [
                null,
                null,
                null,
                null,
                null
            ]
        },
        {
            "attrName": "relative_humidity_1",
            "values": [
                62.0,
                62.0,
                62.0,
                62.0,
                61.0
            ]
        },
        {
            "attrName": "temperature_1",
            "values": [
                26.0,
                26.0,
                26.0,
                27.0,
                26.0
            ]
        }
    ],
    "entityId": "DHT_001",
    "index": [
        "2023-05-24T14:50:16.840+00:00",
        "2023-05-24T14:50:26.865+00:00",
        "2023-05-24T14:50:36.888+00:00",
        "2023-05-24T14:50:46.908+00:00",
        "2023-05-24T14:51:06.962+00:00"
    ]
}
```
 
## Grafana visualisation

-   Navigate to `http://192.168.33.69:3000` and log in with default credentials which are: `admin / admin`

-   Add a new data source using the following parameters:

![image](https://raw.githubusercontent.com/FiwareAtSupCom/LoraWan/main/images/grafana_datasource_add.png)

-   Import the dashboard located in: `grafana_dashboard` folder.

![image](https://raw.githubusercontent.com/FiwareAtSupCom/LoraWan/main/images/grafana_dashboard_import.png)

-   The final result should look like this:

![image](https://raw.githubusercontent.com/FiwareAtSupCom/LoraWan/main/images/grafana_dashboard_visualisation.png)
