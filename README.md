# quetzal_germany

This open source project is a macroscopic passenger transport model for the region of Germany. It supports research aimed at designing an integrated, renewable energy system with mobility behaviour insights. The reference publication can be found here: https://doi.org/10.1186/s12544-022-00568-9

It uses the quetzal transport modelling framework: https://github.com/systragroup/quetzal

## Structure

The method is oriented towards classical four-step transport modelling.

![Structure of quetzal_germany](input/quetzal_germany_structure_chart.PNG "Structure of quetzal_germany")

The directory structure is as follows:
> quetzal_germany/</br>
> -- input/</br>
> -- input_static/</br>
> -- model/</br>
> ---- base/</br>
> ---- scenarioX/</br>
> -- notebooks/</br>
> ---- log/</br>
> -- output/</br>
> ---- base/</br>
> ---- scenarioX/</br>

While input and output data as well as (temporary) model files are stored in seperate folders, Jupyter Notebooks contain all data management and modelling. Briefly, they are structured as follows (`X` as wildcard):
* ``prep1X``: Generation of transport demand zones and all transport networks in high resolution
* ``prep2X``: Aggregation of PT network graph and connection to transport demand sources and sinks
* ``prep3X``: Calculation of shortest paths and enrichment with performance attributes for PT and cars, respectively
* ``prep4X``: Data preparation for generation and destination choice models
* ``calX``: Generation of calibration dataset and estimation of demand model parameters (only applicable with access to calibration data (see below))
* ``model_generation_X``: Trip generation choice for non-compulsory trips either from exogenous data (MiD2017) or endogenously with a logit model
* ``model_destination``: Destination choice for non-compulsory trips
* ``model_volumes_X``: Generation of OD matrix either from exogenous data (VP2030) or endogenously from previous steps
* ``model_mode``: Mode choice and calculation of composite cost for generation and destination choice
* ``model_assignment``: Route assignment and results validation
* ``model_inner-zonal``: Calculation of transport system indicators for inner-zonal traffic
* ``post_X``: Calculation of emissions or energy demand for the entire passenger transport sector
* ``00_launcher``: Automatically runs all preparation and modelling steps in order
* ``00_test_environment``: Run it to see whether your virtual environment is properly set up
* ``val_X``: Various validation notebooks for model results

All scenario parameters are saved in the `input/parameters.xls` file. Other mallable input files are located in the same folder, while unchanged input data sits in `input_static/`.

## Installation

1. Create a virtual environment for quetzal models. Choose one of these methods:
   + Either clone the quetzal package into a local folder and create a virtual environment as described here[^1]: https://github.com/systragroup/quetzal
   + Or create a virtual environment manually with all dependencies using conda:
     * Create an environment with the desired python version, e.g.: `conda create -n quetzal python=3.10`
	 * Activate this environemnt (`conda activate quetzal`)
	 * Install neccesary dependencies: `conda install -c conda-forge geopandas contextily osmnx geopy rtree notebook matplotlib xlsxwriter cython numba scikit-learn scipy xlrd tqdm ray-default pytables sqlalchemy openpyxl==3.0.7`
	 * Install quetzal dependencies that are not available on conda for python > 3.6: `pip install simpledbf biogeme`
	 * Clone the quetzal repository to a desired location (`git clone https://github.com/systragroup/quetzal`) and install it as development version `pip install -e quetzal/`
