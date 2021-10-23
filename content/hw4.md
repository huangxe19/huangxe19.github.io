title: Homework 4
date: 10/23/2021
author: Xige Huang

# Is there life after graduation? - visualization and analysis in dashboard

Data Source for this assignment: <https://ncses.nsf.gov/pubs/nsf19301/data>

I used `dash` by `plotly` to build the dashboard, as I'm becomming familiar with the plots using `plotly` so I imagine it would be more straightforward to integrate the plots into the dashboard.

## Plot1

First I want to have a look of the trend of number of doctorates recipients by years; this may provide us the insight that if post doctoral plans can affect the number of people applying for doctorates degree.

The dataset cleaned for plotting looks like:
![Hi][df1]

The plot will look like:
![Hi][plot1]

The code for a scatter plot by year
```python
px.line(df_recip, x='variable', y='value', color='Major field of study',
       labels={"value": "number of degree awarded",
               "variable": "year"},
        title="Number of Doctoral Degree Awarded",
        template='plotly_white')
```

Furthermore, we will need a dropdown menu to filter for the gender of this plot,
```python
dcc.Dropdown(id="gender1_dropdown",
             options=[{'label': i, 'value': i} for i in df1['Gender'].unique()],
             value = 'Total')
```


## Plot2

Next, we can take a look at the median expected salary vs. the median year to graduation (couting from the time of entering the doctorates program) by field of study. Maybe this can provides some insights on if the difficulty to gruaduate influences people's expectation regarding post-doc plans.

The cleaned dataframe for plotting looks like:
![Hi][df2]

I integrated the two indicators into one grouped histogram, with separated y-axis. The plot should look like:
![Hi][plot2]

The code for the grouped bar plot
```python
df_expsal_study1 = df_expsal_study[df_expsal_study['variable'] == 'Male']
df_median_grad1 = df_median_grad[df_median_grad['variable'] == 'Male']
fig = make_subplots(specs=[[{"secondary_y": True}]])

fig.add_trace(
    go.Bar(x=df_expsal_study1['Field of study'], 
           y=df_expsal_study1['value'],
           name = 'expected salary',
          offsetgroup=1),
          secondary_y=False
)
fig.add_trace(
    go.Bar(x=df_median_grad1['Field of study'], 
           y=df_median_grad1['value'],
           name = 'median year to graduation',
          offsetgroup=2),
          secondary_y=True
)

# Add figure title
fig.update_layout(
    title_text="Expected sal and median years to doctorates degree"
)

# Set x-axis title
fig.update_xaxes(title_text='Field of Study')

# Set y-axes titles
fig.update_yaxes(title_text="expected salary", secondary_y=False)
fig.update_yaxes(title_text="years to degree", secondary_y=True)


fig.show()
```

For this plot, we will need two dropdowns: one filters gender and the other filters the type of post doctoral plan (employment or post-doc study)

```python
dcc.Dropdown(id="gender2_dropdown",
             options=[{'label': i, 'value': i} for i in df_median_grad['variable'].unique()],
             value = 'Female')
```

```python
dcc.Dropdown(id='type1_dropdown',
             options=[{"label": "Employment", "value": "Employment"},
             {"label": "Postdoctoral study", "value": "Postdoctoral study"}],
             value = 'Employment')
```

## Plot 3

Finally, we can take a look at the proportion of different post doctoral plans by field of study.
One thing to note here is that I directly format the data in excel, instead of in pandas.
I have valid reason to do so, hope you read this before you deduct any point for not doing work in pandas:
the data is obviously format for display purpose in excel, therefore it would be more reasonable to convert it to the format for which I desired in excel. For example, there are footnotes for some of the cells, so direcly reading it by pandas will result 'typos' as 'othersb' where 'b' is originally a footnote. Hence, I did all data processing in excel.

The original data in excel:
![Hi][raw_df]

Cleaning the data solely in pandas is too annoying and incovinient, for example the code for which I used for processing the data for the grouped bar plot looks like:

