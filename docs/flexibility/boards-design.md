# Flexibiliteit

## Introductie

Eén van de hoofddoelen van dit **Flexible LoRaWAN Sensor Board** project is een zo flexibel mogelijk modulair systeem bekomen. Hiervoor moet er rekening gehouden worden met dat er op elk afzonderlijk FLWS-board andere sensoren kunnen worden aangesloten. De metingen van deze sensoren moeten vervolgens draadloos verzonden worden aan de hand van LoRaWAN. Dit protocol verzend data geformatteerd in een bitstream. Hierbij moet er ook in acht genomen worden dat dit een batterij gevoede toepassing is. Elke extra bit die verzonden moet worden kost stroom en moet dus tot een minimum beperkt worden.
Om deze vraag te beantwoorden is er een modulair hardware design uitgewerkt samen met het **Sensor Identification System (SIS)**.

---

## Boards Design

Om tot een flexibel en modulair hardware systeem te komen is er gekozen om een opdeling te maken in twee boards. Een hoofd, of main, board die de microcontroller en LoRa radio module bevat, en een tweede connectie, of connector, board waar alle andere componenten op kunnen worden aangesloten.

Om deze twee boards van voeding te voorzien, aan de hand van batterij en zonne-energie, wordt er nog een derde Solar Power Manager board aangekocht.

In dit deel worden de ontwerp beslissingen rond deze drie boards besproken.

1. Main Board met LoRaWAN (SAMDaaNo21)
2. Connector Board
3. Solar Power Manager

Een behuizing hoort niet tot deze iteratie van het project en er zijn dus ook geen vereisten voor opgelegd.
In het hoofdstuk [Enclosure, of Behuizing](./enclosure/considerations.md), is wel een kijk genomen naar enkele overwegingen die in acht kunnen worden genomen. Er moet namelijk toch over nagedacht worden en rekening mee gehouden worden bij het ontwerpen.

#### Blokdiagram

![FLWSB Connector Board blokdiagram v1.0](./assets/boards-design-blockdiagram.jpg 'Figuur 1: Overzicht blokdiagram van de drie boards.')

*Note: in het ontwerp moet rekening gehouden worden met mechanische sterkte. De SAMDaaNo21 wordt gevoed via USB kabel. Deze kan echter loskomen. Dit moet voorkomen worden door het design.*

---

### Main Board met LoRaWAN (SAMDaaNo21)

Het FLWSB Main Board is een microcontroller bord gebaseerd op de ATSAM D21 in de vormfactor vergelijkbaar met een Arduino Nano. Het moet gemakkelijk zijn om dit bord in een breadboard te prikken en zo testen uit te voeren of schakelingen te bouwen.

De LoRa radio module zit hierop reeds geïntegreerd om zo de hoofdfunctie van het verzenden over LoRaWAN centraal te plaatsen.

#### Blokdiagram

![Blokdiagram werking SAMDaaNo21](../schematics/assets/blokdiagram.svg 'Figuur 2: Blokdiagram werking SAMDaaNo21.')

---

### Connector Board

Het connector board dient als verbindingspunt voor de SAMDaaNo21. Het is de tussenschakel om allerlei onderdelen te kunnen verbinden met de SAMD21 microcontroller. De focus ligt hier ook op fast prototyping en flexibiliteit. Dit wilt zeggen dat er enkel connectoren voorzien worden en geen sensoren rechtstreeks op dit bord zullen komen.

#### Veresiten

##### Te connecteren onderdelen

