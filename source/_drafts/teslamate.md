---
title: TeslaMate 配置
date: 2024-05-29
---
# TeslaMate 是什么

TeslaMate 是一款开源软件，通过调用 Tesla 的OpenAPI 数据，用来跟踪和记录 Tesla 车辆的数据。比如可以记录车辆的行驶轨迹、充电历史等数据。

这些数据其中有一部分可以通过 Tesla 的 App 获取，但还有一部分数据在 App 上是不具备的，比如车辆的形式轨迹数据。App 上可以显示出当前的车辆位置，但却无法显示出车辆的历史轨迹。而 TeslaMate 由于已经将 Tesla 车辆的历史数据存放到本地，因此就可以完整的展示车辆的历史轨迹。

# TeslaMate 如何配置
为了部署 TeslaMate，你首先需要有一台可以长期运行的服务器，用来持续抓取 Tesla OpenAPI 的数据。
该服务器的配置要求非常低，但最好为 Linux 系统，且可以部署 docker。
## 获取到 Tesla token
要想访问 Tesla OpenAPI 的数据，必须要先有 Tesla 的 token。我这里直接使用了苹果手机上的 App **Auth for Tesla** ，其他平台需要下载其他的 App。
![image.png|511](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240529225136.png)

在 App 中登录 Tesla 账号后会显示 Token 信息。需要注意的是，该 APP 中包含了 Refresh Token 和 Access Token 两个 Token，需要使用长期有效的 Refresh Token。
## 启动 TeslaMate 服务
TeslaMate 服务启动方式包含了 docker 和进程的方式启动，我这里使用了官方推荐的 docker 启动方式，该方式比进程方式要简单太多了。

在安装完 docker 后，在服务器上创建 docker-compose.yml 文件，内容如下：
```
version: "3"

services:
  teslamate:
    image: teslamate/teslamate:latest
    restart: always
    environment:
      - ENCRYPTION_KEY=$token  # $token 为上一步获取到的 token
      - DATABASE_USER=teslamate
      - DATABASE_PASS=teslamate
      - DATABASE_NAME=teslamate
      - DATABASE_HOST=database
      - MQTT_HOST=mosquitto
    ports:
      - 4000:4000
    volumes:
      - ./import:/opt/app/import
    cap_drop:
      - all

  database:
    image: postgres:15
    restart: always
    environment:
      - POSTGRES_USER=teslamate
      - POSTGRES_PASSWORD=teslamate
      - POSTGRES_DB=teslamate
    volumes:
      - teslamate-db:/var/lib/postgresql/data

  grafana:
    image: teslamate/grafana:latest
    restart: always
    environment:
      - DATABASE_USER=teslamate
      - DATABASE_PASS=teslamate
      - DATABASE_NAME=teslamate
      - DATABASE_HOST=database
    ports:
      - 3000:3000
    volumes:
      - teslamate-grafana-data:/var/lib/grafana

  mosquitto:
    image: eclipse-mosquitto:2
    restart: always
    command: mosquitto -c /mosquitto-no-auth.conf
    # 这里根据实际情况，可以将 MQTT 的端口号暴露出去
    ports:
      - 1883:1883
    volumes:
      - mosquitto-conf:/mosquitto/config
      - mosquitto-data:/mosquitto/data

volumes:
  teslamate-db:
  teslamate-grafana-data:
  mosquitto-conf:
  mosquitto-data:
```

在上述 yaml 配置中，需要注意的是用户名和密码信息需要调整一下，我这里统一使用了 teslamate。

执行 `docker compose up -d` 即可启动 TeslaMate 服务，一共启动了四个容器：
1. teslamate：TeslaMate 的主容器。
2. mosquitto：MQTT 协议的服务端，teslamate 容器中的进程会将数据发布到 MQTT 消息队列上。
3. database：关系型数据库 postgres，用来存储 teslamate 抓取的数据，并最终供 grafana 展示消费。
4. grafana：从 postgres 中读取数据，用来展示 teslamate 的数据。

## 访问 TeslaMate 界面
TeslaMate 提供了一个非常简单的界面，可以查看车辆的基本信息以及对 teslamate 程序进行简单设置。可以通过 `$server:4000`进行访问，其中 `$server` 为服务器所使用的 ip 地址。

