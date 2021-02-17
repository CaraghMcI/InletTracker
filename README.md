# EntranceSat

Valentin Heimhuber, University of New South Wales, Water Research Laboratory, 02/2021


## **Description**

EntranceSat is a Google Earth Engine enabled open source python software package that first uses a novel least cost path finding approach to trace IOCE entrance channels along and across the berm, and then analyses the resulting transects to infer whether an entrance is open or closed. EntranceSat is built on top of the imagery download and pre-processing functionality of the CoastSat toolbox [https://github.com/kvos/CoastSat].

![Alt text](readme_files/EntranceSat_method_illustration.gif)

The underlying approach of the EntranceSat toolkit and it's performance is described in detail in the following publication, which is currently under review:

Valentin Heimhuber, Kilian Vos, Wanru Fu, William Glamore (submitted): EntranceSat: A Google Earth Engine enabled open-source python tool for historic and near real-time monitoring of intermittent estuary entrances, 2021

Output files of EntranceSat include:
- Time series of along-berm and across-berm paths as .shp file for use in GIS software (no tidal correction)
- NIR, SWIR1, NDWI and mNDWI extracted along each along-berm and across-berm path
- A 'dashboard' or overview plot showing the location of the along-berm and across-berm paths along with the spectral transects and level of the tide (if the tide model is integrated)
- A variety of plots that illustrate the key results of the algorithm


## 1. Installation

The EntranceSat toolkit is run in the python environment of Coastsat, with a few additional python packages added to it. It uses the Google Earth Engine API to download and pre-process the satellite imagery based on the corresponding functions of CoastSat [https://github.com/kvos/CoastSat]. The installation instructions provided here are replicated from CoastSat with slight mo.  

### 1.1 Download repository

Download this repository from Github and unzip it in a suitable location.

### 1.2 Create a python environment with Anaconda

To run the toolbox you first need to install the required Python packages in an environment. To do this we will use **Anaconda**, which can be downloaded freely [here](https://www.anaconda.com/download/).

Once you have it installed on your PC, open the Anaconda prompt (in Mac and Linux, open a terminal window) and use the `cd` command (change directory) to go the folder where you have downloaded this repository. Copy the filepath to that folder and replace the below filepath with that.

```
cd C:\Users\EntranceSat
```

Create a new environment named `entrancesat` with all the required packages:


```
conda env create -f environment.yml -n entrancesat
```

All the required packages have now been installed in an environment called `entrancesat`. Now, activate the new environment:

```
conda activate entrancesat
```

To confirm that you have successfully activated EntranceSat, your terminal command line prompt should now start with (entrancesat).

**In case errors are raised:**: open the **Anaconda Navigator**, in the *Environments* tab click on *Import* and select the `environment.yml` file. For more details, this [link](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-with-commands) shows how to create and manage an environment with Anaconda.

### 1.3 Activate Google Earth Engine Python application programming interface or API

First, you need to request access to Google Earth Engine at https://signup.earthengine.google.com/. It takes about 1 day for Google to approve requests.

Once your request has been approved, with the `entrancesat` environment activated, run the following command on the Anaconda Prompt to link your environment to the Google Earth Engine server:

```
earthengine authenticate
```

A web browser will open. Here, login with a gmail account and accept the terms and conditions. Then copy the authorization code into the Anaconda terminal.

Now you are ready to start using the EntranceSat toolbox!

**Note**: remember to always activate the environment with `conda activate entrancesat` each time you are preparing to use the toolbox. Once you do this from the Annaconda command prompt, you can startup Spyder simply by typing spyder in the command line.


### 1.4 Install and activate the FES global tide model (for experienced python users)

EntranceSat uses the FES2014 global tide model reanalysis dataset to estimate the level of the tide for the point in time where each satellite image was acquired, since this tide level determines the depth of the water column in open entrances. FES2014 ranks as one of the best global tide models for coastal areas (based on an assessment on 56 coastal tide gauges, see this paper https://agupubs.onlinelibrary.wiley.com/doi/full/10.1002/2014RG000450.

Importantly, the FES2014 tide levels are provided to the user but it is not currently used in any form of calculation or the determination of open vs. closed entrance states. Since the setup of the tide model is rather technical, some users might therefore choose to not incorporate the FES2014 data into the analysis, which is an option.

	How to install fbriol fes (the FES2014 global tide model) in a python environment:

	1. Download the source folder from here: https://bitbucket.org/fbriol/fes/downloads/ - unzip it and store in any preferred location outside of your EntranceSat repository folder.
	2. Test `import pyfes` in python (this should work because pyfes was installed via the environment.yml file). If this doesn't work yet: With the entrancesat environment activated in the annaconda command propmt, run: `conda install -c fbriol fes` #this works without step 1
	3. Acquire/download the actual model data via Netcdf files provided separately. These Netcdf files include **ocean tide**, **ocean_tide_extrapolated** and **load tide** .nc and are provided through the Aviso+ servers
	as explained here https://bitbucket.org/fbriol/fes/src/master/README.md.
	These are large (8gb+) Netcdf files that contain all the results from the global tide model. They are not included in the installation of pyfes and therefore, have to be downloaded separately and placed in the correct directory locations.
	4. Save the .nc files in the source folder, under /data/fes2014 (e.g. ...\data\fes2014\ocean_tide_extrapolated\).
	5. When using fes in python, it will look for these files in the source folder, but the filepaths to each file have to be provided explicitly (manually) in the ocean_tide_extrapolated.ini file. Update the ocean_tide_extrapolated.ini file to include the absolute path to each tidal component (.nc file)
	6. When using EntranceSat via the EntranceSat_master.py code, set the path to where you unzipped the fbriol source folder in the general 'settings'. e,g, 'filepath_fes' : r"H:\Downloads\fes-2.9.1-Source\data\fes2014". This filepath is used to tell pyfes where to look for the ocean_tide_extrapolated.ini file.
	7. Lastly, to integrate the FES2014 data in EntranceSat, ensure to set the following parameter to true in the settings: 'use_fes_data': True


**Note**: If setup of FES2014 creates any issues, you can always run EntranceSat without this data via setting 'use_fes_data' to False in 'settings'.





## **Using EntranceSat**

### Setting up the algorithm for processing a new estuary entrance site

All user input files (area of interest polygon, transects & tide data) are kept in the folder ".../Entrancesat/user_inputs"

It is recommended that new analysis regions/ IOCEs are added directly to the input_locations.shp file located in this directory via QGIS or ArcGIS. For each site, EntranceSat expects 7 polygons as shown in this example. In the attribute table of this shapefile, each of these 7 unique polygons has to be named accordingly in the 'layer' field ('full_bounding_box', 'A-B Mask', 'C-D Mask', 'B', 'A', 'D', 'C'). Note that the 'estuary_area' polygon is not required at this stage. Each polygon also requires the sitename in the 'sitename' field.

![Alt text](https://github.com/VHeimhuber/EntranceSat/blob/main/readme_files/EntranceSat%20site%20setup%20illustration.jpg)

The full_bounding_box polygon is used for selecting and cropping of the satellite imagery from the Google Earth Engine. This does not necessarily have to incorporate the entire estuary water body. The area of this polygon should not exceed 100 km2.

Points A and C are the seed points for the automated tracing of the across-berm and along-berm paths. B and D are the receiver points. A to B is the across-berm path. C to D is the along-berm path. In the shapefile, points A to D should be built as very small triangles. The first point of this triangle is used as seed/receiver point during analysis. This is a workaround since shapefiles cannot contain polygons and points at the same time.

The A-B and C-D masks are used for limiting the area for least-cost pathfinding to within those polygons. The location of the seed and receiver points and shape of these masks can significantly affect the performance of the pathfinding and it is recommended to play around with these shapes initially by doing test runs based on a limited number of images until satisfactory performance is achieved.


### Running the tool in python

It is recommended the toolkit be run in spyder. Ensure spyder graphics backend is set to 'automatic' for proper plot rendering.
- Preferences - iPython console - Graphics - Graphics Backend - Automatic

EntranceSat is run from the EntranceSat_master.py file.
- Instructions and comments are provided in this file for each step.
- It is recommended steps be run as individual cells for first time users. For each cell/processing step, first ensure that all parameters are set correctly and then run the cell via Ctrl + Enter.

The major steps are briefly outlined here. The numbering corresponds to the numbering provided in EntranceSat_master.py.

#### Step 1: Analysis setup & imagery download
This is where the base parameters for the entrance state analysis are established and the imagery is downloaded. At the top of the script, enter the sitename of your site and ensure it matches the spelling you used in the input_locations.shp file. The code will then extract the polygons from this shapefile for your site. Choose the time period for analysis and the satellites you want to include: e.g. dates = ['1985-01-01', '2020-11-01']; sat_list = ['L5','L7','L8','S2'] for all satellites and longest possible analysis period; Imagery is downloaded via: metadata = SDS_download.retrieve_images(inputs)

#### Step 2: Generate training data  
Generate training data through the interactive user interface shown below. Generally, the more images you classify into open vs. closed entrance states, the better but we recommend at least 10 open and 10 closed entrance states to be included for each group of satellites (Landsat 5,7 and 8) and (Sentinel-2). Use the arrows on the keyboard to 'classify' each image. Equal 'class sizes' can be achieved by using 'skip'. Users can use the 'username' : 'EntranceSat' parameter to generate multiple different sets of training data (although each full entrance state analysis is always only based on a single set of training data)

Use 'skip' if the image is cloudy or otherwise of poor quality. User 'unclear' if you can't tell whether the entrance is open or closed.

![Alt text](https://github.com/VHeimhuber/EntranceSat/blob/main/readme_files/training_data_generator.png)

#### Step 3: Load tide data
Generate tide time series (if use_fes_data = True). In this step, EntranceSat uses the location of seed point A to query into the FES2014 tide reanalysis data and extract tide water levels for the entire analysis period. Two pandas dataframes are created from this data. sat_tides_df contains the tide level for each image along with the name of the image. tides_df contains a 15min timestep tide time series for the full analysis period.  

#### Step 4: Least-cost path finding
This step is the least-cost path finding step, which is the core step of the EntranceSat algorithm. This will process all downloaded images that have less than the user defined threshold of cloud cover over the across-berm mask. For each image, the least-cost path finding algorithm is run and the core bands and spectral indices are then extracted along those transects as input to the second major analysis step. Most of the input parameters to the SDS_entrance.automated_entrance_paths function (settings_entrance) don't need to be changed. The first 8 parameters should be considered more carefully since they have significant impact on the results. The most important parameter here is 'path_index', which defines which band or index will be used as the cost surface for least-cost path finding. Guidance on selecting the most suitable option for this is provided in the aforementioned journal paper. All other parameters can initially be left unchanged.

For each image, EntranceSat creates the following 'dashboard' output and saves it as a png file in the designated directory.
![Alt text](https://github.com/VHeimhuber/EntranceSat/blob/main/readme_files/1989-02-20-23-14-27_L5_DURRAS_swir_based_ABexp_%2025_XBexp_%2025_MAdis_60.png)

**Note**: When you run the SDS_entrance.automated_entrance_paths function, the results are automatically written to a newly established subdirectory, which is named based on some of the parameters provided in settings_entrance. In the postprocessing step, this data is then read-in again based on the parameters provided in settings_entrance. This is done to enable multiple pathfinding parameter configurations to be run and tested.

#### Step 5: Post-processing:

This is a sequence of operations used for ultimately inferring open vs. closed entrance states for the full imagery record based on the training data. Several result files will be written out here including three result plots and a csv file with the processed result data.

###### 5.1 Setup the postprocess_params parameters for post processing and load all required datasets.

The key parameter here is 'spectral_index', which is used for calculating the delta-to-median parameter. Generally, it is recommended to use either NIR in path finding and NDWI for this step, or SWIR in pathfinding and mNDWI for this step. Single bands are not recommended for this step, since they are not very stable over the beach berm and therefore, don't lead to a good 'baseline' for calculating the delta-to-median parameter. The AB_intersection_search_distance is used for locating the minimum or maximum of the along-berm and across-berm spectral transects on either side of their intersection. The choice for this parameter depends on the size of an entrance and the presence/absence of confounding features near the seed and receiver points.

At the beginning of Step 5, all result files (e.g. pickle files) are read in from the result directory that was established automatically during step 4. This means that you only need to run step 4 once and can then pickup where you left off, simply by skipping the SDS_entrance.automated_entrance_paths function via placing a # at the beginning of this line of code. All other parts of the code still need to be run each time you perform an analysis.

##### 5.2 Identify optimal classification threshold and classify the image series into binary entrance states

Based on the data generated by the least-cost path finding step (Step 4), EntranceSat calculates the delta-to-median parameter for each image, which is indicative of the state of the entrance and/or size of a possible opening. The calculation of this parameter is illustrated in the following figure. Details are provided in the publication listed above and in the code itself.

![Alt text](https://github.com/VHeimhuber/EntranceSat/blob/main/readme_files/inferring_entrance_states_method_illustration.png)

After this threshold is identified, the full image series is classified into open vs. closed entrance states via SDS_entrance.classify_image_series_via_DTM

##### 5.3 Check detection (optional)

This is a quality control step. Here, the user has the ability to go over every single automated entrance state detection via a interactive pop-up window. Each image can either be kept or rejected via keyboard inputs. This can be used to get rid of possibly cloudy or otherwise problematic detections - to ultimately create a clean result time series. This step uses the SDS_entrance.check_entrance_state_detection function.

**Note**: You have to go through the entire image series for this step - subsets are not possible. For a first pass assessment or under limited time, it is recommended to skip this step.

##### 5.4 Divide the original data into open and closed entrance states and plot result figures

By running the SDS_entrance.plot_entrancesat_results function, a number of output datasets are generated and stored in a dedicated directory. These outputs include:

-Location plots of the identified least cost paths:
![Alt text](https://github.com/VHeimhuber/EntranceSat/blob/main/readme_files/Figure_1_mndwi_15-02-2021_L5_L7_L8_S2.png)
-Spectral Transect plots:
![Alt text](https://github.com/VHeimhuber/EntranceSat/blob/main/readme_files/Figure_2_mndwi_15-02-2021_L5_L7_L8_S2.png)
-Time series plots of the delta to median parameter:
![Alt text](https://github.com/VHeimhuber/EntranceSat/blob/main/readme_files/Figure_3_mndwi_15-02-2021_L5_L7_L8_S2.png)
-CSV files with all the processed data.
-Shapefile containing all of the least-cost paths corresponding to open entrances.


That's it - you have now successfully used the EntranceSat toolbox to analyse the entrance state of an intermittent estuary or a similar landform.