- SAMDaaNo21 moederbord
- Zonne-energie (Solar Power) manager voor continu gebruik tussen batterij en zonnepaneel.
- Sensoren:
  - [CCS811](https://www.sciosense.com/products/environmental-sensors/ccs811/): TVOC sensor met I2C op 1V8-3V3
  - [SDS011](https://www.tinytronics.nl/shop/nl/sensoren/lucht/stof/nova-sds011-hoge-precisie-laser-stofsensor): fijnstof sensor met UART 3V3 op 5V
  -  [BME280](https://www.bosch-sensortec.com/products/environmental-sensors/humidity-sensors-bme280/): temperatuur, barometer en luchtvochtigheid sensor met I2C op 1V8-3V3
  - ([DHT22](https://www.tinytronics.nl/shop/nl/sensoren/lucht/vochtigheid/dht22-thermometer-temperatuur-en-vochtigheids-sensor): temperatuur en luchtvochtigheid sensor alternatief met Digitale I/O op 3V3-6V)

##### Uitbreidingsmogelijkheden

- 2x Extra I2C 3V3 met 3V3 voeding
- 1x Extra I2C met 5V voeding
- 1x Extra UART 3V3 met 3V3 voeding
- Mogelijke sensoren:
  - [SCD41](https://sensirion.com/products/catalog/SCD41/): CO2 met I2C op 3V3-5V
  - [SGP41](https://sensirion.com/products/catalog/SGP41/): TVOC en NOx met I2C op 1V8-3V3
  - [SPS30](https://sensirion.com/products/catalog/SPS30/): HVAC PM1.0, PM2.5, PM4 en PM10 met I2C en UART op 5V
  - [GY-NEO6MV2](https://www.tinytronics.nl/shop/nl/communicatie-en-signalen/draadloos/gps/modules/gy-neo6mv2-gps-module): GPS met UART 3V3 op 3V3-5V


#### Sensoren

Sensor analyse uit [AirQualitySensor documentatie](https://ap-it-gh.github.io/ssys21-docs-luchtsensor/#/./pagina/Hardware/PCB) (DHT22 toegevoegd):
<table>
    <tbody><tr>
        <th>Naam</th>
        <th width="40%">Eigenschappen</th>
        <th width="30%">Argumentatie</th>
        <th>Links</th>
    </tr>
    <tr>
        <td>CCS811</td>
        <td>
            <ul>
                <li><b>Temperatuur, eCO en eTVOC<sup>2</sup></b></li>
                <li>V<sub>cc</sub> = 1.8V-3.3V</li>
                <li>I<sub>max</sub> = 54mA</li>
                <li><a target="_blank" href="https://en.wikipedia.org/wiki/I%C2%B2C">I²C</a> protocol 3.3V</li>
                <li>Meetbereik temperatuur: -40°C ~ +85°C</li>
                <li>Meetbereik luchtvochtigheid: 10% ~ 95% </li>
                <li>Meetbereik eCO²: 400ppm<sup>1</sup> ~ 32768ppm</li>
                <li>Meetbereik eTVOC: 0ppb<sup>5</sup> ~ 29206ppb</li>
                <li>Leessnelheid: 100kHz (0.01ms)</li>
            </ul>
        </td>
        <td>Een minder bekende sensor maar zeker wel bekend in IoT toepassingen. Het kan temperatuur, eCO² en eTVOC meten. Het heeft geen opwarm tijd dus is direct bruikbaar en is ook een SMD component wat zeker een voordeel is om het zo compact mogelijk te maken.</td>
        <td>
            <a target="_blank" href="https://www.tinytronics.nl/shop/nl/sensoren/ccs811-luchtkwaliteit-sensor">Winkel</a><br>
            <a target="_blank" href="https://www.sciosense.com/wp-content/uploads/documents/SC-001232-DS-2-CCS811B-Datasheet-Revision-2.pdf">Datasheet</a><br>
            <a target="_blank" href="https://github.com/adafruit/Adafruit_CCS811">Bibliotheek</a>
        </td>
    </tr>
    <tr>
        <td>SDS011</td>
        <td>
            <ul>
                <li><b>Fijnstof</b></li>
                <li>V<sub>cc</sub> = 5V</li>
                <li>I<sub>rated</sub> = 70mA ±10mA</li>
                <li><a target="_blank" href="https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter">UART</a> protocol 3.3V</li>
                <li>Baudrate: 9600</li>
                <li>Meetbereik PM2.5<sup>3</sup> &amp; PM10<sup>4</sup>: 0μg/m3 ~ 999.9 μg/m3</li>
                <li>Meetbereik luchtvochtigheid: 10% ~ 95% </li>
                <li>Leessnelheid: 1Hz (1s)</li>
            </ul>
        </td>
        <td>De SDS011 is een veel gebruikte fijnstof sensor voor DIY-projecten. Het zal niet de nauwkeurigste zijn, maar het geeft toch al een sterke indicatie van wat het fijnstof gehalte is in de lucht. Daarnaast het een goedkope modele.<br><br>
        Het werkt met een ventilator die de lucht binnentrekt. Het zal dus eerst moeten opgezet wordne om de huidige lucht erin te trekken vooraleer we kunnen meten.</td>
        <td>
            <a target="_blank" href="https://www.tinytronics.nl/shop/nl/sensoren/nova-sds011-hoge-precisie-laser-stofsensor">Winkel</a><br>
            <a target="_blank" href="https://cdn-reichelt.de/documents/datenblatt/X200/SDS011-DATASHEET.pdf">Datasheet</a><br>
            <a target="_blank" href="https://www.arduinolibraries.info/libraries/sds011-sensor-library">Bibliotheek</a>
        </td>
    </tr>
    <tr>
        <td>BME280</td>
        <td>
            <ul>
                <li><b>Temperatuur, Barometer &amp; Luchtvochtigheid</b></li>
                <li>V<sub>cc</sub> = 1.8 - 3.6V</li>
                <li>I<sub>max</sub> = 4.5mA</li>
                <li><a target="_blank" href="https://en.wikipedia.org/wiki/I%C2%B2C">I²C</a> protocol 3.3V</li>
                <li>Meetbereik temperatuur: -40°C ~ +85°C</li>
                <li>Meetbereik luchtvochtigheid: 0% ~ 100%</li>
                <li>Meetbereik luchtdruk: 300hPa ~ 1100hPa</li>
                <li>Leessnelheid: 1Hz (1s)</li>
            </ul>
        </td>
        <td>Deze digitale IC heeft een tal van metingen aanboord (temperatuur, luchtvochtigheid en druk). Het is ook een SMD component, dus makkelijk integreerbaar op een pcb.</td>
        <td>
            <a target="_blank" href="https://www.tinytronics.nl/shop/nl/sensoren/temperatuur-lucht-vochtigheid/bme280-digitale-barometer-druk-en-vochtigheid-sensor-module">Winkel</a><br>
            <a target="_blank" href="https://www.mouser.com/datasheet/2/783/BST-BME280_DS001-11-844833.pdf">Datasheet</a><br>
            <a target="_blank" href="https://github.com/adafruit/Adafruit_BME280_Library">Bibliotheek</a>
        </td>
    </tr>
    <tr>
        <td>DHT22</td>
        <td>
            <ul>
                <li><b>Temperatuur, Luchtvochtigheid</b></li>
                <li>V<sub>cc</sub> = 3.3-6V DC</li>
                <li>I<sub>max</sub> = ? mA</li>
                <li>Digital signal via single-bus (zie datasheet)</li>
                <li>Meetbereik temperatuur: -40°C ~ +80°C</li>
                <li>Meetbereik luchtvochtigheid: 0% - 100%</li>
                <li>Leessnelheid: 2Hz (~2s)</li>
            </ul>
        </td>
        <td>Een handige temperatuur én vochtigheid sensor.</td>
        <td>
            <a target="_blank" href="https://www.tinytronics.nl/shop/nl/sensoren/lucht/vochtigheid/dht22-thermometer-temperatuur-en-vochtigheids-sensor">Winkel</a><br>
            <a target="_blank" href="https://www.tinytronics.nl/shop/index.php?route=product/product/get_file&file=136/DHT22.pdf">Datasheet</a><br>
            <a target="_blank" href="">Bibliotheek</a>
        </td>
    </tr>
</tbody></table>

---

### Voeding

De flexibiliteit wordt ook doorgevoerd in de voeding van de schakeling.
Er is gekozen voor een voeding op batterij gecombineerd met zonnepanelen.
Zo kan de schakeling op batterij functioneren wanneer er geen zonlicht is, en wanneer die er wel is kan de voeding over gaan op zonne-energie waarmee ook de batterij terug wordt opgeladen.

#### Stroomverbruik

Stroomverbruik per onderdeel en maximum wanneer alles aan is.

| Component | Stroomverbruik in operatie | Start-up | Meet periode | Stroomverbruik standby |
| --- | --- | --- | --- | --- |
| ATSAMD21G18A | I = 1 - 6 mA | n/a | n/a | I<sub>25-85°</sub> = 2.70 - 55.2µA |
| LDL1117S33R | ? | n/a | n/a | I<sub>Q typ.</sub> = 250µA
| CCS811  | I<sub>DD</sub> = 30mA , I<sub>DD peak</sub> = 54mA | 18 - 20ms | ? | I<sub>DD</sub> = 19µA |
| SDS011  | I<sub>rated</sub> = 70mA ±10mA | ? | <10s + 1s read | I < 4mA |
| BME280  | I<sub>DD H/P/T (1V8)</sub> = 340 / 714 / 350µA , I<sub>weather monitoring mode</sub> = 0.16µA | 2ms | <1s | I<sub>DD SB (1V8-3V6)</sub> = 0.2 - 0.5µA |
| DHT22  | I<sub>measuring (3V3)</sub> = 1mA | 1s | >2s | I<sub>stand-by (3V3)</sub> = 40µA |

*I<sub>Q</sub> = [Quiescent current](https://forum.digikey.com/t/what-is-quiescent-current-and-why-is-it-important/3894)*

I<sub>max. totaal</sub> = 6mA + (250µA * 2) + 54mA + 80mA + 714µA + 1mA

__I<sub>max. totaal</sub> = 142.214 mA__

---

#### Zonne-energie manager

Eén van de doelen van het connector board is om continue stroom te kunnen voorzien. Dit door te kunnen schakelen tussen een oplaadbare batterij en een zonnepaneel die da batterij ook kan opladen.
Verder moet het ook voldoende spanning en stroom kunnen voorzien. De SAMDaaNo21 beschikt reeds over een 3V3 lineaire regelaar die gevoed kan worden via een VIN pin waarop ook een 5V USB-C poort zit. De zonne-energie manager moet best 5V kunnen leveren om sensoren op 5V ook te kunnen voeden en ook de VIN van de SAMDaaNo21.

Er wordt gekozen voor de DFRobot Zonne-energie Manager 5V v1.1 DFR0559.
Deze laat toe een heroplaadbare batterij op te laden

![DFROBOT-DFR0559 foto](./assets/DFROBOT-DFR0559/DFR0559-photo.jfif 'Figuur 3: Foto van de DFROBOT-DFR0559.')

##### Eigenschappen

- Model: DFROBOT-DFR0559
- Solar Power Management IC: CN3165
- Solar Input Voltage (SOLAR IN): 4.5V ~ 6V
  - Maximum open circuit voltage < 6.5V
- Battery Input (BAT IN): 3.7V Single cell Li-polymer/Li-ion Battery
- Charge Current(USB/SOLAR IN):
  - 900mA Max trickle charging
  - constant current
  - constant voltage three phases charging
- Charging Cutoff Voltage (USB/SOLAR IN): 4.2V ±1%
- Regulated Power Supply: 5V 1A
- Voltage Regulator IC: MT3608 DC-DC Boost 5V
- Regulated Power Supply Efficiency (3.7V BAT IN): 86% @ 50%Load
- USB/Solar Charge Efficiency: 73% @ 3.7V 900mA BAT IN
- Quiescent Current: <1 mA
- Operation Temperature: -40℃ ～ 85℃
- Dimension: 33.0mm x 63.0mm
- Protection Functions:
  - BAT IN: over charge/over discharge/over current/reverse connection protection
  - 5V/USB OUT: short circuit/over current/over heat protection
  - SOLAR IN: reverse connection protection
- Prijs: €9,50 per stuk op [TinyTronics](https://www.tinytronics.nl/shop/nl/power/bms-en-laders/zonneladers/dfrobot-zonne-energie-manager-5v)
- [DFRobot webshop pagina](https://www.dfrobot.com/product-1712.html)
- [Documentatie](https://wiki.dfrobot.com/Solar_Power_Manager_5V_SKU__DFR0559)
- [Schema](./assets/DFROBOT-DFR0559/DFR0559-schematic.pdf)

##### Features

- Constant voltage MPPT algorithm, maximizing solar panel efficiency
- Designed for 5V solar panel
- Double charging mode: solar/USB charger (900mA max charge current)
- 5V ON/OFF controllable regulated power supply for low-power applications
- All-round protection functions
- USB connector with ESD shell

![DFROBOT-DFR0559 overzicht](./assets/DFROBOT-DFR0559/DFR0559-overview.jpg 'Figuur 4: DFROBOT-DFR0559 overzicht')

![DFROBOT-DFR0559 connecties](./assets/DFROBOT-DFR0559/DFR0559-connections.jpg 'Figuur 5: DFROBOT-DFR0559 connecties')

##### ON/OFF functie

Een zeer handige functie waarover de DFROBOT-DFR0559 beschikt is de mogelijkheid om de 5V output pins aan en uit te schakelen via een enable pin (EN). De USB output blijft hierbij wel nog 5V krijgen. Zo kan een MCU board ononderbroken gevoed worden via USB en via een GPIO pin de enable aansturen en de stroomvoorziening naar sensoren controleren. Via de BAT pin kan ook de batterij capaciteit uitgelezen worden met een analoge pin/ADC.

Hieronder een voorbeeld uit de [documentatie](https://wiki.dfrobot.com/Solar_Power_Manager_5V_SKU__DFR0559):

> This application example use the BME280 environmental sensor to record temperature, humidity and atmospheric pressure, VEML7700 ambient light sensor to record the the ambient illumination, and DS1307 RTC module to record time. Use the analog input A1 to monitor the battery voltage (equivalently the battery capacity). To achieve lower power consumption, use one Arduino digital IO pin to turn ON the power supply, read all the data from the sensors and turn them OFF. Cycle this pattern for a proper interval T to reduce the average power consumption. This can completely get rid of the quiescent power of the peripheral modules. Although single peripheral may consume little power, it can be considerable large for a number of them. This module provides users with effective methods to drive the peripheral modules into discontinuous (pulse) mode to achieve low power operation.
![DFROBOT-DFR0559 application example](./assets/DFROBOT-DFR0559/DFR0559-application-example.png 'Figuur 6: DFROBOT-DFR0559 application example')

--
