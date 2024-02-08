# ESP32 Ready4Sky (R4S) gateway for Redmond+ devices
> **[A script based on a PHP server on a local network with Internet access to display the weather on the gateway screen.](https://github.com/artt652/Weather-for-ESP32-R4S-gate).**<br>
>**In versions starting from 2022.06.03, the device topics in gateway number 0 have been changed from “r4s/#” to “r4s0/#”. In new versions, topics in "r4s/#" are used to track tags by multiple gateways. When upgrading from older versions, you need to enable BLE Monitor in the settings (the gateway deletes the contents of "r4s/#" only when the monitor is turned on), selecting Passive, Active or Auto, check the Hass Discovery and Delete Mqtt Topics items and save the settings. After the reboot, the gateway will delete unnecessary topics and create them again. Then, if necessary, BLE Monitor can be disabled. Then fix the devices in automations, scripts, etc.**

#### Current version is 2024.02.05 для [ESP32](https://github.com/alutov/ESP32-R4sGate-for-Redmond/raw/master/build/r4sGate.bin) и [ESP32C3](https://github.com/alutov/ESP32-R4sGate-for-Redmond/raw/master/C3/build/r4sGate.bin).
* 2024.02.05. At startup, the gateway scans all WIFI APs with the required parameters and selects the AP with the best level. Relevant for mesh systems.
* 2024.02.04. Added selection of WIFI 802.11b/g/n mode in settings.
* 2024.02.02. Corrections by Delonghi. In Auto mode, the image buffer size is selected separately for each URL.
* 2024.01.30. Added the ability to automatically increase the buffer size for loading images within 10-65 kilobytes (Adjust option). The gateway operates more stable at wifi levels below -73dBm.
* 2024.01.25. Corrected Delonghi statistics and availability of BLE devices/tags in Home Assistant. Currently Delonghi in HA:
![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/ecam650.jpg)  
* 2024.01.22. Added mute option as an alternative to beep. The availability of devices in Home Assistant has been fixed. If the gateway is unavailable, all devices connected to it become unavailable. If devices disappear from the BLE monitor, the data from them becomes inaccessible. Added statistics to Delonghi coffee machines.
* 2024.01.14. The project was built using esp-idf version 5.2-beta2. Added Russian program names to the multicooker menu. Added support for the Redmond SkyHeat RCH-4560S heater. Added support for LYWSDCGQ thermometers to BLE Gateway Monitor. You can display up to 4 images in turn on the gateway screen using 4 urls.

## 1. Opportunities
&emsp; The ESP32 r4sGate gateway in a minimal configuration (only ESP32 or ESP32C3 with a 3.3v power supply) allows you to connect BLE-compatible Redmond devices, Xiaomi MiKettle kettles and some other devices to the smart home system (Home Assistant, OpenHab, ioBroker, MajorDoMo, etc. ) via MQTT protocol. Initially, the project was only for Redmond, now other devices are being added. Hence the plus in the name of the project.
<details>
<summary>Why MQTT...</summary>

* This is in fact a standard protocol for smart home systems. Another thing is how it is implemented. From the built-in MQTT server in the ioBroker system, where everything that happens in MQTT is immediately displayed in the admin interface, to the external broker in Home Assistant, where sometimes you have to use third-party utilities to configure it. In the latter, however, MQTT Discovery greatly simplifies the integration of devices into the system.
* In MQTT, devices can exchange data with each other, and not just with the smart home server. The gateway can be configured to display the readings of the main sensors in the house on the screen. In response to the argument that some devices can uncontrollably rewrite the data of others, I note that normal brokers have a shared access system (ACL). Although I don’t have an answer to the question, why even introduce devices into the system that are not trusted.
* In MQTT, devices can exchange information without knowing anything about each other except the topic where they should meet. This is used to determine the maximum signal strength from a tag/beacon among multiple gateways, and the gateway receiving the strongest signal.
* MQTT support is included in the esp-idf development environment and does not require third-party libraries. After unsuccessful attempts to finalize a very good [olehs project](https://github.com/olehs/r4sGate) on arduino, I became a supporter of pure esp-idf.
</details>     

#### List of supported devices:

**Electric kettles:**
* Redmond SkyKettle **RK-M170S**
* Redmond SkyKettle **RK-M173S** / RK-M171S / RTP-M810S
* Redmond SkyKettle **RK-G200S** / RK-G204S / RK-G210S / RK-G211S / RK-G212S / RK-G214S / RK-M216S / RK-M139S
* Redmond SkyKettle **RK-G240S** / RK-G204S / RK-G210S / RK-G211S / RK-G212S / RK-G214S / RK-M216S / RK-M139S
* Xiaomi MiKettle **YM-K1501(Int)** - ProductId 275
* Xiaomi MiKettle **YM-K1501(HK)** - ProductId 131
* Xiaomi MiKettle **V-SK152(Int)** - ProductId 1116
<details>
<summary>Read more about Xiaomi MiKettle...</summary>
&emsp;Xiaomi MiKettle can only be controlled from the **keep warm** mode. In this mode, the kettle maintains a minimum temperature of 40°C set by the gateway with a hysteresis of approximately 4°C, that is, at 36°C the heating is turned on, and at 44°C it is turned off. You can turn boiling on and off (state = ON/OFF), set the heating temperature (heat_temp = 40...95). You can switch the kettle to Idle mode (heat_temp = 0). The last command is executed with a delay. After executing the command, further control of the kettle is unavailable. In contrast to turning it off with the **warm** sensor on the kettle, when you turn it off and on again, the kettle returns to the **keep warm** mode. Perhaps this is a feature of a specific version of MCU 6.2.1.9. For now I left it like that and turned on the kettle through the Redmond outlet. If you turn it off and on again, the kettle goes into heating mode. The gateway installs all the necessary parameters of the kettle itself, and the native application is useful for updating the firmware. The heating time is set to 12 hours (720 minutes), after 256 minutes the gateway resets the counter by briefly turning the boiling on and off. And still control is limited. The main problem is that when you turn on boiling with the **boil** sensor on the kettle, the **keep warm** mode is turned off and you can return it only with the **warm** button on the kettle. For the same reason, I postponed work on Mikettle Pro for now.</details>

**Multicookers**

* Redmond SkyCooker **RMC-M224S**
* Redmond SkyCooker **RMC-M800S**
* Redmond SkyCooker **RMC-M903S**
* Redmond SkyCooker **RMC-M92S**
* Redmond SkyCooker **RMC-M961S**
     
**Coffee makers**

* Redmond SkyCoffee **RCM-M1519S**

**Coffee machines**

* Delonghi **ECAM650.75** (Possibly other models 😉 Primadonna Elite series)

**Sockets**

* Redmond SkyPort **RSP-103S** / RSP-100S

**Electric convectors**

* Redmond SkyHeat **RCH-7001S** / RCH-7002S / RCH-7003S
* Redmond SkyHeat **RCH-4560S**

**Climate stations**

* Redmond SkyClimate **RSC-51S**

**Humidifiers**

* Redmond SkyDew **RHF-3310S**

**Sensors**

* Redmond SkySmoke **RSS-61S** - smoke sensor
* Redmond SkyOpen **RSO-31**  - door open sensor
* Hilink **HLK-LD2410B** - motion and presence sensor

**Irrigation controllers**
     
* Galcon **GL9001A** / Green Apple GATB010-03

**Curtain/blind drivers**
     
* **AM43 blinds** A-OK and similar)

&emsp;The gateway supports 5 simultaneous BLE connections. Device management is also possible from the gateway web interface. The web interface is [simply protected with a password from Raerten](https://github.com/alutov/ESP32-R4sGate-for-Redmond/pull/67). To do this, the string in the form login:password must be encrypted in Base 64 and then entered into the Basic Auth field in the settings. The password string is output to the log when the gateway starts.<br/>
&emsp;Home Assistant Mqtt Discovery is supported. To enable it, you need to check **Hass Discovery** in the settings. It is possible to delete all data created by the gateway in Mqtt and devices in Home Assistant. To do this, select the **Delete Mqtt topics** option in the **Setting** tab and then click **Save setting** . After rebooting the gateway, only devices connected to the gateway will be recreated. Recommended when connecting the gateway for the first time and reconfiguring by removing devices.Поддерживается Home Assistant Mqtt Discovery.<br/> 

<details>
<summary>A simple option for integrating a kettle from Home Assistant into a Yandex smart home is to use the climate entity.</summary>

&emsp; And call it the word **teapot**. All commands of the thermostat device will be available. For example, the command **turn on the kettle** (auto mode, turning on boiling or boiling followed by heating, if heating was turned on before), **turn off the kettle** (turns everything off), **set the kettle temperature to 40°** (if not 0° and not 100° turns on heating, mode heat, if 0° - turns it off, if the temperature is 100°, boiling is turned on, auto mode) or **turn on the heating** (turns on heating at the last set temperature). And finally, the command **turn on cooling** - turns on the backlight, cool mode. Not very pretty, but that's how it is. You can ask what’s wrong with the kettle - the kettle’s thermostat is turned off and what the temperature of the kettle is - it will tell the current temperature.<br>
</details>     

<details>
<summary>It supports calculating the amount of water in the kettle when heating in the range of 65-85°C and more than 3°C from the moment the kettle is turned on.</summary>

&emsp;No modifications to the kettle are required. Calculated based on the energy expended and the temperature difference. The calculated value is reset when the kettle is removed from the stand. The option only works in dummies with statistics. The efficiency of the kettle was initially assumed to be 80%. The accuracy is so-so, I get about ~0.2 liters. To improve accuracy, a mode for adjusting the efficiency value is provided. To do this, you need to pour 1 liter of water into the kettle and select **Boil 1l on** in the web interface . When the mode works, you need to enter the settings mode. The new value will be displayed immediately after the device type. You can write a new value to nvram using the Save setting command. It seems to me that it is unrealistic to obtain greater accuracy, since the efficiency of the kettle changes over time, for example, with the appearance of scale, and, what’s worse, the energy expended is not measured, but is simply calculated by the kettle’s processor based on the rated power of the heater and its operating time. The deviation of the supply voltage during operation from the value during calibration introduces a noticeable error; the dependence there is quadratic. When I boil 1.7 liters of water with a RK-M216S kettle at a voltage at the entrance to the house of 200-204V, the result is 1.8 liters, at a voltage of 210-214V it comes out to 1.6 liters. During calibration there was obviously something in between.<br>
</details>     

&emsp;The BLE Gateway Monitor can be used to monitor up to 24 BLE beacon devices with a static MAC address. The presence/absence of a tag (beacon) and rssi is displayed. Supported are BLE beacons of the Home Assistant application on smartphones (binding by uuid), LYWSD02 watch with thermometer, LYWSD03MMC Xiaomi Mijia 2 thermometers with original firmware, firmware from atc1441in [custom mode}(https://github.com/atc1441/ATC_MiThermometer) and firmware from [pvvx in custom mode](https://github.com/pvvx/ATC_MiThermometer), Xiaomi Mi Scale, Qingping Air Monitor Lite (CGDN1), Elehant counters, as well as Samsung Smart Tag.<br>
&emsp;There are 10 I/O ports, 5 of them can be used to control external devices (Out mode) and read their status (In mode). Three ports can be configured as buttons for turning on/off connected BLE devices (Sw mode, in which case the state of the buttons in mqtt is not displayed), the fourth port can be configured as a button for updating the image from the camera. When configured in input mode, pullup is enabled if possible (pin number less than 34). Another 2 ports are used by the I2C bus, and each of the 3 remaining ports can be used as a pulse width modulated (PWM) output, or as an input for connecting either one DS18B20 sensor with direct power supply, or one DHT22/AM2302 sensor (7 and port 8). Reading procedures are simplified, the checksum is not read or verified, and data is rounded to one decimal place. If the gateway is equipped with an audio emitter, then by connecting the PWM output to it (in m5stack basic this is gpio 25), you can output an audio signal. By changing the duty cycle of the pulses, you can adjust the volume. The frequency is fixed and equal to 3.136 kHz. The I2C bus supports sensors SHT3x/SHT4X(addresses 0x44, 0x45), AHT20(0x38), HTU21(0x40), BMP280/BME280/680/688(0x76, 0x77, 688 not yet tested), SGP30(0x58), SGP4x(0x59 ), SCD4x(0x62), as well as RTC DS3231(0x68) and battery controller IP5306(0x75). It is possible to save the SGP30 calibration data in NVRAM and restore it when the gateway starts. To do this, check the AQ base item in the settings. The [Sensirion library](https://github.com/Sensirion/gas-index-algorithm) is used to calculate VOC in SGP4x. The clock is used to store the date and time from the NTP server, the temperature sensor is output to Mqtt. The IP5306 controller is installed in m5stack and ttgo-t4 (SCL 22, SDA 21), allows you to determine the battery level in 25% increments and its mode (Discarging / Charging / Charged). When on battery power, the screen brightness decreases by 16 times. The gateway also supports the AXP192 power controller and PCF8563 RTC, allowing it to run on the M5Stack Tough, and also supports the ADC of the HX711 load cells. The measurement result from the HX711 can be displayed either in kilograms or as a percentage, depending on the calibration. HX711 is polled at intervals of 4 seconds, other sensors at intervals of 12 seconds. The gateway allows hot plugging of all sensors. Sensors 18B20 and DHT22 appear in Mqtt and Home Assistant immediately after the gateway starts, even if they are not connected, and I2C sensors as they are detected on the bus within 2 polling cycles (24 seconds).
<details>
     <summary>There is also support for IR transmitter (IR Tx, port 6).</summary>

&emsp;Supported protocols are **NEC** (8 and 16 bit address) **RC5** (to work in RC5ext mode you need to invert 6 bits of the command), **RC6** , **Samsung** , Sony **SIRC** (12, 15 and 20 bits), **Panasonic**. You can control it both from the Home Assistant interface and individual topics of address, command and protocol, or by directly writing to the **r4sx/ir6code** topic (where **x** is the gateway number) a string of 8 hex characters 0-9,af, for example, **090a1c3d** , where **09** is protocol(01-nec, 02-necx16, 03-rc5, 04-rc6, 05-samsung, 06-sircx12, 07-sircx15, 08-sircx20, 09-panasonic), **0a1c** - address , **3d** command.
What was tested (the power-on command was of interest):<br>
**NEC:** pioneer vsx-830, power: addr 165, cmd 28, code 0100a51c<br>
**NECx16:** lg dvd dks-2000h, power: addr 11565, cmd 48, code 022d2d30<br> 
**RC6:** philips 40pfs6609, power: addr 0, cmd 12, code 0400000c<br>
**SAMSUNG:** ue32n5300, power: addr 7, cmd 2, code 05000702<br>
**SIRCx12:** sony cmt-sx7, power: addr:16, cmd: 21, code 06001015<br>
**SIRCx20:** sony ubp-x800 power: addr 7258, cmd 21, code 081c5a15<br>
**PANASONIC:** sa-pm20 power: addr 2588, cmd 61, code 090a1c3d<br>
**PANASONIC:** dmp-ub900 power: addr 2816, cmd 61, code 090b003d<br>

I checked everything on the Atom lite, it has IR LED on 12 gpio. RC5 and SIRCx15 have not yet been tested.<br>
</details>

&emsp;In order to expand the capabilities of the gateway, it is possible to connect a 320 * 240 TFT screen on ili9341, ili9342 and ST7789 chips. The screen displays the current time, date, as well as temperature, voltage and current in the house (not a problem when powered by a generator), status (blue - off, red - on) and temperature at the boiler outlet, temperature and humidity outside. Everything comes from Mqtt. Next to the date, the status of BLE devices is displayed in color, 1 ... 5 - from the first to the fifth. Gray - not connected or not defined, blue - off, red - on, yellow - heating, white - program installed. More detailed status and some parameters of connected BLE devices are displayed in turn in the bottom line. It is possible to periodically or on request display a picture on the screen in jpeg format, for example, from a camera. Pictures with a horizontal resolution higher than 320 are displayed at a scale of 1:2. The buffer size for loading images can be changed within 20-65 kilobytes. Screen brightness can be changed by Mqtt. You can also display the weather in text form from the site wttr.in or just text by writing it in the Mqtt topic r4sx/jpg_url. The result was something similar to a clock with a thermometer. It is enough to look at the screen to make sure that everything is in order in the house or on the street today.<br>

## 2. Accessories

![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/myparts.jpg)
Picture 1. Components for assembling the gateway.

&emsp; If the goal is to launch the gateway at minimal cost, you will have to buy spare parts and then assemble the gateway from them. I used an [ESP32 WROOM ESP-32 4 MB with a built-in antenna (bottom left) or an ESP32 WROOM ESP-32U 4 MB with an external one (to the right of the first)](https://aliexpress.ru/item/32961594602.html?item_id=32961594602&sku_id=66888778667&spm=a2g2w.productlist.0.0.4f835c61Fvd1gD) . Issue price $2.5. Then I soldered the microcircuit onto [an adapter board ($0.3)](https://www.aliexpress.com/item/32763489487.html?spm=a2g2w.productlist.i0.2.48d33c75KbnjpB&sku_id=62208988599) and then onto a breadboard. ESP32C3 is also suitable, I have ESP32C3-12F. Due to hardware limitations of this chip, the gateway uses port 8 only as a pulse width modulation (PWM) output. There is approximately 28 kilobytes more free RAM. And even with the screen connected, the ESP32C3-12F still has 6 free gpio. Power supply for 3.3 Hi-Link ($2-$4). [I bought them for $1.65](https://aliexpress.ru/item/32953853140.html?spm=a2g39.orderlist.0.0.32964aa6PePEbg&_ga=2.238912000.104655408.1636114275-428746708.1615828563&_gac=1.87036010.1634012869.Cj0KCQjwwY-LBhD6ARIsACvT72Na1GBQp7leEJDlxPCd0jTye8sF-GiknWzlo4hKElMNbtmI4DYpB_8aAktOEALw_wcB). **You can avoid soldering** if you use [esp32-wroom-devkit (bottom center, $14)]((ttps://aliexpress.ru/item/4000127837743.html?sku_id=10000000372418546&spm=a2g0s.9042311.0.0.274233edNcajyj). True, this board is very redundant for the project; you can get a [simpler one for $3.54](https://aliexpress.ru/item/32928267626.html?item_id=32928267626&sku_id=12000016847177755&spm=a2g2w.productlist.0.0.430c65c8Kf9vOT). In it, esp32 comes along with a board on which there are also converters from 5V to 3.3V, USB-RS232 and a standard mini-USB connector. Through it you can power the esp32 using a five-volt charger from a smartphone, and program it directly from the computer without any adapters. And on the right in the photo is a 3.2" [320 * 240 TFT screen ($18)](https://aliexpress.ru/item/32911859963.html?spm=a2g0s.9042311.0.0.274233edzZnjSp) , which I used in the gateway. You can also use compatible ready-made devices both with a screen ( **TTGO T-Watcher BTC Ticker** , **M5Stack BASIC** , **M5Stack Tough** ) and without ( **m5atom lite**).

## 3. Настройка шлюза

&emsp; Для запуска шлюза нужно [запрограммировать ESP32](https://www.youtube.com/watch?v=Vy-YTSdwy7s). Для прошивки можно использовать программу [flash_download_tools](https://www.espressif.com/en/support/download/other-tools). Файл [**fr4sGate.bin**](https://github.com/alutov/ESP32-R4sGate-for-Redmond/raw/master/build/fr4sGate.bin) в папке build это уже собранный бинарник для esp32 @160MHz с памятью 4 Мбайт, DIO загрузчиком и прошивается одним файлом с адреса **0x0000** в режиме **DIO**.  Если же DIO загрузчик не стартует, можно использовать файл [**fqr4sGate.bin**](https://github.com/alutov/ESP32-R4sGate-for-Redmond/raw/master/build/fqr4sGate.bin) с загрузчиком QIO и программировать его в режиме **QIO**. Как я понял, большинство esp32 можно программировать в любом режиме, но были случаи, что шлюз работал только при прошивке его файлом **fqr4sGate.bin** в режиме **QIO**. Файл [**r4sGate.bin**](https://github.com/alutov/ESP32-R4sGate-for-Redmond/raw/master/build/r4sGate.bin) используется для обновления прошивки через web интерфейс. Файлы для программирования ESP32C3 в папке C3.<br>
&emsp; Затем нужно создать точку доступа на смартфоне с ssid **r4s** и паролем **12345678**, подождать, пока esp32 не подключится к нему, найти в параметрах точки доступа подключенное устройство **r4s0Gate** и его IP, ввести этот адрес в веб-браузере и далее в web во вкладке Setting установить остальные параметры. После чего точка доступа больше не нужна. Esp32 будет пытаться подключиться к сети **r4s** только при недоступности основной сети, например, при неправильном пароле. Если не удается подключиться и к гостевой сети, esp32 перезагружается. Вариант с гостевой сетью в отличие от общепринятой практики запуска точки доступа на esp32, как мне кажется, удобнее, так как в случае падения по каким-то причинам Wi-Fi роутера (а он может быть выделенным только для iot устройств) остальной Wi-Fi не засоряется дружно вплывшими esp32.<br>
&emsp; Далее нужно ввести имя или  MAC адрес Redmond устройства и привязать его к шлюзу. Поиск устройств запускается только тогда, когда есть хотя бы одно определенное, но не соединенное устройство, либо активен BLE монитор. Если имя устройства точно не известно (а редмонды не всегда светятся по BLE как модель один в один), то для начала сканирования нужно ввести в поле **Name** в настройках любое имя, а потом заменить его найденным при сканировании и выбрать в настройках ближайший тип устройства (поле **TYPE**, например, для чайников от RK-G(M)200S до RK-G(M)240S  протокол один и тот же, можно выбрать как RK-G200S, так и RK-G240S). Нужно учитывать, что не все устройства передают имя при пассивном сканировании (например, Xiaomi Mikettle, AM43 Blinds). В любом случае лучше вводить в поле имени MAC адрес, можно как с двоеточиями, так и без. Найти и скопировать адрес можно **BLE Last found name/address** на главной страничке или на странице BLE monitor. Далее для привязки нужно нажать и удерживать кнопку **+** на чайнике или **таймер** на мультиварке  до тех пор, пока устройство не войдет в режим привязки и через некоторое время соединится со шлюзом. Жалюзи AM43 требуют еще и ввода пин кода (Passkey) для соединения.<br> 
&emsp; Предусмотрена возможность подключения к одному MQTT серверу нескольких шлюзов. Для этого нужно в каждом шлюзе установить свой r4sGate Number. Шлюз с номером 0 будет писать в топик r4s0/devaddr/..., шлюз с номером 1 - r4s1/devaddr/... и т.д. Нужно только учесть, что запрос на авторизацию при привязке зависит от номера шлюза и от номера соединения в шлюзе. Это позволяет привязать 2 одинаковых чайника или мультиварки к 2 разным шлюзам или к 2 разным соединениям в пределах одного шлюза. Если же рядом работают два шлюза с одинаковыми параметрами, подключенные к разным системам умного дома (например, сосед за стенкой), для исключения возможности подключения устройства к шлюзу соседа можно использовать опцию авторизации устройств с использованием MAC адреса шлюза, выбрав в настройках **Use MAC in BLE Authentication**. После чего сбросить на устройствах все привязки и затем привязать их к шлюзу заново.<br>&emsp; Для подключения к Mqtt брокеру нужно ввести его адрес и порт, а также логин и пароль. Если шлюз работает с Home Assistant в паре с mosquitto брокером, стоит использовать опцию Hass Discovery. Перед ее использованием рекомендую удалить в Mqtt брокере все топики с r4s, для чего выбрать в настройках **Delete Mqtt topics**. Если же система не поддерживает Mqtt Discovery, придется разбираться с Mqtt.  
<details>
<summary>Подробнее по Mqtt...</summary>
     У меня шлюз подключен к ioBroker. Мои настройки MQTT брокера ниже на картинке 3.<br>

![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mymqtt.jpg)
Картинка 3. Мои Mqtt настройки.<br><br>
     
&emsp; Снят флаг retain, чтобы брокер не запоминал, а считывал состояние устройств при соединении. В Home Assistant  установленный в нем и/или Mqtt брокере флаг retain [может приводить к самопроизвольному включению и выключению устройства](https://mjdm.ru/forum/viewtopic.php?f=8&t=5501&sid=de6b1e2b43f25c8d9ae9af5673ee9417&start=140#p121604).  Также установлен флаг публикации при подписке, что позволяет не вводить все топики вручную. Иногда при публикации сразу большого числа подписок ioBroker почему-то делает некоторые из них с защитой от записи :-), есть у меня такой глюк. Приходится их удалять и перезапускать Mqtt адаптер, чтобы они появились опять.<br>

#### Mqtt топики для чайника (см. картинку 4 ниже):

`r4s0/devaddr/cmd/boil` <-- `0/off/false` - выключение кипячения, `1/on/true` - включение кипячения. Если перед этим был включен подогрев, включается кипячение с последующим подогревом;<br>
`r4s0/devaddr/cmd/heat` <-- `0/off/false` - выключение подогрева, `1/on/true` - включение подогрева с последней запомненной шлюзом температурой. При старте шлюза температура берется из поля Heat в настройках;<br>
`r4s0/devaddr/cmd/heat_temp` <-- `30...90` - включение подогрева, `> 97` - выключение, `< 30` - выключение, если подогрев был включен, последняя температура если был выключен;<br>
`r4s0/devaddr/cmd/boiltime` <-- `-5...5` - время кипения чайника;<br>
`r4s0/devaddr/cmd/nightlight` <-- `0/off/false` - выключение ночника, `1/on/true` - включение ночника;<br>
`r4s0/devaddr/cmd/nightlight_red` <-- `0..255` Уровень красного в ночнике;<br>
`r4s0/devaddr/cmd/nightlight_green` <-- `0..255` Уровень зеленого в ночнике;<br>
`r4s0/devaddr/cmd/nightlight_blue` <-- `0..255` Уровень синего в ночнике;<br>
`r4s0/devaddr/rsp/` - текущее состояние, температура, rssi и т.д.;<br>

> Значения уровней запоминаются в шлюзе и передаются на чайник при включении подсветки.
<br>

![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mymqtt1.jpg)
Картинка 4. Мои Mqtt топики для чайника.

#### Mqtt топики для мультиварки (см. картинку 5 ниже):

`r4s0/devaddr/cmd/state` <-- `0/off/false` - выключение, `1/on/true` - старт программы или подогрев, если программа не установлена;<br> 
`r4s0/devaddr/cmd/prog` <-- номер программы 1-12, 0 - выключение;<br>
     Программы:<br>
     1 - Rice / Рис Крупы, 2 - Slow Cooking / Томление, 3 - Pilaf / Плов, 4 - Frying / Жарка;<br>
     5 - Stewing / Тушение, 6 - Pasta / Паста, 7 - Milk Porridge / Молочная каша, 8 - Soup / Суп;<br>
     9 - Yogurt / Йогурт, 10 - Baking / Выпечка, 11 - Steam / Пар, 12 - Hot / Варка Бобовые;<br>
`r4s0/devaddr/cmd/mode` <-- режим: 1 - овощи, 2 - рыба, 3 - мясо для программ 4,5,11<br>
`r4s0/devaddr/cmd/temp` <-- температура;<br>
`r4s0/devaddr/cmd/set_hour` <-- время работы программы, часы;<br>
`r4s0/devaddr/cmd/set_min` <-- время работы программы, минуты;<br>
`r4s0/devaddr/cmd/delay_hour` <-- время работы программы плюс задержка до старта программы, часы;<br>
`r4s0/devaddr/cmd/delay_min` <-- время работы программы плюс задержка до старта программы, минуты;<br>
`r4s0/devaddr/cmd/warm` <-- подогрев после завершения программы;<br>

> Параметры `delay_hour` и `delay_min` запоминаются в шлюзе и передаются при установке программы, режима или подогрева, а потому устанавливаются перед установкой программы, режима или автоподогрева. При выборе программы устанавливаются температура и время работы программы по умолчанию, после установки mode еще раз корректируются. После установки программы и режима можно при необходимости скорректировать время и температуру. Этот порядок установки параметров обусловлен тем, что по Mqtt нельзя сразу установить одной командой все параметры, если они находятся в разных топиках. При установке же через web все параметры ставятся одной командой. И значения температуры и времени по умолчанию для каждой программы и режима устанавливаются через web только если перед этим они были  равны 0. Программа мультиповар пока не поддерживается, я не вижу смысла. При записи нуля в prog на мультиварку посылается команда выключения, что полезно для сброса программы.
<br>

![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mymqtt2.jpg)
 Картинка 5. Мои Mqtt топики для мультиварки.

#### Mqtt топики для кофеварки:

`r4s0/devaddr/cmd/state` <-- `0/off/false` - выключение, `1/on/true` - включение;<br>
`r4s0/devaddr/cmd/delay_hour` <-- время отложенного старта, часы;<br>
`r4s0/devaddr/cmd/delay_min` <-- время отложенного старта, минуты;<br>
`r4s0/devaddr/cmd/delay` <-- запуск отложенного старта, `0/off/false` - выключение, `1/on/true` - включение;<br>
`r4s0/devaddr/cmd/lock` <-- блокировка, `0/off/false` - выключение, `1/on/true` - включение;<br>
`r4s0/devaddr/cmd/strength` <-- крепость, `0/off/false` - выключение, `1/on/true` - включение;<br>
`r4s0/devaddr/rsp/` - текущее состояние, rssi и т.д.;<br>

> Значения времени отложенного старта запоминаются в шлюзе и передаются на кофеварку при включении этого режима.

#### Mqtt топики для розетки:

`r4s0/devaddr/cmd/state` <-- `0/off/false` - выключение, `1/on/true` - включение;<br>
`r4s0/devaddr/cmd/lock` <-- блокировка, `0/off/false` - выключение, `1/on/true` - включение;<br>
`r4s0/devaddr/rsp/` - текущее состояние, rssi и т.д.;

&emsp; Начиная с версии 2020.10.27 появилась возможность использовать совмещенные топики для команд и ответов. Опция включается в настройках. Мне это пригодилось при интеграции устройств в Google Home с помощью драйвера iot iobroker-а. Как я понял, этот драйвер не принимает раздельные топики команд/ответов. Кроме того, так как Google Home понимает `true / false` вместо `ON / OFF`, то нужно в настройках драйвера iot `Conversation to Google Home = function (value) {}` ввести строку вида `switch (value) {case **ON**: return true ; break; default: return false;}`. Если же автозамена недоступна, то начиная с версии 2020.11.07 можно использовать опцию `**true / false** Response`. Опция не работает совместно с Hass Discovery, там она не нужна.<br>
</details>

#### Веб интерфейс

Устройствами можно управлять также и в веб интерфейсе шлюза. Примеры главной страницы и страницы настроек ниже на картинках 6 и 7.

![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/myweb.jpg) 
Картинка 6. Главная страничка.

![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/myweb1.jpg) 
Картинка 7. Страничка настроек.
 
## 4. BLE Monitor
&emsp;Монитор позволяет отслеживать метки(маяки) со статическими MAC адресами. Выводится наличие/отсутствие метки(маяка) и rssi.<br>
**Дополнительно поддерживаются:**
* BLE маяки приложения Home Assistant на смартфонах (привязка по uuid, тайм-аут у меня 60 секунд)
* Термометры Xiaomi LYWSD03MMC с оригинальной прошивкой, [прошивкой от atc1441 в режиме custom](https://github.com/atc1441/ATC_MiThermometer) и [прошивкой от pvvx в режиме custom и Mija](https://github.com/pvvx/ATC_MiThermometer). Ключи для LYWSD03MMC оригинальной версии можно взять из [облака Xiaomi](https://github.com/PiotrMachowski/Xiaomi-cloud-tokens-extractor?tab=readme-ov-file). (400 секунд)
* Термометры Xiaomi LYWSDCGQ (400 секунд)
* Часы с термометром Xiaomi LYWSD02 (400 секунд) 
* Весы Xiaomi Mi Scale  (400 секунд)
* Qingping Air Monitor Lite(CGDN1) (400 секунд)
* Счетчики воды и газа Elehant  (400 секунд)
* Samsung Smart Tag (120 секунд).
     
&emsp;Для активации монитора нужно во вкладке **Setting** установить в опции **BLE Monitoring** значения **Active** или **Passive** для активного или пассивного сканирования и нажать **Save setting**. Активный сканер дает больше информации, но расходует больше энергии на сканируемых устройствах. Для сканирования меток рекомендуется пассивный режим. Нужно учитывать, что в режиме **Auto** при поиске устройств перед соединением сканер всегда работает в активном режиме, а потом переходит в пассивный режим.<br> 
&emsp; После установки опции в меню появится вкладка **BLE Monitor**, открыв которую, можно увидеть найденные устройства. В поле **Gap** выводится временной интервал между двумя последними пришедшими пакетами, в поле **Last** время с момента прихода последнего пакета. Тайм-аут по умолчанию (если в поле **Timeout** пусто) 300 секунд, после чего устройство считается потерянным, а его данные удаляются из таблицы. В дальнейшем эта строка может быть перезаписана данными с другого устройства. **Для вывода данных в Mqtt в поле Timeout нужно ввести ненулевое значение и подтвердить ввод, нажав Ok**. Все значения сохранятся в энергонезависимой памяти, а Mqtt Discovery передаст все в Home Assistant. Хотя сканирование идет постоянно, но при установке значения **Timeout** нужно учитывать, что для поддержания соединений тоже нужно время, в течение которого возможны пропуски пакетов. Лишние данные в Mqtt и Home Assistant можно удалить, выбрав при включенном BLE мониторе во вкладке **Setting** опцию **Delete Mqtt topics** и нажав **Save setting**.<br>
&emsp;Samsung Smart Tag, не привязанные к аккаунту SmartThings, для отслеживания не пригодны, так как через несколько минут отключаются. Рекламное сообщение привязанного к аккаунту Smart Tag содержит статический UUID 0xFD5A, динамический MAC адрес и шифрованный идентификатор, из-за наличия в нем RND байт меняющийся вместе с MAC адресом. Остальные поля (статус, счетчик рекламных сообщений, регион, состояние батареи) не уникальны. Рекламное сообщение содержит также цифровую подпись. Стандартные BLE трекеры, насколько мне известно, способны опознать наличие этих меток по UUID, но не способны однозначно идентифицировать каждую метку, если их больше одной. Шлюз использует проверку цифровой подписи рекламного сообщения для идентификации этих меток(маяков), что требует ввода ключа. После ввода ключ проверяется и, если все нормально, нужно ввести значение тайм-аута. Только после ввода тайм-аута и нажатия **Ok** ключ и тайм-аут запоминаются в NVS (Non-volatile storage - энергонезависимая память).
<details>
<summary>Как получить ключ...</summary>

&emsp; Ключ (Signing Key) представляет собой ASCII строку из 64 символов (это 32 байта в шестнадцатеричном виде, 16 байт ключа шифрования AES128CBC и 16 байт начального вектора).  Действует от момента привязки Smart Tag к аккаунту SmartThings и до момента отвязки от аккаунта или возврата устройства к заводским установкам. Генерируется и меткой Smart Tag,  и сервером SmartThings, по каналу связи не передается. Первые 16 байт используются шлюзом как идентификатор устройства в системе умного дома. Для генерации ключа нужно записать лог Bluetooth HCI в момент привязки Smart Tag к аккаунту SmartThings. Если метка уже привязана, перед записью лога ее нужно удалить из аккаунта. Для привязки к аккаунту нужно устройство Galaxy c версией android не ниже 8.0. Я использовал Galaxy S7. Прежде всего нужно включить режим разработчика. Открываем **Настройки > Сведения о телефоне > Сведения о ПО** и 8 (кажется) раз нажимаем номер сборки. Может потребоваться ввести пин код телефона. В настройках должно появиться меню **Параметры разработчика**. Заходим в меню и включаем **Журнал отслеживания Bluetooth**. Далее я на всякий случай выключал и включал Bluetooth, затем перегружал телефон. Заходим в приложение **SmartThings** и добавляем устройство **Smart Tag**. Затем опять идем в **Настройки > Параметры разработчика** и выбираем **Создать отчет об ошибках > Интерактивный отчет**. Через некоторое время придет уведомление о созданном отчете. Далее его нужно сохранить на компьютере с windows. Я выбирал сохранить в приложении telegram в папке **Избранное**, а затем уже в telegram на компьютере сохранил архив. Затем нужно извлечь из архива (папка **FS/data/log/bt/**) файлы **btsnoop_hci.log** и **btsnoop_hci.log.last**. В одном из этих файлов должен быть лог привязки. Далее в папку с этими файлами загружаем архив с [**консольной утилитой stsk**](https://github.com/alutov/ESP32-R4sGate-for-Redmond/raw/master/utils/stsk.zip) и достаем программу из архива. На всякий случай отмечу, что вирусов в архиве нет, а размер программы 28160 байт. Открываем командную строку windows, заходим в папку с файлами и набираем **stsk btsnoop_hci.log** и **stsk btsnoop_hci.log.last**. Программа выведет найденные Smart Tag и сгенерирует ключи для них. Последний ключ будет скопирован в буфер обмена windows:<br>
     
![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/stsk.jpg) 
Картинка 8. Программа stsk.

&emsp;  Программа не обрабатывает разбитые на части пакеты данных, соответственно, если встречает такой пакет, то не находит в логе ничего или выдает ошибку. Столкнулся с этим пока только один раз на s21. Возможно это просто сбой, т.к wireshark тоже не совсем корректно восстановил сбойный пакет. И пока не нашел алгоритма опознания и сборки таких пакетов данных. В этом случае нужно повторить всю процедуру еще раз.<br> 
</details>


![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/blemon1.jpg) 
Картинка 9. Страничка BLE Monitor.
     
&emsp;Добавлено определение лучшего сигнала от метки(маяка) среди нескольких шлюзов. Алгоритм работает так. Каждый шлюз мониторит топик **r4s/DevId/rssi**. Если у него сигнал от метки(маяка)  с большим уровнем, он прописывает свой уровень в этот топик, а также свой номер в топик **r4s/DevId/gtnum**. После чего шлюз периодически, раз в 6 секунд сохраняет свой уровень сигнала в топике, то есть становится ведущим. Остальные шлюзы проверяют уровень и есть ли его обновление. Если какой-то шлюз обнаруживает отсутствие обновления уровня более 30 секунд, или же его уровень больше,  он становится ведущим. Лучший уровень и номер шлюза можно увидеть во второй строчке RSSI на странице BLE монитора. В сущностях каждого устройства тоже есть лучший уровень и номер шлюза:<br>
![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/blemon2.jpg)      
Картинка 10. Сущности метки(маяка) в Home Assistant.

## 5. Поддержка экрана

&emsp;В первой версии шлюза оставался запас как оперативной (~100кБайт), так и программируемой (~400кБайт) свободной памяти, что позволяло расширить возможности прошивки, в частности, добавить поддержку экрана. К тому же у меня уже была собранная esp32 с экраном [3.2" 320x240 на чипе ili9341](https://www.aliexpress.com/item/32911859963.html?spm=a2g0s.9042311.0.0.274233edzZnjSp), работающая с прошивкой с сайта [wifi-iot](https://wifi-iot.com/). Возможно и использование для шлюза уже готовых устройств на чипах ili9341, ili9342 или ST7789. В шлюзе я использовал только необходимые процедуры из [Bodmer](https://github.com/Bodmer/TFT_eSPI), адаптированные не совсем хорошо, но как есть для esp-iot. Пины для поключения экрана по умолчанию: MOSI-23, MISO-25, CLK-19, CS-16, DC-17, RST-18, LED-21. Пины можно переназначить в настройках. Если PWR, RST, LED установить 0, то шлюз эти пины использовать не будет. Есть также опция поворота экрана на 180&deg;, а также возможность регулировки яркости дисплея по Mqtt, иcпользуя топик **r4s/screen**. Программа проверяет наличие экрана на шине SPI при старте. Предусмотрена возможность вывода на экран и картинки в формате jpeg. Для этого нужно указать url картинки. У моей камеры url такой: **http://192.168.1.7/auto.jpg?usr=admin&pwd=admin**. Картинка грузится в буфер размером 20-65 килобайт в оперативной памяти. Время обновления и размер буфера можно установить в настройках. Есть возможность загрузки картинки по https. Проверка сертификата отключена. Есть возможность управления параметрами загрузки до 4 картинок по Mqtt, используя топики **r4sx/jpg_url1...r4sx/jpg_url4** и **r4sx/jpg_time**. Для очистки url картинки в соответствующий топик нужно ввести символ **#**. Если в Mqtt эти топики не прописаны, а также после сохранения настроек, эти параметры копируются из настроек в Mqtt. Установка нулевого интервала обновления возвращает кота на экран. Длина буфера ссылки пока 384 байта. Добавлены загрузка и отображение в текстовом виде погоды с сайта wttr.in. В принципе, это может любой сайт, отдающий текст и допускающий форматирование. Если ссылка не содержит строки **http://** или **https://**, то шлюз считает это сообщение простым текстом и отображает на экране. Доступно 2 шрифта и 10 вариантов цветов. Управляющие символы: \ \ или \n - перевод строки, \F  - шрифт 26 пикселей и перевод строки, \f - шрифт 16 пикселей и перевод строки, \0 ... \9 - цвета. Поддерживается кириллица, проверял, правда, только из mosquitto. Он поддерживает юникод по факту, как другие брокеры, не знаю. Пример вывода изображения на экран на картинке 11.
<br>

![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mytft3.jpg)
Картинка 11. Изображение.<br><br>

&emsp;На картинке 12 пример вывода на экран Mqtt погоды с сайта wttr.in:
<td>https://wttr.in/Донецк?format=\F\6+%25l%20\\\4Темп:+\0%25t(%25f)\\\4Давл:\0+%25P\\\4Влажн:\0+%25h\\\6+%25c+%25w+UV:+%25u\f\4Восход:\0+%25D+\4Закат:\0+%25d</td><br> 

![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mytft10.jpg)
Картинка 12. Погода.<br><br>

&emsp;На картинке 13 пример вывода на экран Mqtt строки (символ градуса можно вывести на экран используя обратный апостроф):
<td>\F\0` English \1color \2text\3 example\n\4Русский \5цветной \6текст\n\7text1 \8text2 \9text3\f\0` English \1color \2text\3 example\n\4Русский \5цветной \6текст\n\7text1 \8text2 \9text3</td><br>    
  
![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mytft9.jpg)
Картинка 13. Текст.<br><br>

&emsp; Стоит отметить, что сама TFT плата влияет на распространение как WiFi, так и BLE. И даже если антенна esp32 выглядывает из-под экрана, чувствительнось такого бутерброда заметно меньше обычной esp32. Рекомендую использовать с экраном вариант esp32 с внешней антенной. У меня в шлюзе с экраном замена esp32 на вариант с разъемом и установка внешней антенны дала прирост уровней WIFI и BLE примерно на 15-20dBm.<br>
&emsp; Если же экран не нужен, то нужно после программирования и настройки  esp32 подсоединить ее к источнику питания и спрятать где-нибудь на кухне.<br>

## 6. Совместимые устройства
Если хочется запустить шлюз максимально быстро, без пайки, да еще и с приличным корпусом, стоит присмотреться к совместимым устройствам. Их нужно только перепрограммировать. Ниже перечислены только проверенные мной устройства. Для прошивки использовалась программа [flash_download_tools](https://www.espressif.com/en/support/download/other-tools).

#### [TTGO T-Watcher](http://www.lilygo.cn/prod_view.aspx?TypeId=50033&Id=1160) (LILYGO® TTGO T4 в корпусе).<br>
![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mytft4.jpg)
<br>Картинка 14. TTGO T-Watcher.<br><br>
Я проверял работоспособность шлюза на TTGO T4 версии 1.3. Прошивается он через встроенный USB разъем, перед прошивкой устройства нужно соединить контакты 6 и 7 (gpio0 и gnd) в нижнем ряду разъема (картинка 15). Возможна прошивка и без установки перемычек, зависит от программы. Настройки экрана для версии 1.3: 12-MISO, 23-MOSI, 18-CLK, 27-CS, 32-DC, 5-RES, 4-LED, 0-PWR, и для версии 1.2: 12-MISO, 23-MOSI, 18-CLK, 27-CS, 26-DC, 5-RES, 4-LED, 0-PWR. В версии 1.2 нет управления включением и выключением экрана. Кнопки сверху вниз 38-Port1, 37-Port2, 39-Port3. Шина I2C: SCL-22, SDA-21.<br>
![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mytft5.jpg)
<br>Картинка 15. Соединить 6 и 7 пины разъема перед прошивкой TTGO T-Watcher.<br><br>

#### [M5Stack BASIC Kit](https://docs.m5stack.com/en/core/basic)<br>
![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mytft6.jpg)
<br>Картинка 16. M5Stack BASIC Kit.<br><br>
Как я понял, старые версии M5Stack Basic шли с экраном на ili9341, и на этих версиях [работала и старая версия шлюза](https://github.com/alutov/ESP32-R4sGate-for-Redmond/issues/16). 
Настройки экрана для этой версии: 19-MISO, 23-MOSI, 18-CLK, 14-CS, 27-DC, 33-RES, 32-LED, 0-PWR. Новые версии уже идут с экраном на ili9342. Начиная с версии 2021.10.29 добавлена поддержка экрана на ili9342. Я проверял работоспособность шлюза на новой версии M5Stack BASIC Kit. Прошивается он через встроенный USB разъем, перед прошивкой устройства нужно соединить последний контакт в верхнем ряду и 4 в нижнем ряду (gnd и gpio0) разъема (картинка 17). Возможна прошивка и без установки перемычек, зависит от программы. Настройки экрана для новой версии: 23-MISO, 23-MOSI, 18-CLK, 14-CS, 27-DC, 33-RES, 32-LED, 0-PWR. Кнопки слева направо 39-Port1, 38-Port2, 37-Port3. Шина I2C: SCL-22, SDA-21.<br>
Настройки для [**M5Stack Tough**](https://docs.m5stack.com/en/core/tough): 23-MISO, 23-MOSI, 18-CLK, 5-CS, 15-DC, 44-RES, 47-LED, 46-PWR. Шина I2C: SCL-22, SDA-21. Без I2C экран не запустится.<br>  
     
![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mytft7.jpg)
<br>Картинка 17. Соединить последний контакт в верхнем ряду и 4 в нижнем ряду (gnd и gpio0) перед прошивкой M5Stack BASIC Kit.<br><br>

#### [ATOM-LITE-ESP32-DEVELOPMENT-KIT](https://docs.m5stack.com/en/core/atom_lite)<br>
![PROJECT_PHOTO](https://github.com/alutov/ESP32-R4sGate-for-Redmond/blob/master/jpg/mytft8.jpg)
<br>Картинка 18. ATOM-LITE-ESP32-DEVELOPMENT-KIT.<br><br>
     Прошивается атом по usb без установки перемычек. Кнопку использовал для включения-выключения одного из устройств (39-Port1), светодиод пока в прошивке не задействован. IR LED на 12 gpio можно использовать для дистанционного управления. Устройство компактное (24 * 24 * 10 mm), devkit esp32 по размерам больше.
     
     
## 7. Сборка проекта и лицензия
&emsp; Для сборки бинарных файлов использовал [Espressif IoT Development Framework.](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/). Добавлена лицензия MIT. Добавлена конфигурация для сборки в среде PlatformIO, спасибо [bvp](https://github.com/bvp),  его сообщение [здесь](https://github.com/alutov/ESP32-R4sGate-for-Redmond/pull/89).
<details>
<summary>Подробнее...</summary>

## PlatformIO
Это платформа для сборки прошивок для микроконтроллеров. Управляет инструментарием сборки, и зависимостями проекта. Всё нужное скачает сама. Со списком поддерживаемых платформ можно ознакомиться [тут](https://registry.platformio.org/search?t=platform), а фреймворков - [тут](https://registry.platformio.org/search?t=tool&q=keyword%3Aframework).

platformio.ini - файл конфигурации для PlatformIO
Собрать так - `pio run -t build` или просто `pio run`
Загрузить прошивку - `pio run -t upload`
Потребуется только поправить `upload_port` и `monitor_port`.
Для Win32 значение будет вида `COM4` (поставить свой номер порта, на котором находится прошивальщик).
Для Linux - будет `/dev/ttyUSB0` (так же поставить свой номер порта, на котором находится прошивальщик).
Для macOS - как в прилагаемом примере.

## Clang-format
В файле описываются правила форматирования кода, согласно которым код приводится к нужному стилю. Необходим установленный `clang-format`.     
</details>

<br>
