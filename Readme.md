# LoudML-Grafana-Influx-Telegraf

As of 21.06.2022 : updated by Alberto, any queries email me `albertopopescu@outlook.com`

This tutorial is for InfluxDB version 1.x. 
LoudML supports InfluxDB version 1.x and 2.x. LoudML also supports Grafana version 8.x.
For your project you only need to get the docker-compose file and LoudMLData folder, and modify it to suit your requirements.


## Stage I ENVIRONMENT

- Windows 10 Enterprise version 21H2
- Docker version 20.10.13, build a224086
- Influx version 1.8.10
- Telegraf version 1.22.4
- Grafana version 7.5.8
- LoudML version 1.7.2


## Stage II WINDOWS-DOCKER SETUP

`WINDOWS`
1. Enable HYPER-V in windows features: https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v
2. Install WSL2: https://www.omgubuntu.co.uk/how-to-install-wsl2-on-windows-10

Visualise:
![image](https://user-images.githubusercontent.com/63293696/172639073-ed240188-bd36-4c24-a096-9ef1b94d0261.png)

`Docker`

Visualise:
![image](https://user-images.githubusercontent.com/63293696/172639961-ab026550-c581-46d8-9e9a-e60db502d3f6.png)

## Stage III DOCKER-COMPOSE.YML FILE
1. Create a file called 'docker-compose.yml' using below structure:

```yaml
# This is a Docker Compose file to work with Loud ML and an InfluxDB stack.
version: "3.6"
services:
    loudml:
    influxdb:
    telegraf:
    grafana:
volumes:
```
2. Service loudml:

```yaml
  loudml:
    image: loudml/loudml:addVersionExample1.6.0
    container_name: addName 
    restart: always 
    volumes: 
      - addYourPcPathToConfigFile/loudml/config.yml:/etc/loudml/config.yml:ro 
      - var_loudml
    environment: 
      influx: http://addParameterThatIsExactlyTheServiceNameOfInfluxDBFoundInDockerComposeFile:addDockerPortOfInfluxDBServiceFoundInDockerComposeFile
      influx_database: addNameForADatabaseToBeCreatedOnInfluxDB 
    ports:
      - "addPortToBeOpenedOnYourNetworkDefault8077:addPortToBeOpenedOnDockerNetworkDefault8077" 
    depends_on:
      - addNameOfTheInfluxDBServiceFoundInDockerComposeFile
```
3. Service influx:

```yaml
influx:
    image: influxdb:addVersionExample1.8.10 
    container_name: addName
    restart: always
    environment:
      - INFLUX_USERNAME=addUserName 
      - INFLUX_PASSWORD=addPassword
      - INFLUXDB_HTTP_SHARED_SECRET=addToken
    ports:
      - "addPortToBeOpenedOnYourNetworkDefaul8086:addPortToBeOpenedOnDockerNetworkDefault8086"
    volumes:
      - addYourPcPathToConfigFile/influxdb/config/influxdb.conf:/var/lib/influxdb/influxdb.conf 
      - addYourPcPathToDataFolder/influxdb/data:/var/lib/influxdb/data
      - var_influxdb
```
4. Service telegraf:

```yaml
  telegraf:
    image: telegraf:addVersionExample1.22.4
    container_name: addName
    restart: always
    volumes:
      - "addYourPcPathToConfigFile/telegraf/telegraf.conf:/etc/telegraf/telegraf.conf:ro" 
    depends_on:
      - influxdb
    links: 
      - "influxdb:addAlternativeNameToAccessInfluxdbBy"
```
5. Service grafana:

```yaml
 grafana:
    image: grafana/grafana-enterprise:addVersion
    container_name: addName
    restart: always
    ports:
      - addPortToBeOpenedOnYourNetworkDefaul3000:addPortToBeOpenedOnDockerNetworkDefault3000
    user: '104'
    environment:
    - GF_INSTALL_PLUGINS=addPluginUrlExample>>http://www.github.com/vsergeyev/loudml-grafana-app/blob/master/loudml-grafana-app-1.7.2.zip?raw=true;loudml-grafana-app
    volumes:
      - addYourPcPathToConfigFile/grafana/grafana.ini:/etc/grafana/grafana.ini:rw 
      - var_grafana
    depends_on:
      - influxdb
    links: 
      - "influxdb:addAlternativeNameToAccessInfluxdbBy"
```
6. Volumes:

```yaml
volumes:
  var_loudml:
    external: if-set-to-FALSE-docker-compose-will-create-the-volumes
  var_influxdb:
    external: if-set-to-TRUE-docker-compose-will-check-for-external-volumes-already-created
  var_grafana:
    external: false
```

Stage IV GRAFANA SETUP

1. Allow grafana to use unsigned plugins, to be able to use LoudML, by adding this line into grafana.ini, and then enable it.

```ini
[plugins]
allow_loading_unsigned_plugins = true
```
- Enable LoudML plugin: Grafana >> Configuration >> Plugins >> LoudML >> Config >> Click 'Enable' button. Reminder: ensure StageIV-Step1 is complete!
2. Setup Influx datasource

- Query Language : InfluxQL

- HTTP
  - URL : http://influxdb:8086 (http://`<nameOfInfluxServiceInDockerComposeFile>:<numberOfPortInfluxServiceOnDockerComposeFile>`)
  - Access: Server(Default)

- Custom HTTP Headers
  - Header: Authorization Value: Token addNameOfYourTokenFromInfluxDB (Leave a space between 'Token' and 'yourActualToken')

- InfluxDB Details
  - Database: _internal (Add any database/bucket you have created on Influx; '_internal' is the default database created by InfluxV1)
  - Username: admin (The username for Influx service defined in docker-compose file or influxdb.conf file)
  - Password: admin (The password for Influx service defined in docker-compose file or influxdb.conf file)
  - HTTP Method: GET
  
  Click 'Save and Test' >> If it works, a green label pops up: Data source is working. >>If it doesn't work, most likely you have some other authentication method checked or/and your url doesn't have the service name of influxdb specified in docker-compose as it's domain name.

3. Setup LoudML datasource
- HTTP
  - Loud ML Server URL : http://loudml:8077 (http://`<nameOfLoudMLServiceInDockerComposeFile>:<numberOfPortLoudMLServiceOnDockerComposeFile>`)
  - Access : Server (Default)
Click 'Save and Test'
4. Setup Model on LoudML
- Name : addAnyName
- Model type : donut
- Bucket : readYourData (this is the bucket name specified in the loudml.yml for the influx database from where you get your data to apply the ML model on : if you see the example provided here note that the first bucket on the buckets list is for the database from where you get the data to train you model, whereas, the second bucket on the lsit is where the trained data goes based on which your predictions are made)
- Max training hyper-params iterantions : 10 (number of variable parameters)
- GroupedBy bucket interval : 5s (setting the interval to 1s for the model - I believe - should be the same or close to the one set on the telegraf (5s in our case - in telegraf.conf) for querying the data to influxdb)
- Span : 100 (number of training iterations (100 times 5s over 60 = approx 8mins to train), adjust as you need but you should need more than 100 to train your model)

- Feature
  - Name : addAnyName
  - Measurement : cpu  (add the value you inserted in the 'select measurement' field in the dashboard)
  - Field : usage_user (add the value inserted in the 'field' field in the dashboard)
  - Metric : mean (collects the mean values)
  - Default : 0 (when there are missing value replace with 0, missing values can also occur this model reads data quicker (every 5s) than data is written by telegraf to influxdb (every 5s))

- Predictions 
  - Interval : 5s (intervall for predictions, you probably want your predictions to be of the same interval with the frequency with which telegraf writes data to influxdb (every 5s))
  - Offset : 5s (An offset is a per-row “bias value” that is used during model training)

- Anomalies
  - Min threshold : 0 (what goes below this value is considered anomaly, stored into the 'annotation_db: youAnnotationsEnterHere' and is excluded from your training data)
  - Max threshold : 0 (what goes above this value is considered anomaly, stored into the 'annotation_db: youAnnotationsEnterHere' and is excluded from your training data)

5. Dashboard setup InfluxDB and LoudML
- Query 
  - InfluxDB (Select the name set for Influx as datasource on Grafana)
    ![image](https://user-images.githubusercontent.com/63293696/172900728-1ce9fc62-4760-43a1-946f-19de5c2a1aff.png)

- Panel 
  - Visualization
    - Loud ML Graph
  - Display 
    - Loud ML Server : Loud ML Datasource (Select the name set for LoudML as datasource on Grafana)
    - Input Bucket : influxdb (add the name of the input bucket in loudml.yml)
    - Output Bucket : loudml  (add the name of the output bucket in loudml.yml)
    ![screencapture-localhost-3000-d-LwSVJ797z-new-dashboard-copy-2022-06-09-10_36_27](https://user-images.githubusercontent.com/63293696/172905276-41babeba-df12-47ca-a6e8-49248ef2bcc2.png)

6. LoudML CLI commands
- create model: 

Visualise:
- Datasource: Influx
    ![image](https://user-images.githubusercontent.com/63293696/172815599-334e69b7-0f33-4b0b-aecf-afb57944d74c.png)
- Datasource: LoudML
    ![image](https://user-images.githubusercontent.com/63293696/172817494-4d301deb-6181-496f-8e63-82dbac53bc0c.png)
- LoudML: Model
    ![image](https://user-images.githubusercontent.com/63293696/172817621-856f127a-9313-4bdc-94f3-cc2a42cc92f7.png)
- LoudML: Dashboard Setup
    ![screencapture-localhost-3000-d-LwSVJ797z-new-dashboard-copy-2022-06-09-10_36_27](https://user-images.githubusercontent.com/63293696/172818850-3d2d017a-8ed3-4354-8b76-cf188ae7c762.png)
- Influx: Databases
    ![image](https://user-images.githubusercontent.com/63293696/172905942-3b787747-18c6-4163-8d47-b93c3feef95c.png)

    


