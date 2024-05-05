#Importing required libraries and packages
import pandas as pd
import altair as alt 
import numpy as np 
from datetime import datetime
import warnings
warnings.filterwarnings('ignore')
df=pd.read_csv("Boonsong Lekagul waterways readings.csv")
df.head()
df.isnull()
df['measure'].nunique()
alt.data_transformers.enable('default',max_rows=None)
#Adding month and year colums
#Extracting month and year from each sample in the 'sample date' column
df['sample date'] = pd.to_datetime(df['sample date'])
df['year'] = df['sample date'].dt.year
df['month'] = df['sample date'].dt.month
df.head()
#Water Temperature Trend
#Q1
#i) Trend 1 (Water Temperature)
#Line chart showing water temperatures every month in every year 
water_temp_data = df[df['measure'] == 'Water temperature']
slider = alt.binding_range(min=min(df['year']), max=max(df['year']), step=1,name='Year')
selector = alt.selection_point(fields=['year'], bind=slider)
water_temp_trend = alt.Chart(water_temp_data).mark_line().encode(x=alt.X('month(sample date):N'),y=alt.Y('value:Q'),color='year:N').properties(width=800,title='Water Temperature : Year - Month').add_selection(selector).transform_filter(selector)
water_temp_trend
#Heatmap showing intensity of mean water temperature every month in each location
hexagon = "M0,-2.3094010768L2,-1.1547005384 2,1.1547005384 0,2.3094010768 -2,1.1547005384 -2,-1.1547005384Z"
water_temp_heatmap=alt.Chart(water_temp_data).mark_point(size=200,shape=hexagon).encode(x=alt.X('location:N'),y=alt.Y('month(sample date)'),fill=alt.Color('mean(value):Q', scale=alt.Scale(scheme='inferno',reverse=True)),stroke=alt.value('black'),strokeWidth=alt.value(0.1),tooltip=['month(sample date):O','mean(value):Q']).properties(width=600,height=280,title='Water Temperature : Location - Month')
water_temp_heatmap
#Trend of Top 5 Water Quality Indicators
#Q1
#i) Trend 2 (Top 5 Water Quality Indicators Trend-line Location-wise)
input_features_dropdown = alt.binding_select(options=['Achara','Boonsri','Busarakhan','Chai','Decha','Kannika','Kohsoom','Sakda','Somchair','Tansanee'],name='Location')
my_selection = alt.selection_point(fields=['location'], bind=input_features_dropdown)
#Grouping measures based on mean values and sorting them in descending order and choosing the first five
top_measures = df.groupby(['measure'])['value'].mean().sort_values(ascending=False).head(5).reset_index()
df_top_measures = df[df['measure'].isin(top_measures['measure'].unique())]
#Creating line chart to show trend lines of top 5 water quality indicators 
top5_trend_line = alt.Chart(df_top_measures).mark_line(strokeWidth=4).encode(x='year:O',y='mean(value):Q',color='measure',tooltip=[ 'year:O', 'mean(value):Q']).properties(width=750, height=60, title='Top Five Water Quality Indicators -  Trend line Over Years : Location Wise').add_selection(my_selection).transform_filter(my_selection)
alt.layer(top5_trend_line).facet(row='measure')
#Anomalous Iron Values in 2003
#Q1 
#ii) Anomaly (Iron 2003)
#Scatter chart to show all anomalous values in the dataset 
anomaly_chart1 = alt.Chart(df).mark_circle().encode(alt.X('year:O'),alt.Y('value'),size='value',color='measure',tooltip=['measure:N']).properties(width=600,title='Spike in Iron Values in 2003')
#Creating a dataframe taking rows where measure = iron in the original dataframe 
iron_values_df = df[df['measure'] == 'Iron']
#Creating a stacked bar chart to show anomalous iron values compared to other years and locations 
anomaly_chart2 = alt.Chart(iron_values_df).mark_bar(stroke='brown',strokeWidth=2).encode(alt.X('year:O'),alt.Y('mean(value):Q'),alt.Color('location:N'),tooltip=['location:N','sample date:T','mean(value):Q']).properties(width=600)
anomaly_chart1 & anomaly_chart2
#Missing Data 
#Q2
#i) Missing Data Chart 1 (Percentage of measures recorded in all years and locations)
total_measures_overall=df['measure'].nunique()
#Extracting unique measures 
measures_per_location_year = df.groupby(['location', 'year'])['measure'].unique().reset_index(name='measures_list')
common_measures = set(measures_per_location_year['measures_list'].iloc[0])
#Extracting measures that are recorded in every year and location using 'intersection'
for measures_list in measures_per_location_year['measures_list']:
    common_measures = common_measures.intersection(set(measures_list))
