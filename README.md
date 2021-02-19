# quetzal_germany
This open source project is a macroscopic transport model for the region of Germany. It supports research aimed at designing an integrated, renewable energy system with mobility behaviour insights.

It uses the quetzal transport modelling framework: https://github.com/systragroup/quetzal

The method is oriented towards classical four-step transport modelling. Focus lies on mode choice by using a purpose-segmented nested logit model.

## Structure

The directory structure is straight-forward:
> quetzal_germany/
> -- input/
> -- input_static/
> -- model/
> -- output/

While input and output data as well as (temporary) model files are stored in seperate folders, Jupyter Notebooks contain all data management and modelling. Briefly, they are structured as follows:
* ``prep1X``: Generation of all transport networks in high resolution
* ``prep2X``: Aggregation of PT network graph and connection to transport demand sources and sinks
* ``prep3X``: Calculation of shortest paths and enrichment with performance attributes
* ``calX``: Generation of calibration dataset and estimation of parameters
* ``model2X``: Generation of OD matrix
* ``model3X``: Mode choice modelling
* ``model4X``: Route assignment
* ``valX``: Validation of the model results

## Usage

### Installation

1. Create a virtual environment for quetzal models: Clone the quetzal package into a local folder and create a virtual environment as described here: https://github.com/systragroup/quetzal
2. Clone this repository into a local folder: In your terminal, navigate to the position where you want to store the code. Type `git clone <this repo's URL>`. Navigate into the folder `quetzal_germany`.
3. Download static input files from Zenodo[^1]:
3.1. Download the whole data package into a folder named `input_static/` within the `quetzal_germany` repository
3.2. Move the file `links.geojson` from `input_static/` to `model/de_pt_network_bus/`
4. Activate your quetzal environment
5. Open the local project in Jupyter Notebook (in your terminal type `jupyter notebook`) and start running the notebooks

[^1]: If you wonder why these files are not hosted in this very repository: Large input data files require different handling and some of them also require a license different to this repo's licensing.

### First model run

The repository comes with the transport networks but without level-of-service (LoS) attributes assigned to origin-destination relations. This is because LoS attribution is strongly assumption-driven and the LoS shortest path table is too large to be hosted online efficiently. Thus, you need to create LoS tables for road and PT connections by running all `prep3X` notebooks and adjusting the assumptions made, if needed. However, adjustments to LoS attributes must be consistent with assumptions made for calibration data (`calX` notebooks).

After creating LoS tables you are able to run the mode choice model. If you don't have access to travel demand data (see below), you can only run `model3X` - mode choice. This will output mode choice probabilities, which you can then visualise using the `valX` notebooks.

### Example for custom region

Notebook `prep10` creates the four step model (always abbreviated with `sm`) with a zones table that you specify. By default, it contains all NUTS3 zones of Germany, but you can limit it to the desired region or refine it with higher resolution data.

Notebooks `prep11` to `prep15` create road and PT networks from openstreetmap and German-wide GTFS feeds, respectively. They will be saved in `sm.road_links/sm.road_nodes` and `sm.links/sm.nodes`, respectively. Additionally, a list `sm.pt_route_types` is created. Make sure you uncomment the cell where you spatially restrict the network graph, if you want a smaller region. Notebook `prep16` creates distances from all population points in the latest census to your PT stops. Make sure to spatially restrict this one too.

The rest works straight-forward with the notebooks' comments and should work for every self-defined region. At the end of each notebook in the 'save' cell you find all DataFrames (as `sm`'s attributes) that will be relevant in later steps. One additional attribute is always present: `sm.epsg` which defines the coordinate reference system.

### Data accessibility

This repository together with externally hosted data packages contains all openly licensed data sources which are necessary for mode choice modelling in Germany.

Though, for estimating calibration parameters (beyond the estimation results given in `input/`) and assignment of absolute traffic flows, there are additional data sources needed:
* Calibration uses a large national survey of mobility behaviour "[Mobilität in Deutschland 2017](http://www.mobilitaet-in-deutschland.de/) B2" (MiD2017)
* Assignment uses modified origin destination matrices from the underlying model of the German federal governments transport study "[Bundesverkehrswegeplan 2030](https://www.bmvi.de/SharedDocs/DE/Artikel/G/BVWP/bundesverkehrswegeplan-2030-inhalte-herunterladen.html)"

You can apply for access to both data sets using the national [Clearing House Transport order form](https://daten.clearingstelle-verkehr.de/order-form.html)

## Next steps:

* Calculation of mode choice elasticities
* Scenario modelling
* Linkage to an energy system model
