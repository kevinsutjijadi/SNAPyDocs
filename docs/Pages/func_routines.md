---
title: Network Analysis
description: Spatial Network Analysis for Python Module
---

# Spatial Network Analysis Routine Functions

Built in aggregate functions for multiple runs of the network analysis tools

### Routines <i>Functions</i>

#### ReachAggregate <i> func </i>
`#!python ReachAggregate(GraphSm:GraphSims, MeasureDf:pd.DataFrame, **kwargs)`

:   returns GraphSims : GraphSims class object <br>
    <i>GraphSims object</i>

    Runs a set of Reach functions available on the GraphSim functions. Reads the set of runs from a dataframe. See MeasureDf explanation for input sets. Outputs GraphSims with self.EntriesDf appended with results, with naming fill be

    !!! info "result return"
        results field on self.EntriesDf would be a concatenation of prefix-origin-destination-suffix string, which could have long characters. Saving the format as 
    
    !!! warning "processing duration"
        function has built in multithreading capabilities using multiprocessing.Pool. Please take notice of self.baseSet['Threads'], as the value is also used in this function.
        Larger models, larger search distance, larger detour ratio will be cause exponentially longer processing time.

##### Parameters

:   <b>GraphSm</b> : GraphSim <i>required</i>
    :   GraphSim object, initialized, can also be from previous analysis runs.

:   <b>MeasureDf</b> : DataFrame <i>required</i>
    :   Pandas Dataframe of sets of runs. Make sure in format of columns:
        <br>'OriginField' - 'DestinationField' - SeachDistance - WeightField

        Note that OriginField and DestinationField columns are matching with the EntriesDF's column name.<br>
        Example:

        | Function | Function.1 | Dist | WeightField |
        | - | - | - | - |
        | ResidentialV | ShopFNB | 1200 | Capacity |
        | ResidentialV | BusStop | 400  | Routes |
        | ShopFnB | Office | 700 | GFA |
        | ShopFnB | Office | 1500 | GFA |

        !!! info "columns"
            Note that the first and second columns names are matching the self.EntriesDf column names. its rows are the values of the features. Note that Pandas DataFrame cannot have matching/same column name, so if the keys for Origin and Destination are the same, the suffix of '.1' on the columns are appended, which is accounted in the functions.

        !!! warning "same destinations" 
            The resulting field is constructed from OptSuffix + DestinationField + OptPrefix, where OptSuffix and prefix can be accessed on the **kwargs settings. If there are 2 or more same destinations, the subsequent result names will be appended '_n', with n representing intergers.


:   <b>**kwargs</b> : Dict/keys-values
    :   Parameter arguments, that contain:

        - `#!python "OptSuffix" string `   default ''

            suffix string for all output.
        
        - `#!python "OptPrefix" string `   default "weight"

            prefix string for all outputs.
        
        !!! info "additional arguments" 
            Additional arguments can be added in this kwargs that are corresponding to the Reach's arguments. For example, CalcType, CalcComp, etc.

##### Use Example
:   
    ```python
    import SNAPy as sna
    import geopandas as gpd
    import pandas as pd

    dfNetwork = gpd.read_file('testdata\\NetworkClean.shp') # network dataframe
    dfEntries = gpd.read_file('testdata\\Features.gpkg', layer='Features') # entrance dataframe

    nwSim = sna.GraphSims(dfNetwork, dfEntries, settings)

    dfMeasures = pg.read_csv('testdata\\Measures.csv') # routine dataframe

    ReachAggregate(nwSim, dfMeasures, OptSuffix='R1_', CalcType='NWD') # results does not need to be contained

    nwSim.NetworkDf.to_file("output.gpkg", layer="Entries", crs="EPSG:32748", driver='GPKG') # geoDataFrame can be saved to GIS files
    ```

<br>
#### BetweenessPAggregate <i> func </i>
`#!python BetweenessPAggregate(GraphSm:GraphSims, PairsDf:DataFrame, MeasureDf:DataFrame=None, **kwargs)`

:   returns GraphSims : GraphSims class object <br>
    <i>GraphSims object</i>

    Runs a set of Reach functions available on the GraphSim functions. Reads the set of runs from a dataframe. See MeasureDf explanation for input sets. Outputs GraphSims with self.NetworkDf appended with results. PairsDf is required for iteration, but MeasureDf, which is used for interpolating to form simulation weights, does not; if left none, make sure the weight fields already exists on Graphsims.Entries.

    !!! warning "PairsDf and MeasureDf"
        use the same column order as in the parameter documentation, it does not match keys!

    
    !!! warning "processing duration"
        function has built in multithreading capabilities using multiprocessing.Pool. Please take notice of self.baseSet['Threads'], as the value is also used in this function.
        Larger models, larger search distance, larger detour ratio will be cause exponentially longer processing time.

##### Parameters

:   <b>GraphSm</b> : GraphSim <i>required</i>
    :   GraphSim object, initialized, can also be from previous analysis runs.

