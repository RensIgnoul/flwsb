# Frontend

## Weather Station Dashboard

Het dashboard voor het weerstation bestaat uit verschillende time-series tabellen om de metingen te visualiseren. Er wordt ook een kleine geomap getoont dat de uiteindelijke locatie van het weerstation laat zien en er zal ook een tabel zijn met alle weerstations (en mogelijk extra info).

Er zal met een Python-script een API-call gestuurd worden naar de Grafana-API met alle nodige gegevens om het dashboard te genereren. Het script is vrij simpel en stuurt enkel wat json naar de API.

```python
import requests
import json

# Define your Grafana API endpoint, API key, and InfluxDB datasource URL
GRAFANA_API_URL = 'http://localhost:3000/api/dashboards/db'
GRAFANA_API_KEY = 'eyJrIjoieTZCQTdtNzhRMEdpUDFpdWJ5d2JHSEU1ckRYY0FsbnoiLCJuIjoidGVzdGtleSIsImlkIjoxfQ=='
INFLUXDB_URL = 'http://localhost:8086'
INFLUXDB_BUCKET = 'TestDataBucket'
INFLUXDB_MEASUREMENT = 'TestMeasurement'

# Define the panels to include in the dashboard
PANELS = [
    {
	'title': 'Weather Station Location',
	'type': 'geomap',
	'query': '''from(bucket: "TestDataBucket2")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurement")
  |> filter(fn: (r) => r["_field"] == "longitude" or r["_field"] == "latitude")
  |> filter(fn: (r) => r["id"] == "0")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
	'w': 5,
	'x': 0,
	'y': 0,
"fieldConfig": {
    "defaults": {
      "custom": {
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        }
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "color": {
        "mode": "thresholds"
      }
    },
    "overrides": []
  },
    },
    {
	'title': 'Weather Stations',
	'type': 'table',
	'query': '''from(bucket: "TestDataBucket2")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurement")
  |> filter(fn: (r) => r["_field"]=="name")
  |> group(columns: ["name"])
  |> keep(columns: ["id", "_value"])''',
	'w':19,
	'x':5,
	'y':0,
"fieldConfig": {
    "defaults": {
      "custom": {
        "align": "auto",
        "displayMode": "auto",
        "inspect": False,
        "filterable": False
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "color": {
        "mode": "thresholds"
      }
    },
    "overrides": []
  },
    },
    {
        'title': 'Temperature',
	'type': 'timeseries',
        'query':'''from(bucket: "TestDataBucket2")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurement")
  |> filter(fn: (r) => r["_field"] == "temp")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'y_axis_label': 'Temperature (C)',
	'w': 8,
	'x': 0,
	'y': 8,
        'fillOpacity': 10,
        'fixedColor': 'light-yellow',
  "fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic",
        "fixedColor": "light-yellow"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Weather Station ${__field.labels.source}",
      "unit": "celsius"
    },
    "overrides": []
  },
    },
    {
        'title': 'Rain',
	'type': 'timeseries',
        'query': '''from(bucket: "TestDataBucket2")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurement")
  |> filter(fn: (r) => r["_field"] == "rain")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'y_axis_label': 'Rain (mm)',
	'w': 8,
	'x': 8,
        'y': 8,
	'fillOpacity': 10,
        'fixedColor': 'blue',
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic",
        "fixedColor": "blue"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Weather Station ${__field.labels.source}",
      "unit": "lengthmm"
    },
    "overrides": []
  },
    },
    {
        'title': 'Humidity',
	'type': 'timeseries',
        'query': '''from(bucket: "TestDataBucket2")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurement")
  |> filter(fn: (r) => r["_field"] == "humidity")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'y_axis_label': 'Humidity (%)',
	'w': 8,
	'x': 16,
        'y': 8,
	'fillOpacity': 10,
	'fixedColor': 'light-green',
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic",
        "fixedColor": "light-green"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Weather Station ${__field.labels.source}",
      "unit": "humidity"
    },
    "overrides": []
  },
    },
    {
        'title': 'Wind Direction',
	'type': 'timeseries',
        'query': '''from(bucket: "TestDataBucket2")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurement")
  |> filter(fn: (r) => r["_field"] == "wind_direction")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'y_axis_label': 'Wind Direction (degrees)',
	'w': 8,
	'x': 0,
        'y': 16,
	'fillOpacity': 10,
        'fixedColor': 'green',
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic",
        "fixedColor": "green"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Weather Station ${__field.labels.source}",
      "unit": "degree"
    },
    "overrides": []
  },
    },
    {
        'title': 'Wind Speed',
	'type': 'timeseries',
        'query': '''from(bucket: "TestDataBucket2")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurement")
  |> filter(fn: (r) => r["_field"] == "wind_speed")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'y_axis_label': 'Wind Speed (m/s)',
	'w': 8,
	'x': 8,
        'y': 16,
	'fillOpacity': 10,
        'fixedColor': 'light-blue',
  "fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic",
        "fixedColor": "light-blue"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Weather Station ${__field.labels.source}",
      "unit": "velocityms"
    },
    "overrides": []
  },
    },
    {
        'title': 'Wind Gust',
	'type': 'timeseries',
        'query': '''from(bucket: "TestDataBucket2")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurement")
  |> filter(fn: (r) => r["_field"] == "wind_gust")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
	'y_axis_label': 'Wind Gust (m/s)',
	'w': 8,
	'x': 16,
        'y': 16,
	'fillOpacity': 10,
        'fixedColor': 'super-light-blue',
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic",
        "fixedColor": "super-light-blue"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Weather Station ${__field.labels.source}",
      "unit": "velocityms"
    },
    "overrides": []
  },
    }
    ,
]

# Define the dashboard JSON structure
dashboard = {
    'title': 'Test Dashboard 2.2',
    'panels': [],
    'editable': True,
    'hideControls': False,
    'timezone': 'browser',
}

# Add each panel to the dashboard
for panel in PANELS:
    panel_data = {
        'title': panel['title'],
        'type': panel['type'],
        'datasource': 'InfluxDB',
	'gridPos': 
	    {
		'h': 8,
		'w': panel['w'],
		'x': panel['x'],
		'y': panel['y']
	    }
	,
	'fieldConfig': panel['fieldConfig'],
        'targets':[
            {
                'query': panel['query'],
                'refId': 'A',
                'hide': False,
                'type': panel['type'],
            },
	]
    }
    if panel['type'] == 'timeseries':
      panel_data['yaxes'] = [{
	    'label': panel['y_axis_label'],
            'format': 'short',
            'show': True
        }, {
            'format': 'short',
            'show': True
        }]
        panel_data['x_axis'] = {'show': True}

    dashboard['panels'].append(panel_data)

# Add the rows to the dashboard
# Define the payload to send to the Grafana API
payload = {
    'dashboard': dashboard,
    'overwrite': True,
}

# Send the request to the Grafana API
response = requests.post(
    GRAFANA_API_URL,
    headers={
        'Authorization': f'Bearer {GRAFANA_API_KEY}',
        'Content-Type': 'application/json',
    },
    json=payload,
)

# Print the response from the Grafana API
print(response.text)
```

