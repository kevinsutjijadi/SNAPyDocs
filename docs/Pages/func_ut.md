---
title: Utilities
description: Spatial Network Analysis for Python Module
---

# Utility functions

utilities in geometry and processing, with parts that help on data preparation and several additional related utilitary functions.

### Data Preparations <i>Functions</i>

#### NetworkSegmentIntersections <i> func </i>
`#!python NetworkSegmenIntersections(df, dfi=None, EndPoints=True, tol=1e-3)`

:   returns 
    :   ndf : GeoDataFrame of segmented network
        pts : GeoDataFrame of endpoints and intersections

    Segments network lines on each intersection points as a part of data preparation for GraphSims class.

    !!! info "result pts"
        this result compiles all end points of segmented network lines, appending a new field named 'intersection' as boolean, with True values representing a point that have multiple segmented line endpoints and False as just a singular end point/dead end.
    
    !!! warning "processing duration"
        function does not have built in multithread processing, but as most of the process are vectorized numpy operations, it can run somewhat efficiently.

##### Parameters

:   <b>df</b> : GeoDataFrame <i>required</i>
    :   Geopandas GeoDataFrame object, of unsegmented network.

:   <b>dfi</b> : GeoDataFrame default:None
    :   Geopandas GeoDataFrame object, of unsegmented network, used for inserting different set of network to segment the df variable with. If there are some type of line defined by a field on df GeoDataFrame, please filter it first. If left None, df will use itself as the segmenter.

:   <b>dfi</b> : GeoDataFrame default:None
    :   Geopandas GeoDataFrame object, of unsegmented network, used for inserting different set of network to segment the df variable with. If there are some type of line defined by a field on df GeoDataFrame, please filter it first. If left None, df will use itself as the segmenter.

:   <b>EndPoints</b> : Boolean default:True
    :   Switch for the function to calculate and return pts. If False the pts result will return an empty list.

:   <b>tol</b> : float default:1e-3
    :   Tolerance level for nearby points considered as an intersection/meet.

##### Use Example
:   
    ```python
    import SNAPy as sna
    import geopandas as gpd

    dfNetwork = gpd.read_file('testdata\\Network.gpkg') # network dataframe

    dfNetworkSg, IxPts = sna.NetworkSegmenIntersections(dfNetwork)

    dfNetworkSg.to_file("SegmentedNetwork.gpkg", layer="Network", crs="EPSG:32748", driver='GPKG') # geoDataFrame can be saved to GIS files
    dfNetworkSg.to_file("SegmentedNetwork.gpkg", layer="Points", crs="EPSG:32748", driver='GPKG')

    nwSim = sna.GraphSims(dfNetworkSg, dfEntries, settings) # segmented lines to Graphsims
    ```

#### NetworkSegmentDistance <i> func </i>
`#!python NetworkSegmentDistance(df, dist:float=50.0)`

:   returns 
    :   ndf : GeoDataFrame of segmented network

    Segments network lines to an approximate length according to projection units. I.e. a "metre" unit projection, where a line is 150m, with "dist" distance variable of 50m, it will seperate the line into 3 segments.

    !!! info "Minimum length and segmentation rounding"
        line lengths less than 1.5x than the segment distance will not be segmented. Segmentation rounding uses modulus operation to determine number of segments.
    
    !!! warning "Does not segment intersection"
        Recommended to use this function AFTER using NetworkSegmentIntersection. Note that segmenting network into smaller segments will cause some/major computing time for any analysis.

##### Parameters

:   <b>df</b> : GeoDataFrame <i>required</i>
    :   Geopandas GeoDataFrame object, of a network.

:   <b>dist</b> : float default:50
    :   float distance for base network segment distance.


##### Use Example
:   
    ```python
    import SNAPy as sna
    import geopandas as gpd

    dfNetwork = gpd.read_file('testdata\\Network.gpkg') # network dataframe

    dfNetworkSg, IxPts = sna.NetworkSegmenIntersections(dfNetwork)

    dfNetworkSg2 = sna.NetworkSegmenIntersections(dfNetworkSg, 100)

    dfNetworkSg2.to_file("SegmentedNetwork.gpkg", layer="Network", crs="EPSG:32748", driver='GPKG') # geoDataFrame can be saved to GIS files

    nwSim = sna.GraphSims(dfNetworkSg, dfEntries, settings) # segmented lines to Graphsims
    ```


<br><br>
@February2024