#Taking the count of measures and its percentage out of the total
num_common_measures=len(common_measures)
num_uncommon_measures=total_measures_overall - len(common_measures)
percentage_common = (num_common_measures / total_measures_overall) * 100
percentage_not_common = (num_uncommon_measures / total_measures_overall) * 100
#Creating pie chart to show the proportion of measures recorded in every year and location
pie_data = pd.DataFrame({
    'Category': ['Measures Recorded in Every Year and Location', 'Measures Not Recorded in Every Year and Location'],
    'Number':[num_common_measures,num_uncommon_measures],
    'Percentage': [percentage_common, percentage_not_common]
})
pie_chart = alt.Chart(pie_data).mark_arc().encode(theta='Percentage:Q',color=alt.Color('Category:N', scale=alt.Scale(range=['#FFA500','#FF4500'])),tooltip=['Category:N','Number:Q','Percentage:Q']).properties(width=400,height=400,title='Percentage of Measures tested in Every Year and Location')
pie_chart
#Q3
#i) Missing Data Chart 2 
#Heatmap to spot missing data and to see number of records taken every year in each location
custom_order = ['Boonsri', 'Busarakhan', 'Chai', 'Kannika','Kohsoom','Sakda','Somchair','Achara','Decha','Tansanee'] 
heatmap=alt.Chart(df).mark_square(size=800).encode(alt.X('year:O'),alt.Y('location',sort=custom_order),color='location',tooltip=['location','year:O','count()'])
text_chart = heatmap.mark_text(baseline='middle', align='center', size=11).encode(text=alt.Text('count():Q'),color=alt.value('black'))
final_chart = heatmap + text_chart
final_chart.properties(height=450, width=600, title='Locations with Missing Data') 
#Change in Collection Frequency
#Q3
#ii) Change in Collection Frequency 
#Stacked bar chart showing change in collection freuquency over the years and locations
count_df = df.groupby(['year', 'location']).size().reset_index(name='count')
collection_frequency = alt.Chart(count_df).mark_bar().encode(x='year:O',y='count:Q',color='location:N',tooltip=['year:O', 'location:N', 'count:Q']).properties(width=600,height=400,title='Stacked Bar Chart - Change in Collection Frequency ')
collection_frequency
#Unrealistic Values (Outliers)
#Q3
#iii) Unrealistic Values (Outliers)
#Using Isolation Forest with contamination rate = 0.001 to flag outlier rows
from sklearn.ensemble import IsolationForest
def detect_and_visualize_outliers(df, value_column='value', contamination=0.001, random_seed=43):
  measures_agg = df.groupby(['sample date', 'location', 'measure']).agg(value=('value', 'mean')).reset_index()
  features = df[['value']]
  outlier = IsolationForest(n_estimators=200, contamination=contamination, random_state=random_seed)
  df['value_outliers'] = outlier.fit_predict(features)
  value_outliers = df[df['value_outliers'] == -1][['measure', 'value_outliers']].groupby(['measure']).count().reset_index()
  return df,value_outliers
df, value_outliers = detect_and_visualize_outliers(df)
contamination=0.001
#Bar chart showing count of outliers in each outlier measure 
value_outliers_chart = alt.Chart(value_outliers).mark_bar().encode(x=alt.X('measure', sort='-y',title= 'Measure'),y=alt.Y('value_outliers',title='Count of Outliers'),tooltip=['value_outliers', 'measure'],color='measure:N').properties(width=300, height=250, title=f'Outliers {contamination * 100}% Contamination')
value_outliers_chart
#Creating boxplots of outlier measures to check for distribution of outliers in each location 
df['outliers']=np.where((df['measure']=='Total dissolved salts')|(df['measure'] =='Total coliforms')|(df['measure'] =='Manganese')|(df['measure']=='Zinc')|(df['measure']=='Iron')|(df['measure']=='Copper')|(df['measure']=='Aluminium')|(df['measure']=='Fecal coliforms'),'outliers','Others') 
outliers_col=df[df['outliers']=='outliers']
selected_measures = ['Total dissolved salts', 'Total coliforms', 'Manganese', 'Iron', 'Zinc', 'Copper', 'Aluminium', 'Fecal coliforms']
outlier_df = {}
for measure in selected_measures:
    outlier_df[measure.lower().replace(' ', '_')] = df[df['measure'] == measure].copy()
def create_outlier_chart(outlier_df, measure_name):
    return alt.Chart(outlier_df).mark_boxplot(extent='min-max').encode(x=alt.X('location:N'),y=alt.Y('mean(value):Q', scale=alt.Scale(zero=False)),color='location',tooltip=['location:N', 'mean(value)'],).properties(title=f'{measure_name}: Outlier plot')
tds = create_outlier_chart(outlier_df['total_dissolved_salts'], 'Total dissolved salt')
tc = create_outlier_chart(outlier_df['total_coliforms'], 'Total coliforms')
mg = create_outlier_chart(outlier_df['manganese'], 'Manganese')
z = create_outlier_chart(outlier_df['zinc'], 'Zinc')
ir = create_outlier_chart(outlier_df['iron'], 'Iron')
cpr = create_outlier_chart(outlier_df['copper'], 'Copper')
alm = create_outlier_chart(outlier_df['aluminium'], 'Aluminium')
fc = create_outlier_chart(outlier_df['fecal_coliforms'], 'Fecal coliforms')
((tds|tc|mg|alm)&(z|ir|cpr|fc)).properties(title='Outlier measure: Plot over regions')