## Sensor Board Dashboard

Voor het dashboard voor de sensor boards wordt dezelfde layout gebruikt en zal ook dezelfde soort API-call gebruikt worden.

```python
import requests
import json

# Define your Grafana API endpoint, API key, and InfluixDB datasource URL
GRAFANA_API_URL = 'http://localhost:3000/api/dashboards/db'
GRAFANA_API_KEY = 'eyJrIjoieTZCQTdtNzhRMEdpUDFpdWJ5d2JHSEU1ckRYY0FsbnoiLCJuIjoidGVzdGtleSIsImlkIjoxfQ=='
INFLUXDB_URL = 'http://localhost:8086'
INFLUXDB_BUCKET = 'TestDataSensor'
INFLUXDB_MEASUREMENT = 'TestMeasurementSensor'
# Define the time range for the data

#Define  the panels to incluse in the dashboard

PANELS = [
    {
	'title': 'Sensor Board Location',
	'type': 'geomap',
	'query':'',
	'w': 5,
	'x': 0,
	'y': 0,
"fieldConfig": {
    "defaults": {
      "custom": {
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        }
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "color": {
        "mode": "thresholds"
      }
    },
    "overrides": []
  },
    },
    {
	'title': 'Sensor Board',
	'type': 'table',
	'query':'',
	'w': 19,
	'x': 5,
	'y': 0,
  "fieldConfig": {
    "defaults": {
      "custom": {
        "align": "auto",
        "displayMode": "auto",
        "inspect": False,
        "filterable": False
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "color": {
        "mode": "thresholds"
      }
    },
    "overrides": []
  },
    },
    {
	'title':'Temperature (BME280)',
	'type':'timeseries',
	'query':'''from(bucket: "TestDataSensor")
  |> range(start: -1y, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurementSensor")
  |> filter(fn: (r) => r["_field"] == "temp")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
	'w': 8,
	'x': 0,
	'y': 8,
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Board ${__field.labels.id}",
      "unit": "celsius"
    },
    "overrides": []
  },
    },
    {
	'title':'Air Pressure (BME280)',
	'type':'timeseries',
	'query':'''from(bucket: "TestDataSensor")
  |> range(start: -1y, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurementSensor")
  |> filter(fn: (r) => r["_field"] == "air_pressure")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
	'w': 8,
	'x': 8,
	'y': 8,
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Board ${__field.labels.id}",
      "unit": "pressurehpa"
    },
    "overrides": []
  },
    },
    {
        'title':'Humidity (BME280)',
        'type':'timeseries',
        'query':'''from(bucket: "TestDataSensor")
  |> range(start: -1y, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurementSensor")
  |> filter(fn: (r) => r["_field"] == "humidity")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'w': 8,
        'x': 16,
        'y': 16,
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Board ${__field.labels.id}",
      "unit": "humidity"
    },
    "overrides": []
  },
    },
    {
        'title':'Particles (SPS30)',
        'type':'timeseries',
        'query':'''from(bucket: "TestDataSensor")
  |> range(start: -1y, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurementSensor")
  |> filter(fn: (r) => r["_field"] == "particles")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'w': 8,
        'x': 8,
        'y': 8,
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Board ${__field.labels.id}",
      "unit": "µg/m³"
    },
    "overrides": []
  },
    },
    {
        'title':'Ppm (SGP41)',
        'type':'timeseries',
        'query':'''from(bucket: "TestDataSensor")
  |> range(start: -1y, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurementSensor")
  |> filter(fn: (r) => r["_field"] == "PPM")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'w': 8,
        'x': 8,
        'y': 16,
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Board ${__field.labels.id}",
      "unit": "ppm"
    },
    "overrides": []
  },
    },
    {
        'title':'CO2 (SCD41)',
        'type':'timeseries',
        'query':'''from(bucket: "TestDataSensor")
  |> range(start: -1y, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurementSensor")
  |> filter(fn: (r) => r["_field"] == "CO2")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'w': 8,
        'x': 16,
        'y': 16,
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Board ${__field.labels.id}",
      "unit": "ppm"
    },
    "overrides": []
  },
    },
    {
        'title':'VOC (SGP41)',
        'type':'timeseries',
        'query':'''from(bucket: "TestDataSensor")
  |> range(start: -1y, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurementSensor")
  |> filter(fn: (r) => r["_field"] == "VOC")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'w': 8,
        'x': 0,
        'y': 24,
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Board ${__field.labels.id}"
    },
    "overrides": []
  },
    },
    {
        'title':'NOx (SGP41)',
        'type':'timeseries',
        'query':'''from(bucket: "TestDataSensor")
  |> range(start: -1y, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "TestMeasurementSensor")
  |> filter(fn: (r) => r["_field"] == "NOx")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")''',
        'w': 8,
        'x': 0,
        'y': 24,
"fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": False,
        "showPoints": "auto",
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "axisColorMode": "text",
        "scaleDistribution": {
          "type": "linear"
        },
        "axisCenteredZero": False,
        "hideFrom": {
          "tooltip": False,
          "viz": False,
          "legend": False
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "color": {
        "mode": "palette-classic"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": None
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      },
      "displayName": "Board ${__field.labels.id}"
    },
    "overrides": []
  },
    },
]

# Define the dashboard JSON structure
dashboard = {
    'title': 'Test Dashboard Sensor',
    'panels': [],
    'editable': True,
    'hideControls': False,
    'timezone': 'browser',
    'time': {
	'from':'now-24h',
	'to':'now'
    }
}

# Add each panel to the dashboard
for panel in PANELS:
    panel_data = {
        'title': panel['title'],
        'type': panel['type'],
        'datasource': 'InfluxDB',
        'gridPos': 
            {
                'h': 8,
                'w': panel['w'],
                'x': panel['x'],
                'y': panel['y']
            }
        ,
        'fieldConfig': panel['fieldConfig'],
        'targets':[
            {
                'query': panel['query'],
                'refId': 'A',
                'hide': False,
                'type': panel['type'],
            },
        ]
    }
    dashboard['panels'].append(panel_data)

# Define the payload to send to the Grafana API
payload = {
    'dashboard': dashboard,
    'overwrite': True,
}

# Send the request to the Grafana API
response = requests.post(
    GRAFANA_API_URL,
    headers={
        'Authorization': f'Bearer {GRAFANA_API_KEY}',
        'Content-Type': 'application/json',
    },
    json=payload,
)

# Print the response from the Grafana API
print(response.text)
```
