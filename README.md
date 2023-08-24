# RF_FRP_v1
Repo for the functioning first dataset and version of the FRP RAP RF project

Readme for calling Fire Radiative Power (FRP) Random Forest (RF) models in python:

Note: I would recommend reading the formatted word document and not the text document.

Background into models:

The models were trained on a RAP grid cell domain size to produce an FRP value at a specified hour. This means that each satellite FRP value was taken and then mapped to the nearest RAP grid cell and likewise, that same RAP grid cell meteorological values were paired with the FRP value and used in the RF training. For best results, they should be run with the same input variables and same resolution domain as possible. 

There are two subdomain trained models: CONUS and West. CONUS was trained on inputs covering the entire CONUS domain. West was trained on inputs on and to the west of -105. The satellite FRP was collected from both GOES and polar orbiters. Because of the resolution discrepancy, models are trained to run on either the GOES or polar FRP inputs. Polar inputs were aggregated to the RAP grid domain each hour. In the instance where the rap gridcell contains both GOES and polar data, a “both” product was trained. This took an average between the polar aggregated FRP with the GOES value on RAP grid cells where both FRPs were greater than 0. If only one satellite existed at the current hour for that grid cell, then that satellite’s value was taken.

Satellite mFRP calculations:

Polar satellites: Each rounded same-hour satellite FPR value is aggregated over the closest RAP grid cell. For example, if there are eight polar FRP measurements in the same hour over a same single RAP grid cell, all eight points are added together for one value. Then a mean is taken spanning either (i) the UTC day-before, hours 0-23, for that grid cell or (ii) the rolling 24-hours up-to and at that hour. Example For a polar satellite FRP at 4:20 UTC on July 7th 2018: The previous day mFRP (i) would be calculated from aggregated polar values in that grid cell on July 6th 2018 hours 0-23. The rolling 24-hour average (ii) would be calculated from July 6th 2018 5:0 UTC up to July 7th 2018 4:00 UTC. 

GOES satellites: Each rounded same-hour satellite FRP value is averaged over the RAP grid cell. For instance, should multiple GOES FRP measurements in the same hour over a same single RAP grid cell exist, then an average of them is taken. Then a mean is taken spanning either (i) the UTC day-before, hours 0-23, for that gridcell or (ii) the rolling 24-hours up-to and at that hour. Example For a GOES satellite FRP at 4:20 UTC on July 7th 2018: The previous day mFRP (i) would be calculated from averaged GOES values in that gridcell on July 6th 2018 hours 0-23. The rolling 24-hour average (ii) would be calculated from July 6th 2018 5:0 UTC up to July 7th 2018 4:00 UTC. 

Finally, models were trained over the entire 2018 year. 

Below is a list of trained model names and the required inputs to produce an hourly FRP value. Each input is a singular float, representative of the desired surface grid RAP value and in the RAP standard unit value. All times are kept in UTC and the model has been trained from UTC times. It is important to note that the models cannot be switched between inputs. Think of these models as a lock fit to a specific set of input keys. If one input is given in a different unit, a total-fire FRP value instead of RAP grid cell dependent, etc., then the model will not perform as trained to do.

How to use the model:

The models are saved as NAME.sav files in the RF_v1_models folder. Most importantly, the model inputs must match those specified in training the models. Firstly, the mFRP satellite inputs must have been calculated/derived in the same manner as those used in training the model (see subsection satellite mFRP calculations). Secondly, inputs must be on the RAP resolution used in training. A latitude and longitude map has been included in this package to confirm the grid resolution. For these versions of the RF model, they are only trained on CONUS. It is unknown how they will behave on the larger RAP domain and worth studying.

This is lat_lon_map_RAP.txt and is (2,337,451)

These RF models were trained in python with the scikit-learn package (importing sklearn in python). Please refer to the documentation for installation: https://scikit-learn.org/stable/

The trained models will need to be called and opened in python using the following command:

	import pickle
loaded_model = pickle.load(open(MODEL_FILE_NAME_HERE, 'rb'))

The models can be found either in the zip file or subfolder labeled models. The models’ names have either west or conus, based on the region the model was trained over, and polar or goes, based on the satellite FRP points that the model was trained with. Once a model is opened, the following command will execute the model the input features, DATAFRAME_OF_INPUT_FEATURES_HERE, typically provided in a pandas data frame structure:

	predictors = loaded_model.predict(DATAFRAME_OF_INPUT_FEATURES_HERE)

