---
id: litvis

narrative-schemas:
  - ../narrative-schemas/courseworkPG.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

# Postgraduate Coursework Template

{(questions|}

1. How are traffic accidents in the US distributed throughout the states and what is the role of population?
2. Which days of the week are most "dangerous"? How are traffic accidents distributed throughout the day?
3. How severity of the accidents is related to the time of a day? In which hours and days do the most severe accidents occur the most?

{|questions)}

### 1st Visualization - Research question 1

##### Bubble plot: Relationship between accidents, drivers and population by state

```elm{v interactive }
bubbleScaterPlot : Spec
bubbleScaterPlot =
    let
        enc =
            encoding
                << position X [ pName "drivers/100K", pQuant, pAxis [ axTitle "Drivers per 100,000 population", axGrid False ] ]
                << position Y [ pName "accidents/100K", pQuant, pAxis [ axTitle "Accidents per 100,000 population", axGrid False ] ]
                << color [ mStr "#800" ]
                << size
                    [ mName "Population/State", mQuant, mScale [ scRange (raNums [ 0, 5000 ]), scType scPow, scExponent 0.87 ] ]
                << tooltips
                    [ [ tName "accidents/100K", tQuant, tTitle "Accidents per 100,000 population" ], [ tName "drivers/100K", tQuant, tTitle "Drivers per 100,000 population" ], [ tName "Population/State", tQuant, tTitle "Actual population" ], [ tName "State", tNominal ] ]
    in
    toVegaLite [ width 500, height 500, dataFromUrl "scatter_graph_per_100K.csv" [], circle [ maOpacity 0.4 ], enc [] ]
```

### 2nd Visualization - Research question 2

#### Part 1

##### Stream graph: Distribution of accidents by hour and day of the week

```elm{v interactive}
streamGraph : Spec
streamGraph =
    let
        data =
            dataFromUrl "stream_graph.csv"

        weekColours =
            categoricalDomainMap
                [ ( "Monday", "#ffffb3" )
                , ( "Tuesday", "#fdb462" )
                , ( "Wednesday", "#b3de69" )
                , ( "Thursday", "#80b1d3" )
                , ( "Friday", "#8dd3c7" )
                , ( "Saturday", "#bebada" )
                , ( "Sunday", "#fb8072" )
                ]

        enc =
            encoding
                << position X [ pName "hour", pOrdinal, pAxis [ axTitle "Hours of day" ] ]
                << position Y [ pName "accidents", pQuant, pAxis [], pStack stCenter ]
                << color [ mName "day_of_week", mTitle "Days of week", mScale weekColours ]
                << tooltips
                    [ [ tName "accidents", tQuant, tTitle "Accidents" ], [ tName "hour", tQuant, tTitle "Hour of the day" ], [ tName "day_of_week", tNominal, tTitle "Day of week" ] ]
    in
    toVegaLite
        [ width 600
        , height 350
        , data [ parse [ ( "hour", foNum ) ] ]
        , area [ maInterpolate miMonotone ]
        , enc []
        ]
```

#### Part 2

##### Pivot table heatmap: Intensity of accidents by hour and day of the week

```elm{v interactive}
histoDayHour : Spec
histoDayHour =
    let
        data =
            dataFromUrl "table_2.csv"

        enc =
            encoding
                << position X [ pName "day_of_week_number", pOrdinal, pAxis [ axTitle "Day of week" ] ]
                << position Y [ pName "hour", pOrdinal, pAxis [ axTitle "Hour of day" ] ]
                << color
                    [ mName "accidents"
                    , mQuant
                    , mScale [ scType scSqrt ]
                    , mTitle "Accidents"
                    ]
                << tooltips
                    [ [ tName "accidents", tQuant, tTitle "Number of accidents" ], [ tName "hour", tQuant, tTitle "Hour of the day" ], [ tName "day_of_week", tNominal, tTitle "Day of week" ] ]
    in
    toVegaLite
        [ width 350
        , height 350
        , data [ parse [ ( "hour", foNum ), ( "day_of_week_number", foNum ) ] ]
        , enc []
        , rect []
        ]
```

Note: Days of week are represented like numbers from Monday (0) to Sunday (6).

### 3rd Visualization - Research question 3

#### Part 1

##### Line graph: Scaled distribution of accidents by hour for every severity group

