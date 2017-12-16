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

### 1> Assess the Dataset
number of samples in each dataset ?
number of columns in each dataset ?
duplicate rows in each dataset ?
datatypes of columns ?
features with missing values ?
number of non-null unique values for features in each dataset ?
what those unique values are and counts for each ?

#### Counting
samples, duplicate rows, rows with missing, unique values
```
df.info()
sum(df.duplicated())
df.isnull().sum(axis=0)
df.nunique()
```
### 2> Cleaning
#### Drop extraneous columns
Drop features that aren't consistent (not present in both datasets) or aren't relevant to our aim.
 - Columns to Drop:
   - From 2008 dataset: 'Stnd', 'Underhood ID', 'FE Calc Appr', 'Unadj Cmb MPG'
   - From 2018 dataset: 'Stnd', 'Stnd Description', 'Underhood ID', 'Comb CO2'
```
df_08.drop(['Stnd', 'Underhood ID', 'FE Calc Appr', 'Unadj Cmb MPG'], axis=1, inplace=True)
df_18.drop(['Stnd', 'Stnd Description', 'Underhood ID', 'Comb CO2'], axis=1, inplace=True)
```
#### Rename Columns
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
#### Querying and Dropping
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
#### Fixing Data-Types
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
 - 2)Fix air_pollution_score datatype
   - 2008: convert string to float.
   - 2018: convert int to float.
```



```   
 - 3)Fix city_mpg, hwy_mpg, cmb_mpg datatypes
   - 2008 and 2018: convert string to float.
   
   
 - 4)Fix greenhouse_gas_score datatype
   - 2008: convert from float to int.
































