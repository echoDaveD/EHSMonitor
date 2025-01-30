# EHSMonitor

## About

This application uses the NASA protocol to monitor a Samsung EHS Mono HT Quiet unit and publish its values via MQTT. See `EHSController.swift` for published MQTT topics.

## Hrdware Setup

Connect the Samsung Indoor Unit or Wifi Kit F1/F2 Connectors to an RS485 Adapter.
F1 -> A+
F2 -> B-

more detailed description under [wiki](https://wiki.myehs.eu/wiki/F1/F2_connector)

## Run from Dockerfile
1. Clone this repository
2. Build Docker Image `docker build -t ehsmonitor EHSMonitor/.`
3. Create an Folder to store the Configuration file `mkdir EHSMonitor_dockervolume`
4. Copy the Sample Configuration file `cp EHSMonitor/Resources/ExampleConfiguration.json EHSMonitor_dockervolume/Configuration.json`
5. Edit the `Configuration.json` with your favorite editor
6. Run the Docker Container in dettached mode `docker run --restart=always --device=/dev/ttyUSB0 -v /root/EHSMonitor_dockervolume:/media/persistvol --name ehsmonitor -dt ehsmonitor`
   - `--restart=always` auto start the container on docker start/system restart
   - `--device=/dev/ttyUSB0` passthrough your USB Device to the container, if your USB rs485 Adapter is on another device, provide here yours
   - `-v /root/EHSMonitor_dockervolume:/media/persistvol` The docker container storage is not persistant, so you need to mount your `EHSMonitor_dockervolume` folder to your container under `media/persistvol` to provide you Configuration file.
   - `--name ehsmonitor` name your container instance, without it, docker will generate an generic one
   - `-dt ehsmonitor` -d = dettached mode (background)

### Additional Start Command

After the docker is running you can start the EHSMonitor with this command.
If you have used the Dockerfile, not need for this Command.

Start the EHSMonitor within your docker instance `docker exec -it ehsmonitor .build/debug/EHSMonitor --config /media/persistvol/Configuration.json > /dev/null 2>&1 &`
   - `-it ehsmonitor` your instance name, if you did not provide `--name ehsmonito` on the `docker run` command, type `docker ps` to get the generic name
   - ` > /dev/null 2>&1 &` pipe the output to null so it runs in background.

By default the build command will build the executable in a debug configuration. As this is an early development release, this is fine. The executable will be located at `./.build/debug/EHSMonitor` inside your local copy of the repository.

Run the executlabe via `./.build/debug/EHSMonitor --config $PathToConfigurationFile`

## Upgrade docker image

- Stop docker container `docker stop ehsmonitor`
- Build the new docker image (it is recommended to create a new docker image, so can always go back) `docker build -t ehsmonitor_new EHSMonitor/.`
- Run the new docker image `docker run --restart=always --device=/dev/ttyUSB0 -v /root/EHSMonitor_dockervolume:/media/persistvol --name ehsmonitor_new -dt ehsmonitor_new`

If Anything is Fine, you can delete the old container/image and repeat it without the _new suffix

- Remove docker container  `docker rm ehsmonitor` 

## Home Assistant Integration

To integrate the EHSMonitor Measurements into HomeAssistant, it is necessary to set up the MQQT client in HomeAssistant first.
After this just copy the contents [Resources/Homeassistant/mqtt.yaml](Resources/Homeassistant/mqtt.yaml) into your HomeAssitant Configuration file for mqtt.

After this, reference the mqtt entities files in your `configuration.yaml`
```
# mqtt konfiguration
mqtt: !include mqtt.yaml
```

After This restart Home Assistant and the Entities should be present.


## Possible Errors

- Swift is not installed
   - Install it via `apt install swift`
   
- EHSMonitor cannot resolve your MQTT Brokers DNS Address
   -  attach to the docker container `docker exec -it ehsmonitor sh`
   - Add your DNS/IP Mapping in `/etc/host` in your docker container

      sample: 

      `192.168.2.69 mqttbroker`
   - Add your DNS/IP Mapping in `/etc/host` in lxc container/VM/Host
      sample: 
      ```
      # --- BEGIN PVE ---
      192.168.2.69 mqttbroker localhost
      # --- END PVE ---
      ```

## Changelog

### v0.2.0 - 30.01.2025
- Added support for additional Samsung EHS enum
   - threeWayValve2 (getENUM_IN_3WAY_VALVE_2)
   - zone1PowerStatus (getENUM_IN_OPERATION_POWER_ZONE1)
   - zone2PowerStatus (getENUM_IN_OPERATION_POWER_ZONE2)
   - zone2Temperature (getVAR_IN_TEMP_ZONE2_F)
   - zone2TargetTemperatur (getVAR_IN_TEMP_TARGET_ZONE2_F)
   - zone2TargetFlowTemperature (getVAR_IN_TEMP_WATER_OUTLET_TARGET_ZONE2_F)
   - zone1FlowTemperature (getVAR_IN_TEMP_WATER_OUTLET_ZONE1_F)
   - zone2FlowTemperature (getVAR_IN_TEMP_WATER_OUTLET_ZONE2_F)
   - roomTemperature (getVAR_in_temp_room_f)
   - roomTargetTemperature (getVAR_in_temp_target_f)
- added Autostart config in Dockerfile
- added Homeassistent MQQT Entity definitions
 