```elm{v interactive}
severityHourLinegraph : Spec
severityHourLinegraph =
    let
        data =
            dataFromUrl "line_graph_1.csv"

        severityColours =
            categoricalDomainMap
                [ ( "1", "#1b9e77" )
                , ( "2", "#d95f02" )
                , ( "3", "#7570b3" )
                , ( "4", "#e6ab02" )
                ]

        enc =
            encoding
                << position X [ pName "hour", pOrdinal, pAxis [ axTitle "Hours of day" ] ]
                << position Y [ pName "accidents", pQuant, pScale [ scType scLog ], pAxis [ axGrid False, axTitle "Number of accidents" ] ]
                << color [ mName "severity", mTitle "Severity level", mScale severityColours ]
                << tooltips
                    [ [ tName "accidents", tQuant, tTitle "Accidents" ], [ tName "hour", tQuant, tTitle "Hour of the day" ], [ tName "severity", tNominal, tTitle "Level of severity" ] ]
    in
    toVegaLite
        [ width 600
        , height 350
        , data [ parse [ ( "hour", foNum ) ] ]
        , line []
        , enc []
        ]
```

#### Part 2

##### Faceted barplots: Accidents of level 4 severity by hour and day of the week

```elm{v interactive}
severity4HourDayLinegraph : Spec
severity4HourDayLinegraph =
    let
        data =
            dataFromUrl "level4severity_graph.csv"

        weekColours =
            categoricalDomainMap
                [ ( "Monday", "#ffffb3" )
                , ( "Tuesday", "#fdb462" )
                , ( "Wednesday", "#b3de69" )
                , ( "Thursday", "#80b1d3" )
                , ( "Friday", "#8dd3c7" )
                , ( "Saturday", "#bebada" )
                , ( "Sunday", "#fb8072" )
                ]

        enc =
            encoding
                << position X
                    [ pName "hour"
                    , pOrdinal
                    , pAxis [ axTitle "Hours of day" ]
                    ]
                << position Y [ pName "accidents", pQuant, pAxis [ axTitle "Number of level 4 severity accidents" ] ]
                << color [ mName "day_of_week", mTitle "Day of week", mNominal, mLegend [], mScale weekColours ]
                << column [ fName "day_of_week", fOrdinal ]
                << tooltips
                    [ [ tName "accidents", tQuant, tTitle "Accidents" ], [ tName "hour", tQuant, tTitle "Hour of the day" ], [ tName "day_of_week", tNominal, tTitle "Day of week" ] ]
    in
    toVegaLite
        [ height 200
        , width 200
        , columns 2
        , data [ parse [ ( "hour", foNum ) ] ]
        , bar []
        , enc []
        ]
```

## Insights

#### Bubble plot

**How are traffic accidents in the US distributed throughout the states and what is the role of population?**

An intuitive leap is that traffic accidents do not seem to be related to the amount of cars that are used in a geographical region (state). The Y axis represents the number of accidents per 100,000 population, the X axis the number of licensed drivers per 100,000 population and each bubble is a state that its size represents the state's population - the wider the bubble the larger the population.

On the one hand, there is not a clear relationship (either positive or negative) between the accidents and the drivers as most states have a number of drivers between 55,000 and 75,000 per 100,000 population leading to a perpendicular pattern to the X axis. On the other hand, wider bubbles are concentrated in the upper part of the graph while the smaller ones are close to the X axis. That means that the larger the population of a state the higher the amount of accidents occur in this state. Although, we can spot a very small bubble, Nebraska, that has high accident rate and can be considered as an outlier. Despite the fact that it has a relatively small population, it demonstrates the second highest amount of accidents per 100,000 population, only bellow California.

#### Stream graph - Pivot table heatmap

**Which days of the week are most "dangerous"?**

The first part of the question is very clear and straightforward to answer. During the weekdays, the amount of accidents is much higher than that of the weekends. This indicates that during the weekends people tend to spend more time at home than moving around by car. It is also clear that the main reason of having more accidents during the weekdays, is that all the employed people have to go to work, either by car or using the transit. This increases the probability of a traffic accident to occur, if it either involves vehicles, or a vehicle and a pedestrian. To sum up, workdays are much more delicate to traffic accidents than weekends.

**How are traffic accidents distributed throughout the day?**