```python
df2 = pd.read_excel('postdoc_expected_sal.xlsx', header = 3)
df2 = df2.iloc[[3,7,11,12,16,18,19,21]]
df_expsal_emp = df2[['Field of study', 'Employment', 'Unnamed: 2']]
df_expsal_emp.columns = ['Field of study', 'Male', 'Female']
df_expsal_emp.loc[:, 'Type'] = 'Employment'
df_expsal_emp = pd.melt(df_expsal_emp, id_vars = ['Field of study', 'Type'])
df_expsal_emp['Field of study'] = df_expsal_emp['Field of study'].replace({'Psychology and social sciences ':'Psychology and social sciences', 'Other non-S&E fieldsb':'Other non-S&E fields'})
df_expsal_study = df2[['Field of study', 'Postdoctoral study', 'Unnamed: 4']]
df_expsal_study.columns = ['Field of study', 'Male', 'Female']
df_expsal_study.loc[:, 'Type'] = 'Postdoctoral study'
df_expsal_study = pd.melt(df_expsal_study, id_vars = ['Field of study', 'Type'])
df_expsal_study['Field of study'] = df_expsal_study['Field of study'].replace({'Psychology and social sciences ':'Psychology and social sciences','Other non-S&E fieldsb':'Other non-S&E fields'})
df3 = pd.read_excel('doc_median_year.xlsx', header = 3)
df3 = df3.iloc[[2,3]]
df3 = df3.T
df3.columns = df3.iloc[0]
df3 = df3.iloc[2:]
df3['Years since entering doctoral programe'] = df3.index
df3.columns = ['Male', 'Female','Field of study']
df3['Field of study'] = df3['Field of study'].replace({'Otherb':'Other non-S&E fields', 'Life sciencesa':'Life sciences'})
df_median_grad = df3
df_median_grad.loc[:, 'Type'] = 'Median Years to Graduate'
df_median_grad = pd.melt(df_median_grad, id_vars = ['Field of study', 'Type'])
df_expsal = pd.concat([df_expsal_study, df_expsal_emp])
```
I think one important thing is that we know how to process data wisely to save time.

Therefore I just read in the pre-processed data, and it looks like:
![Hi][df3]

The desired pie chart should look like: 
![Hi][plot3]

from code
```python    
fig = px.pie(filter_df, values = filter_df['Value'], names=filter_df['Category'],
             title = 'Post-doctoral plans by type, field, and gender')

fig.show()
```

For this pie chart we will need three dropdowns: one filters gender, the second filters the type of indicator for post doctoral plan (for example, location or activity for job), and the third filters the field of study.

```python    
dcc.Dropdown(id='gender3_dropdown',
             options=[{'label': i, 'value': i} for i in df4['Gender'].unique()],
             value = 'Total')

dcc.Dropdown(id='field_dropdown',
             options=[{'label': i, 'value': i} for i in df4['Field of Study'].unique()],
             value = 'Life sciences')
             
dcc.Dropdown(id='ind_dropdown',
             options=[{'label': i, 'value': i} for i in df4['Type'].unique()],
             value = 'Postgraduation status')
```


## Dashboard
Then I just need to integrate the plots and dropdown menus. Specifically, update of the plots by the dropdowns should be done by
`@app.callback` following the specific output and input. Moreover, because this is not a very complicated dash app, I format the components directly by adding `style = {}` following the components, instead of using `css` formating.

