---
title: Calculation Analysis
description: Spatial Network Analysis for Python Module
---

# Other Analysis

further and additional analysis based of extrapolation and calculation from network analysis's results.

### Supplemental Calculations <i>Functions</i>

#### SimTimeDistribute <i> func </i>
`#!python SimTimeDistribute(Gdf: GeodataFrame, SetDt: nested tuple, spread: float =1.0, ApdAtt: string ='HrTrf_')`

:   returns GeoDataFrame object <br>
    <i>GeoDataFrame of network segment traffic values</i>

    Runs a set of skewed distribution to distribute one or more betweeness patronage results to form traffic intensity distribution over each segment. Method can be read further at <u>[Methods]{https://github.com/kevinsutjijadi/SNAPyDocs/Methods/}</u>. Will result in information of traffic per hour.

    !!! info "result return"
        results field on self.EntriesDf would be a concatenation of prefix-origin-destination-suffix string, which could have long characters. Saving the format as 
    
    !!! warning "processing duration"
        function is a singlethread processing which can take some time in larger models. Test smaller/single distrubutions ones first. 

##### Parameters

:   <b>Gdf</b> : GeoDataFrame <i>required</i>
    :   Geodataframe of network from betweeness patronage results, can use GraphSim.NetworkDf directly. Function will output/construct a different geodataframe, so that the origin won't change/appended.

:   <b>SetDt</b> : Nested Tuple/list <i>required</i>
    :   Nested tuple/list of distributions, can be from csv or other table format, but without columns. the following data are:
        <br>BtwnP Field Names - location - Shape - Skew

        location, shape, and skew are parameters form <u>[Skew Normal distribution]{https://en.wikipedia.org/wiki/Skew_normal_distribution}</u>, which can be obtained from model building/regessions from observation or other second hand data.

:   <b>Spread</b> : float <i>default 1.0</i>
    :   time spread of calculations in the integral, in the unit of hours. if 1.0, will results in total traffic per hour for each hour of the day, and 0.5 will results in total traffic per hour for each 30 min of the day.

:   <b>ApdAtt</b> : string <i>default 'HrTrf_'</i>
    :   suffix for result columns.

##### Use Example
:   
    ```python
    sets = [
        ['Btw_Commute', 8.0, 1, 0.8],
        ['Btw_Activity', 12.0, 2.5, 1.1],
        ['Btw_Commute', 17.0, 1.2, 1.05],
    ]

    sp = 0.5

    rslt = sna.SimTimeDistribute(nwSim.NetworkDf, sets, spread=sp)
    ```

    
<br><br>
@October2023