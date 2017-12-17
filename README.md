# project-02-DA

__Case-02.__ Fuel-Economy Dataset
  - package:
  - func: 
-------------------------------------------------------------------------------------------------------------------------------------
__Data:__ Fuel-Economy Dataset (https://www.epa.gov/compliance-and-fuel-economy-data/data-cars-used-testing-fuel-economy) provided by the EPA-US.Environmental Protection Agency (http://www.fueleconomy.gov/feg/download.shtml/). The data is collected from vehicle testing at EPA's National vehicle lab in Michigan. The EPA provide this data to the US government each year to publish their Fuel-Economy guide. we analyize the two dataset from 2008 and 2018. 
 - Model: vehicle make and model 
 - Displ: engine displacement in liters(the size of an engine in liters) 
 - Cyl: number of engine cylinders 
 - Trans: transmission type plus number of gears 
 - Drive: 2-wheel Drive, 4-wheel drive/all-wheel drive 
 - Fuel: fuel type
 - Cert Region: certification region code
 - Stnd: vehicle emissions standard code (https://www.epa.gov/greenvehicles/federal-and-california-light-duty-vehicle-emissions-standards-airpollutants)
 - Underhood ID: engine family or test group ID. This is a 12-digit ID number that can be found on the underhood emission label of every vehicle. It's required by the EPA to designate its "test group" or "engine family."  (http://www.fueleconomy.gov/feg/findacarhelp.shtml#airPollutionScore) 
 - Veh Class: EPA vehicle class (http://www.fueleconomy.gov/feg/findacarhelp.shtml#epaSizeClass) 
 - Air Pollution Score (Smog Rating): (https://www.epa.gov/greenvehicles/smog-rating) 
 - City MPG: Estimated city mpg (miles/gallon) 
 - Hwy MPG: Estimated highway mpg (miles/gallon)
 - Cmb MPG: Estimated combined mpg (miles/gallon) 
 - Greenhouse Gas Score (Greenhouse Gas Rating): (https://www.epa.gov/greenvehicles/greenhouse-gas-rating) 
 - SmartWay: based on the greenhouse gas (GHG) and smog ratings found on vehicle fuel economy labels - Yes, No, or Elite (https://www.epa.gov/greenvehicles/consider-smartwayvehicle)
 - Comb CO2: combined city/highway CO2 tailpipe emissions in grams per mile 

__Story:__ 
The fuel-economy of an automobile is the 'fuel efficiency relationship' between 
 - the distance traveled and 
 - the amount of fuel consumed by the vehicle
> Consumption can be expressed in terms of volume of fuel to travel a distance, or the distance travelled per unit volume of fuel consumed.

__Possible Questions:__ Are more models using alternative sources of fuel? By how much? How much have vehicle classes improved in fuel economy? What features are associated with better fuel economy? For all the models produced in 2008 that are still being produced in 2018, how much has the mpg improved and which vehicle improved the most? 

<img src="https://user-images.githubusercontent.com/31917400/34065415-63386fc6-e1f9-11e7-86c1-b23629421c80.jpg" />

## 1> Assess the Dataset
number of samples in each dataset ?
number of columns in each dataset ?
duplicate rows in each dataset ?
datatypes of columns ?
features with missing values ?
number of non-null unique values for features in each dataset ?
what those unique values are and counts for each ?

### Counting
samples, duplicate rows, rows with missing, unique values
```
df.info()
sum(df.duplicated())
df.isnull().sum(axis=0)
df.nunique()
```
## 2> Cleaning
### Drop extraneous columns
Drop features that aren't consistent (not present in both datasets) or aren't relevant to our aim.
 - Columns to Drop:
   - From 2008 dataset: 'Stnd', 'Underhood ID', 'FE Calc Appr', 'Unadj Cmb MPG'
   - From 2018 dataset: 'Stnd', 'Stnd Description', 'Underhood ID', 'Comb CO2'
```
df_08.drop(['Stnd', 'Underhood ID', 'FE Calc Appr', 'Unadj Cmb MPG'], axis=1, inplace=True)
df_18.drop(['Stnd', 'Stnd Description', 'Underhood ID', 'Comb CO2'], axis=1, inplace=True)
```
### Rename Columns
Change the "Sales Area" column label in the 2008 dataset to "Cert Region" for consistency.
```
df_08.rename(columns = {'Sales Area':'Cert Region'}, inplace=True)
```
Rename all column labels to replace spaces with underscores and convert everything to lowercase. Being consistent with lowercase and underscores makes things easier.  
```
df_08.rename(columns = lambda x: x.strip().lower().replace(" ", "_"), inplace=True)
```
**Confirm column labels for 2008 and 2018 datasets are identical**
```
df_08.columns == df_18.columns
(df_08.columns == df_18.columns).all()
```
### Querying and Dropping
For consistency, only compare cars certified by 'California standards', filter both datasets using 'query()' to select only rows where cert_region is CA. Then, drop the cert_region columns, since it will no longer provide any useful information. 
```
df_08 = df_08.query('cert_region=="CA"')
df_18 = df_18.query('cert_region=="CA"')

df_08['cert_region'].unique()   #confirm
df_18['cert_region'].unique()   #confirm

df_08 = df_08.drop('cert_region', axis=1) 
df_18 = df_18.drop('cert_region', axis=1)

df_08.shape   #confirm
df_18.shape   #confirm
```
Drop any rows in both datasets that contain missing values.
```
df_08.isnull().sum(axis=0)
df_08.isnull().sum(axis=0)

df_08.dropna(axis=0, inplace=True)
df_18.dropna(axis=0, inplace=True)

df_08.isnull().sum().any()   #confirm
df_18.isnull().sum().any()   #confirm
```
Drop any duplicate rows in both datasets (dedupping).
```
df_08.duplicated().sum()
df_18.duplicated().sum()

df_08.drop_duplicates(inplace=True)
df_18.drop_duplicates(inplace=True)

print(df_08.duplicated().sum())   #confirm
print(df_18.duplicated().sum())   #confirm
```
### Fixing Data-Types
> NOTE: extract int from str in pandas
 - > You can convert to string then extract the integer using regular expressions: df['column'].str.extract('(\d+)')

What changes should be made to make them practical and consistent in both datasets? 
 - 1)Fix 'cyl' datatype (it's categorical): 
   - 2008: extract int from str using 'regular expression', then convert to int. 
   - 2018: convert float to int...using 'astype(int)'.
```
df_08['cyl'].value_counts()
df_08['cyl'].str.extract('(\d+)').astype(int)
df_08['cyl'].value_counts()

df_18['cyl'] = df_18['cyl'].astype(int)
```
 - 2)Fix 'air_pollution_score' datatype
   - 2008: convert string to float.
   - 2018: convert int to float.
```
df_08.air_pollution_score = df_08.air_pollution_score.astype(float)
............ERROR
```
<img src="https://user-images.githubusercontent.com/31917400/34075216-a8f0156e-e2b7-11e7-8b33-ff6a05107727.jpg" width="350" height="100" />

Looks like this isn't going to be as simple as converting the datatype. According to the error above, the value at row 582 is "6/4"
```
df_08.iloc[582]
```
<img src="https://user-images.githubusercontent.com/31917400/34075494-7ad54fe2-e2c0-11e7-8b16-649adcaa4c5d.jpg" width="250" height="150" />

The mpg columns and greenhouse gas scores also seem to have the same problem - maybe that's why these were all saved as strings! According to the document, all vehicles with more than one fuel type, or hybrids, like the one above (it uses ethanol AND gas) will have a string that holds two values - one for each. so...First, let's get all the hybrids in 2008
```
hb_08 = df_08[df_08['fuel'].str.contains('/')]; hb_08
hb_18 = df_18[df_18['fuel'].str.contains('/')]; hb_18
```
The 2008 dataset only has one! 

<img src="https://user-images.githubusercontent.com/31917400/34075553-30f23b9a-e2c2-11e7-8671-78e306a7d9be.jpg" width="600" height="40" />

The 2018 has MANY more...

<img src="https://user-images.githubusercontent.com/31917400/34075554-339828fa-e2c2-11e7-9e27-fac2b6fc0706.jpg" width="600" height="250" />

Take each hybrid row and split them into two new rows - one with values for the first fuel type (values before the "/"), and the other with values for the second fuel type (values after the "/") and separate them with two dataframes! First, create two copies of the 2008 hybrids dataframe.
```
df1 = hb_08.copy()  # data on 'first' fuel type of each hybrid vehicle
df2 = hb_08.copy()  # data on 'second' fuel type of each hybrid vehicle

df3 = hb_18.copy()
df4 = hb_18.copy()
```
Define columns to split by "/"
```
split_columns_08 = ['fuel', 'air_pollution_score', 'city_mpg', 'hwy_mpg', 'cmb_mpg', 'greenhouse_gas_score']

split_columns_18 = ['fuel', 'city_mpg', 'hwy_mpg', 'cmb_mpg'] 
# No need to split for air_pollution_score or greenhouse_gas_score here because these columns are already 'ints' in the 2018 dataset.
```
Apply split function to each column of each dataframe-copy then combine dataframes to add to the original dataframe
```
for c in split_columns_08:
    df1[c] = df1[c].apply(lambda x: x.split("/")[0])
    df2[c] = df2[c].apply(lambda x: x.split("/")[1])
new_rows = df1.append(df2); new_rows

for c in split_columns_18:
    df3[c] = df3[c].apply(lambda x: x.split('/')[0])
    df4[c] = df4[c].apply(lambda x: x.split('/')[1])
new_rows2 = df3.append(df4); new_rows2
```
Drop the original hybrid rows and add in our newly separated rows
```
df_08.drop(hb_08.index, inplace=True)
df_08 = df_08.append(new_rows, ignore_index=True)
df_08[df_08['fuel'].str.contains('/')]    #check '/' are gone...

df_18.drop(hb_18.index, inplace=True)
df_18 = df_18.append(new_rows2, ignore_index=True)
df_18[df_18['fuel'].str.contains('/')]    #check '/' are gone...
```
Now fianlly, comfortably continue the changes needed for 'air_pollution_score'.
```
# convert string to float for 2008 air pollution column
df_08.air_pollution_score = df_08.air_pollution_score.astype(float)

# convert int to float for 2018 air pollution column
df_18.air_pollution_score = df_18.air_pollution_score.astype(float)
```
 - 3)Fix city_mpg, hwy_mpg, cmb_mpg datatypes
   - 2008 and 2018: convert string to float.
```
mpg_columns = ['city_mpg','hwy_mpg','cmb_mpg']
for c in mpg_columns:
    df_18[c] = df_18[c].astype(float)
    df_08[c] = df_08[c].astype(float)
```
 - 4)Fix greenhouse_gas_score datatype
   - 2008: convert from float to int.
```
df_08['greenhouse_gas_score'] = df_08['greenhouse_gas_score'].astype(int)

#confirm using 'dtypes'
df_08.dtypes
df_18.dtypes
df_08.dtypes == df_18.dtypes
```
## 3> Exploring with Visuals
 - Q0: correlation?
 - Q1: Are more unique models using alternative sources of fuel? By how much?
 - Q2: How much have vehicle classes improved in fuel economy (increased in mpg)?
 - Q3: What are the characteristics of SmartWay vehicles? Have they changed over time? (mpg, greenhouse gas)
 - Q4: What features are associated with better fuel economy (mpg)?
 - Q5: For all of the models that were produced in 2008 that are still being produced in 2018, how much has the mpg improved and which vehicle improved the most? (This is a question regarding models that were updated since 2008 and still being produced in 2018. In order to do this, we need a way to compare models that exist in both datasets.)
 
#### Q0. What's the correlation between 'displacement' and 'combined mpg'? and the correlation between 'greenhouse gas score' and 'combined mpg'?
```
df_08['displ'].corr(df_08['cmb_mpg'])   # -0.81795323414579513
df_18['displ'].corr(df_08['cmb_mpg'])   # 0.011908849032669692

df_08['greenhouse_gas_score'].corr(df_08['cmb_mpg'])   # 0.94547041103218687
df_18['greenhouse_gas_score'].corr(df_08['cmb_mpg'])   # -0.015115270816720715
```
#### Q1. Are more unique models using alternative sources of fuel? By how much?
<img src="https://user-images.githubusercontent.com/31917400/34084014-7c5aabe4-e371-11e7-973a-b24dacf0d456.jpg" width="450" height="180" />

These are the sources of fuel. Which ones are alternative sources? Looks like the alternative sources of fuel available in 2008 are CNG and ethanol, and those in 2018 ethanol and electricity.
```
alt_08 = df_08.query('fuel in ["CNG", "ethanol"]')['model'].nunique()
alt_18 = df_18.query('fuel in ["Ethanol", "Electricity"]').model.nunique()
total_08 = df_08.model.nunique()  # 377
total_18 = df_18.model.nunique()  # 357

plt.bar(["2008", "2018"], [alt_08, alt_18])
plt.title("Number of Unique Models Using Alternative Fuels")
plt.xlabel("Year")
plt.ylabel("Number of Unique Models")
```
<img src="https://user-images.githubusercontent.com/31917400/34084097-eaa84fd8-e372-11e7-8e8c-7510e9160ab0.jpg" width="250" height="150" />

#### Q2. How much have vehicle classes improved in fuel economy (increased in mpg)?
<img src="https://user-images.githubusercontent.com/31917400/34084286-5869ae52-e375-11e7-818d-9fed4d596c99.jpg" width="600" height="250" />

These are average fuel economy for each vehicle class for both years.
```
inc = veh_18 - veh_08
inc.dropna(inplace=True)

plt.subplots(figsize=(8, 5))
plt.bar(inc.index, inc)
plt.title('Improvements in Fuel Economy from 2008 to 2018 by Vehicle Class')
plt.xlabel('Vehicle Class')
plt.ylabel('Increase in Average Combined MPG');
```
<img src="https://user-images.githubusercontent.com/31917400/34084344-3d70b180-e376-11e7-9c18-76fe09bf8a93.jpg" width="350" height="200" />

#### Q3. What are the characteristics of SmartWay vehicles? Have they changed over time? (mpg, greenhouse gas)
<img src="https://user-images.githubusercontent.com/31917400/34084809-9b7a3fbe-e37e-11e7-974e-ab72634b03c4.jpg" width="450" height="100" />
<img src="https://user-images.githubusercontent.com/31917400/34084880-8dfb5e8a-e37f-11e7-85c7-2e5a76c215c4.jpg" />

#### Q4. What features are associated with better fuel economy (mpg)?
<img src="https://user-images.githubusercontent.com/31917400/34084014-7c5aabe4-e371-11e7-973a-b24dacf0d456.jpg" width="600" height="250" />

We explore trends between 'cmb_mpg' and the other features in this dataset, or filter this dataset like in the previous question and explore the properties of that dataset. We select all vehicles that have the top 50% fuel economy ratings than compare. 
```
top_08 = df_08.query('cmb_mpg > cmb_mpg.mean()')
top_08.describe()
top_18 = df_18.query('cmb_mpg > cmb_mpg.mean()')
top_18.describe()
```
<img src="https://user-images.githubusercontent.com/31917400/34084941-bdac730c-e380-11e7-91c3-540cd8cd9e56.jpg" />

#### Q5. For all of the models that were produced in 2008 that are still being produced in 2018, how much has the mpg improved and which vehicle improved the most? 
 >This is a question regarding models that were updated since 2008 and still being produced in 2018. In order to do this, we need a way to compare models that exist in both datasets.
#### Merging





























### 00. Python RegEx

