> [!NOTE] 注意
> 该界面必须要登录，否则 teslamate 无法抓取到数据

在登录后显示界面如下：
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240529231233.png)
## 访问 grafana
接下来可以通过 grafana 来查看 TeslaMate 的详细数据，访问 `$server:3000`，默认的登录用户名和密码均为 admin。

> [!NOTE] 注意
> 刚开始登录 grafana 的时候可能无法显示数据，这是因为 teslamate 还没有从 Tesla 中获取到有效的数据。很多的数据，需要在行车的过程中产生。


在 grafana 中有很多的 dashboard，但这些 dashboard 都是英文展示的，用户体验上也不是特别友好。要想真正使用可以自己修改 dashboard 来实现。

![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240529231643.png)
# 与 Home Assistant 集成
如果你恰巧有 Home Assistant，TeslaMate 可以与 Home Assistant 集成。

> [!NOTE] 注意
> [TeslaMate 的官方文档](https://docs.teslamate.org/docs/integrations/home_assistant#ui-lovelaceyaml)存在错误，并不能按照文档直接配置成功。
## MQTT 对外暴露
TeslaMate 通过 MQTT 协议对 HA 暴露数据，因此需要 MQTT 的端口号先要对外暴露，需要检查下 MQTT 的配置。
```
  mosquitto:
    image: eclipse-mosquitto:2
    restart: always
    command: mosquitto -c /mosquitto-no-auth.conf
    # 将 MQTT 的端口号暴露出去
    ports:
      - 1883:1883
    volumes:
      - mosquitto-conf:/mosquitto/config
      - mosquitto-data:/mosquitto/data
```
如果好奇要查看 MQTT 中的 topic 列表，可以使用命令 `mosquitto_sub -v -t '#'`，其中 `mosquitto_sub` 命令在 ubuntu 下的安装方式为 `apt install mosquitto-clients`。

也可以使用 Mac 下的工具 MQTTX 来订阅 MQTT 中的数据，在订阅的地方指定 Topic 为 `teslamate/cars/1/#` 接口消费所有 TeslaMate topic 的数据。
![image.png|416](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240529233857.png)

## HA 配置文件
### configuration.yaml 
增加如下内容：
```yaml
# teslamate 使用
proximity:
  home_tesla:
    zone: home
    devices:
      - device_tracker.tesla_location
    tolerance: 10
    unit_of_measurement: km

mqtt: !include mqtt_sensors.yaml
sensor: !include sensor.yaml
binary_sensor: !include binary_sensor.yaml
```
需要注意的是不要增加官方文档中的如下片段，否则加载配置时会报错：
```yaml
tesla:  
username: !secret tesla_username  
password: !secret tesla_password  
scan_interval: 3600
```
### mqtt_sensors.yaml
用来配置 MQTT 的 topic 中数据和 HA 中的设备和实体之间的对应关系，需要修改下数组中第一个元素中的 name 和 model 信息，跟你的车型对应起来。
```yaml
- sensor:
    name: Display Name
    object_id: tesla_display_name # entity_id
    unique_id: teslamate_1_display_name # internal id, used for device grouping
    availability: &teslamate_availability
      - topic: teslamate/cars/1/healthy
        payload_available: 'true'
        payload_not_available: 'false'
    device: &teslamate_device_info
      identifiers: [teslamate_car_1]
      configuration_url: https://teslamate.zxxz.io/
      manufacturer: Tesla
      model: Model Y
      name: Tesla Model Y
    state_topic: "teslamate/cars/1/display_name"
    icon: mdi:car

- device_tracker:
    name: Location
    object_id: tesla_location
    unique_id: teslamate_1_location
    availability: *teslamate_availability
    device: *teslamate_device_info
    json_attributes_topic: "teslamate/cars/1/location"
    icon: mdi:crosshairs-gps
    
- device_tracker:
    name: Active route location
    object_id: tesla_active_route_location
    unique_id: teslamate_1_active_route_location
    availability: &teslamate_active_route_availability
      - topic: "teslamate/cars/1/active_route"
        value_template: "{{ 'offline' if value_json.error else 'online' }}"
    device: *teslamate_device_info
    json_attributes_topic: "teslamate/cars/1/active_route"
    json_attributes_template: "{{ value_json.location | tojson }}"
    icon: mdi:crosshairs-gps

- sensor:
    name: State
    object_id: tesla_state
    unique_id: teslamate_1_state
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/state"
    icon: mdi:car-connected

- sensor:
    name: Since
    object_id: tesla_since
    unique_id: teslamate_1_since
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/since"
    device_class: timestamp
    icon: mdi:clock-outline

- sensor:
    name: Version
    object_id: tesla_version
    unique_id: teslamate_1_version
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/version"
    icon: mdi:alphabetical

- sensor:
    name: Update Version
    object_id: tesla_update_version
    unique_id: teslamate_1_update_version
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/update_version"
    icon: mdi:alphabetical

- sensor:
    name: Model
    object_id: tesla_model
    unique_id: teslamate_1_model
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/model"

- sensor:
    name: Trim Badging
    object_id: tesla_trim_badging
    unique_id: teslamate_1_trim_badging
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/trim_badging"
    icon: mdi:shield-star-outline

- sensor:
    name: Exterior Color
    object_id: tesla_exterior_color
    unique_id: teslamate_1_exterior_color
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/exterior_color"
    icon: mdi:palette

- sensor:
    name: Wheel Type
    object_id: tesla_wheel_type
    unique_id: teslamate_1_wheel_type
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/wheel_type"

- sensor:
    name: Spoiler Type
    object_id: tesla_spoiler_type
    unique_id: teslamate_1_spoiler_type
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/spoiler_type"
    icon: mdi:car-sports

- sensor:
    name: Geofence
    object_id: tesla_geofence
    unique_id: teslamate_1_geofence
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/geofence"
    icon: mdi:earth

- sensor:
    name: Shift State
    object_id: tesla_shift_state
    unique_id: teslamate_1_shift_state
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/shift_state"
    icon: mdi:car-shift-pattern

- sensor:
    name: Power
    object_id: tesla_power
    unique_id: teslamate_1_power
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/power"
    device_class: power
    unit_of_measurement: kW
    icon: mdi:flash

- sensor:
    name: Speed
    object_id: tesla_speed
    unique_id: teslamate_1_speed
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/speed"
    unit_of_measurement: "km/h"
    icon: mdi:speedometer

- sensor:
    name: Heading
    object_id: tesla_heading
    unique_id: teslamate_1_heading
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/heading"
    unit_of_measurement: °
    icon: mdi:compass

- sensor:
    name: Elevation
    object_id: tesla_elevation
    unique_id: teslamate_1_elevation
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/elevation"
    unit_of_measurement: m
    icon: mdi:image-filter-hdr

- sensor:
    name: Inside Temp
    object_id: tesla_inside_temp
    unique_id: teslamate_1_inside_temp
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/inside_temp"
    device_class: temperature
    unit_of_measurement: °C
    icon: mdi:thermometer-lines

- sensor:
    name: Outside Temp
    object_id: tesla_outside_temp
    unique_id: teslamate_1_outside_temp
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/outside_temp"
    device_class: temperature
    unit_of_measurement: °C
    icon: mdi:thermometer-lines

- sensor:
    name: Odometer
    object_id: tesla_odometer
    unique_id: teslamate_1_odometer
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/odometer"
    unit_of_measurement: km
    icon: mdi:counter

- sensor:
    name: Est Battery Range
    object_id: tesla_est_battery_range_km
    unique_id: teslamate_1_est_battery_range_km
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/est_battery_range_km"
    unit_of_measurement: km
    icon: mdi:gauge

- sensor:
    name: Rated Battery Range
    object_id: tesla_rated_battery_range_km
    unique_id: teslamate_1_rated_battery_range_km
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/rated_battery_range_km"
    unit_of_measurement: km
    icon: mdi:gauge

- sensor:
    name: Ideal Battery Range
    object_id: tesla_ideal_battery_range_km
    unique_id: teslamate_1_ideal_battery_range_km
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/ideal_battery_range_km"
    unit_of_measurement: km
    icon: mdi:gauge

- sensor:
    name: Battery Level
    object_id: tesla_battery_level
    unique_id: teslamate_1_battery_level
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/battery_level"
    device_class: battery
    unit_of_measurement: "%"
    icon: mdi:battery-80
    
- sensor:
    name: Usable Battery Level
    object_id: tesla_usable_battery_level
    unique_id: teslamate_1_usable_battery_level
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/usable_battery_level"
    unit_of_measurement: "%"
    icon: mdi:battery-80

- sensor:
    name: Charge Energy Added
    object_id: tesla_charge_energy_added
    unique_id: teslamate_1_charge_energy_added
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/charge_energy_added"
    device_class: energy
    unit_of_measurement: kWh
    icon: mdi:battery-charging

- sensor:
    name: Charge Limit Soc
    object_id: tesla_charge_limit_soc
    unique_id: teslamate_1_charge_limit_soc
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/charge_limit_soc"
    unit_of_measurement: "%"
    icon: mdi:battery-charging-100

- sensor:
    name: Charger Actual Current
    object_id: tesla_charger_actual_current
    unique_id: teslamate_1_charger_actual_current
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/charger_actual_current"
    device_class: current
    unit_of_measurement: A
    icon: mdi:lightning-bolt

- sensor:
    name: Charger Phases
    object_id: tesla_charger_phases
    unique_id: teslamate_1_charger_phases
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/charger_phases"
    icon: mdi:sine-wave

- sensor:
    name: Charger Power
    object_id: tesla_charger_power
    unique_id: teslamate_1_charger_power
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/charger_power"
    device_class: power
    unit_of_measurement: kW
    icon: mdi:lightning-bolt

- sensor:
    name: Charger Voltage
    object_id: tesla_charger_voltage
    unique_id: teslamate_1_charger_voltage
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/charger_voltage"
    device_class: voltage
    unit_of_measurement: V
    icon: mdi:lightning-bolt

- sensor:
    name: Scheduled Charging Start Time
    object_id: tesla_scheduled_charging_start_time
    unique_id: teslamate_1_scheduled_charging_start_time
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/scheduled_charging_start_time"
    device_class: timestamp
    icon: mdi:clock-outline

- sensor:
    name: Time To Full Charge
    object_id: tesla_time_to_full_charge
    unique_id: teslamate_1_time_to_full_charge
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/time_to_full_charge"
    unit_of_measurement: h
    icon: mdi:clock-outline

- sensor:
    name: TPMS Pressure Front Left
    object_id: tesla_tpms_fl
    unique_id: teslamate_1_tpms_fl
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/tpms_pressure_fl"
    unit_of_measurement: bar
    icon: mdi:car-tire-alert

- sensor:
    name: TPMS Pressure Front Right
    object_id: tesla_tpms_fr
    unique_id: teslamate_1_tpms_fr
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/tpms_pressure_fr"
    unit_of_measurement: bar
    icon: mdi:car-tire-alert

- sensor:
    name: TPMS Pressure Rear Left
    object_id: tesla_tpms_rl
    unique_id: teslamate_1_tpms_rl
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/tpms_pressure_rl"
    unit_of_measurement: bar
    icon: mdi:car-tire-alert

- sensor:
    name: TPMS Pressure Rear Right
    object_id: tesla_tpms_rr
    unique_id: teslamate_1_tpms_rr
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/tpms_pressure_rr"
    unit_of_measurement: bar
    icon: mdi:car-tire-alert

- sensor:
    name: Active route destination
    object_id: tesla_active_route_destination
    unique_id: teslamate_1_active_route_destination
    availability: *teslamate_active_route_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/active_route"
    value_template: "{{ value_json.destination }}"
    icon: mdi:map-marker

- sensor:
    name: Active route energy at arrival
    object_id: tesla_active_route_energy_at_arrival
    unique_id: teslamate_1_active_route_energy_at_arrival
    availability: *teslamate_active_route_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/active_route"
    value_template: "{{ value_json.energy_at_arrival }}"
    unit_of_measurement: "%"
    icon: mdi:battery-80

- sensor:
    name: Active route distance to arrival (mi)
    object_id: tesla_active_route_distance_to_arrival_mi
    unique_id: teslamate_1_active_route_distance_to_arrival_mi
    availability: *teslamate_active_route_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/active_route"
    value_template: "{{ value_json.miles_to_arrival }}"
    unit_of_measurement: mi
    icon: mdi:map-marker-distance

- sensor:
    name: Active route minutes to arrival
    object_id: tesla_active_route_minutes_to_arrival
    unique_id: teslamate_1_active_route_minutes_to_arrival
    availability: *teslamate_active_route_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/active_route"
    value_template: "{{ value_json.minutes_to_arrival }}"
    unit_of_measurement: min
    icon: mdi:clock-outline

- sensor:
    name: Active route traffic minutes delay
    object_id: tesla_active_route_traffic_minutes_delay
    unique_id: teslamate_1_active_route_traffic_minutes_delay
    availability: *teslamate_active_route_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/active_route"
    value_template: "{{ value_json.traffic_minutes_delay }}"
    unit_of_measurement: min
    icon: mdi:clock-alert-outline

- binary_sensor:
    name: Healthy
    object_id: tesla_healthy
    unique_id: teslamate_1_healthy
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/healthy"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:heart-pulse

- binary_sensor:
    name: Update Available
    object_id: tesla_update_available
    unique_id: teslamate_1_update_available
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/update_available"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:alarm

- binary_sensor:
    name: Locked
    object_id: tesla_locked
    unique_id: teslamate_1_locked
    availability: *teslamate_availability
    device: *teslamate_device_info
    device_class: lock
    state_topic: "teslamate/cars/1/locked"
    payload_on: "false"
    payload_off: "true"

- binary_sensor:
    name: Sentry Mode
    object_id: tesla_sentry_mode
    unique_id: teslamate_1_sentry_mode
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/sentry_mode"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:cctv

- binary_sensor:
    name: Windows Open
    object_id: tesla_windows_open
    unique_id: teslamate_1_windows_open
    availability: *teslamate_availability
    device: *teslamate_device_info
    device_class: window
    state_topic: "teslamate/cars/1/windows_open"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:car-door

- binary_sensor:
    name: Doors Open
    object_id: tesla_doors_open
    unique_id: teslamate_1_doors_open
    availability: *teslamate_availability
    device: *teslamate_device_info
    device_class: door
    state_topic: "teslamate/cars/1/doors_open"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:car-door

- binary_sensor:
    name: Trunk Open
    object_id: tesla_trunk_open
    unique_id: teslamate_1_trunk_open
    availability: *teslamate_availability
    device: *teslamate_device_info
    device_class: opening
    state_topic: "teslamate/cars/1/trunk_open"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:car-side

- binary_sensor:
    name: Frunk Open
    object_id: tesla_frunk_open
    unique_id: teslamate_1_frunk_open
    availability: *teslamate_availability
    device: *teslamate_device_info
    device_class: opening
    state_topic: "teslamate/cars/1/frunk_open"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:car-side

- binary_sensor:
    name: Is User Present
    object_id: tesla_is_user_present
    unique_id: teslamate_1_is_user_present
    availability: *teslamate_availability
    device: *teslamate_device_info
    device_class: presence
    state_topic: "teslamate/cars/1/is_user_present"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:human-greeting

- binary_sensor:
    name: Is Climate On
    object_id: tesla_is_climate_on
    unique_id: teslamate_1_is_climate_on
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/is_climate_on"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:fan

- binary_sensor:
    name: Is Preconditioning
    object_id: tesla_is_preconditioning
    unique_id: teslamate_1_is_preconditioning
    availability: *teslamate_availability
    device: *teslamate_device_info
    state_topic: "teslamate/cars/1/is_preconditioning"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:fan

- binary_sensor:
    name: Plugged In
    object_id: tesla_plugged_in
    unique_id: teslamate_1_plugged_in
    availability: *teslamate_availability
    device: *teslamate_device_info
    device_class: plug
    state_topic: "teslamate/cars/1/plugged_in"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:ev-station

- binary_sensor:
    name: Charge Port Door OPEN
    object_id: tesla_charge_port_door_open
    unique_id: teslamate_1_charge_port_door_open
    availability: *teslamate_availability
    device: *teslamate_device_info
    device_class: opening
    state_topic: "teslamate/cars/1/charge_port_door_open"
    payload_on: "true"
    payload_off: "false"
    icon: mdi:ev-plug-tesla
```
### sensor.yaml
增加跟官方文档一致的配置
```yaml
 - platform: template
   sensors:
    tesla_est_battery_range_mi:
      friendly_name: Estimated Range (mi)
      unit_of_measurement: mi
      icon_template: mdi:gauge
      value_template: >
       {{ (states('sensor.tesla_est_battery_range_km') | float / 1.609344) | round(2) }}

    tesla_rated_battery_range_mi:
      friendly_name: Rated Range (mi)
      unit_of_measurement: mi
      icon_template: mdi:gauge
      value_template: >
       {{ (states('sensor.tesla_rated_battery_range_km') | float / 1.609344) | round(2) }}

    tesla_ideal_battery_range_mi:
      friendly_name: Ideal Range (mi)
      unit_of_measurement: mi
      icon_template: mdi:gauge
      value_template: >
       {{ (states('sensor.tesla_ideal_battery_range_km') | float / 1.609344) | round(2) }}

    tesla_odometer_mi:
      friendly_name: Odometer (mi)
      unit_of_measurement: mi
      icon_template: mdi:counter
      value_template: >
       {{ (states('sensor.tesla_odometer') | float / 1.609344) | round(2) }}

    tesla_speed_mph:
      friendly_name: Speed (MPH)
      unit_of_measurement: mph
      icon_template: mdi:speedometer
      value_template: >
       {{ (states('sensor.tesla_speed') | float / 1.609344) | round(2) }}

    tesla_elevation_ft:
      friendly_name: Elevation (ft)
      unit_of_measurement: ft
      icon_template: mdi:image-filter-hdr
      value_template: >
       {{ (states('sensor.tesla_elevation') | float * 3.2808 ) | round(2) }}

    tesla_tpms_pressure_fl_psi:
      friendly_name: Front Left Tire Pressure (psi)
      unit_of_measurement: psi
      icon_template: mdi:car-tire-alert
      value_template: >
       {{ (states('sensor.tesla_tpms_fl') | float * 14.50377) | round(2) }}

    tesla_tpms_pressure_fr_psi:
      friendly_name: Front Right Tire Pressure (psi)
      unit_of_measurement: psi
      icon_template: mdi:car-tire-alert
      value_template: >
       {{ (states('sensor.tesla_tpms_fr') | float * 14.50377) | round(2) }}

    tesla_tpms_pressure_rl_psi:
      friendly_name: Rear Left Tire Pressure (psi)
      unit_of_measurement: psi
      icon_template: mdi:car-tire-alert
      value_template: >
       {{ (states('sensor.tesla_tpms_rl') | float * 14.50377) | round(2) }}

    tesla_tpms_pressure_rr_psi:
      friendly_name: Rear Right Tire Pressure (psi)
      unit_of_measurement: psi
      icon_template: mdi:car-tire-alert
      value_template: >
       {{ (states('sensor.tesla_tpms_rr') | float * 14.50377) | round(2) }}

    tesla_active_route_distance_to_arrival_km:
      friendly_name: Active route distance to arrival (km)
      unit_of_measurement: km
      icon_template: mdi:map-marker-distance
      value_template: >
        {{ (states('sensor.tesla_active_route_distance_to_arrival_mi') | float * 1.609344) | round(2) }}
```
### binary_sensor.yaml
```
 - platform: template
   sensors:
    tesla_park_brake:
      friendly_name: Parking Brake
      icon_template: mdi:car-brake-parking
      value_template: >-
       {% if is_state('sensor.tesla_shift_state', 'P') %}
         ON
       {% else %}
         OFF
       {% endif %}
```
### ui-lovelace.yaml
```
views:
  - path: car
    title: Car
    badges: []
    icon: mdi:car-connected
    cards:
      - type: vertical-stack
        cards:
          - type: glance
            entities:
              - entity: sensor.tesla_battery_level
                name: Battery Level
              - entity: sensor.tesla_state
                name: Car State
              - entity: binary_sensor.tesla_plugged_in
                name: Plugged In
          - type: glance
            entities:
              - entity: binary_sensor.tesla_park_brake
                name: Park Brake
              - entity: binary_sensor.tesla_sentry_mode
                name: Sentry Mode
              - entity: sensor.tesla_speed
                name: Speed
          - type: glance
            entities:
              - entity: binary_sensor.tesla_healthy
                name: Car Health
              - entity: binary_sensor.tesla_windows_open
                name: Window Status
          - type: horizontal-stack
            cards:
              - type: button
                entity: binary_sensor.tesla_locked
                name: Charger Door
                show_state: true
                state:
                  - value: locked
                    icon: mdi:lock
                    color: green
                    tap_action:
                      action: call-service
                      service: lock.unlock
                      service_data:
                        entity_id: lock.tesla_model_3_charger_door_lock
                  - value: unlocked
                    icon: mdi:lock-open
                    color: red
                    tap_action:
                      action: call-service
                      service: lock.lock
                      service_data:
                        entity_id: lock.tesla_model_3_charger_door_lock
              - type: button
                entity: lock.tesla_door_lock
                name: Car Door
                show_state: true
                state:
                  - value: locked
                    icon: mdi:lock
                    color: green
                    tap_action:
                      action: call-service
                      service: lock.unlock
                      service_data:
                        entity_id: lock.tesla_model_3_door_lock
                  - value: unlocked
                    icon: mdi:lock-open
                    color: red
                    tap_action:
                      action: call-service
                      service: lock.lock
                      service_data:
                        entity_id: lock.tesla_model_3_door_lock
      - type: vertical-stack
        cards:
          - type: map
            dark_mode: true
            default_zoom: 12
            entities:
              - device_tracker.tesla_location
          - type: thermostat
            entity: climate.tesla_model_3_hvac_climate_system
      - type: entities
        entities:
          - entity: sensor.tesla_display_name
            name: Name
          - entity: sensor.tesla_state
            name: Status
          - entity: sensor.tesla_since
            name: Last Status Change
          - entity: binary_sensor.tesla_healthy
            name: Logger Healthy
          - entity: sensor.tesla_version
            name: Software Version
          - entity: binary_sensor.tesla_update_available
            name: Available Update Status
          - entity: sensor.tesla_update_version
            name: Available Update Version
          - entity: sensor.tesla_model
            name: Tesla Model
          - entity: sensor.tesla_trim_badging
            name: Trim Badge
          - entity: sensor.tesla_exterior_color
            name: Exterior Color
          - entity: sensor.tesla_wheel_type
            name: Wheel Type
          - entity: sensor.tesla_spoiler_type
            name: Spoiler Type
          - entity: sensor.tesla_geofence
            name: Geo-fence Name
          - entity: proximity.home_tesla
            name: Distance to Home
          - entity: sensor.tesla_shift_state
            name: Shifter State
          - entity: sensor.tesla_speed
            name: Speed
          - entity: sensor.tesla_speed_mph
            name: Speed (MPH)
          - entity: sensor.tesla_heading
            name: Heading
          - entity: sensor.tesla_elevation
            name: Elevation (m)
          - entity: sensor.tesla_elevation_ft
            name: Elevation (ft)
          - entity: binary_sensor.tesla_locked
            name: Locked
          - entity: binary_sensor.tesla_sentry_mode
            name: Sentry Mode Enabled
          - entity: binary_sensor.tesla_windows_open
            name: Windows Open
          - entity: binary_sensor.tesla_doors_open
            name: Doors Open
          - entity: binary_sensor.tesla_trunk_open
            name: Trunk Open
          - entity: binary_sensor.tesla_frunk_open
            name: Frunk Open
          - entity: binary_sensor.tesla_is_user_present
            name: User Present
          - entity: binary_sensor.tesla_is_climate_on
            name: Climate On
          - entity: sensor.tesla_inside_temp
            name: Inside Temperature
          - entity: sensor.tesla_outside_temp
            name: Outside Temperature
          - entity: binary_sensor.tesla_is_preconditioning
            name: Preconditioning
          - entity: sensor.tesla_odometer
            name: Odometer
          - entity: sensor.tesla_odometer_mi
            name: Odometer (miles)
          - entity: sensor.tesla_est_battery_range_km
            name: Battery Range (km)
          - entity: sensor.tesla_est_battery_range_mi
            name: Estimated Battery Range (mi)
          - entity: sensor.tesla_rated_battery_range_km
            name: Rated Battery Range (km)
          - entity: sensor.tesla_rated_battery_range_mi
            name: Rated Battery Range (mi)
          - entity: sensor.tesla_ideal_battery_range_km
            name: Ideal Battery Range (km)
          - entity: sensor.tesla_ideal_battery_range_mi
            name: Ideal Battery Range (mi)
          - entity: sensor.tesla_battery_level
            name: Battery Level
          - entity: sensor.tesla_usable_battery_level
            name: Usable Battery Level
          - entity: binary_sensor.tesla_plugged_in
            name: Plugged In
          - entity: sensor.tesla_charge_energy_added
            name: Charge Energy Added
          - entity: sensor.tesla_charge_limit_soc
            name: Charge Limit
          - entity: binary_sensor.tesla_charge_port_door_open
            name: Charge Port Door Open
          - entity: sensor.tesla_charger_actual_current
            name: Charger Current
          - entity: sensor.tesla_charger_phases
            name: Charger Phases
          - entity: sensor.tesla_charger_power
            name: Charger Power
          - entity: sensor.tesla_charger_voltage
            name: Charger Voltage
          - entity: sensor.tesla_scheduled_charging_start_time
            name: Scheduled Charging Start Time
          - entity: sensor.tesla_time_to_full_charge
            name: Time To Full Charge
          - entity: sensor.tesla_tpms_pressure_fl
            name: Front Left Tire Pressure (bar)
          - entity: sensor.tesla_tpms_pressure_fl_psi
            name: Front Left Tire Pressure (psi)
          - entity: sensor.tesla_tpms_pressure_fr
            name: Front Right Tire Pressure (bar)
          - entity: sensor.tesla_tpms_pressure_fr_psi
            name: Front Right Tire Pressure (psi)
          - entity: sensor.tesla_tpms_pressure_rl
            name: Rear Left Tire Pressure (bar)
          - entity: sensor.tesla_tpms_pressure_rl_psi
            name: Rear Left Tire Pressure (psi)
          - entity: sensor.tesla_tpms_pressure_rr
            name: Rear Right Tire Pressure (bar)
          - entity: sensor.tesla_tpms_pressure_rr_psi
            name: Rear Right Tire Pressure (psi)
          - entity: sensor.tesla_active_route_destination
            name: Active Route Destination
          - entity: sensor.tesla_active_route_energy_at_arrival
            name: Active Route Energy at Arrival
          - entity: sensor.tesla_active_route_distance_to_arrival_km
            name: Active Route Distance to Arrival (km)
          - entity: sensor.tesla_active_route_distance_to_arrival_mi
            name: Active Route Distance to Arrival (mi)
          - entity: sensor.tesla_active_route_minutes_to_arrival
            name: Active Route Minutes to Arrival
          - entity: sensor.tesla_active_route_traffic_minutes_delay
            name: Active Route Traffic Minutes Delay
```
修改完上述配置后即可重启 HA 服务。
## HA 中的配置 MQTT
在 **配置** -> **设备与服务** -> **添加集成** 中输入 MQTT，点击后输入如下信息。因为我的 MQTT 服务和 HA 在同一台机器上，所以可以输入 127.0.0.1 进行访问。
![image.png|505](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601010743.png)
点击提交后可以看到 Tesla Model Y 的配置说明已经成功，可以在 MQTT 中找到 Tesla 的数据。

![20240602001104.png|361](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240602001104.png)
## HA 的 dashboard 配置
在

# 资料
[Home Assistant接入teslamate相关问题](https://www.xiaote.com/r/62d8e6063bba567764159459)
[中用更中看，Lovelace 界面定制与 HomeAssistant 美化](https://post.smzdm.com/p/axlxed09/#:~:text=%E4%B8%AD%E7%94%A8%E6%9B%B4%E4%B8%AD%E7%9C%8B%EF%BC%8CLovelace%20%E7%95%8C%E9%9D%A2%E5%AE%9A%E5%88%B6%E4%B8%8E%20HomeAssistant%20%E7%BE%8E%E5%8C%96)