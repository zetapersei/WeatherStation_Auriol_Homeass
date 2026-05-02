# ESPHome configuration

- copy this file [WeatherStationSensor.h](WeatherStationSensor.h) to yourESPHome project folder.
- copy & paste the below example into ESPHome. Change it by your configuration.

The application need to know wind_station_id / rain_station_id. We can switch it to scan mode (set wind_station_id=0 and
rain_station_id=0). After that we can check logs and when we find sensor records with real values, take sensor ID and
save it to wind_station_id / rain_station_id.

```yaml
substitutions:
  device_name: weatherstation
  comment: "Meteostation"
  ip: "192.168.178.x"
  station_pin: "14"   
  wind_station_id: "0"
  rain_station_id: "0"


api:
  encryption:
    key: "KEY_ESPHOME_INTEGRATION"

web_server:
  port: 80
  auth:
    username: your_user
    password: "your_password"

ota:
  - platform: esphome
    password: "your_password"

esp8266:
  board: esp12e

esphome:
  name: '${device_name}'
  comment: '${comment}'
  libraries:
    - "WeatherStationDataRx=https://github.com/Zwer2k/WeatherStationDataRx.git"
  includes:
    - WeatherStationSensor.h
  
  on_boot:
    priority: -10
    then:
      - lambda: |-
          auto ws = new WeatherSensor(${station_pin}, ${wind_station_id}, ${rain_station_id});
          App.register_component(ws);
          id(global_ws) = (void*)ws;

wifi:
  ssid: "your_ap_ssid"
  password: "your_AP_password"
  manual_ip:
    static_ip: "${ip}"
    gateway: 192.168.178.1
    subnet: 255.255.255.0



globals:
  - id: global_ws
    type: 'void*'
    restore_value: no
    initial_value: 'nullptr'

sensor:
  - platform: template
    name: "${device_name} Temperature"
    unit_of_measurement: "°C"
    device_class: TEMPERATURE
    state_class: measurement
    lambda: |-
      if (id(global_ws) == nullptr) return {};
      auto ws = (WeatherSensor*)id(global_ws);
      return ws->temperature->state;

  - platform: template
    name: "${device_name} Humidity"
    unit_of_measurement: "%"
    device_class: HUMIDITY
    state_class: measurement
    lambda: |-
      if (id(global_ws) == nullptr) return {};
      auto ws = (WeatherSensor*)id(global_ws);
      return ws->humidity->state;

  - platform: template
    name: "${device_name} Wind Speed"
    unit_of_measurement: "m/s"
    device_class: WIND_SPEED
    state_class: measurement
    lambda: |-
      if (id(global_ws) == nullptr) return {};
      auto ws = (WeatherSensor*)id(global_ws);
      return ws->wind_speed->state;

  - platform: template
    name: "${device_name} Rain Volume"
    unit_of_measurement: "mm"
    device_class: PRECIPITATION
    state_class: total_increasing
    lambda: |-
      if (id(global_ws) == nullptr) return {};
      auto ws = (WeatherSensor*)id(global_ws);
      return ws->rain_volume->state;

  - platform: template
    name: "Direzione Vento"
    unit_of_measurement: "°"
    icon: "mdi:compass"
    lambda: |-
        if (id(global_ws) == nullptr) return {};
        return ((WeatherSensor*)id(global_ws))->wind_direction->state;

  - platform: template
    name: "Batteria Sensore Esterno"
    unit_of_measurement: "%"
    device_class: BATTERY
    entity_category: diagnostic
    lambda: |-
        if (id(global_ws) == nullptr) return {};
        return ((WeatherSensor*)id(global_ws))->battery_wind_station->state;

  - platform: template
    name: "Raffica di Vento"
    id: wind_gust_sensor
    unit_of_measurement: "m/s"
    device_class: WIND_SPEED
    state_class: measurement
    icon: "mdi:weather-windy-variant"
    lambda: |-
        if (id(global_ws) == nullptr) return {};
        return ((WeatherSensor*)id(global_ws))->wind_gust->state;
      

logger:
  level: DEBUG

```
