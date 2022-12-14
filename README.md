# Polimi-GIS-CAMS-ARPA

The goal of this case study is to retrieve and process the CAMS (Copernicus Atmosphere Monitoring Service) dataset and ARPA (Agenzia Regionale per la Protezione dell'Ambiente) data, and the ROI of this project is mainly focused on territory of Italy and the Lombardy region. 

Additionally, this project also includes an attempt to assess the CAMS data within the Lomabrdy region as well, considering the ARPA data as ground truth. 

As for the air quality data, the ARPA (Interpolated) dataset provides them in the year of 2018, 2019, 2020 and 2021 (temporary access), and the CAMS dataset offers its data in the year of 2020, 2021 and 2022 currently (3-year rolling archive). There are also ground sensors data from the ARPA dataset, but it is not recommended to use those point vector data to perform data assessment since it can produce errors due to its insufficient quantity of available data. 

## Data Sources

1. CAMS: [European Air Quality Forecast](https://ads.atmosphere.copernicus.eu/cdsapp#!/dataset/cams-europe-air-quality-forecasts?tab=overview)
2. ARPA Ground Sensors: [ARPA Elenco Data Pubblicati](https://www.dati.lombardia.it/widgets/8ask-gxyr)
3. ARPA Interpolated: Sistema Modellistico ARPA Lombardia
4. NUTS: [NUTS-BN|RG-2021-4326](https://gisco-services.ec.europa.eu/distribution/v2/nuts/nuts-2021-files.html)
5. DUSAF Landuse: [DUSAF 6.0 (2018)](https://www.geoportale.regione.lombardia.it/metadati?p_p_id=detailSheetMetadata_WAR_gptmetadataportlet&p_p_lifecycle=0&p_p_state=normal&p_p_mode=view&_detailSheetMetadata_WAR_gptmetadataportlet_uuid=%7B18EE7CDC-E51B-4DFB-99F8-3CF416FC3C70%7D)

## Folder Structure
```
root/ 
├── ARPA/                                         # Folder containing ARPA interpolated data
│   ├── Dominio_1km.csv                           # Sheet file containing UTM coordinates to pair with data in 2020 and 2021
│   ├── Dominio_4km.csv                           # Sheet file containing UTM coordinates to pair with data in 2018 and 2019
│   ├── 2018                                      # ARPA interpolated data in 2018
│   │   ├── medie_giornaliere_2018.txt            # .txt file containing all the ARPA interpolated data of NO2, O3, PM 10 and PM 2.5 in 2018
│   ├── 2019
│   │   ├── medie_giornaliere_2019.txt            # .txt file containing all the ARPA interpolated data of NO2, O3, PM 10 and PM 2.5 in 2019
│   ├── 2020
│   │   ├── NO2
│   │   │   ├── NO220200101.txt                   # .txt file containing all the ARPA interpolated data of NO2 on 1st January, 2018
│   │   │   ├── ...
│   │   ├── O3
│   │   ├── PM10
│   │   ├── PM25
│   ├── 2021
│   │   ├── NO2
│   │   │   ├── NO220210101.txt
│   │   │   ├── ...
│   │   ├── O3
│   │   ├── PM10
│   │   ├── PM25
├── ARPA Ground Sensors/                          # Folder containing ARPA ground sensors data
│   ├── 2018                                      # ARPA ground sensor data in 2018
│   │   ├── NO2
│   │   ├── O3
│   │   ├── PM10
│   │   ├── PM25
│   ├── 2019
│   ├── 2020
│   ├── 2021
├── CAMS/                                         # Folder containing CAMS data
│   ├── 2020                                      # CAMS data in 2020
│   │   ├── NO2
│   │   │   ├── NO2202001.nc                      # netcdf file containing all the data of NO2 in January, 2020, as well as processed data which will be exported and saved here
│   │   │   ├── ...
│   │   ├── O3
│   │   ├── PM10
│   │   ├── PM25
│   ├── 2021
│   ├── 2022
├── DUSAF/                                        # DUSAF shapefile data
├── Functions/                                    # Functions dump
├── NUTS/                                         # NUTS shapefile data
```

## Data Process

### 1. Retrieve data from CAMS and ARPA

- CAMS: Through API (3-year Rolling Archive). The data are stored in .nc format. 
- ARPA (Ground Sensor): Through Socrata API method to retrieve data in the current year or download the data from previous year in .csv format. The data from ground sensors are not used in later process in this project but might be useful in other case studies. 
- ARPA (Interpolated): Through direct URL request (Available from 2018-2021). The data are stored in .txt format compressed in zip files. 

### 2. Data Process of ARPA (Interpolated)

1. Unzip the files either manually or through scripts. 
2. Import the data and the sheet file which contains the information of geographical coordinates (UTM).
3. Join the data with the sheet file, as a result of which, we will obtain a geopandas dataframe which contains the concentration of air pollutants at each geographical coordinate. 
4. Calculate the monthly averages and export the data to .csv files. 

### 3. Data Process of CAMS

1. Import the data, adjust its longitude format and timestamp format and calculate the monthly averages. 
2. Mask the data with Lombardy shapefile. 
3. Export the data to .nc files. 

### 4. Comparison between ARPA and CAMS

1. Import the .csv files and the .nc files exported in the last two steps above. 
2. Calculate the averages and standard variances of each dataset and perform the F-Test and T-Test to these values, to verify if there is a significant difference between them. 
3. Calculate the Pearson correlation coefficient between these two datasets, to check if they have a strong relationship or no. 
4. Export these statistical data to .csv files. 

### 5. CAMS Upscaling

1. Upscale the CAMS dataset from spatial resolution 0.1°x0.1° up to 0.01°x0.01°. 
2. Perform the T-Test and F-Test to verift that the upscaled model is feasible, whose averages and standard variances are not significantly different from those of the original one. 
3. Export the upscaled dataset. 

### 6. Comparison between ARPA and Upscaled CAMS

1. Import the ARPA dataset and the upscaled CAMS dataset. 
2. Calculate the residuals at each pixel between the ARPA dataset and the upscaled CAMS dataset. 
3. Calculate the biases between these two datasets. 

## Credit

1. [CAMS](https://ads.atmosphere.copernicus.eu/cdsapp#!/dataset/cams-europe-air-quality-forecasts): for providing precious satellite modelled data and API method to retrieve its data automatically. 
2. Sistema Modellistico ARPA Lombardia: for providing URL to retrieve professionally interpolated ARPA data. 
3. [D-DUST Project](https://github.com/gisgeolab/D-DUST/tree/WP2): for providing essential guidance to retrieve ARPA ground sensor data via API. 
4. [Wekeo](https://www.wekeo.eu/): for providing useful tutorials regarding CAMS data process and visualization. 
