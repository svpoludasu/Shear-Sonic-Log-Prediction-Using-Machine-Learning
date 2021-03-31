# Shear Sonic Log Prediction Using Machine Learning
## About
This project was done as a part of the [SPE Machine Learning Challenge](https://www.spegcs.org/events/5965/) conducted by the SPE Gulf Coast Section in February 2021. This code has won the ***First prize*** in the **Metrics Driven** award and also the **Innovative Approach** award. As part of the agreement, the data provided to us during the competition cannot be shared. This code is also hosted by [SPEGCS repository](https://github.com/SPEGCS/ML-Challenge-Feb-March-2021).

## Problem Statement
Typically, shear sonic travel time (DTS) in conjunction with compressional sonic travel time (DTS) and bulk density are the key factors used in estimating rock mechanical properties. These estimates of rock mechanical properties provide us with valuable insights on subsurface characterization. DTS logs are often missing from the log suite due to their financial or operational constraints. The goal of the “SPE Machine Learning Challenge” is to develop data-driven models by processing “easy-to-acquire” conventional logs, use the data-driven models to generate synthetic DTS logs. **THE DATASET IS NOT RELEASED FOR USE OUTSIDE THE COMPETIOTION**.

## EDA and Feature Selection
Input dataset consists of well logs (.las files) from 234 wells. [Lasio](https://pypi.org/project/lasio/) was used to read the .lasfile and individual dataframes were created for each well.  Each well consisted of different number of logs varying from 5 to 40. The histogram below shows the distribution of number of logs.

![image](https://user-images.githubusercontent.com/69025410/113067965-f5c47100-9182-11eb-83a5-9e0e428b23c2.png)

Due to the confidential nature of the data, the actual latitude and longitudes were anonymized by keeping the relative locations constant. The image below shows the distribution of all the 234 wells. We see that there are two clusters of wells based on their locations.

![image](https://user-images.githubusercontent.com/69025410/113068736-8780ae00-9184-11eb-9afb-0c67a21e8312.png)

The naming schemes or log mnemonics don’t seem to be consistent among all the wells. This data set consists of 694 uniquely named logs and Only Depth and DTSM(target variable) are available for all the 234 wells. Other logs are either named differently or unavailable.

![image](https://user-images.githubusercontent.com/69025410/113069140-4b018200-9185-11eb-9690-3361a0418cac.png)

Another way of identifying logs would be grouping them based on their respective units. Even when the logs are named differently, their units would still be the same. Lasio has the capability of reading the header file and extracting the units of the logs within the .las files. We can see a better distribution of logs when grouped based on units. We can also observe that the number of logs is much higher than the total number of wells, implying multiple logs within the wells have the same units.

![image](https://user-images.githubusercontent.com/69025410/113069488-18a45480-9186-11eb-8dcf-903f6327409c.png)

We can tweak the grouping to only count a unit once per well, we can get a better idea of log availability. As we see below, the number of unique logs available is much lesser than before.

![image](https://user-images.githubusercontent.com/69025410/113069645-6de06600-9186-11eb-9a27-4e3def631255.png)

Grouped based on the log units, we can select the following features
- **Latitude (LAT)**: Relative location to include the effects of spatial variation. Available for all 234 wells.
- **Longitude (LONG)**: Relative location to include the effects of spatial variation. Available for all 234 wells.
- **Measured Depth (DEPT)**: Available for all the wells
- **Shear sonic travel time (DTSM)**: Available for all the wells, and this is the **TARGET variable**
- **Compressional sonic travel time (DTCO)**: Available for 223 Wells
- **Bulk Density (RHOB)**: Available for 169 wells. Some of the logs are density correction logs based on their descriptions, these logs are typically the correction factors used to increase accuracy of the bulk density logs. Hence, these correction logs need to be removed as a feature.
- **Photo-electric Factor (PEF)**: Available for 160 wells. The PEF logs are named differently and multiple logs are available in a single well, so we might have to create a new compund average feature.
- **Gamma Ray (GR)**: Available for 230 wells. Multiple GR logs are present within a well, so a compound average GR feature needs to be created. Also, some of the logs within this group are Spectral Gamma Ray logs (A different kind of gamma ray log measuring natural radiation within the rock) which needs to be removed from the unit group.
- **Neutron Porosity (NPHI)**: Available for 199 wells. Multiple NPHI logs are present within a well, so a compound average GR feature needs to be created.
- **Resistivity (RES)**: Available for 181 wells, but many different variations are available so depending on the magnitudes, we might have to eliminate the entire feature or create a compound average feature.
- **Caliper logs (CALI)**: Caliper logs are available for 187 wells. Multiple CALI logs might be present within a well, so a compound average GR feature needs to be created.

We can see the number of null values within each selected feature below. These values can be imputed using an inbuilt imputer. I chose to use an iterative imputer for this case.

![image](https://user-images.githubusercontent.com/69025410/113071540-92d6d800-918a-11eb-9713-53c8ac23324e.png)

A simple representation of the ML pipeline used for this analysis.

![image](https://user-images.githubusercontent.com/69025410/113071987-81da9680-918b-11eb-9254-2b7ebafff097.png)

## Model Selection
Based on the cross-validation scores in conjunction with training time of different ML algorithms, I selected the top two Algorithms as XGBoost and LightGBM and submitted the results for the preliminary leaderboard. Both of these models are tree-style gradient boosting models, Light GBM has very fast training time whereas XGBoost is has higher accuracy. This was confirmed based on the error scores on the leaderboard 2, Light GBM had an error score of 29.66 where as XGboost had 29.49. So, **XGBoost** will be used for ***hyperparameter tuning*** and ***final predictions***.

![image](https://user-images.githubusercontent.com/69025410/113072023-9880ed80-918b-11eb-8acc-d803e36073f6.png)

## Predictions for Blind Test Set
The blind test data set consisted of 10 wells. These test wells are spatially distributed within the train dataset space as seen below.

![image](https://user-images.githubusercontent.com/69025410/113072329-6a4fdd80-918c-11eb-8954-d3abbad09d9f.png)

This blind test dataset is put through the same pipeline as Train dataset to condition it for predicting the DTS. The predicted DTSM had RMSE value of 22.905 as published by the organizers.

![image](https://user-images.githubusercontent.com/69025410/113072490-c1ee4900-918c-11eb-8c44-89d2eb18ed4d.png)




