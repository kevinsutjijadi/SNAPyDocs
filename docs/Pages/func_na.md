---
title: Network Analysis
description: Spatial Network Analysis for Python Module
---

# Spatial Network Analysis Functions

Network Analysis functions

## GraphSims   <i> class </i>
`#!python GraphSims(NetworkDf:GeoDataFrame, EntriesDf:GeoDataFrame, **kwargs)`

:   The core process for spatial network analysis is operated/structured within the `#!python Graphsims` Class, which reads Geopandas Geodataframe for entries and network data. The class initialization prepackages and compiles further processing and its subsequent results. main class for network analysis processing, initialization stage will have some heavy processes included. the processes in order
    
    - checking if network and entrances are the same projection and is_geographic (in meters)
    - checking geometry type (linestring for network and points for entries. if network has multilinestring, it will convert the layer to linestring)
    if the input data does not meet required prerequisite above, the initialization process will stop and raise exceptions.

    - initial compiling of network nodes (junction points), if more than 30% of nodes are deadends, initialization process will offer to segment lines on intersections (SNAPy.NetworkSegmentIntersections(NetworkDf))
    - adding Feature-ID of network and entrances as attribute if not found
    - building graph on SGACy (Spatial Graph Analisis in Cython)
    - building entries data, appending with entrance/relation to graph related information.
    - appending point coordinate as attributes unto EntriesDf



    ##### Parameters

    :   <b>NetworkDf</b> : GeoDataFrame
        :   Geopandas geodataframe of network data, containing edges in LineString data form. Can contain weight multiplier for costs and also vertex classification; with the default value for cost is geometrical length. 

    :   <b>EntriesDf</b> : GeoDataFrame
        :   Geopandas geodataframe of Entries data, containing nodes in Point data form.

    :   <b>**kwargs</b> : Dict/keys-values
        :   Parameter arguments, that contain:

            - `#!python "EntDist" float `   default 100.0

                search distance to closest edge of network, further distanced entrances will not be accounted
            
            - `#!python "EntID" string `   default "fid"

                string of attribute/column name for Entrance Feature ID. If not found, will be automatically made in initialization process

            - `#!python "EdgeID" string `   default "fid"

                string of attribute/column name for Network Feature ID. If not found, will be automatically made in initialization process
            
            - `#!python "AE_Lnlength" string `   default None

                string of attribute/column name for Network cost multiplier, if None, all edges will use the value of its geometrical length
            
            - `#!python "AE_LnlengthR" string `   default None

                string of attribute/column name for Network cost in reverse multiplier, if None, all edges will use the value of 'AR_Lnlength'
            
            - `#!python "AE_EdgeCost" string `   default None

                Additional cost multiplier, if None will revert to 1.0
            
            - `#!python "Threads" int `   default 0

                specify threads number for multiprocessing. For single core processing, input 1. The default value is 0, if which, will be replaced with (CPU count - 1), which almost fully utilizes CPU processing.
            
            - `#!python "SizeBuffer" float `   default 0.05

                buffer multipler for edges and nodes array size in GraphCy, only important if additional addition to edges or nodes are done during process.

    ##### Use Example
    :   
        ```python
        import SNAPy as sna
        import geopandas as gpd

        dfNetwork = gpd.read_file('testdata\\NetworkClean.shp') # network dataframe
        dfEntries = gpd.read_file('testdata\\Features.gpkg', layer='Features') # entrance dataframe

        nwSim = sna.GraphSims(dfNetwork, dfEntries) # main class for loading network data
        ```

### GraphSims <i>Items</i>

:   #### .baseSet <i> dict </i>

    :   Dictionary of settings

        Dictionary of processing settings used in initialization or other functions. Complete elements of the dictionary can be read/found at class parameter description.

:   #### .NetworkDf <i> GeoDataFrame </i>

    :   GeoDataFrame of network from input, with parity with edges data in SGACy graph.

        Functions that have results on edges such as .BetweenessPatronage, etc will save their results in a specified field name. Which then the variable can be obtained and modified as a GeoDataFrame, which can be edited, saved, filtered.

