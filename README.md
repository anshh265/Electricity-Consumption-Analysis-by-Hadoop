# Electricity Consumption Analysis using Hadoop

A Big Data Analytics project that processes smart-meter electricity
readings with **Apache Hadoop 3.4.2, HDFS, YARN, and Java MapReduce**.
Four MapReduce jobs calculate overall consumption statistics, hourly
averages, environmental bucket averages, and normal-versus-abnormal
reading summaries. The project also includes an interactive HTML
dashboard for exploring the dataset and checking the Hadoop results.

## Project Team

  Name                 Roll No.
  ------------------ ----------
  Ansh Nandanwar             41
  Vinshi Manwatkar           38

## Project Overview

The project studies **5,000 half-hourly electricity readings** recorded
between **1 January 2024 and 14 April 2024**. The input CSV contains
eight columns, including electricity consumption, weather variables,
historical consumption, a derived consumption difference, and an anomaly
label.

The data is stored in HDFS and processed through four independent
MapReduce jobs. Each job uses a Mapper to parse and emit key-value pairs
and a Reducer to aggregate the values for each key. The resulting output
files are exported to a local folder and used as the Hadoop result
reference for the dashboard.

### Objectives

-   Calculate total, average, minimum, and maximum electricity
    consumption.
-   Compare average electricity consumption across the 24 hours of a
    day.
-   Examine consumption across temperature, humidity, and wind-speed
    ranges.
-   Compare normal and abnormal readings using counts and average
    values.
-   Present the analysis through an interactive dashboard.

## Dataset

**Input file:** `electricity_consumption.csv`

  -----------------------------------------------------------------------
  Column                              Description
  ----------------------------------- -----------------------------------
  `Timestamp`                         Date and time of the reading, at
                                      30-minute intervals

  `Electricity_Consumed`              Recorded electricity consumption

  `Temperature`                       Normalised temperature value

  `Humidity`                          Normalised humidity value

  `Wind_Speed`                        Normalised wind-speed value

  `Avg_Past_Consumption`              Historical average consumption for
                                      the corresponding slot

  `Consumption_Difference`            Current consumption minus
                                      historical average

  `Anomaly_Label`                     `Normal` or `Abnormal`
  -----------------------------------------------------------------------

### Dataset profile

-   **Rows:** 5,000
-   **Time span:** 1 January 2024 -- 14 April 2024
-   **Sampling interval:** 30 minutes
-   **Missing values:** None reported
-   **Normal readings:** 4,750 (95%)
-   **Abnormal readings:** 250 (5%)

> **Important --- normalised values:** The numeric input columns are
> min-max normalised to the range 0--1. The values in the outputs and
> dashboard are relative, normalised values---not kWh. No conversion to
> physical units is made because the original scaling metadata is not
> available.

## Technology Stack

  -----------------------------------------------------------------------
  Component                           Technology
  ----------------------------------- -----------------------------------
  Distributed storage                 Hadoop Distributed File System
                                      (HDFS)

  Processing                          Hadoop MapReduce

  Resource management                 YARN

  Programming language                Java

  Big Data platform                   Apache Hadoop 3.4.2

  Dashboard                           HTML, CSS, JavaScript

  Charts                              Chart.js 4.4.1

  Fonts                               Google Fonts (Bricolage Grotesque,
                                      IBM Plex Sans, IBM Plex Mono)
  -----------------------------------------------------------------------

The Hadoop jobs were executed on a **single-node Hadoop cluster on
Windows**, with NameNode, DataNode, ResourceManager, and NodeManager
services. Each job used one Map task and one Reduce task for this input.

## Hadoop Processing Pipeline

``` text
electricity_consumption.csv
          |
          v
         HDFS
 /electricity/input
          |
          v
    YARN / MapReduce
          |
   +------+------+----------------+
   |             |                 |
   v             v                 v
Electricity   Hourly          Environmental
  Stats     Consumption        Analysis
   |             |                 |
   +-------------+-----------------+
                 |
                 v
          Anomaly Analysis
                 |
                 v
       HDFS result directories
                 |
                 v
       output/hadoop_results
                 |
                 v
        Interactive Dashboard
```

### HDFS locations

  Purpose                HDFS path
  ---------------------- -------------------------------------
  Input directory        `/electricity/input`
  Statistics output      `/electricity/output/stats`
  Hourly output          `/electricity/output/hourly`
  Environmental output   `/electricity/output/environmental`
  Anomaly output         `/electricity/output/anomaly`

Each completed job writes its reducer output to `part-r-00000` in its
respective HDFS output directory.

## MapReduce Jobs

### 1. `ElectricityStats`

**Purpose:** Calculate overall consumption statistics.

-   Mapper skips the CSV header and emits a constant key (`stats`) with
    each `Electricity_Consumed` value.
-   Reducer calculates the total, average, minimum, and maximum.

**Output:** Four summary values in `stats.txt`.

