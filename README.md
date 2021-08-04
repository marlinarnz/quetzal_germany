# quetzal_germany
This open source project is a macroscopic transport model for the region of Germany. It supports research aimed at designing an integrated, renewable energy system with mobility behaviour insights.

It uses the quetzal transport modelling framework: https://github.com/systragroup/quetzal

The method is oriented towards classical four-step transport modelling. Focus lies on mode choice by using a purpose-segmented nested logit model.

## Structure

![Structure of quetzal_germany](input/quetzal_germany_structure_chart.pdf "Structure of quetzal_germany")

The directory structure is straight-forward:
> quetzal_germany/</br>
> -- input/</br>
> -- input_static/</br>
> -- model/</br>
> -- output/</br>

While input and output data as well as (temporary) model files are stored in seperate folders, Jupyter Notebooks contain all data management and modelling. Briefly, they are structured as follows (`X` as wildcard):
* ``prep1X``: Generation of transport demand zones and all transport networks in high resolution
* ``prep2X``: Aggregation of PT network graph and connection to transport demand sources and sinks. You can either cluster stops and connect the clusters to zone centroids or aggregate the PT network to relevant trips and stops
* ``prep3X``: Calculation of shortest paths and enrichment with performance attributes for PT and cars, respectively
* ``calX``: Generation of calibration dataset and estimation of parameters (only applicable with access to calibration data (see below))
* ``model_volumes``: Generation of OD matrix (steps trip generation and distribution together; only applicable with access to volume data (see below))
* ``model_logit``: Mode choice
* ``model_assignment``: Route assignment and results validation
* ``model_emissions``: Calculation of emissions from German passenger transport (post-processing; only applicable with access to inner-zone data (see below))

## Usage

### Installation

1. Create a virtual environment for quetzal models: Clone the quetzal package into a local folder and create a virtual environment as described here *[1]*: https://github.com/systragroup/quetzal
2. Activate your quetzal environment, if not done yet
3. Clone this repository into a local folder: In your terminal, navigate to the position where you want to store the code. Type `git clone <this repo's URL>`. Navigate into the folder `quetzal_germany`.
4. Download static input files from Zenodo *[2]* into a folder named `input_static/` within the `quetzal_germany` repository (see Structure): [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.5159308.svg)](https://doi.org/10.5281/zenodo.5159308)
5. Open the local project in Jupyter Notebook (in your terminal type `jupyter notebook`) and start running the notebooks

*[1]*: If you face problems importing geopandas, consider uninstalling package `rtree` and reinstalling a version up to 0.9.3 (`conda install -c conda-forge rtree=0.9.3`) or uninstalling the whole environment and reinstalling it with the requirements file posted in this [issue](https://github.com/systragroup/quetzal/issues/45).
*[2]*: If you wonder why these files are not hosted in this very repository: Large input data files require different handling and some of them also require a license different to this repo's licensing.

### First model run

This repository (together with static input data) contains road and public transport (incl. air) networks, aggregated networks, and estimation results for the choice model. Thus, you can simply execute all `prep3X` notebooks to generate the level-of-service (LoS)-shortest-paths-stack and then run the model notebooks in order of classic transport modelling (generation, distribution, mode choice, assignment). You can adjust network-related assumptions in the `prep30` notebooks, if you want to simulate an alternative transport system. 

Detailed descriptions what the notebooks do are to be found as comments. Your StepModel object (always abbreviated with `sm`) is where the magic happens. It saves all tables as attributes (pandas `DataFrame`s) and provides all transport modelling specific functions from the quetzal library. Quetzal provides wrapper function for classic steps in aggregated transport modelling (trip generation, assignment, etc.), which execute a set of more specific functions. Due to a higher degree of customisation, this model mostly uses quetzal's specific functions in many places.

If you don't have access to travel demand data (see below), you can only run `prepX` notebooks and `model_logit` - mode choice.

### Example for custom region (or new generation of networks)

Notebook `prep10` creates the four step model (`sm`) with a zones table that you specify. By default, it contains all NUTS3 zones of Germany, but you can limit it to the desired region or refine it with higher resolution data.

Notebooks `prep11` to `prep14` create road and PT networks from OpenStreetMap and German-wide GTFS feeds, respectively. They will be saved in `sm.road_links`/`sm.road_nodes` and `sm.links`/`sm.nodes`, respectively. Additionally, a list `sm.pt_route_types` is created. Make sure you uncomment the cell where you spatially restrict the network graph, if you want a smaller region. Notebook `prep15` creates distances from all population points in the latest census to your PT stops (make sure to spatially restrict this one too). This data is used to parametrise PT access and egress links.

Notebooks `prep2X` aggregate yor network and create access/egress links between zones' demand centroids and the PT stops or road nodes, respectively. There are two methods available for PT network aggregation, which is necessary in order to reduce computation time for path finders and all other methods:
* Clustering short-distance stops and then creating access/egress links (efficient computation, but information loss in network connectivity and travel time)
* Aggregation of PT network to relevant trips and stops with simultaneous connection to zone centroids (size and quality of the network depend on your definition of 'relevant')

The notebooks are named correspondingly (`prep20` and `prep21`).

The rest works straight-forward with the notebooks' comments and should work for every self-defined region with minor adjustments. At the end of each notebook in the 'save' cell(s) you find all DataFrames (as `sm`'s attributes) that will be relevant in later steps. One additional attribute is always present: `sm.epsg` which defines the coordinate reference system.

### Results

Results of the inter-zonal model (NUTS3 resolution) are computed and validated in `model_assignment`, while results for whole Germany need inner-zonal traffic as well. This is computed under use of restrictively licensed data (see below; notebook `model_assignment_inner-zonal`). Combined results, which are eligible for comparison to other studies, are created in `model_emissions`.

### Data accessibility

This repository together with externally hosted data packages contains all openly licensed data sources which are necessary for transport modelling in Germany.

Though, for estimating calibration parameters (beyond the estimation results given in `input/`) and assignment of absolute traffic flows, there are additional data sources needed:
* Calibration uses a large national survey of mobility behaviour "[Mobilität in Deutschland 2017](http://www.mobilitaet-in-deutschland.de/) B2" (MiD2017)
* Assignment uses modified origin destination matrices from the underlying model of the German federal governments transport study "[Bundesverkehrswegeplan 2030](https://www.bmvi.de/SharedDocs/DE/Artikel/G/BVWP/bundesverkehrswegeplan-2030-inhalte-herunterladen.html)"

You can apply for access to both data sets using the national [Clearing House Transport order form](https://daten.clearingstelle-verkehr.de/order-form.html) . All `csv` and `xlsx` data tables go into the folder `input/transport_demand`, which added to the `.gitignore`.