The second part of the question is linked to the justification given in the previous paragraph, as a different accident pattern is observed for the weekdays and the weekends. During the week, the hours indicating the higher accident rate are during the rush hours of the day. From 07:00 to 11:00 most of the employed people are heading to their work places and therefor there is a rise on the accident rate. For the same reason, from 16:00 to 18:00 most of the people return to their home after finishing their daily work, and again the accident rate increases. As for the weekend, there is no such pattern and in general it seems that people's movement is very limited as there are very few incidents throughout the day. There is only a slight rise in the number of accidents around 13:00 and 14:00 but still is more than 4 times less compared to the most dangerous hours of the workdays.

#### Line graph

**How severity of the accidents is related to the time of a day?**

For the 3rd question it was decided to further investigate the severity of the accidents. We have 4 severity levels, a number from 1 to 4 included, where 1 indicates the least impact on traffic (i.e., short delay as a result of the accident) and 4 indicates a significant impact on traffic. It is discovered that all kind of accidents have a similar trend. The least frequent accidents are the ones with a severity of 1, then 4,3 and the most frequent is 2. The severity 2 and 3 accidents have almost identical ups and downs in their accident rate throughout the day following the overall accident distribution that we commented on before. The severity 1 has a very small amount of accidents and therefore will not be taken into acount as the accident rate at its highest is 24.

The most interesting accidents are the most severe ones because they are the only accidents that are increased during the night, from 21:00 to 23:00.**(Gjerde, Normann, Christophersen, Samuelsen, & Mørland, 2011)** concluded that alcohol and drugs consumption have a high correlation with the most fatal accidents. Consequentially, drug or alcohol consumption by drivers might be the reason of this unexpected increase in the number of accidents that occured only for the most severe accidents during late night hours.

#### Faceted barplots

**In which hours and days do the most severe accidents occur the most?**

For that reason, it is considered a necessity to further investigate any extra patterns for the most severe accidents. Faceted barplots visualization shows 7 histograms, one for each day of the week, indicating how many severity 4 accidents are happening for every hour of the day. It is clear that from Friday and Saturday are the most dangerous days during the midnight. Assuming that alcohol and drugs consumption can cause more fatal traffic accidents than average **(Gjerde, Normann, Christophersen, Samuelsen, & Mørland, 2011)**, it would be expected the severity 4 accidents to increase these days were people tend to go out the most. Nonetheless, severity 4 accidents also appear to have an increase during the high risk rush hour periods of the weekdays.

## Design justification

As Nathan Yau states: "Letting the computer do everything for you has the advantage of time saving while adding the human in the loop, by telling the computer exactly what to do, results to better and more accurate representations" **(Yau, 2013)**.

In this project it was decided to choose the second approach. From the very beggining I tried to combine in the best way possible the human-computer cooperation that when is succesfully applied gives significant results. First, it was decided to select only those accidents that happened in 2016 out of the years 2016, 2017 and 2018, to reduce the size of the dataset. It was determined that 3 million observations would dellay the analysis without having a proportionate increase in the visual performance. At the same time, the licensed drivers dataset has information for the year 2015 and it was considered better to keep the closest to that year accidents data to avoid any biased assumptions. Additionally, pre-processing and data manipulation were implemented in Python and the resulted tables (csv files) were imported for each graph accordingly. The data management part included multiple aggregate and group-by functions **(Gray, Bosworth, Lyaman, & Pirahesh)**, that contributed to identify patterns, followed by a "data tidying" process that facilitates modeling, exploration and visualization **(Wickham, 2014)**.

At this point it is important to mention that the graphs for which there is not a "Scales" section, are those that linear scaling was applied.

#### 1. Bubble plot

##### Visual cues

**Position:** This bubble scatter plot uses position as a visual clue by placing every state according to its value of drivers and accidents in the cartesian coordinate system. This helps us judge the position of a state compared to the position of other states and helps us detect outliers, clusters and trends **(Yau, 2013)**.

**Size:** Additionally to the simple scatter plot, a size visual cue was added in order to represent the discriptive numeric variable of population by State **(Kirk, 2019)**. The bigger the circle, the greater the population value. This is the 3rd element of this graph and it helps us see the role of population.

**Color:** Given that our visualization is about traffic accidents, the selected color is red as, according to color psychology, it is considered one of the "negative" ones along with brown and grey **(Bartram, Patra, & Stone, 2017)**. It was chosen to have opacity in order to deal with the accrued cluster and to make it easier for the reader to recognize each state.

#### 2. Stream graph

##### Visual cues

**Size:** The size of each colored area represents the number of accidents for the given hour. The wider the area the more accidents occured. It is useful as it gives us an estimate about the days and hours that are most dangerous.