:   <b>PairsDf</b> : DataFrame <i>required</i>
    :   Pandas Dataframe of sets of runs. Make sure in format of columns:
        <br>Name - OriginWeightFields - DestinationWeightFields - SearchDistance* - DetourR* - AlphaExp*
        <br> * optional

        Note that OriginField and DestinationField columns are matching with the EntriesDF's column name.<br>
        Example:

        | RsltName | OriginWeightField | DestinationWeightField | Distance | DetourR |
        | - | - | - | - | - |
        | Commute_01 | ComOr01 | ComDes | 1200 | 1.0 |
        | Commute_02 | ComOr02 | ComDes  | 700 | 1.1 |
        | Activity_01 | Cap01 | CapW01 | 700 | 1.5 |
        | Event_01 | Stadium | PT01 | 400 | 1.0 |

        !!! info "SearchDistance, DetourR, and AlphaExp"
            these settings are optional, with if columns is not found, will be using default values that can be declared on kwargs.

        !!! warning "column order" 
            Use the same column order as the documentation. Naming of those columns does not effect processing

:   <b>MeasureDf</b> : DataFrame <i>default: None</i>
    :   Pandas Dataframe of sets of runs. Make sure in format of columns:
        <br>'TypeMatch' - BaseWeightField - *Extrapolated Fields

        For extrapolating columns for betweeness matching with PairsDf. DataFrame can be expanded to n columns/fields.<br>
        Example:

        | Function | WeightField | ComOr01 | ComDes | Cap01 | CapW01 |
        | - | - | - | - | - | - |
        | ResidentialV | Residents | 0.8 | 0.0 | 0.5 | 0.5 |
        | ShopFNB | Capacity | 0.1  | 0.0 | 0.8 | 2.0 |
        | ShopFnB | Capacity | 0.1 | 0.0 | 0.8 | 2.0 |
        | BusStop | RouteW | 0 | 1.0 | 0.0 | 0.0 |
        | Office | Capacity | 0.8 | 0.0 | 0.6 | 0.6 |

        !!! info "columns"
            Note that the first column name should match the self.EntriesDf column names. its rows are the values of the features.

:   <b>**kwargs</b> : Dict/keys-values
    :   Parameter arguments, that contain:

        - `#!python "OptSuffix" string `   default ''

            suffix string for all output.
        
        - `#!python "OptPrefix" string `   default "weight"

            prefix string for all outputs.
        
        !!! info "additional arguments" 
            Additional arguments can be added in this kwargs that are corresponding to BetweenessPatronage arguments. For example, DetourR, AlphaExp, etc. `GateIds`, `TermIds`, `TouchIds`, and `Include_Destination` are forwarded into every BetweenessPatronage call in the aggregate loop.

##### Use Example
:   
    ```python
    import SNAPy as sna
    import geopandas as gpd
    import pandas as pd

    dfNetwork = gpd.read_file('testdata\\NetworkClean.shp') # network dataframe
    dfEntries = gpd.read_file('testdata\\Features.gpkg', layer='Features') # entrance dataframe

    nwSim = sna.GraphSims(dfNetwork, dfEntries, settings)

    dfPairs = pd.read_csv('testdata\\Pairs.csv')
    dfMeasures = pd.read_csv('testdata\\Measures.csv') # routine dataframe

    BetweenessPAggregate(nwSim, dfPairs, dfMeasures) # results does not need to be contained

    nwSim.NetworkDf.to_file("output.gpkg", layer="Entries", crs="EPSG:32748", driver='GPKG') # geoDataFrame can be saved to GIS files
    ```

<br>
#### GateFlowSum <i> func </i>
`#!python GateFlowSum(GraphSm:GraphSims, names=None, RsltAttr='total')`

:   returns DataFrame : summed gate-to-gate matrix, also stored at `GraphSm.GateFlow[RsltAttr]`

    Adds selected `GateFlow` matrices. Each BetweenessPatronage / BetweenessPAggregate run already stores a **gate × gate** matrix (plus terminals and `open`) keyed by the run name. Movements are separate matrices, not axes. Use this to build a daily total from weekday runs. `TouchFlow[RsltAttr]` is the sum of the matching touch scalars.

##### Parameters

:   <b>GraphSm</b> : GraphSims <i>required</i>
    :   GraphSims object that already has `GateFlow` matrices from BetweenessPatronage or BetweenessPAggregate.

:   <b>names</b> : iterable <i>default None</i>
    :   Keys of matrices to add. If None, every stored `GateFlow` matrix is included.

:   <b>RsltAttr</b> : string <i>default 'total'</i>
    :   Name of the summed matrix written to `GraphSm.GateFlow` and the summed scalar written to `GraphSm.TouchFlow`.

##### Use Example
:   
    ```python
    import SNAPy as sna

    BetweenessPAggregate(nwSim, dfPairs, dfMeasures, GateIds=gate_ids, TermIds=term_ids)

    daily = sna.GateFlowSum(nwSim, names=['AM_in', 'PM_out', 'offpeak'], RsltAttr='daily')
    nwSim.GateFlow['daily']
    ```


<br><br>
@October2026