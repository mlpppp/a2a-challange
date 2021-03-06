# A2A Challange - An Introduction

The content of this repository is part of a project submitted for a challenge organized by A2A Group, the italian multiutiliy leader, that aims to find a good way in order to manage turns of a fleet of vehicles for the trash cleaning.
In order to reach our objectives we followed the strategy of **Cluster-first** - **Route-second** for the Vehicle Routing Problem, VRP, wich executes a 2-steps solutions. In the first step we made a Geo-Spatial Clustering and for the second step a Routing optimization using the Travelling Salesman Problem, TSP, solution. TSP is a semplification of the VRP and it allows us to tract the VRP in a more suitable way, from the computational point of view.

# Contributors 
Thanks goes to these wonderful people <img src="https://cdn1.iconfinder.com/data/icons/addiction-drugs-2/24/addiction_Coffee_1-24.png" />
<table>
    <tr>
        <td>Bryan Christian Murrone</td>
        <td><img src="https://cdn3.iconfinder.com/data/icons/technology-1-1/512/technology-machine-electronic-device-23-16.png"/> ideas, clustering validation</td>
    </tr>
    <tr>
        <td>Marco Distrutti</td>
        <td><img src="https://cdn2.iconfinder.com/data/icons/business-process-1/512/epc-16.png"/> coding, algorithms, routing engine</td>
    </tr>
    <tr>
        <td>Santosh Anand</td>
        <td><img src="https://cdn2.iconfinder.com/data/icons/miscellaneous-37/100/improving_quality_chart-16.png"/> coding, results reviewer</td>
    </tr>
</table>

# Prerequisites

In order to execute all scripts in this repository you need python with ortools and scikit learns packages. The EqualSize KMeans implementation is imported by the following github repository: [ndanielsen/Same-Size-K-Means](https://github.com/ndanielsen/Same-Size-K-Means), the suggested version of scikit learn to install is 18.X, we successfully tested it with the version 21.X

```sh
    $ pip install ortools
    $ pip install 'scikit-learn>=0.18.0,<=0.21.3'
```

# Repo structure

- **input folder**: follow the instructions inside this folder. Important input source for the data preparation and clustering algorithms
- **output folder**: 
    - **data-preparation**: all the results of the data preparation step. In this step a python script executes the split division between vehicles turns and creates first_visit.<YYYYmmdd>.csv and second_visit.<YYYYmmdd>.csv
    - **clustering**: all the results of a clustering execution are organized in files, whose prefix is the name of the algorithm concatenated with the number of cluster produced, for each clustering results the produced files are a clustered dataframe, centroids csv, tsp result statistics (with the waypoints to use in the map) and silhouette coefficent.
- **plot**: contains combined results of clusters of size from 15 to 20 for each clustering algorithm.
- **routing_map**: this folder contains a very simple javascript application using **Leaflet Routing Machine** in order to compute the waypoints produced by the TSP algorithm and present them a the map. In the last paragraph is explained better. 
- **root folder**: 
    - file names **a2a_*** indicate our custom python modules used by the Python Notebooks and serve features in different areas:
        - **a2a_clustering**: executes dataframe transformation for the scikit-learn training models, computes clusters centroids and contains a custom implementation of the Sweep Algorithm (a clustering based on the polar angles started from the DEPOT)
        - **a2a_validation**: executes the silhouette coefficent for each observation in the clusterized dataframe and computes the coefficent average between all clusters. All the results are presented in form of a graph splitted in two sections: silhouette trend and a 2D representation of the clustering. [Inspired solution from scikit-learn documentation](https://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_silhouette_analysis.html)
        - **a2a_travellingsalesman.py**: executes the TSP problem and produce a pandas datarame with all statistics organized for aech cluster, we deeply used the google [ortools](https://developers.google.com/optimization/introduction/overview) in order to execute this algorithm
    - file names **clust_*** indicate jupyter notebooks with a very simple machine learning pipeline in which we execute the clustering first, and in a second step the TSP to the clusterized dataframe. These scripts are responsible even to store all the steps results (clusterized dataframe, centroids, silhouette charts, tsp statistics dataframe) in the output folder with the following file prefix: <algorithm_name>_<n_clusters>_<result_type>

# Distance matrix

We computed the distance matrix using the travel distances. The reason why we computed this distance matrix with the travel distance is for the Route Optimization part that aims to find an ordered set of geographical nodes, the route, in respect of the real distances given from the streets routability. After then, we putted all the results on a map using a Routing Engine, as a **Routing Server**, and the **Leaflet Routing Machine** as a frontend Javascript library for the map routes renderizations.

**Distance Matrix** estimations are made by the **Open Source Routing Machine** project developing a custom NodeJS (**NodeJS** v8.0.0 & **node-osrm** v5.15.0) application that executes all the routes estimations between all pairs using the Route Service API provided by the OSRM node package. For larger datasets the fastest computation provided by the Table API, executed with the whole geo spatial dataset in a single call, is more suitable in order to scale well with environments, in contrast it has a less accuracy in terms of estimations and the only way to execute an optimization based on distance in meters, without the routability heuristics, is to set a custom configuration with the speed of vehicles constants in all routes, which causes an unreliability in time estimates. With 3148 nodes (3147 bins and 1 depot) we decided to use the Route API.

The produced file is a csv that will be loaded by pandas dataframes: ./input/distance_duration_matrix.csv

The distance/duration matrix contains for each pairs, in the geospatial graph, a pair composed by **(meters,sedonds)**.

# Travelling Salesman Problem

We used the Google ORTools for the TSP computation and the implementation resides in the **a2a_travellingsalesman.py**, this module contains some methods for the distance matrix reading and various helper for the final solution. The main procedure is the **solve_tsp** that accept a dataframe with all observations clusterized (a column "Cluster_label" in the dataframe have to describe it) and returns a new dataframe with the statistics for each clusters:

 - **total meters**: both in decimal and string format
 - **total time of travelling**:which time is needed to execute all the computed path
 - **total time of emptying**:which time is needed to execute all the computed path plus 60seconds for each bins
 - **number of bins**
 - **waypoints**: an array of json representation of the path nodes (see the next paragraph for their usage)
 - **TOTALS** and **AVERAGE** for all stats

# Routing Engine and Map

For the "Routes on Map" activity we used the OSRM Routing Engine configured in the same way we created the distance matrix. A very usefull docker container which saves our time was the [osrm-backend](https://hub.docker.com/r/osrm/osrm-backend/) runned with the Contraction Hierarchies technique on the data provided by [GeoFabrik Open StreetMap datasource](https://download.geofabrik.de/europe/italy.html).

After this container installation and using the waypoints given from the TSP produced Dataframe is very simple to render the routes on the map. We used **Leaflet Routing Machine** for the frontend renderization.

<img src="https://github.com/kayne87/a2a-challange/blob/master/output/routes/path.PNG?raw=true" />

<img src="https://github.com/kayne87/a2a-challange/blob/master/output/routes/path_with_directions.png?raw=true" />

# Configure the Routing Engine with a custom map

In the following snippet, from ./routing_map/index.js, you have to replace the **waypointsRaw** variable content with any waypoints results from the tsp csv files and have an OSRM listening on port 5000, you can configure even this from the same script.

```js
    var waypointsRaw = [{"serial": -1, "coords": [45.5069182, 9.2684501]},...{...}];
    L.tileLayer.provider('Hydda.Full')
    var waypoints = []
    var counter = 0
```

In the output/route folder we have uploaded 4 paths created in this way.