:   #### .EntriesDf <i> GeoDataFrame </i>

    :   GeoDataFrame of entries from input. From initialization process or reparametrization will be automatically appended intersection point coordinate as 'xPt_X' and 'xPt_Y', and 'xLn_ID' for connected edge id.

        built-in entries point based functions such as .Reach, etc will save their results in a specified field name. Which then the variable can be obtained and modified as a GeoDataFrame, which can be edited, saved, filtered.


:   #### .Gph <i> Graph </i>

    :   networkx Graph object from class initialization or reparametrization

:   #### .NodeDf <i> GeoDataFrame </i>

    :   GeodataFrame of network nodes, with the attribute 'JunctCnt', interger, of number of junctions/connections with edges of network. Will automatically initialize if used NetworkSegmentIntersections, but will be None initially. To access please use GraphSims.getNodes().

:   #### .pdkLayers, .pdkLyrNm, .pdkCenter

    :   Pydeck related values, see more at [visualisation related variables](../func_vis/#related-itemsvalues-in-graphsims-to-mapping)


### GraphSims <i>Functions</i>

:   #### .BetweenessPatronage <i> func </i>
    `#!python BetweenessPatronage(self, OriID:Tuple=None, DestID:Tuple=None, **kwargs)`

    :   returns self.NetworkDf : GeoDataFrame <br>
        <i>GraphSims.NetworkDf with the specified results column</i>

        Function for betweeness patronage, calculating segment traffic weighting from origin capacity, distributed through weightable destinations within the network in a set distance. Further elaboration on the method can be accessed on [method documentation](../methods/#betweeness-patronage-functions).
        Output appended on network/edges data on self.NetworkDf, but results on entries-destination on self.EntriesDf can also be aqcuired with setting Include_Destination : True in kwargs.

        !!! info "result return"
            results will be appended to self.NetworkDf, therefore the results can be accesed later so that the function does not need to be assigned into a new variable.
        
        !!! info "origin-destination pairings"
            It is acceptable to have the same id in origins and destinations, as the algorithm will skip/pass the calculations if the origin and destination has the same id.
        
        !!! warning "processing duration"
            Further distances will require exponentially longer time. Function has built in multithreading capabilities using multiprocessing.Pool. Please take notice of self.baseSet['Threads'], as the value is also used in this function.

    ##### Parameters

    :   <b>OriID</b> : list, tuple, np.array, geoseries <i>default None</i>
        :   Set of registered IDs of entries acting as origins. Any iterable format that is compatible to geoDataFrame.isin() function. If None, all entries will be set as origins.

    :   <b>DestID</b> : list, tuple, np.array, geoseries <i>default None</i>
        :   Set of registered IDs of entries acting as destinations. Any iterable format that is compatible to geoDataFrame.isin() function. If None, all entries will be set as destinations.

    :   <b>**kwargs</b> : Dict/keys-values
        :   Parameter arguments, that contain:

            - `#!python "OriWgt" string `   default 'weight'

                name of the attribute from self.EntriesDf to access weight value for origin points. In the context of Betweeness Patronage, the value will represent the trip unit/person.
                There is a feature that skips any processing if the weighting value is 0.0

                !!! warning "default values"
                    if the attribute name is not found, there will be an automatic adjustment/appandage with the default value of 1.0
            
            - `#!python "DestWgt" string `   default "weight"

                name of the attribute from self.EntriesDf to access weight value for destination points. In the context of Betweeness Patronage, the value will represent destination preference.
                There is a feature that skips any processing if the weighting value is 0.0

                !!! warning "default values"
                    if the attribute name is not found, there will be an automatic adjustment/appandage with the default value of 1.0
            
            - `#!python "RsltAttr" string `   default "PatronBtwns"

                Attribute name for the result. will be appended to self.NetworkDf. (And self.EntriesDf if Include_Destination set True)
            
            - `#!python "SearchDist" float `   default 1200.0

                Search distance from origin to destination in network distance. Note that origin-destination pairings that are close to SearchDist and have DetourR that may result in distances further than SearchDist will still append the path found.
            
            - `#!python "DetourR" float `   default 1.0

                detour ratio for redundant paths, the value will represent the maximum redundant/alternative path length from the shortest path within the maximum search distance. Default value is 1.0 which is provided a switch to only find one shortest path. (if two exact shortest path exist, the pathfinding algorithm will use the smalled FID edge number).
            
            - `#!python "AlphaExp" float `   default 0.0

                Exponent value for inverse distance function. if using default value of 0.0, the calculation turns into a linear inverse distance function. See more at [method documentation](../methods/#betweeness-patronage-functions).
            
            - `#!python "AttrEdgeID" string `   default self.baseSet['EdgeID']

                Edge ID attribute field name, default will be referring to self.baseSet['EdgeID'], which has a default of 'fid'.

            - `#!python "AttrEntID" string `   default self.baseSet['EntID']

                Entry ID attribute field name, default will be referring to self.baseSet['EntID'], which has a default of 'fid'.
            
            - `#!python "Include_Destination" bool `   default False

                Included distribution of origin weight on destination entries. Note that this will also create a new attribute/field on self.EntriesDf but it won't be included as an ouptut of this function. To access results use GraphSims.EntriesDf.
            
            - `#!python "Threads" int|None `   default None

                number of threads for processing, if None, threads will refer to self.baseSet['Threads']. Note that this will set self.baseSet['Threads'], impacting other processes. Set 0 to automatically set cpu_count - 1
            
            - `#!python "PathLim" int `   default 2000

                Number of alternative path between an origin-destination pair. Paths generated are already very close to be sorted by distance. Process stopped if number of found paths exceed PathLim, so that further paths within DetourR range will not be detected/skipped.

    
    ##### Use Example
    :   
        ```python
        ### continuation from class initialization
        ### nwSim = sna.GraphSims()


        nwSim.BetweenessPatronage(OriWgt='Capacity', DestWgt='Weight', SearchDist=700, DetourR=1.1) # results does not need to be contained
        nwSim.NetworkDf.sample # results can be accessed from the NetworkDf attribute
        ```

        Function can be run in loops

        ```python
        ### continuation from class initialization
        ### nwSim = sna.GraphSims()

        runs = (
            ('Ent01', 'Dst02', 350.0, 1.0, 'run01'),
            ('Ent02', 'Dst02', 700.0, 1.2, 'run02'),
            ('Ent03', 'Dst03', 700.0, 1.0, 'run03'),
        )

        for rn in runs:
            nwSim.BetweenessPatronage(OriWgt=rn[0], DestWgt=rn[1], SearchDist=rn[2], DetourR=rn[3], RsltAttr=rn[4]) # results does not need to be contained
        nwSim.NetworkDf.to_file("output.gpkg", layer="Entries", crs="EPSG:32748", driver='GPKG') # geoDataFrame can be saved to GIS files
        ```

:   #### .Reach <i> func </i>
    `#!python Reach(self, OriID:Tuple=None, DestID:Tuple=None, Mode:str='N', **kwargs)`

    :   returns self.EntriesDf : GeoDataFrame <br>
        <i>GraphSims.EntriesDf with the specified results column</i>

        Function for Reach related calculations. Further elaboration on the method can be accessed on method documentation. With the resulting information are counts/weighted sums/sums of destinations within reach, the results can also be interpreted as Centralities. Further explanation on reach documentation can found on [method documentation](../methods/#reach-functions). Contains multiple Modes:
        
        - Mode="N" : Count Sum
            for counting destinations within reach distance, results in one new attribute of interger numbers

        - Mode="W" : Weighted Sum
            for counting and summing value based on specific weights of each destination feature, results in two attribute (with the second has a suffix of "_2"). First is interger count, second is the summed weight

        - Mode="WD" : Weighted Distance Sum
            for counting and summing value based on specific weights adjusted with distance function of each destination feature, results in two attribute (with the second has a suffix of "_2"). First is interger count, second is the summed weight.

            !!! info "CalcExp Value"
                CalcExp value is negative by default. It can handle negative 

        - Mode="ND" : Count and Distance
            for counting and finding nearest distance to destination. Results in two attribute (with the second has a suffix of "_2"). First is interger count, second is the minimum distance.

        - Mode="NDW" : Count, min Distance, and Weight Sum
            for counting and finding nearest distance to destination, and also sum of weight to all destinations in reach. Results in three attribute (with the minimum distance suffix of "_D", and Weight Sum of "_W").

        !!! info "result return"
            results will be appended to self.EntriesDf, therefore the results can be accesed later so that the function does not need to be assigned into a new variable.
        
        !!! warning "processing duration"
            function has built in multithreading capabilities using multiprocessing.Pool. Please take notice of self.baseSet['Threads'], as the value is also used in this function.
            Larger models, larger search distance, will be cause exponentially longer processing time.

    ##### Parameters

    :   <b>OriID</b> : Tuple <i>default None</i>
        :   Tuple of Origin point IDs, referencing `#!python GraphSims.EntriesDf` ID attribute specified at class initialization. If None, all 

    :   <b>DestID</b> : Tuple <i>default None</i>
        :   Geopandas geodataframe of Entries data, containing nodes in lines/polylines data form. Can contain weight

    :   <b>Mode</b> : string <i>default "N"</i>
        :   modes of calculation, see function description.

    :   <b>**kwargs</b> : Dict/keys-values
        :   Parameter arguments, that contain:
            
            - `#!python "DestWgt" string `   default "weight"

                name of the attribute from self.EntriesDf to access weight value for destination points. In the context of Betweeness Patronage, the value will represent destination preference.
                There is a feature that skips any processing if the weighting value is 0.0

                !!! warning "default values"
                    if the attribute name is not found, there will be an automatic adjustment/appandage with the default value of 1
            
            - `#!python "RsltAttr" string `   default "Reach"

                Attribute name for the result. will be appended to self.NetworkDf.
            
            - `#!python "SearchDist" float `   default 1200.0

                maximum path distance, corresponding to the data's units
            
            - `#!python "CalcExp" float `   default 0.35

                Exponent value for inverse distance function. if using default value of 0.0, the calculation turns into a linear inverse distance function. only for "WD" mode.

            - `#!python "CalcComp" float `   default 0.6

                Compunding multiplier for "WD" mode.
            
            - `#!python "Threads" int|None `   default None

                number of threads for processing, if None, threads will refer to self.baseSet['Threads']. Note that this will set self.baseSet['Threads'], impacting other processes. Set 0 to automatically set cpu_count - 1
    
    ##### Use Example

    :   
        ```python
        ### continuation from class initialization
        ### nwSim = sna.GraphSims()


        nwSim.Reach(Mode='WD", SearchDist=700, CalcExp=0.1) # results does not need to be contained
        nwSim.EntriesDf.sample # results can be accessed from the EntriesDf attribute
        ```

:   #### .Straightness <i> func </i>
    `#!python Straightness(self, OriID:list=None, DestID:list=None, Mode='A', **kwargs)`

    :   returns self.EntriesDf : GeoDataFrame <br>
        <i>GraphSims.EntriesDf with the specified results column</i>

        Returns average of Straightness value from all objects is flight distance from origin point. Only accounts of objects with findable paths within the network. Untracable paths will not be accounted. Further elaboration on the method can be accessed on method documentation.

        !!! info "result return"
            results will be appended to self.EntriesDf, therefore the results can be accesed later so that the function does not need to be assigned into a new variable.
        
        !!! warning "processing duration"
            function has built in multithreading capabilities using multiprocessing.Pool. Please take notice of self.baseSet['Threads'], as the value is also used in this function.
            Larger models, larger search distance, will be cause exponentially longer processing time.

    ##### Parameters

    :   <b>OriID</b> : Tuple <i>default None</i>
        :   Tuple of Origin point IDs, referencing `#!python GraphSims.EntriesDf` ID attribute specified at class initialization. If None, all 

    :   <b>DestID</b> : Tuple <i>default None</i>
        :   Geopandas geodataframe of Entries data, containing nodes in lines/polylines data form. Can contain weight

    :   <b>**kwargs</b> : Dict/keys-values
        :   Parameter arguments, that contain:
            
            - `#!python "DestWgt" string `   default "weight"

                name of the attribute from self.EntriesDf to access weight value for destination points. In the context of Betweeness Patronage, the value will represent destination preference.
                There is a feature that skips any processing if the weighting value is 0.0

                !!! warning "default values"
                    if the attribute name is not found, there will be an automatic adjustment/appandage with the default value of 1
            
            - `#!python "RsltAttr" string `   default "Straightness"

                Attribute name for the result. will be appended to self.NetworkDf.
            
            - `#!python "SearchDist" float `   default 1200.0

                maximum path distance, corresponding to the data's units
            
            - `#!python "CalcExp" float `   default 0.35

                Exponent value for inverse distance function. if positive will calculate by distance exponent, with further destinations will result in larger weights. use negative numbers for inverse distance function.
            
            - `#!python "Threads" int|None `   default None

                number of threads for processing, if None, threads will refer to self.baseSet['Threads']. Note that this will set self.baseSet['Threads'], impacting other processes. Set 0 to automatically set cpu_count - 1

    
    ##### Use Example

    :   
        ```python
        ### continuation from class initialization
        ### nwSim = sna.GraphSims()


        nwSim.Straightness(SearchDist=700, CalcExp=-0.1) # results does not need to be contained
        nwSim.EntriesDf.sample # results can be accessed from the EntriesDf attribute
        ```


:   #### .PathReach <i> func </i>
    `#!python PatyReach(self, OriID:list, distance:float=800, joined:bool=False, incl_nodes=False, showmap=False, skip_layerinit=True, pdkupdate=False, **kwargs)`

    :   returns edges : GeoDataFrame <br>

        Returns a new dataframe containing geometries of edges that are within the reach radius from each origin entry, where each feature in output will have an appended origin ID.
        
        !!! warning "multiple outputs!"
            the arguments 'incl_nodes', 'showmap', 'pdkupdate' will result in different return data and types. Read more on parameter description

    ##### Parameters

    :   <b>OriID</b> : Tuple <i>default None</i>
        :   iterable object of Origin Entry Ids.

    :   <b>distance</b> : float <i>default 800</i>
        :   Distance limit of pathreach. Lines/edges will be split on this distance.
    
    :   <b>joined</b> : bool <i>default False</i>
        :   Joins lines from all origins. Note that this will override/exclude origin entry id from results.

    :   <b>incl_nodes</b> : bool <i>default False</i>
        :   Output includes nodes on intersection, containing distance from origin information.
    
    :   <b>showmap</b> : bool <i>default False</i>
        :   Outputs Pydeck Deck/Map as the first input. Shows the edges, nodes on intersection with labels of distance, and origin point. Note that there are still edges and node output also appended in output
    
    :   <b>skip_layerinit</b> : bool <i>default True</i>
        :   Skips .Map_BaseLayerInit, so that output map will not include lines and other entries.

    :   <b>pdkupdate</b> : bool <i>default False</i>
        :   Outputs a list of Pydeck.Layers to replace or update previous pydeck.deck for update/interactive visualization. See more on [Samples](../smpl/)

    :   <b>**kwargs</b> : Dict/keys-values
        :   Parameter arguments, empty.

    
    ##### Use Example

    :   
        ```python
        ### continuation from class initialization
        ### nwSim = sna.GraphSims()

        EdgesDf, NodeDf = nwSim.PathReach((1,), SearchDist=800, incl_nodes=True)
        # unlike other GraphSims functions, PathReach results are not contained within the class.
        ```

<br><br>
@September2024