DATAFRAME_OF_INPUT_FEATURES_HERE in this case is a data frame where each row represents a desired RAP grid cell at some hour in time for the RF model to run and derive the hourly FRP value. Each corresponding column of the row is the necessary module input feature. For example, we have N collection of RAP grid cells at some hour that have an mFRP greater than zero for yesterday. All N rows will have 9 columns of corresponding inputs for Temp at N, Hour at N, Lat at N, RH at N, etc. that are inputted into the RF model. Predictors is the output and will be size N, where each row is the corresponding hourly FRP to the inputs of the Nth row.

The RF model that produces predictors can be called in a loop to run over the desired number of hours in a forecast period. To have the best possible FRP hourly forecast, it is important to update the RAP hourly weather variables (and hour of day input value). The lat and lon will remain consistent as will the mFRP input. These updated values can be pulled from the forecast at the current hour, or if easier, pulled from the forecast the hour before. It has been experimented to keep them consistent through time, but that produces poorer results because that is not how the RF model was trained. 

Python code for basic inference of random forest models:

The code to call any of the random forest models mentioned in the Random Forest Models section is the following:

RF_V1_demo.py

To run, take the pre-made input csv file for the fire. This csv contains columns with the random forest model inputs and 

The code is documented. Look for the areas that specify you change the variables and/or models. Only one random forest model should be called at a time. It is good practice to make sure that the input names of the variables are consistent with 

Random Forest (RF) model files, located in the subdirectory called “models”:

The models are listed by the model credentials and then the name. Additionally, there are the training and testing sizes for each random forest model. Lastly, the variable inputs and their importance values (summed to 1) are listed. These variable inputs are the listed inputs for each model. These are the only inputs allowed to be used in the model, no more and no less.

WEST GOES where FRP_yesterday>0 : RF_west_goes_2018_v1.sav
Training Features Shape: (113870, 7)
Training Labels Shape: (113870,)
Testing Features Shape: (48802, 7)
Testing Labels Shape: (48802,)
Mean Absolute Error for FRP: 63.7
Mean Absolute Error for FRP HRRR Gaussian Curve: 178.47
Accuracy: 30.39 %.
0.6031643492238596
Variable: goes_FRP_yesterday   Importance: 0.34
Variable: lon                  Importance: 0.19
Variable: wind_gust            Importance: 0.12
Variable: temp                 Importance: 0.11
Variable: RH                   Importance: 0.1
Variable: lat                  Importance: 0.08
Variable: hour                 Importance: 0.07
﻿
WEST polar where FRP_yesterday>0 : RF_west_polar_2018_v1.sav
Training Features Shape: (26026, 7)
Training Labels Shape: (26026,)
Testing Features Shape: (11154, 7)
Testing Labels Shape: (11154,)
Mean Absolute Error for FRP: 251.79
Mean Absolute Error for FRP HRRR Gaussian Curve: 407.12
Accuracy: -594.61 %.
0.004223948189165161
Variable: polar_FRP_yesterday  Importance: 0.21
Variable: wind_gust            Importance: 0.2
Variable: temp                 Importance: 0.17
Variable: RH                   Importance: 0.15
Variable: lon                  Importance: 0.12
Variable: lat                  Importance: 0.07
Variable: hour                 Importance: 0.07

CONUS GOES where FRP_yesterday>0 : RF_conus_goes_2018_v1.sav
Training Features Shape: (221887, 7)
Training Labels Shape: (221887,)
Testing Features Shape: (95095, 7)
Testing Labels Shape: (95095,)
Mean Absolute Error for FRP: 40.11
Mean Absolute Error for FRP HRRR Gaussian Curve: 118.85
Accuracy: 38.09 %.
0.6072659065827651
Variable: goes_FRP_yesterday   Importance: 0.35
Variable: lon                  Importance: 0.17
Variable: wind_gust            Importance: 0.12
Variable: temp                 Importance: 0.11
Variable: RH                   Importance: 0.09
Variable: lat                  Importance: 0.08
Variable: hour                 Importance: 0.07

CONUS polar where FRP_yesterday>0 : RF_conus_polar_2018_v1.sav
Training Features Shape: (38536, 7)
Training Labels Shape: (38536,)
Testing Features Shape: (16516, 7)
Testing Labels Shape: (16516,)
Mean Absolute Error for FRP: 176.13
Mean Absolute Error for FRP HRRR Gaussian Curve: 275.81
Accuracy: -441.46 %.
0.04208104063434315
Variable: wind_gust            Importance: 0.2
Variable: RH                   Importance: 0.19
Variable: polar_FRP_yesterday  Importance: 0.19
Variable: temp                 Importance: 0.14
Variable: lon                  Importance: 0.12
Variable: lat                  Importance: 0.08
Variable: hour                 Importance: 0.08