2. Activate your quetzal environment, if not done yet
3. Clone this repository into a local folder: In your terminal, navigate to the position where you want to store the code. Type `git clone <this repo's URL>`. Navigate into the folder `quetzal_germany`.
4. Download static input files from Zenodo[^2] into a folder named `input_static/` within the `quetzal_germany` repository (see directory structure): [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4518680.svg)](https://doi.org/10.5281/zenodo.4518680)
5. Open the local project in Jupyter Notebook (in your terminal type `jupyter notebook`) and start running the notebooks

You can test your virtual environment by running the `00_test_environment` notebook. Read potential error messages and install concerning packages using conda, or refer to the [quetzal issues](https://github.com/systragroup/quetzal/issues) page to see if this error has occured before.

[^1]: If you face problems importing geopandas, consider uninstalling package `rtree` and reinstalling a version up to 0.9.3 (`conda install -c conda-forge rtree=0.9.3`) or browsing through quetzal [issues](https://github.com/systragroup/quetzal/issues) for discussions on installation.
[^2]: Large input data files are not hosted in this very repository, as they require different handling in git and licensing.

## First model run

You can execute your first model run from the `00_launcher` notebook by running only `prep10`, `prep3X` and `model_X` notebooks in the order provided there. Note: If you want to run the model on a laptop with at least 8GB RAM, you should sparsify the OD set or use the NUTS3-level zoning system. Please read the cooresponding section in `00_launcher`. Otherwise, you'll need 80GB RAM.

This repository (together with static input data) contains road and public transport (incl. air) networks ready for simulation, and estimation results for the demand models. It also includes OSM data from `prep4X` notebooks. Thus, you can simply generate the level-of-service (LoS)-shortest-paths-stack and then run the classic transport modelling steps. However, the order shifts, as you need the composite cost from mode choice in generation and destination choice steps (see order in `00_launcher`).

Running notebooks with the `00_launcher` generates log files for printed output and error messages, respectively. These can be found under `notebooks/log/`.

You can adjust all assumptions in the `parameters.xls` file, if you want to simulate an alternative transport system (see Scenarios).

Detailed descriptions what the notebooks do are to be found as comments. Briefly: Your StepModel object (always abbreviated with `sm`) is where the magic happens. It saves all tables as attributes (pandas `DataFrame`s) and provides all transport modelling specific functions from the quetzal library. Quetzal provides wrapper function for classic steps in aggregated transport modelling (trip generation, assignment, etc.), which execute a set of more specific functions. Due to a higher degree of customisation, this model uses quetzal's specific functions in many places.

## Results

Results of the transport model are computed in `model_assignment`. If you have access to validation data, this notebook will also visualise validation plots. Files and figures are saved under the respective scenario name in `output`.

Some relevant results are aggregated numbers that are printed in the corresponding model step (e.g. average yearly mileage per car in `model_assignment`). These statements can be found in the output log file under `/notebooks/log/out_<scenario name>_<notebook name>.txt`

## Scenarios

You can define own scenarios "the quetzal way": Open the `parameters.xls` file and add a new column with your scenario name. Name it under "general/description" and refer to "base" as a "general/parent" scenario. All values, which you don't change in your new column are taken from the parent column.

You can now adjust parameters and run the model with new values. To do so, either use the `00_launcher` by typing your scenario name (column name in `parameters.xls`) in the list of scenarios (fourth cell). All scenario names in this list will be executed in parallel. The other option is running the notebooks manually and defining the variable `scenario` to your name (very first cell).

## Network generation and example for custom region

Notebook `prep10` creates the four step model (`sm`) with a zones table that you specify. By default, it contains all NUTS3 zones of Germany, but you can limit it to the desired region or refine it with higher resolution data. The disaggregated notebook uses an aggregation of "Gemeindeverband"-zones, which constitute the default model.

Notebooks `prep11` to `prep14` create road and PT networks from OpenStreetMap and German-wide GTFS feeds, respectively. They will be saved in `sm.road_links`/`sm.road_nodes` and `sm.links`/`sm.nodes`, respectively. Additionally, a list `sm.pt_route_types` is created. Make sure you uncomment the cell where you spatially restrict the network graph, if you want a smaller region. Notebook `prep15` creates distances from all population points in the latest census to your PT stops (make sure to spatially restrict this one too). This data is used to parametrise PT access and egress links between zone's demand centroids and transport networks.

Notebooks `prep2X` aggregate your network and create access/egress links between zones' demand centroids and the PT stops or road nodes, respectively. There are two methods used for PT network aggregation, which is necessary in order to reduce computation time for path finders and all other methods:
* Clustering short-distance stops
* Aggregation of PT network to relevant trips and stops with simultaneous connection to zone centroids (size and quality of the network depend on your definition of 'relevant')
* Subsequently, the road network gets connected

The rest works straight-forward with the notebooks' comments and should work for every self-defined region with minor adjustments. At the end of each notebook in the 'save' cell(s) you find all DataFrames (as `sm`'s attributes) that will be relevant in later steps. One additional attribute is always present: `sm.epsg` which defines the coordinate reference system.

## Data accessibility

This repository together with externally hosted data packages contains all openly licensed data sources which are necessary for transport modelling in Germany.

Though, for estimating calibration parameters anew (beyond the those given in `input/`), you need a German-wide mobility survey with trips on "Gemeindeverband"-level: "[Mobilität in Deutschland 2017](http://www.mobilitaet-in-deutschland.de/) B2" (MiD2017)

Optionally, you can generate the origin destination matrix exogenously, using origin destination matrices from the underlying model of the German federal governments transport study "[Bundesverkehrswegeplan 2030](https://www.bmvi.de/SharedDocs/DE/Artikel/G/BVWP/bundesverkehrswegeplan-2030-inhalte-herunterladen.html)".

You can apply for access to both data sets using the national [Clearing House Transport order form](https://daten.clearingstelle-verkehr.de/order-form.html) . All `csv` and `xlsx` data tables go into the folder `input/transport_demand`, which added to the `.gitignore`.