The complete code for the layout of the app:
```python  
app.layout = html.Div([ 
    html.Div(children = [
        html.H1(children = ["Is there life after graduate school?"],
                style = {'color':'#FFFFFF',
                         'font-size': '48px',
                         'font-weight': 'bold',
                         'text-align': 'center'}),
        html.P(children = ["Analysis of doctorate recipients and post doctoral plans"],
                style = {'color':'#CFCFCF',
                         'margin': '4px auto',
                         'text-align': 'center'}),
    ],style = {'background-color': '#222222', 
               'height': '288px', 'padding': '16px 0 0 0'}), # end of header
     
    html.Div(
            children=[
                html.Div(
                    children=[
                        html.Div(children=["Gender"],
                                 style = {'margin-bottom': '6px',
                                          'font-weight': 'bold',
                                          'color': '#079A82'}),
                        dcc.Dropdown(
                            id="gender1_dropdown",
                            options=[{'label': i, 'value': i} for i in df1['Gender'].unique()],
                            value = 'Total'),
                    ], style={'width': '40%', 'display': 'inline-block'}),
            ],style = {'height': '100px','width': '912px',
                       'display':'flex', 
                       'justify-content': 'space-evenly',
                       'padding-top': '24px', 'margin': '-80px auto 0 auto',
                       'background-color': '#FFFFFF',
                       'box-shadow': '0 4px 6px 0 rgba(0, 0, 0, 0.18)',}), # end of plot1 dropdown
    
    html.Div(
    children = [
    html.Div(
        children = [
            dcc.Graph(id='doc_recip')],
              style = {'margin-bottom': '24px', 'box-shadow': '0 4px 6px 0 rgba(0, 0, 0, 0.18)'})
    ]
             , style={'margin-right': 'auto','margin-left': 'auto',
              'max-width': '1024px','padding-right': '10px',
              'padding-left': '10px', 'margin-top': '32px'}), # end of plot1
    
    html.Div(
            children=[
                 html.Div(
                    children=[
                        html.Div(children=["Gender"], style = {'margin-bottom': '6px',
                                                               'font-weight': 'bold', 'color': '#079A82'}),
                        dcc.Dropdown(
                            id="gender2_dropdown",
                            options=[{'label': i, 'value': i} for i in df_median_grad['variable'].unique()],
                            value = 'Female')
                    ], style={'width': '30%', 'float': 'center'}),
                html.Div(
                    children=[
                        html.Div(children=["Type of post-doc plan"], style = {'margin-bottom': '6px',
                                                               'font-weight': 'bold', 'color': '#079A82'}),
                        dcc.Dropdown(
                            id='type1_dropdown', 
                            options=[{"label": "Employment", "value": "Employment"},
                                     {"label": "Postdoctoral study", "value": "Postdoctoral study"}],
                            value = 'Employment')
                    ], style={'width': '30%', 'display': 'inline-block'}),
                
                    
            ],style = {'height': '100px','width': '912px',
                       'display':'flex', 'justify-content': 'space-evenly',
                       'padding-top': '24px', 'margin': '50px auto 0 auto',
                       'background-color': '#FFFFFF',
                       'box-shadow': '0 4px 6px 0 rgba(0, 0, 0, 0.18)',}), # end of plot2 dropdown
    
    html.Div(
    children = [
    html.Div(
        children = [
            dcc.Graph(id='year_sal')],
              style = {'margin-bottom': '24px', 'box-shadow': '0 4px 6px 0 rgba(0, 0, 0, 0.18)'})
    ]
             , style={'margin-right': 'auto','margin-left': 'auto',
              'max-width': '1024px','padding-right': '10px',
              'padding-left': '10px', 'margin-top': '32px'}), # end of plot2
    
    html.Div(
            children=[
                html.Div(
                    children=[
                        html.Div(children=["Gender"], style = {'margin-bottom': '6px',
                                                               'font-weight': 'bold', 'color': '#079A82'}),
                        dcc.Dropdown(id='gender3_dropdown', 
                                     options=[{'label': i, 'value': i} for i in df4['Gender'].unique()],
                                     value = 'Total')
                    ], style={'width': '30%', 'float': 'center'}),
                html.Div(
                    children=[
                        html.Div(children=["Field of study"], 
                                 style = {'margin-bottom': '6px', 'font-weight': 'bold', 'color': '#079A82'}),
                        dcc.Dropdown(id='field_dropdown', 
                                     options=[{'label': i, 'value': i} for i in df4['Field of Study'].unique()],
                                     value = 'Life sciences')
                    ], style={'width': '30%', 'display': 'inline-block'}),
                
                html.Div(
                    children=[
                        html.Div(children=["Indicator of postdoctoral plan"], 
                                 style = {'margin-bottom': '6px', 'font-weight': 'bold', 'color': '#079A82'}),
                        dcc.Dropdown(id='ind_dropdown', 
                                     options=[{'label': i, 'value': i} for i in df4['Type'].unique()],
                                     value = 'Postgraduation status')
                    ], style={'width': '30%', 'display': 'inline-block'}),
                    
            ],style = {'height': '100px','width': '912px',
                       'display':'flex', 'justify-content': 'space-evenly',
                       'padding-top': '24px', 'margin': '50px auto 0 auto',
                       'background-color': '#FFFFFF',
                       'box-shadow': '0 4px 6px 0 rgba(0, 0, 0, 0.18)',}), # end of plot3 dropdown
    
    html.Div(
    children = [
    html.Div(
        children = [
            dcc.Graph(id='post_plan')],
              style = {'margin-bottom': '24px', 'box-shadow': '0 4px 6px 0 rgba(0, 0, 0, 0.18)'})
    ], style={'margin-right': 'auto','margin-left': 'auto',
              'max-width': '1024px','padding-right': '10px',
              'padding-left': '10px', 'margin-top': '32px'}), # end of plot3
    
    
])
# end of app layout


@app.callback(
    dash.dependencies.Output('doc_recip', 'figure'),
    Input(component_id='gender1_dropdown', component_property='value'))
def plot_recip(selected_gender):
    filter_df = df1[df1['Gender'] == selected_gender]
    fig = px.line(filter_df, x='variable', y='value', color='Major field of study',
       labels={
                     "value": "number of degree awarded",
                     "variable": "year"
                 },
        title="Number of Doctoral Degree Awarded",
        template='plotly_white')
    fig.update_layout(legend=dict(font = dict(size=10), 
                                  orientation="h",
                                  yanchor="top", y=-0.25,
                                  xanchor="right", x=1))
    return fig


@app.callback(
    dash.dependencies.Output('year_sal', 'figure'),
    Input(component_id='gender2_dropdown', component_property='value'),
    Input(component_id='type1_dropdown', component_property='value'))

def plot_year_sal(gender2_dropdown, type1_dropdown):
    filter_year = df_median_grad[df_median_grad['variable'] == gender2_dropdown]
    filter_expsal = df_expsal[(df_expsal['variable'] == gender2_dropdown)&(df_expsal['Type'] == type1_dropdown)]
    
    fig = make_subplots(specs=[[{"secondary_y": True}]])
    fig.add_trace(
    go.Bar(x=filter_expsal['Field of study'], 
           y=filter_expsal['value'],
           name = 'median epected salary',
          offsetgroup=1),
          secondary_y=False)
    fig.add_trace(
    go.Bar(x=filter_year['Field of study'], 
           y=filter_year['value'],
           name = 'median year to degree',
          offsetgroup=2),
          secondary_y=True)
    
    # Add figure title
    fig.update_layout(
    title_text="Median expected salary and median years to doctorates degree, 2017")
    
    # Set x-axis title
    fig.update_xaxes(title_text='Field of Study')

    # Set y-axes titles
    fig.update_yaxes(title_text="expected salary", secondary_y=False)
    fig.update_yaxes(title_text="years to degree", secondary_y=True)

    return(fig)

@app.callback(
    dash.dependencies.Output('post_plan', 'figure'),
    Input(component_id='gender3_dropdown', component_property='value'),
    Input(component_id='ind_dropdown', component_property='value'),
    Input(component_id='field_dropdown', component_property='value'))

def plot_year_sal(gender3_dropdown, ind_dropdown, field_dropdown):
    filter_df = df4[(df4['Field of Study'] == field_dropdown) &(df4['Gender'] == gender3_dropdown)&(df4['Type'] == ind_dropdown)]
    
    fig = px.pie(filter_df, values = filter_df['Value'], names=filter_df['Category'],
                title = 'Post-doctoral plans by type, field, and gender')

    return(fig)
```

The final dashboard looks like:
![Hi][dash1]
![Hi][dash2]
![Hi][dash3]


[df1]: {static}/images/df1.png
[plot1]: {static}/images/plot1.png
[df2]: {static}/images/df2.png
[plot2]: {static}/images/plot2.png
[raw_df]: {static}/images/rawexcel.png
[df3]: {static}/images/df3.png
[plot3]: {static}/images/plot3.png
[dash1]: {static}/images/dash1.png
[dash2]: {static}/images/dash2.png
[dash3]: {static}/images/dash3.png