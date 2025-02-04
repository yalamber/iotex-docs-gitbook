# Deploy the backend service

You can configure your own backend service to receive the data from your Pebbles. The backend service will allow you to archive, visualize, and even send the data to a IoTeX Blockchain Smart Contract.

## Hardware Requirements

To run the backend services, you will need a machine with least 2CPU and 4GB memory with Debian or Ubuntu Linux. You can use an AWS or GCP virtual server, or if you want to use your local PC/laptop, make sure it has a public IP \(e.g., using ngrok\), where pebble can send data to.

### Install Docker CE <a id="install-docker-ce"></a>

You can install Docker CE by following the official documentation at [https://docs.docker.com/engine/installation/\(opens new window\)](https://docs.docker.com/engine/installation/)

Optionally, you can your Linux user to the `Docker` Group: that will allow you to use the `docker` command without `sudo`:

```text
sudo groupadd docker; sudo usermod -aG docker $USER
```

To make sure it has been applied you need to log out and login again then type:

```text
cat /etc/group | grep docker
```

should return something like `docker:x:999:ubuntu`.

### Install Docker Compose <a id="install-docker-compose"></a>

You can install Docker Compose by following the official documentation at [https://docs.docker.com/compose/install/\(opens new window\)](https://docs.docker.com/compose/install/)

### Install the SDK for MQTT <a id="install-the-sdk-for-mqtt"></a>

```text
sudo apt-get update
sudo apt install python3-pip
pip3 install AWSIoTPythonSDK

```

### Configure git <a id="configure-git"></a>

Make sure your `git`, installation is correctly configured \(e.g., with the correct SSH key if required\), so that the `git clone` command will work properly.

## Install the backend

### Run Pebble Backend Configuration

To start the backend configuration run:

```text
cd ~/pebble-backend-master
./setup-dev.sh

```

this will download all docker images and code repos first, and may take a while before completion.

### Start the backend service

To start the backend service run:

```text
./start-dev.sh
```

Make sure all the docker images are up and running by running

```text
docker ps
```

and you should see something like below:

```text
CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS              PORTS
                                  NAMES
116b6aa864c8        minio/minio:latest               "/usr/bin/docker-ent…"   14 minutes ago      Up 14 minutes       0.0.0.0:9000->9000/tcp
                                  docker-compose_minio_1
a651095d850d        iotex-hmq:local                  "/hmq -c /config/con…"   14 minutes ago      Up 14 minutes       0.0.0.0:1884->1883/tcp
                                  docker-compose_hmq_1
a67d65f71c76        thingsboard/tb-gateway:latest    "/bin/sh ./start-gat…"   14 minutes ago      Up 14 minutes
                                  docker-compose_thingsboard-gateway_1
9ce61f993ca5        iotex-blockchain-data:local      "pebble-data-contain…"   14 minutes ago      Up 14 minutes
                                  docker-compose_api-server_1
cbb292f45664        thingsboard/tb-postgres:latest   "start-tb.sh"            14 minutes ago      Up 14 minutes       0.0.0.0:1883->1883/tcp, 0.0.0.0:5683->5
683/udp, 0.0.0.0:8080->9090/tcp   docker-compose_thingsboard_1

```

Make sure port `1884` is exposed for MQTT and port `8080` is exposed for Thingsboard, from any other machine type \(replace `1.2.3.4`with the actual IP address of your backend server\):

```text
telnet 1.2.3.4 1884
telnet 1.2.3.4 8080
```

### Configure thingsboard

Login in Thingsboard via [http://1.2.3.4:8080/](https://docs.iotex.io/developer/hardware/pebble-backend-configuration.html) with the default username: `tenant@thingsboard.org` and password `tenant` \(feel free to change your password after login\).

In Thingsboard, create a Gateway device:

![](http://docs-old.iotex.io/img/developer/pebble-backend/create-gateway-2.jpg)

![](http://docs-old.iotex.io/img/developer/pebble-backend/create-gateway-3.jpg)

Copy the `Access Token` of the gateway device and paste it in the configuration file of the gateway:

![](http://docs-old.iotex.io/img/developer/pebble-backend/create-gateway-4.jpg)

```bash
sudo nano ~/pebble-var/conf/tb-gateway/conf/tb_gateway.yaml
```

Replace `xxxxxxxxxxxxx` with the newly copied token

```yaml
thingsboard:
  host: thingsboard
  port: 1883
  security:
    accessToken: xxxxxxxxxxxxx
storage:
  type: memory
  read_records_count: 100
  max_records_count: 100000

connectors:
  - name: MQTT Broker Connector
    type: mqtt
    configuration: mqtt.json

```

Restart the gateway service with:

```bash
docker restart docker-compose_thingsboard-gateway_1
```

Congrats! You just setup the pebble backend, which is ready to receive data from Pebbles!

### Visualize the Data on Thingsboard

#### Receive Pebble MQTT data <a id="receive-pebble-mqtt-data"></a>

To visualize data from your Pebble, you need to change the Pebble firmware configuration to have it send messages to your backend service public endpoint instead of the default endpoint `trypebble.io:8080`, to do so you must change the `Asset Tracker > IoTeX Hosted MQTT broker hostname` firmware setting accordingly: see the ["Configure the Firmware" ](../../develop-and-build-the-firmware/configure-the-firmware.md)doc to learn how to configure the Pebble firmware.

#### Inject Mock Data <a id="inject-mock-data"></a>

Alternatively, if you want to visualize some data when you do not have a Pebble device available, you can use the `mock` tool to generate and send some test data compliant with the Pebble data specs. Run the following on the Pebble Backend server:

```bash
cd pebble-backend-master/scripts
./mock-dev.sh
```

This script continuously produce data points according to pebble data specification, and inject it into the local `1884` port. If you run into an issue, e.g., not see data flow in, you can debug by running  directly: 

```bash
python3 ./run-dev.py -e localhost -p 1884 -id device/pebble-1/data -pf ../data/sample.txt 
```

#### Visualize the data <a id="visualize-the-data"></a>

Import the predefined dashboard:

![](http://docs-old.iotex.io/img/developer/pebble-backend/import-dashboard-1.jpg)

drop example/dashboard/pebble\_1.json to the box

![](http://docs-old.iotex.io/img/developer/pebble-backend/import-dashboard-2.jpg)

![](http://docs-old.iotex.io/img/developer/pebble-backend/import-dashboard-3.jpg)

Click the Modify Icon at the bottom right of the screen:

![](http://docs-old.iotex.io/img/developer/pebble-backend/import-dashboard-4.jpg)

Select the device `pebble-1`from the dropdown:

![](http://docs-old.iotex.io/img/developer/pebble-backend/import-dashboard-5.jpg)