### 2. `HourlyConsumption`

**Purpose:** Calculate average consumption for each hour of the day.

-   Mapper reads the timestamp and extracts the hour (`00`--`23`).
-   Mapper emits the hour as the key and consumption as the value.
-   Reducer calculates the average consumption for each hour.

**Output:** 24 hourly averages in `hourly.txt`.

### 3. `EnvironmentalAnalysis`

**Purpose:** Compare consumption across environmental ranges.

-   Mapper reads consumption, temperature, humidity, and wind speed.
-   Each reading is emitted under three keys: one for temperature, one
    for humidity, and one for wind speed.
-   Each environmental value is assigned to a bucket of width `0.1` (for
    example, `0.3–0.4`).
-   Reducer calculates average consumption for every factor-and-range
    key.

**Output:** 30 averages in `environmental.txt` --- 10 buckets for each
of the three factors.

### 4. `AnomalyAnalysis`

**Purpose:** Compare normal and abnormal readings.

-   Mapper groups records by `Anomaly_Label`.
-   It passes current consumption, past consumption, and consumption
    difference to the reducer.
-   Reducer calculates the reading count and average of all three values
    for each label.

**Output:** Two summary rows in `anomaly.txt`, one for `Normal` and one
for `Abnormal`.

## Results and Findings

The following values are from the completed Hadoop outputs.

### Overall statistics

  Metric                       Result
  --------------------- -------------
  Total consumption       1883.387250
  Average consumption        0.376677
  Minimum consumption             0.0
  Maximum consumption             1.0

### Hourly analysis

-   **Highest hourly average:** 02:00 --- `0.398156`
-   **Lowest hourly average:** 01:00 --- `0.356704`
-   All 24 hourly averages fall between approximately `0.357` and
    `0.398`.
-   The hourly averages are relatively close, indicating that
    consumption is broadly steady across the day in this dataset.

### Environmental analysis

The environmental job generated 30 bucket averages: 10 each for
temperature, humidity, and wind speed. The bucket averages do not show a
consistent increasing or decreasing pattern across these factors.

These bucket comparisons describe associations in this dataset; they do
not establish that a weather variable causes a change in electricity
consumption. Bucket sizes can also affect the stability of individual
averages.

### Anomaly analysis

  -----------------------------------------------------------------------
  Label                Count        Average   Average past        Average
                                consumption    consumption     difference
  ----------- -------------- -------------- -------------- --------------
  Normal               4,750       0.376553       0.459978      -0.083424

  Abnormal               250       0.379039       0.435187      -0.056148
  -----------------------------------------------------------------------

Abnormal readings account for **5%** of the dataset. The average current
consumption for the two labels is very similar, so the average alone
does not distinguish normal from abnormal readings.

### Execution summary

All four MapReduce jobs completed successfully. Their recorded execution
times were between **19 and 22 seconds per job** on the local
single-node setup. These timings demonstrate successful execution in the
project environment; they are not a distributed performance benchmark.

## Interactive Dashboard

**File:** `Electricity Consumption Dashboard.html`

Open the HTML file in a modern web browser to explore the dashboard. It
is a client-side dashboard and does not require a separate backend
server.

### Dashboard features

-   Summary KPI cards for readings, average consumption, observed range,
    and abnormal share.
-   Filters for reading label (All, Normal, Abnormal) and month.
-   Consumption comparisons by hour, weekday, day, and month.
-   Weekday-by-hour heatmap.
-   Temperature, humidity, and wind-speed bucket charts.
-   Past-consumption comparisons and consumption-difference
    distribution.
-   Normal-versus-abnormal profiles and anomaly-share charts.
-   A Hadoop verification section that compares the embedded Hadoop
    output summaries with calculations from the dashboard's embedded
    dataset.

### Dashboard data and dependencies

The HTML file contains the dataset records and the Hadoop result values
embedded in its JavaScript. The dashboard recalculates its visual
analysis in the browser; it does **not** read files directly from HDFS
or automatically load the local `output/hadoop_results` directory. The
Hadoop verification section compares the embedded Hadoop results against
calculations from the embedded records.

Chart.js 4.4.1 is loaded from the Cloudflare CDN, and the selected fonts
are loaded from Google Fonts. An internet connection is therefore needed
for those external resources to load. The HTML file itself can be opened
locally.

## Running the Hadoop Jobs

The following is the workflow used in the Windows Hadoop environment. It
assumes Hadoop is installed and configured and that the Java source
files and JARs are available in the project directory.

### 1. Start Hadoop services

Start HDFS and YARN using the scripts/configuration in your Hadoop
installation. Confirm the required daemons are running:

``` bat
jps
```

The project environment used Hadoop at `C:\hadoop`.

### 2. Create the HDFS input directory and upload the CSV

``` bat
hdfs dfs -mkdir -p /electricity/input
hdfs dfs -put electricity_consumption.csv /electricity/input/
```

