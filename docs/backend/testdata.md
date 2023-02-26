# Backend

## Testdata

Om de backend te testen moet deze gevuld worden met testdata.
De AI studenten zijn modellen aan het trainen met [bestaande historische weerdata uit Duitsland](https://figshare.com/articles/dataset/PPNN_full_data_feather_format_/13516301/1). Ze baseren zich op de paper [Neural Networks for Postprocessing Ensemble Weather Forecasts](https://journals.ametsoc.org/view/journals/mwre/146/11/mwr-d-18-0187.1.xml) door Stephan Rasp and Sebastian Lerch. Om deze testdata in de InfluxDb database te krijgen wordt gebruik gemaakt van onderstaand [Python script](./assets/InfluxDBConnect.py). In dit gedeelte wordt de werking van dit script uitgelegd.

```python
from datetime import datetime
import influxdb_client, os, time
from influxdb_client import InfluxDBClient, Point, WritePrecision
from influxdb_client.client.write_api import SYNCHRONOUS
from influxdb import DataFrameClient
import pandas as pd
import numpy as np

token = ''
org = "AP"
url = "http://localhost:8086"
bucket = "weather-ap"


client = influxdb_client.InfluxDBClient(url=url, token=token, org=org)
columns = list(data.columns)
columns.remove("date")
write_api = client.write_api(write_options=SYNCHRONOUS)

data = pd.read_feather("../data/data_RL18.feather")
data.station = pd.to_numeric(data.station, downcast = 'integer')

# drop soil moisture predictions due to missing values
# Note that self is a minor change compared to the paper, but does not have a significant effect
data = data.drop(['sm_mean', 'sm_var'], axis=1)

data_frame = data
data_frame = data_frame[(data_frame['date'].dt.year==2015)&(data_frame['station']==10)]

data_frame.index = data_frame['date']
# DataFrame must have the timestamp column as an index for the client.
write_api.write(bucket = bucket,record= data_frame,data_frame_measurement_name='obs',data_frame_tag_columns=columns)


my_token = ''
my_org = "AP"
url = "http://localhost:8086"
bucket = "weather-ap"
query= '''
from(bucket: "weather-ap")
|> range(start:-15y, stop: now())
|> filter(fn: (r) => r._measurement == "obs")
|> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
'''


client = InfluxDBClient(url="http://localhost:8086", token=my_token, org=my_org, debug=False)
system_stats = client.query_api().query_data_frame(org=my_org, query=query)
print(system_stats.head())
```

### Python en InfluxDb

InfluxDB bestaat uit buckets, die aangesproken kunnen worden met organisaties en tokens.

Met de python library influxdb_client kan op deze manier connectie gemaakt worden met een bestaande influxdb instance.

```python
import influxdb_client
from influxdb_client import InfluxDBClient
from influxdb_client.client.write_api import SYNCHRONOUS
```

### Dataframes inladen

[pandas.read_feather](https://pandas.pydata.org/docs/reference/api/pandas.read_feather.html)

[pandas dataframe](https://pandas.pydata.org/docs/user_guide/dsintro.html#dataframe)

Met pandas kunnen verschillende databestanden ingeladen worden in dataframes. Hierna kan het dataframe omgevormd worden tot het gewenste formaat, waarna het onder eender welk bestandstype opgeslagen kan worden of kan het bvb naar een database gestuurd worden.
De dataframe wordt hier toegewezen aan de 'data' variabele.

Vanuit feather:

```
data = pd.read_feather("../data/data_RL18.feather")
data.station = pd.to_numeric(data.station, downcast = 'integer')
data = data.drop(['sm_mean', 'sm_var'], axis=1)
```

Enkele veelgebruikte functionaliteiten:

```
data.columns --> returnt array van alle kolomnamen in de dataframe

data["column_name"] --> selecteren van enkele kolom

data.count() --> geeft overzicht van het aantal elementen in pandas object

data.isna() --> returnt pandas object waar elke waarde vervangen is door True of False, naargelang dit een missende waarde is of niet

data[[condition1==True&&condition2==False]] --> conditionele selectie van waarden uit pandas object

data.mean() --> gemiddelde van waarden in dit object, 1 van de verschillende aggregatiefuncties.
```

### Schrijven naar InfluxDB

De tijdsgebonden kolom wordt ingesteld als index (vb. date), en er wordt een \_measurement aangemaakt, die dan bestaat uit meerdere \_fields. Deze \_fields worden gemapt aan de kolommen van de dataframe, en bevatten data voor alle gewenste features binnen een bepaalde tijdsperiode.

De token en organisatie worden mee gegeven en met een write-api kan zo tijdgebonden data doorgestuurd worden naar de bucket. In het geval van een dataframe, moet deze eerst nog omgevormd worden om door influxdb geaccepteerd te worden.

```python
client = influxdb_client.InfluxDBClient(url=url, token=token, org=org)
columns = list(data_frame.columns)
columns.remove("date")
write_api = client.write_api(write_options=SYNCHRONOUS)
data_frame.index = data_frame['date']

write_api.write(bucket = bucket,record= data_frame,data_frame_measurement_name='obs',data_frame_tag_columns=columns)
```

### Lezen uit InfluxDB

Met behulp van de [Flux](https://fluxcd.io/flux/) syntax, kunnen queries opgesteld worden om data uit een influxDB op te halen. Er wordt een tijdsrange ingesteld, alsook de measurements en fields die opgehaald moeten worden. Vervolgens kan dan opnieuw met influxdb_client een query gedaan worden naar de ifdb instance.

```python
query= '''
from(bucket: "weather-ap")
|> range(start:-15y, stop: now())
|> filter(fn: (r) => r._measurement == "obs")
|> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
'''


client = InfluxDBClient(url="http://localhost:8086", token=my_token, org=my_org, debug=False)
system_stats = client.query_api().query_data_frame(org=my_org, query=query)
```

### Genereren van nieuwe testdata

Door technische problemen met de huidige testomgeving lukte het niet om de testdata in te laden, de testdata moest dus handmatig in een database gezet worden. Dit werd gedaan door een Python-script dat random getallen genereerd en dan naar een bestand schrijft, dit bestand moet dan via de InfluxDB UI ingegegven worden. De random getallen zijn gebaseerd op realistische waarden van temperatuur, luchtdruk, luchtvochtigheid, etc ...

```python
import random
from datetime import datetime, timedelta
filename = "testdata.csv"
file = open(filename, "w")
sb = ""
for i in range(10000):
    temp = round(24 + random.uniform(0, 1) * (28 - 24), 1)
    humidity = random.randint(75, 83)
    wind_speed = round(1 + random.uniform(0, 1) * (4 - 1), 1)
    wind_gusts = round(3 + random.uniform(0, 1) * (8 - 3), 1)
    wind_direction = random.randint(50, 100)
    rain = round(50 + random.uniform(0, 1) * (100 - 50), 1)
    currentTime = datetime.now()
    prevTime = currentTime - timedelta(seconds=(10000 - i))
    epochSeconds = prevTime.timestamp()
    timestamp = str(int(epochSeconds)) + "000000000"
    line = f"TestMeasurement temp={temp},humidity={humidity},wind_speed={wind_speed},wind_gust={wind_gusts},wind_direction={wind_direction},rain={rain} {timestamp}\n"
    sb += line
file.write(sb)
```

Dit script genereerd een .csv bestand dat 10000 lijnen testdata bevat. Hoe de timestamp gegenereerd wordt zal nog aangepast moeten worden zodat de data meer verspreid is, momentaal maakt het script data dat om de seconde verandert, dit is niet realistisch (maar momenteel voldoende om verder mee te werken voor het dashboard).

Om dit bestand dan in te laden zal in de InfluxDB UI bij "Line protocol" het bestand moeten uploaden.

![InfluxDB Load Data.](./assets/influxdb-ui-load-data.png "Figuur 1: Het inladen van data met InfluxDb.")

![InfluxDB Line Protocol.](./assets/influxdb-ui-line-protocol.png "Figuur 2: Selecteren van line protocol.")

![InfluxDB Select File.](./assets/influxdb-ui-select-file.png "Figuur 3: De file uploaden naar de database.")