Readmes for the codes that processed the data to train the RF models, located in the subdirectory called “processing_scripts”:

Python scripts to combine satellite FRP’s with corresponding RAP weather variables by opening the satellite file, checking its location, and then finding the nearest RAP grid point and exports the satellite file’s worth of FRP points into a csv:

process_RAP_noaa_V1.py
process_RAP_jpss_V1.py
process_RAP_modis_V1.py
process_rap_goes_V1.py

This is an example of the format of the output FRP-RAP merged csv’s:



Python script to combine all of the merged csvs into hourly npy files as well as csv files. The npy files are set up on the RAP 2D grid with N dimensions for each variable input. This script calculates one hourly FRP per RAP grid point in one of the three ways, polar, goes, or both, mentioned in the background section. Additionally, mFRP values are calculated for each of the three methods for the day before and for the rolling 24-hour average.

hera_proc_satellites_RAP_v5.py

Below is a description of the npy setup where merged is the name of the object that will be stored to the npy file, idx is the 2D RAP grid, and N=0,1,…18 is the variable: 
       mergedf[(0, idx[0], idx[1])] =  #FRP goes averaged for rap grid only
        mergedf[(1, idx[0], idx[1])] = frp_regrid.lat[i]
        mergedf[(2, idx[0], idx[1])] = frp_regrid.lon[i]
        mergedf[(3, idx[0], idx[1])] = frp_regrid.temp2m[i]
        mergedf[(4, idx[0], idx[1])] = frp_regrid.RH[i]
        mergedf[(5, idx[0], idx[1])] = frp_regrid.Uwind[i]
        mergedf[(6, idx[0], idx[1])] = frp_regrid.Vwind[i]
        mergedf[(7, idx[0], idx[1])] = frp_regrid.windspeed[i] #wind gust
        mergedf[(8, idx[0], idx[1])] = frp_regrid.precip[i] # instant
        mergedf[(9, idx[0], idx[1])] = frp_regrid.vis[i] # instant
        mergedf[(10, idx[0], idx[1])] = frp_regrid.FRP[i] #this will be the 24 hour average to currenti
        mergedf[(11, idx[0], idx[1])] =  #this will be yesterdays 24 hour average
        mergedf[(12, idx[0], idx[1])] =  #count for mergedf[0] current hour goes avg value
        mergedf[(13, idx[0], idx[0])] = #FRP polar summation of points only
        mergedf[(14, idx[0], idx[0])] = #avgd polar and goes FRP points gridded
        mergedf[(15, idx[0], idx[1])] = frp_regrid.FRP[i] #this will be the 24 hour average to current polar
        mergedf[(16, idx[0], idx[1])] =  #this will be yesterdays 24 hour average polar
        mergedf[(17, idx[0], idx[1])] = frp_regrid.FRP[i] #this will be the 24 hour average to current goes
        mergedf[(18, idx[0], idx[1])] =  #this will be yesterdays 24 hour average goes

Below is an example of what the output files naming convention look like:
YYMMDDHH_RAP_FRP_proc.csv --> 18110100_RAP_FRP_proc.csv
YYMMDDHH_RAP_FRP_proc.npy --> 18110100_RAP_FRP_proc.npy

Python script to make a single csv with appended future values, when present, in same RAP grid cell as current hour FRP. Each row represents a single satellite hourly FRP point on the RAP grid domain. Three csv’s are created, based on non-zero entries either from goes, polar, or both. NOTE: a polar or goes or both point can occur within a different satellite csv, but the file name indicates which file contains the complete list of that satellite (or merged both product) contains the entire list of frp points. The name of the csv indicates the priority of which satellite must have non-zero values and stored all possible hourly FRP points.

polar_goes_npy_to_csv.py

Below are the three produced csv files:
RAP_polar_FRP.csv
RAP_both_FRP.csv
RAP_goes_FRP.csv

The csv files and npy files are not necessary to run the model. The model just needs the specified inputs, likely pulled directly from the model, but, if need be, below is a python script that reads the hourly npy files and sets up a csv file for case studies in 2018. It takes a 3x3 mean of RAP pixels FRP and weather variables. For the input, specify the date range and fire center latitude and longitude value and the script will pull the nearest points. The exported csv files for the Williams Flat fire, for instance, was used in the RF_V1_demo.py:

	plot_data_from_npys.py