Check the uploaded file:

``` bat
hdfs dfs -ls /electricity/input
hdfs dfs -cat /electricity/input/electricity_consumption.csv
```

### 3. Run the MapReduce jobs

Each job takes an HDFS input path and a distinct HDFS output path.
Hadoop requires the output directory not to exist before a job starts.
If rerunning a job, remove its previous output directory first.

``` bat
hdfs dfs -rm -r /electricity/output/stats
hadoop jar ElectricityStats.jar ElectricityStats /electricity/input /electricity/output/stats
```

``` bat
hdfs dfs -rm -r /electricity/output/hourly
hadoop jar HourlyConsumption.jar HourlyConsumption /electricity/input /electricity/output/hourly
```

``` bat
hdfs dfs -rm -r /electricity/output/environmental
hadoop jar EnvironmentalAnalysis.jar EnvironmentalAnalysis /electricity/input /electricity/output/environmental
```

``` bat
hdfs dfs -rm -r /electricity/output/anomaly
hadoop jar AnomalyAnalysis.jar AnomalyAnalysis /electricity/input /electricity/output/anomaly
```

If an output directory does not exist yet, the corresponding `-rm -r`
command may report that it cannot find the path; run the job after
confirming the directory is absent.

### 4. Inspect the Hadoop outputs

``` bat
hdfs dfs -cat /electricity/output/stats/part-r-00000
hdfs dfs -cat /electricity/output/hourly/part-r-00000
hdfs dfs -cat /electricity/output/environmental/part-r-00000
hdfs dfs -cat /electricity/output/anomaly/part-r-00000
```

### 5. Export the outputs to the local project folder

Create the destination folder if needed:

``` bat
mkdir output\hadoop_results
```

Export each reducer output:

``` bat
hdfs dfs -get -f /electricity/output/stats/part-r-00000 "output\hadoop_results\stats.txt"
hdfs dfs -get -f /electricity/output/hourly/part-r-00000 "output\hadoop_results\hourly.txt"
hdfs dfs -get -f /electricity/output/environmental/part-r-00000 "output\hadoop_results\environmental.txt"
hdfs dfs -get -f /electricity/output/anomaly/part-r-00000 "output\hadoop_results\anomaly.txt"
```

The exported files can be inspected with:

``` bat
type output\hadoop_results\stats.txt
type output\hadoop_results\hourly.txt
type output\hadoop_results\environmental.txt
type output\hadoop_results\anomaly.txt
```

> **Build note:** The Java classes are compiled against the Hadoop
> libraries installed on the machine and packaged into JAR files. The
> exact `javac` classpath depends on the local Hadoop installation and
> environment configuration.

## Suggested Repository Structure

A clean repository layout for the project is:

``` text
Electricity-Consumption-Analysis/
│
├── README.md
├── Electricity Consumption Dashboard.html
├── Electricity_Consumption_Analysis.pptx
│
├── src/
│   ├── ElectricityStats.java
│   ├── HourlyConsumption.java
│   ├── EnvironmentalAnalysis.java
│   └── AnomalyAnalysis.java
│
├── jars/
│   ├── ElectricityStats.jar
│   ├── HourlyConsumption.jar
│   ├── EnvironmentalAnalysis.jar
│   └── AnomalyAnalysis.jar
│
├── data/
│   └── electricity_consumption.csv
│
└── output/
    └── hadoop_results/
        ├── stats.txt
        ├── hourly.txt
        ├── environmental.txt
        └── anomaly.txt
```

This is a suggested organisation; adjust the paths to match the files
included in your repository. If the original dataset is not intended for
public distribution, do not commit it. The dashboard currently embeds
its analysis data, so keep in mind that publishing the HTML also
publishes those embedded records.

## Limitations

-   All numeric values are normalised; physical consumption units such
    as kWh cannot be inferred from the available scaling information.
-   The Hadoop cluster is single-node and the dataset contains 5,000
    records. The project demonstrates the MapReduce workflow, not
    large-cluster scalability.
-   Environmental bucket averages and correlations indicate association,
    not causation.
-   The dashboard is a static client-side HTML application. It does not
    launch Hadoop jobs or connect to HDFS at runtime.
-   Dashboard chart rendering uses external CDN-hosted Chart.js and
    Google Fonts resources.

## Conclusion

The project demonstrates an end-to-end Big Data workflow: storing a
smart-meter CSV in HDFS, processing it with four Java MapReduce jobs on
Hadoop, exporting the reducer outputs, and presenting the findings
through an interactive dashboard. The analysis found relatively steady
hourly consumption, no consistent trend across the environmental
buckets, and a 5% abnormal-reading share whose average consumption is
close to that of normal readings.

------------------------------------------------------------------------

**Project:** Electricity Consumption Analysis using Hadoop\
**Course:** Big Data Analytics\
**Platform:** Apache Hadoop 3.4.2 · HDFS · YARN · Java MapReduce
