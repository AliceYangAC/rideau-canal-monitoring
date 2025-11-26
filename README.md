# Rideau Canal Skateway - IoT Simulation & Dashboard

![Python](https://img.shields.io/badge/Python-3.13-blue?logo=python)
![Dash](https://img.shields.io/badge/Built%20with-Plotly%20Dash-0174DF?logo=plotly)

![Azure](https://img.shields.io/badge/Cloud-Azure-blue?logo=microsoft-azure)
![IoT Hub](https://img.shields.io/badge/Azure%20Service-IoT%20Hub-0078D7?logo=azure)
![Stream Analytics](https://img.shields.io/badge/Azure%20Service-Stream%20Analytics-0078D7?logo=azure)
![Cosmos DB](https://img.shields.io/badge/Azure%20Service-Cosmos%20DB-0078D7?logo=azure)
![Blob Storage](https://img.shields.io/badge/Azure%20Service-Blob%20Storage-0078D7?logo=azure)

A real‑time safety dashboard powered by IoT Hub, Azure Stream Analytics, Cosmos DB, and Blob Storage. Sensor data flows from IoT devices congregated into IoT Hub, through Stream Analytics, into Cosmos DB for live aggregation, and archived in Blob Storage for historical records. The dashboard, built with Dash (Plotly) and deployed on Vercel, visualizes current conditions, safety status, and hourly trends across multiple canal locations.

---

## Student Information
- **Name:** Alice Yang
- **Student ID:** 041200019
- **Course:** CST8916 - Fall 2025

## Repository Links

### 1. Main Documentation Repository
- **URL:** [documenation repo (here)](https://github.com/aliceyangac/rideau-canal-monitoring)
- **Description:** Complete project documentation, architecture, screenshots, and guides

### 2. Sensor Simulation Repository
- **URL:** [sensor simulation repo](https://github.com/aliceyangac/rideau-canal-sensor-simulation)
- **Description:** IoT sensor simulator code

### 3. Web Dashboard Repository
- **URL:** [dashboard repo](https://github.com/aliceyangac/rideau-canal-dashboard)
- **Description:** Web dashboard application

## Scenario Overview

Thousands of citizens in the city head to the Rideau Canal once it freezes over in the winter. However, skating safety can vary wildly depending on factors like ice thickness and surface temperature. If real-time monitoring is not enabled, skaters may not get timely information and end up on the ice in unsafe conditions. In addition, the city would not have visibility into historical trends to inform municipal decisions regarding how to open or close the canal.

### IoT Sensor Simulation

Simulate sensor devices at three canal locations — Dow’s Lake, Fifth Avenue, and NAC — capturing key environmental metrics:

- Ice Thickness (cm)
- Surface Temperature (°C)
- Snow Accumulation (cm)
- External Temperature (°C)

### Real‑Time Data Processing

Stream sensor data through Azure IoT Hub and process it with Azure Stream Analytics, applying 5‑minute aggregation windows to generate timely, consolidated insights.

### Fast Data Access

Store aggregated results in Azure Cosmos DB, enabling low‑latency queries for the dashboard and supporting real‑time safety classification.

### Historical Archiving

Persist aggregated sensor data in Azure Blob Storage, providing a durable record for historical trend analysis and reporting.

### Web Dashboard Visualization

Deliver a user‑friendly dashboard hosted on Vercel, built with Dash (Plotly), to:

- Display live sensor readings and safety badges for each location.
- Show system‑wide skating condition status.
- Render historical trend charts for ice thickness and surface temperature.

## System Architecture

![alt text](architecture/diagram.png)
```ascii
+-------------------+
| IoT Devices       |
| (Simulated:       |
|  Dow's Lake,      |
|  Fifth Ave, NAC)  |
+---------+---------+
          |
          v
+-------------------+
| Azure IoT Hub     |
| (telemetry ingest)|
+---------+---------+
          |
          v
+-------------------+
| Azure Stream      |
| Analytics         |
| (5-min tumbling   |
|  window agg)      |
+---------+---------+
          |
          +-------------------+
          |                   |
          v                   v
+-------------------+   +-------------------+
| Cosmos DB         |   | Blob Storage      |
| (fast query for   |   | (historical data  |
|  dashboard)       |   |  archive)         |
+---------+---------+   +---------+---------+
          |                       |
          v                       |
+-------------------+             |
| Vercel            |             |
| (Dash dashboard   |             |
|  visualization)   |             |
+-------------------+             |
          |                       |
          v                       v
+-------------------+   +-------------------+
| Live safety       |   | Historical trend  |
| badges & status   |   | charts            |
+-------------------+   +-------------------+
```
---
### Azure Services

1. IoT Hub
2. Stream Analytics
3. Cosmos DB
4. Blob Storage

### Data Flow

#### IoT Devices

- Simulated IoT devices are deployed at 3 Rideau Canal Skateway areas:
   1. Dow’s Lake
   2. Fifth Avenue
   3. NAC
- Each device monitors four environmental metrics:
   1. Ice Thickness (cm)
   2. Surface Temperature (°C)
   3. Snow Accumulation (cm)
   4. External Temperature (°C)

#### Azure IoT Hub

- 3 devices transmit raw telemetry data to Azure IoT Hub, which acts as the ingestion point for all sensor traffic.

#### Azure Stream Analytics

- Input: Incoming IoT data is processed in real-time using 5-minute tumbling aggregation windows.
- Stream Analytics processes averages, minimums, maximums, and counts for each metric per location.
- Output: Aggregated results is stored in Blob and Cosmos DB containers

#### Azure Cosmos DB

- Aggregated results are written to Cosmos DB, optimized for fast querying with extremely low latency
- Dashboard queries the latest readings from Cosmos DB to display live skate conditions and safety metrics.

#### Azure Blob Storage

- Aggregated results are written to Cosmos DB as historical data
- Supports trend analysis and long-term reporting.

#### Vercel App

- Dash (Plotly) dashboard is hosted on Vercel, visualizing:
   1. Real-time readings and safety badges per location
   2. Historical trend charts on ice temperature and surface temperature
   3. Overall safety/system status
- Queries Cosmos DB for live data and Blob Storage for historical records.

## Implementation Overview

### IoT Sensor Simulation (link to repo)

**URL:** [sensor simulation repo](https://github.com/aliceyangac/rideau-canal-sensor-simulation)

I modified the IoT Hub demo code from class for one simulated device to instead connect 3 devices to IoT Hub, `dows-lake`, `fifth-avenue`, and `nac`. The Python script simulates telemetry data from three different locations, Dow's Lake, Fifth Avenue, and NAC, sending data to Azure IoT Hub every 10 seconds on loop until the connection is broken by keyboard. The connection strings for each location are loaded from a `.env` file located in the same directory as this script.

### Azure IoT Hub configuration

I registered the 3 devices, `dows-lake`, `fifth-avenue`, and `nac`, in IoT hub and retrieved their connection strings for my script. Once the script was active, I was able to track the influx of messages via Azure CLI `az iot hub monitor-events --hub-name rideau-canal-iot-hub` and within the portal via Logs, Monitor, or Overview.

### Stream Analytics job (include query)

The query is included under `/stream-analytics` and can be seen below:
```sql
SELECT 
    CONCAT(c.location, '-', MAX(TRY_CAST(c.timestamp AS datetime))) as id,
    c.location,
    MAX(TRY_CAST(c.timestamp AS datetime)) as timestamp,
    AVG(c.ice_thickness) AS avg_ice_thickness,
    MIN(c.ice_thickness) AS min_ice_thickness,
    MAX(c.ice_thickness) AS max_ice_thickness,
    AVG(c.surface_temperature) AS avg_surface_temperature,
    MIN(c.surface_temperature) AS min_surface_temperature,
    MAX(c.surface_temperature) AS max_surface_temperature,
    MAX(c.snow_accumulation) AS max_snow_accumulation,
    AVG(c.external_temperature) AS avg_external_temperature,
    COUNT(1) AS reading_count
INTO SensorAggregations
FROM "rideau-canal-iot-hub" AS c
GROUP BY TumblingWindow( minute , 5 ), c.location

SELECT 
    CONCAT(c.location, '-', MAX(TRY_CAST(c.timestamp AS datetime))) as id,
    c.location,
    MAX(TRY_CAST(c.timestamp AS datetime)) as timestamp,
    AVG(c.ice_thickness) AS avg_ice_thickness,
    MIN(c.ice_thickness) AS min_ice_thickness,
    MAX(c.ice_thickness) AS max_ice_thickness,
    AVG(c.surface_temperature) AS avg_surface_temperature,
    MIN(c.surface_temperature) AS min_surface_temperature,
    MAX(c.surface_temperature) AS max_surface_temperature,
    MAX(c.snow_accumulation) AS max_snow_accumulation,
    AVG(c.external_temperature) AS avg_external_temperature,
    COUNT(1) AS reading_count
INTO "historical-data"
FROM "rideau-canal-iot-hub" AS c
GROUP BY TumblingWindow( minute , 5 ), c.location
```

The two queries go `INTO` two different destination sinks; the `SensorAggregations` Cosmos DB container and the `historical-data` Blob Storage container. It is the same query for both that aggregates averages, minimums, and maxes for the telemetry metrics of ice thickness, surface temperature, snow accumulation, and external temperature over a 5 min tumbling window of the IoT device messages. In addition I set a unique ID per aggregation that is the `{location}-{timestamp}`.

### Cosmos DB setup

I created a Cosmos DB named `RideauCanalDB` with the container `SensorAggregations` that serves as a output sink from Stream Analytics. It has a partition key, `/location`.

### Blob Storage configuration

I created a Storage Account called `rideaucanalstorage` with the container `historical-data` that serves as the second output sink from Stream Analytics.  has the file path pattern `aggregations/{date}/{time}` and with the format JSON (line separated).

### Web Dashboard (link to repo)

**URL:** [dashboard repo](https://github.com/aliceyangac/rideau-canal-dashboard)

Dash (Plotly) is the framework I used for this simulated Rideau Canal Skateway dashboard. It functions on top of a Flask server, with the web components built on top of React. The HTML components and callback architecture make up the application's state by linking the `dcc.Interval` timer directly to the update_dashboard function. This allows the app to auto-refresh the real-time location cards and historical trend charts every 30 seconds. Dash’s native integration with Pandas and Plotly made it easy to display the figures required for the data visualization.

### Vercel deployment

I deployed the app on Vercel by connecting the Github repo to a new project. Then, I configured the environment variables in the settings to include the `COSMOS_CONN_STR`, `COSMOS_KEY`, `BLOB_CONN_STR`, and `PORT` required for the web app to render.

## Repository Links

### 1. Main Documentation Repository

- **URL:** [documenation repo](https://github.com/aliceyangac/rideau-canal-monitoring)
- **Description:** Complete project documentation, architecture, screenshots, and guides

### 2. Sensor Simulation Repository

- **URL:** [sensor simulation repo](https://github.com/aliceyangac/rideau-canal-sensor-simulation)
- **Description:** IoT sensor simulator code

### 3. Web Dashboard Repository

- **URL:** [dashboard repo](https://github.com/aliceyangac/rideau-canal-dashboard)
- **Description:** Web dashboard application

## Video Demonstration

- **Video Demo:** [YouTube](https://youtu.be/FqjUoSnanKY)

## Setup

### Prerequisites

Ensure you have the following:

#### Local Development

- **Python 3.13+**
- **pip** for dependency management

#### Azure Cloud Resources

- **Azure IoT Hub**
- **Azure Stream Analytics**
- **Azure Cosmos DB**
- **Azure Blob Storage**
- **Vercel (Free)**

---

### High-Level Installation (Pre-Deployment)

#### 1. Clone the Repos

```bash
git clone https://github.com/aliceyangac/rideau-canal-dashboard.git
git clone https://github.com/aliceyangac/rideau-canal-sensor-simulation.git
```

#### 2. Create and Activate Virtual Environment per Repo

Make a unique `venv` in each repo.

```bash
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
```

#### 3. Install Dependencies per Repo

Ensure you `pip install` into the `venv` in both repos.

```bash
pip install -r requirements.txt
```

#### 4. Configure Environment Variables

Copy `.env.example` as `.env` in both folders and replace the placeholder values later once you set up Azure services.

#### 5. Detailed Setup/Next Steps

Follow the instructions in the READMEs below for deploying Azure services.

- **URL:** [sensor simulation repo](https://github.com/aliceyangac/rideau-canal-sensor-simulation)
- **URL:** [dashboard repo](https://github.com/aliceyangac/rideau-canal-dashboard)

## Results and Analysis

### Screenshots

- IoT Hub with 3 registered devices
![alt text](screenshots/image.png)
- IoT Hub metrics showing messages received
![alt text](screenshots/image-1.png)
![alt text](screenshots/image-6.png)
- Stream Analytics query editor with your query
![alt text](screenshots/image-8.png)
![alt text](screenshots/image-9.png)
- Stream Analytics job in "Running" state
![alt text](screenshots/image-5.png)
- Cosmos DB Data Explorer with sample documents
![alt text](screenshots/image-2.png)
- Blob Storage container with archived files
![alt text](screenshots/image-7.png)
- Dashboard running locally (showing live data)
![alt text](screenshots/image-3.png)
- Dashboard deployed on Vercel
![alt text](screenshots/image-11.png)
![alt text](screenshots/image-10.png)

### Data analysis

The dashboard successfully visualizes real-time and historical sensor data from three Rideau Canal locations. Recent aggregated metrics such as ice thickness, surface temperature, snow accumulation, and external temperature are clearly displayed per location. Safety status badges accurately reflect current (within 5 minutes) skating conditions based on defined thresholds, while trend charts reveal hourly fluctuations and help identify potential trending risks in the skating conditions. In conclusion, the system supports both immediate decision-making and trend analysis.

### System performance observations

The pipeline demonstrates reliable E2E data streaming from IoT device simulation data sources, IoT Hub data ingestion, Stream Analytics data processing, Cosmos DB & Blob Storage data sinks, and dashboard data visualization. Cosmos DB non-relational data queries return live data with extremely low latency, and Blob Storage is reliable for displaying historical trends. The 30 second auto-refresh intervals and countdown logic operate consistently with Dash's native `dcc.Interval` feature, and the dashboard remains responsive under typical load of new aggregated metrics every 5 minutes.

## Challenges and Solutions

When deploying code from repo, I could not successfully get a working dashboard application through Azure Web App Service. I plan to attempt deploying from container next, but I believe the reason is because Dash is built on Flask, which requires `gunicorn` to startup in App Service. I will continue to troubleshoot the issue. For now, with your permission, I was able to successfully deploy from repo onto Vercel, and the app worked out the box after configuring the environment variables in the settings.

## AI Tools Disclosure

I used Copilot to generate CSS for the aesthetic for the dashboard, for debugging (particularly when trying to deploy on App Service), and to summarize parts of the report, like the overview or troubleshooting sections. This is because I wanted to include potential troubleshooting scenarios that I did not personally encounter, but others may. I made sure to reword the summaries to better reflect my thoughts.

## References

[Dash documentation](https://dash.plotly.com/)