**Color:** In this graph, each color represents a value of the categorical variable "Day of week" with an adjusted low overall saturation. Large regions, like areas, are favored to have lower saturation compared to small regions, like lines **(Munzner, 2015)**. For that reason [qualitative color scaling](**https://colorbrewer2.org/**) was used so that the varying shades to provide an easy visual seperation for the reader.

#### 3. Pivot table heatmap

##### Visual cues

**Color:** This table uses colors to represent the descrete numeric variable of the number of accidents by hour and day of week. A dark blue cell indicates a high number of accidents while a light yellow cell a small amount of accidents. It is similar to the "Viridis" multi-hue palette, that was found to have high interpretation performance by users, as a quantitative color encoding combination, in terms of speed and accuracy **(Liu & Heer, 2018)**.

#### 4. Line graph

##### Visual cues

**Direction:** This time series line graph is created to see how accidents icrease and decrease by hours of a day. When the line goes upwards it means that the slope is positive and when it goes downwards it means that the slope is negative. It is effective as we can visualize how the amount of accidents change over time in a continious way.

**Color:** Color hue is used for the representation of the numeric categorical variable of the severity of every accident. For that reason, high saturated and easy seperable colors were chosen for each severity category **(Wilke, 2019)**.

##### Scales

**Logarithmic:** The first edition of this graph used linear scaling to represent the data, but this proved to be confusing for the reader as level 1 and 4 severity accidents were located at the botom of the graph. This made it impossible to recognize patterns for those categories. A logarithic scalling was applied to achieve to visualize minor changes of those classes **(Wilkinson, 2012)**. This helped me to observe that the most severe accidents tend to increase during the late night hours and also lead to the second part of the research question.

#### 5. Faceted barplots

##### Visual cues

**Color:** The same colors as for the stream graph are used for each day to help the reader link a specific color with a specific day.

**Length:** The length of each bar shows the magnitude of the value. The Y axis (number of accidents) starts from 0 and the longer the distance of a bar from Y = 0 the greater the absolute value. Length is considered one of the most blatant visualization methods for magnitude comparison **(Cleveland & Mcgill, 1984)**.

## Validation

In a diagram, if the title isn't descriptive enough, people turn to other text to fulfil their need of understanding the design **(Borkin, et al., 2016)**. For that reason, all the graphs are accompanied by revealing titles, to help the reader understand from the beggining of what the graph tries to demonstrate. Lastly, interaction is added to every design in order to make them more precise and clear.

#### Bubble plot: Relationship between accidents, drivers and population by state

This graph visualized succesfully the relationship between 3 variables in a single design. Feature based depictions are used a lot when both dimensions, in a two dimensional scatter plot, are already mapped, as is enables us to add an extra feature to the pattern recognition process **(Szafir, Haroz, Gleicher, & Franconeri, 2016)**. It showed that the range of the amount of drivers by state is small and not related to the accidents. Contrarily, states indicating a high number of accidents seem to have a larger population as well. Another virtue is that the interaction helps the reader understand which bubble is equivalent to which state.

Nonetheless, it is still challenging to identify which state is represented by which bubble as it is still necessary to mouse over. In addition to that, some states completely overlap others. It might be more effective to represent the same features by creating a map of the US, bordered by state, to solve the overlapping problem.

#### Stream graph: Distribution of accidents by hour and day of the week

This data vis showed the distribution of accident rates by hour for every day of the week in a single design. Colors are chosen in a way to be appealing and easily interpretable for the eye and the size of each colored area helps us get an estimate of how the accidents are distributed. The priority of a stream graph is getting a general sense of the patterns more than precision value reading and so the Y acis is completely deleted **(Kirk, 2019)**. Size is not the most effective comparison method, thus the reader is not aware of the exact number of accidents per hour **(Munzner, 2015)**. To address this issue an interaction was added that stated the exact numbers of the features.

New York Times, published in 2008 a stack graph of movie ticket sales that drew a lot of attention. Studies have shown that unusual graphs tend to be more memorable than more popular ones like histograms, lines etc **(Borkin, et al., 2013)**. Additionally, "junk" embellished charts achieve the same interpetation accuracy with the minimal ones and can be recalled better after 2-3 weeks **(Bateman, et al., 2010)**. Nonetheless, streamgraphs are hard to read by an average exposed in data vis individual **(Byron & Wattenberg, 2008)**.

As a result, I implemented a small sampled experiment by showing this graph to 8 friends of mine. I supported them with a brief description of what the graph shows without further explanation of how the graph represents the features. I found out that 6 of them succesfully understood its structure while the other 2 were confused. It is important to mention that the succesful group are either economists or engineers while the 2 that failed to interpret are involved in primary school education. It seems that the person's background plays a major role in understanding more complex graphs. For that reason, a pivot table representing the same feature relationship was constructed.

#### Pivot table heatmap: Intensity of accidents by hour and day of the week

This design demonstrates the number of accidents by hour for every day of the week but this time using the color cue for representing the icrease and decrease in the accident's rate and not the size cue as before. It is a popular graph and a lot easier to understand and interpret. It fails to show the exact number accidents but the legend bar in the right along with the color saturation and the interaction by mousing over, helps to overcome this problem. A drawback could be the fact that colorblind population might have trouble recognizing the differences of the colors leading to low interpretation. As for the aforementioned experiment, all the participants achieved to understand what it shows, making it preferable from a practical point of view but maybe less memorable compared to the stream graph. Lastly, it is not clear which day is which number in the X axis although a note is added to the design.

#### Line graph: Scaled distribution of accidents by hour for every severity group

This is a simple data vis design representing the accident rate by time of day, specifying it's severity via color cue. It is not appropriate for value comparison as the logarithmic scaling intends to discover the trends of every severity level. In this case, the absolute difference in accident rates between each severity level is biased and the user should not be confused as the lines are not linearly placed to the cartesian coordinate system.

As shown in the graph bellow, linear scaling failed to establish this trend succesfully as small changes in low accident values for the 4 severity accidents were hidden in a flat line representation.Lastly, time series graphs achieve an interprentation accuracy equal to which can not be fould to other visual designs **(Tufte, 1985)**.

##### Rejected design

```elm{v interactive}
severityHourLinegraphRejected : Spec
severityHourLinegraphRejected =
    let
        data =
            dataFromUrl "line_graph_1.csv"

        severityColours =
            categoricalDomainMap
                [ ( "1", "#1b9e77" )
                , ( "2", "#d95f02" )
                , ( "3", "#7570b3" )
                , ( "4", "#e6ab02" )
                ]

        enc =
            encoding
                << position X [ pName "hour", pOrdinal, pAxis [ axTitle "Hours of day" ] ]
                << position Y [ pName "accidents", pQuant, pAxis [ axGrid False, axTitle "Number of accidents" ] ]
                << color [ mName "severity", mTitle "Severity level", mScale severityColours ]
                << tooltips
                    [ [ tName "accidents", tQuant, tTitle "Accidents" ], [ tName "hour", tQuant, tTitle "Hour of the day" ], [ tName "severity", tNominal, tTitle "Level of severity" ] ]
    in
    toVegaLite
        [ width 600
        , height 350
        , data [ parse [ ( "hour", foNum ) ] ]
        , line []
        , enc []
        ]
```

#### Faceted barplots: Accidents of level 4 severity by hour and day of the week

This group of barplots was considered the best option for answering the 2nd part of the 3rd research question. It effectively shows how the most severe accidents are distributed every day by time. I was interested in investigating minor patterns and link the results to the probability of alcohol playing a role in the existence of relatively high number of accidents during late night hours **(Gjerde, Normann, Christophersen, Samuelsen, & Mørland, 2011)**. This design is allocated horizontally as it makes it easier to compare the length of the barplots. As shown bellow, the initial design was a stacked barplot but it was confusing as 7 different colors (one for each day) for 24 bars (one for each hour) made it unappropriate for comparison.

##### Rejected design

```elm{v interactive}
severity4HourDayStackedegraph : Spec
severity4HourDayStackedegraph =
    let
        data =
            dataFromUrl "level4severity_graph.csv"

        weekColours =
            categoricalDomainMap
                [ ( "Monday", "#ffffb3" )
                , ( "Tuesday", "#fdb462" )
                , ( "Wednesday", "#b3de69" )
                , ( "Thursday", "#80b1d3" )
                , ( "Friday", "#8dd3c7" )
                , ( "Saturday", "#bebada" )
                , ( "Sunday", "#fb8072" )
                ]

        enc =
            encoding
                << position X [ pName "hour", pOrdinal, pAxis [ axTitle "Hours of day" ] ]
                << position Y [ pName "accidents", pQuant, pStack stNormalize, pAxis [ axGrid False, axTitle "Number of level 4 severity accidents" ] ]
                << color [ mName "day_of_week", mTitle "Day of week", mScale weekColours ]
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

Having each day in a different sub-graph makes it easier to interpret the design. Nonetheless, it is proven that faceted graphs require more time to be processed, probably because of the multiple charts that are being compared, although they end up with reasonable accuracy performance **(Kim & Heer, 2018)**. Lastly, the designer's initial intention was to sort each barplot from Monday to Sunday but it was not achieved to be implemented.

## References

1. Bartram, L., Patra, A., & Stone, M. (2017). Affective Color in Visualization. Proceedings of the 2017 CHI Conference on Human Factors in Computing Systems. doi: 10.1145/3025453.3026041

2. Bateman, S., Mandryk, R. L., Gutwin, C., Genest, A., Mcdine, D., & Brooks, C. (2010). Useful junk? Proceedings of the 28th International Conference on Human Factors in Computing Systems - CHI 10. doi: 10.1145/1753326.1753716

3. Borkin, M. A., Vo, A. A., Bylinskii, Z., Isola, P., Sunkavalli, S., Oliva, A., & Pfister, H. (2013). What Makes a Visualization Memorable? IEEE Transactions on Visualization and Computer Graphics, 19(12), 2306–2315. doi: 10.1109/tvcg.2013.234

4. Borkin, M. A., Bylinskii, Z., Kim, N. W., Bainbridge, C. M., Yeh, C. S., Borkin, D., … Oliva, A. (2016). Beyond Memorability: Visualization Recognition and Recall. IEEE Transactions on Visualization and Computer Graphics, 22(1), 519–528. doi: 10.1109/tvcg.2015.2467732

5. Byron, L., & Wattenberg, M. (2008). Stacked Graphs – Geometry & Aesthetics. IEEE Transactions on Visualization and Computer Graphics, 14(6), 1245–1252. doi: 10.1109/tvcg.2008.166

6. Cleveland, W. S., & Mcgill, R. (1984). Graphical Perception: Theory, Experimentation, and Application to the Development of Graphical Methods. Journal of the American Statistical Association, 79(387), 531–554. doi: 10.1080/01621459.1984.10478080

7. COLORBREWER 2.0. (n.d.). Retrieved from https://colorbrewer2.org/

8. Gjerde, H., Normann, P. T., Christophersen, A. S., Samuelsen, S. O., & Mørland, J. (2011). Alcohol, psychoactive drugs and fatal road traffic accidents in Norway: A case–control study. Accident Analysis & Prevention, 43(3), 1197–1203. doi: 10.1016/j.aap.2010.12.034

9. Gray, J., Bosworth, A., Lyaman, A., & Pirahesh, H. (n.d.). Data cube: a relational aggregation operator generalizing GROUP-BY, CROSS-TAB, and SUB-TOTALS. Proceedings of the Twelfth International Conference on Data Engineering. doi: 10.1109/icde.1996.492099

10. Kim, Y., & Heer, J. (2018). Assessing Effects of Task and Data Distribution on the Effectiveness of Visual Encodings. Computer Graphics Forum, 37(3), 157–167. doi: 10.1111/cgf.13409

11. Kirk, A. (2019). Data visualisation: a handbook for data driven design. London: SAGE Publications Ltd.

12. Liu, Y., & Heer, J. (2018). Somewhere Over the Rainbow. Proceedings of the 2018 CHI Conference on Human Factors in Computing Systems - CHI 18. doi: 10.1145/3173574.3174172

13. Munzner, T. (2015). Visualization analysis & design. Boca Raton: CRC Press, Taylor & Francis Group.

14. Szafir, D. A., Haroz, S., Gleicher, M., & Franconeri, S. (2016). Four types of ensemble coding in data visualizations. Journal of Vision, 16(5), 11. doi: 10.1167/16.5.11

15. Tufte, E. R. (1985). The Visual display of quantitative information. Cheshire, CT: Graphics Press.

16. Wickham, H. (2014). Tidy Data. Journal of Statistical Software, 59(10). doi: 10.18637/jss.v059.i10

17. Wilke, C. O. (2019). Fundamentals of Data Visualization: A Primer on Making Informative and Compelling Figures. OReilly Media, Incorporated.

18. Wilkinson, L. (2012). Grammar of graphics. Place of publication not identified: Springer.

19. Yau, N. (2013). Data points: visualization that means something. Indianapolis: John Wiley & Sons.
