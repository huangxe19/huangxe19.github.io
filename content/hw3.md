title: Homework 3
date: 09/30/2021
author: Xige Huang

# Creating effective visualizations using best practices

Using <https://github.com/cliburn/bios-823-2021/blob/main/homework/assignments.md>


__Outline of this post__:

* [Interactive scatter plot using Plotly](#section1)
* [Interactive dot plot using Plotly](#section2)
* [Boxplot using Seaborn](#section3)



## Scatter Plot<a name="section1"></a>

Because the data is time series data, therefore it's always useful to take a first look at the trend throughout the years. 

I use `Plotly` to plot two scatter lines, which allows user to select the two countries they want to compare.

The first reason I chose plotly is that plotly allows intreactive filtering of the data by widgets. Moreover, the 'compare data on hover' feature is especially useful to see the exact numbers of the plots by moving through the lines.

![Hi][scatter_plot]

First read the data
```python
df = pd.read_csv('malaria_deaths_age.txt')
df = df.drop(columns = 'Unnamed: 0')

# aggregate all age groups into 'All age'
age_agg = df.groupby(['entity', 'year']).agg('sum').reset_index()
age_agg['age_group'] = 'All age'
df1 = pd.concat([df, age_agg], axis=0)
```

The code to define the widgets:
```python
age = widgets.Dropdown(
    options=list(df1['age_group'].unique()),
    value='All age',
    description='Age Group:',
)

entity1 = widgets.Dropdown(
    options=list(df1['entity'].unique()),
    value='Afghanistan',
    description='Country1:',
    continuous_update=True
)

entity2 = widgets.Dropdown(
    options=list(df1['entity'].unique()),
    value='Zimbabwe',
    description='Country2:',
    continuous_update=True
)

container = widgets.HBox([entity1, entity2])
```

Then create the default plot:

```python

trace1 = go.Scatter(x= df1[(df1['entity'] == entity1.value) & (df1['age_group'] == age.value)]['year'], 
                    y = df1[(df1['entity'] == entity1.value) & (df1['age_group'] == age.value)]['deaths'], 
                    name = 'Country1')
trace2 = go.Scatter(x= df1[(df1['entity'] == entity2.value) & (df1['age_group'] == age.value)]['year'], 
                    y = df1[(df1['entity'] == entity2.value) & (df1['age_group'] == age.value)]['deaths'], 
                    name = 'Country2')
g = go.FigureWidget(data=[trace1, trace2],
                    layout=go.Layout(
                        title=dict(
                            text=' Malaria deaths per 100,000 people across time'
                        ),
                        xaxis_range=[min(df1['year']), max(df1['year'])],
                        template= 'seaborn'
                    ))
```

and define the function to update the dataset used for plotting by the interactive input
```python
def response(change):
    filter_list1 = [i and j for i, j in
                    zip(df1['entity'] == entity1.value, df1['age_group'] == age.value)]
    temp_df1 = df1[filter_list1]
    filter_list2 = [i and j for i, j in
                    zip(df1['entity'] == entity2.value, df1['age_group'] == age.value)]
    temp_df2 = df1[filter_list2]
        
    with g.batch_update():
        g.data[0].x = temp_df1['year']
        g.data[0].y = temp_df1['deaths']
        g.data[1].x = temp_df2['year']
        g.data[1].y = temp_df2['deaths']
        g.layout.xaxis.title = 'Year'
        g.layout.yaxis.title = 'Number of Deaths'


entity1.observe(response, names="value")
age.observe(response, names="value")
entity2.observe(response, names="value")
```

Finally, update the plot
```python
container2 = widgets.HBox([age])
widgets.VBox([container2,
              container,
              g])
```


## Dot Plot (Bubble Plot)<a name="section2"></a>

I use plotly again because it can plot 'animated' graph conviniently by slider by looking at the documentation <https://plotly.com/python/sliders/>. 

The second plot I made demonstrate how gdp and deaths number evolves by time (labelled by continent).
![Hi][dot_plot]

The data of population is from United Nations, Department of Economic and Social Affairs. Because the exact population is only updated to the year 2010, and therefore I only use the data from `alaria_deaths.csv` up to 2010. The data of GDP is from worldbank.org.


Read the data of number of death of malaria and the mapping table of country code and continent.
```python
deaths = pd.read_csv('malaria_deaths.csv')
deaths.columns = ['entity', 'country code', 'year', 'deaths']

codes = pd.read_csv('country_code.csv')
codes = codes.iloc[:, [0]+[2]+[4]+[5]]
# standarize variable names
codes.columns = ['continent', 'entity', 'country code', 'numeric code']
```

Then read, standarize, and merge the population and GDP data by either country code or numeric country code.
```python
# read population data
pop = pd.read_excel('WPP2010_DB2_F01_TOTAL_POPULATION_BOTH_SEXES.XLS', header = 16)
pop = pop.drop(list(range(0, 9)))
pop = pop.drop(columns=['Index', 'Variant', 'Notes'])
pop = pop.iloc[: , [0] + [1] + list(range(-21, 0))]
pop = pop.rename(columns={'Major area, region, country or area':'entity'})
pop = pd.melt(pop, id_vars = ['entity', 'Country code'])
pop.columns = ['entity', 'numeric code', 'year', 'population']
pop['year'] = pd.to_numeric(pop['year'])

pop1 = pop.merge(codes, on = 'numeric code', how = 'left')
pop1 = pop1.drop(columns=['entity_y', 'numeric code'])
pop1 = pop1.rename(columns={'entity_x':'entity'})

# read gdp data
gdp = pd.read_csv('GDP_country.csv', header = 2)
gdp = gdp.drop(columns=['Indicator Name', 'Indicator Code'])
gdp = gdp.iloc[:,[0] + [1] +list(range(-32, -11))]
gdp = gdp.rename(columns={'Country Name':'entity'})
gdp = pd.melt(gdp, id_vars = ['entity', 'Country Code'])
gdp.columns = ['entity', 'country code', 'year', 'gdp']
gdp['year'] = pd.to_numeric(gdp['year'])

gdp1 = gdp.merge(codes, on = 'country code', how = 'left')
gdp1 = gdp1.dropna(subset=['numeric code'])
gdp1 = gdp1.drop(columns=['entity_y', 'numeric code'])
gdp1 = gdp1.rename(columns={'entity_x':'entity'})

# merge data

# merge deaths and population
death_pop = deaths.merge(pop1, on = ['country code', 'year'], how = 'left')
death_pop = death_pop.drop(columns=['entity_y'])
death_pop = death_pop.rename(columns={'entity_x':'entity'})
death_pop = death_pop.dropna(subset=['population'])

# merge with GDP
df2 = death_pop.merge(gdp1, on=['year', 'country code', 'continent'], how = 'left')
df2 = df2.drop(columns=['entity_y'])
df2 = df2.rename(columns={'entity_x':'entity'})
df2 = df2.dropna(subset=['country code'])
df2['gdp'] = df2['gdp']/1000000
```

Finally, the code to create the interactive plot with the slider is as simple as:

```python
fig = px.scatter(df2, 
                 x="gdp",
                 y="deaths",
                 animation_frame="year", 
                 animation_group="entity",
                 size="population", 
                 color="continent",
                 hover_name="entity", 
                 size_max=60,
                labels={
                     "gdp": "gdp in $1,000,000",
                     "deaths": "deaths per 100,000 people"
                 },
                title="Deaths by Malaria vs. GDP")
fig.show()
```



## Boxplot<a name="section3"></a>

Then I chose `seaborn`, as it produce simple and clear visulizations. For example, by the plot below we can see that the number of incidence of malaria has a clear decreasing trend over years.

After reading in the data on the number of incidence of malaria, I named it `inc`.

```python
ax = sns.boxplot(x="Year", y="Incidence", data=inc)
```

![Hi][boxplot]

We can zoom in by setting the limit of y-axis to take a closer look at the difference between the years

```python
plt.rcParams["figure.figsize"] = [7.50, 7.50]
plt.rcParams["figure.autolayout"] = True
ax = sns.boxplot(x="Year", y="Incidence", data=inc)
plt.ylim(0, 700)

plt.title("Incidence of Malaria(per 1,000 people) at risk by year")

plt.show()
```

![Hi][boxplot_zoomed]



[scatter_plot]: {static}/images/scatter.png
[dot_plot]: {static}/images/dot.png
[boxplot]: {static}/images/box.png
[boxplot_zoomed]: {static}/images/box_zoomed.png