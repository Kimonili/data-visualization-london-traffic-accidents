---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import VegaLite exposing (..)
```

### Severity of total accidents in the US by state

```elm {v}
myHorizontalBar : Spec
myHorizontalBar =
    let
        cfg =
            configure
                << configuration (coTitle [ ticoAnchor anStart ])

        trans =
            transform
                << calculateAs
                    "datum.Severity == '2' ? datum.State : 0"
                    "amountOf2"
                << calculateAs
                    """if(datum.Severity == 2,1,
                          if(datum.Severity == 4, 2,
                           if(datum.Severity == 3,3,
                             if(datum.Severity == 1, 4, 5))))"""
                    "severityOrder"

        severityColours =
            categoricalDomainMap
                [ ( "1", "rgb(239,133,55)" )
                , ( "2", "rgb(81,157,62)" )
                , ( "3", "rgb(59,118,175)" )
                , ( "4", "rgb(213,126,190)" )
                ]

        enc =
            encoding
                << position X
                    [ pAggregate opCount
                    , pQuant
                    , pAxis
                        [ axValues (nums [ 0, 0.25, 0.5, 0.75, 1 ]), axTitle "", axFormat "%", axZIndex 1, axDomain False ]
                    , pStack stNormalize
                    ]
                << position Y
                    [ pName "State", pOrdinal, pSort [ soByField "amountOf2" opSum, soAscending ] ]
                << color
                    [ mName "Severity", mNominal, mScale severityColours ]
                << order
                    [ oName "severityOrder"
                    , oOrdinal
                    , oSort [ soAscending ]
                    ]
    in
    toVegaLite
        [ width 550
        , height 500
        , dataFromUrl "us_accidents.csv" []
        , bar []
        , enc []
        , trans []
        ]
```

### Bubble Scatter plots for each State of US

```elm{v interactive }
bubbleScaterPlot : Spec
bubbleScaterPlot =
    let
        enc =
            encoding
                << position X [ pName "Total drivers", pQuant, pAxis [ axTitle "Number of licensed drivers", axGrid False ] ]
                << position Y [ pName "Accidents", pQuant, pScale [ scType scLog ], pAxis [ axTitle "Number of accidents", axGrid False ] ]
                << color [ mStr "#800" ]
                << size
                    [ mName "Population/State", mQuant, mScale [ scRange (raNums [ 0, 7000 ]), scType scPow, scExponent 1.0 ] ]
                << tooltips
                    [ [ tName "Accidents", tQuant ], [ tName "Population/State", tQuant, tTitle "Population" ], [ tName "Total drivers", tQuant ], [ tName "State", tNominal ] ]
    in
    toVegaLite [ width 600, height 370, dataFromUrl "scatter_graph.csv" [], circle [ maOpacity 0.4 ], enc [] ]
```

### Traffic accidents by hour and day of the week

```elm{v interactive}
streamGraph : Spec
streamGraph =
    let
        enc =
            encoding
                << position X [ pName "hour", pQuant, pAxis [ axTitle "Hours of day" ] ]
                << position Y [ pName "accidents", pQuant, pAxis [], pStack stCenter ]
                << color [ mName "day_of_week", mTitle "Days of week" ]
                << tooltips
                    [ [ tName "accidents", tQuant, tTitle "Accidents" ], [ tName "hour", tQuant, tTitle "Hour of the day" ], [ tName "day_of_week", tNominal, tTitle "Day of week" ] ]
    in
    toVegaLite
        [ width 600
        , height 350
        , dataFromUrl "stream_graph.csv" []
        , area
            [ maInterpolate miMonotone ]
        , enc []
        ]
```

### Accidents by date for every month

```elm{v}
histoDateMonth : Spec
histoDateMonth =
    let
        enc =
            encoding
                << position X [ pName "Month", pQuant, pBin [ biMaxBins 12 ], pAxis [ axTitle "Months" ] ]
                << position Y [ pName "Date", pQuant, pBin [ biMaxBins 30 ], pAxis [ axTitle "Date of month" ] ]
                << color
                    [ mAggregate opCount
                    , mQuant
                    , mScale [ scType scSqrt ]
                    ]
    in
    toVegaLite
        [ width 350
        , height 350
        , dataFromUrl "us_accidents.csv" []
        , enc []
        , rect []
        ]
```

### Accidents by hour for every day of the week

```elm{v interactive}
histoDayHour : Spec
histoDayHour =
    let
        enc =
            encoding
                << position X [ pName "day_of_week_number", pOrdinal, pBin [ biMaxBins 14 ], pAxis [ axTitle "Day of week" ] ]
                << position Y [ pName "hour", pOrdinal, pBin [ biMaxBins 48 ], pAxis [ axTitle "Hour of day" ] ]
                << color
                    [ mName "accidents"
                    , mQuant
                    , mScale [ scType scSqrt ]
                    ]
                << tooltips
                    [ [ tName "accidents", tQuant, tTitle "Accidents" ], [ tName "hour", tQuant, tTitle "Hour of the day" ], [ tName "day_of_week", tNominal, tTitle "Day of week" ] ]
    in
    toVegaLite
        [ width 350
        , height 350
        , dataFromUrl "table_2.csv" []
        , enc []
        , rect []
        ]
```

### Accidents by severity and hour

```elm{v interactive}
severityHourLinegraph : Spec
severityHourLinegraph =
    let
        severityColours =
            categoricalDomainMap
                [ ( "1", "rgb(239,133,55)" )
                , ( "2", "rgb(81,157,62)" )
                , ( "3", "rgb(59,118,175)" )
                , ( "4", "rgb(213,126,190)" )
                ]

        enc =
            encoding
                << position X [ pName "hour", pQuant, pAxis [ axTitle "Hours of day" ] ]
                << position Y [ pName "accidents", pQuant, pScale [ scType scLog ], pAxis [ axGrid False, axTitle "Number of accidents" ] ]
                << color [ mName "severity", mTitle "Severity level", mScale severityColours ]
                << tooltips
                    [ [ tName "accidents", tQuant, tTitle "Accidents" ], [ tName "hour", tQuant, tTitle "Hour of the day" ], [ tName "severity", tNominal, tTitle "Level of severity" ] ]
    in
    toVegaLite
        [ width 600
        , height 350
        , dataFromUrl "line_graph_1.csv" []
        , line []
        , enc []
        ]
```

### Accidents by severity and month

```elm{v interactive}
severityMonthLinegraph : Spec
severityMonthLinegraph =
    let
        severityColours =
            categoricalDomainMap
                [ ( "1", "rgb(239,133,55)" )
                , ( "2", "rgb(81,157,62)" )
                , ( "3", "rgb(59,118,175)" )
                , ( "4", "rgb(213,126,190)" )
                ]

        enc =
            encoding
                << position X [ pName "month", pQuant, pAxis [ axTitle "Month" ] ]
                << position Y [ pName "accidents", pQuant, pScale [ scType scLog ], pAxis [ axGrid False, axTitle "Number of accidents" ] ]
                << color [ mName "severity", mTitle "Level of severity", mScale severityColours ]
                << tooltips
                    [ [ tName "accidents", tQuant, tTitle "Accidents" ], [ tName "month", tQuant, tTitle "Month" ], [ tName "severity", tNominal, tTitle "Level of severity" ] ]
    in
    toVegaLite
        [ width 600
        , height 350
        , dataFromUrl "line_graph_2.csv" []
        , line []
        , enc []
        ]
```

```elm{v interactive}
severity4HourDayStackedegraph : Spec
severity4HourDayStackedegraph =
    let
        data =
            dataFromUrl "level4severity_graph.csv"

        enc =
            encoding
                << position X [ pName "hour", pOrdinal, pAxis [ axTitle "Hours of day" ] ]
                << position Y [ pName "accidents", pQuant, pStack stNormalize, pAxis [ axGrid False, axTitle "Number of level 4 severity accidents" ] ]
                << color [ mName "day_of_week", mTitle "Day of week" ]
                << tooltips
                    [ [ tName "accidents", tQuant, tTitle "Accidents" ], [ tName "hour", tQuant, tTitle "Hour of the day" ], [ tName "day_of_week", tNominal, tTitle "Day of week" ] ]
    in
    toVegaLite
        [ width 600
        , height 350
        , data [ parse [ ( "hour", foNum ) ] ]
        , bar []
        , enc []
        ]
```

### US states map

```elm {l v}
stateMap : Spec
stateMap =
    let
        stateMapData =
            dataFromUrl "https://vega.github.io/vega-lite/data/us-10m.json"
                [ topojsonFeature "states" ]

        accidentsStateData =
            dataFromUrl "accidentRatebyState.csv" []

        trans =
            transform
                << lookup "id" accidentsStateData "states" (luFields [ "Accidents" ])

        enc =
            encoding
                << color [ mName "Accidents", mQuant, mScale [ scType scSqrt ] ]
    in
    toVegaLite [ width 500, height 300, stateMapData, trans [], enc [], geoshape [] ]
```
