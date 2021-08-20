
# Company Master (using sql queries)

Company Master is the data project which was previously done by using
python programming language. As we are currently studying the databases
we planned to do this project using the SQL queries.

## Dataset

raw data which is .csv file, it's the real world data of company registration in the state of maharashtra.
below is the link you can download the data from there.

Two dataset has been used for this project:

| Dataset | Link |
| --- | --- |
| Company Master data of Maharashtra | https://tinyurl.com/3453m26x |
| District vs zip data | https://tinyurl.com/39tauusa |


## Queries

To import the .csv data in postgresql. 

create the database schema.

```sql
    create table company_master(
        corporate_identification_number varchar(100),
        company_name varchar(100),
        company_status varchar(50),
        company_class varchar(50),
        company_category varchar(100),
        company_sub_category varchar(100),
        date_of_registration varchar(20),
        registered_state varchar(50),
        authorized_cap varchar(100),
        paidup_capital varchar(100),
        industrial_class varchar(100),
        principal_business_activity_as_per_cin varchar(200),
        registered_office_address varchar(200),
        registrar_of_companies varchar(100),
        email_addr varchar(120),
        latest_year_ar varchar(20),
        latest_year_bs varchar(20)
    )
```

import the data from csv file to the newly created table.

```sql
    \copy company_master from '~/Maharashtra.csv' delimiter ',' csv header; 

```

once the data is imported check using the select command.

```sql
    select * from company_master
```

following data contains more than 4 lakhs record, I am going to considered past
22 years of data, that is from 2000 to 2021.

Query to extract the data respective to the year and create a new table.

```sql
    create table dataset as
    (select * from company_master
    where date_of_registration >= '2000-01-01' and
            date_of_registration < '2021-12-31')
```

**Tasks** 

plot a Histogram on authorized_cap according to the interval, so I created one query which basically updates
the price value into the groups number from (1 to 5) and to store the query I used subqueries and 
created the table with that query.

```sql
    create table Histogram as (
	select company_name,
		case when (authorized_cap <= '100000') then 1
			 when (authorized_cap > '100000' and authorized_cap <= '1000000') then 2
			 when (authorized_cap > '1000000' and authorized_cap <= '10000000') then 3
			 when (authorized_cap > '10000000' and authorized_cap <= '100000000') then 4
			 when (authorized_cap > '100000000') then 5
		end
		as new_authorized_cap
	from dataset
)
```

plot the bar chart for the number of company registration vs year

```sql
    create table barplot as (
        select A.year_company, count(A.year_company)
        from 
            (select extract(year from date_of_registration::date) as year_company
            from dataset) as A
        group by A.year_company
        order by A.year_company
    )
```

plot the bar_chart for the number_of_registration vs district

here, for this task we need to import new dataset of district_zip.csv
for that first create new table schema

```sql
    create table districts(
        pincode varchar(50),
        district varchar(50)
    )
```

now, our data is created we need to import the .csv data as we did earlier for
the Maharashtra.csv data

```sql
    \copy districts from '~/district_zip.csv' delimiter ',' csv header;
```

select query to check if data is loaded properly

```sql
    select * from districts
```

barchart for the year 2015 between number_of_companies vs districts

to solve this, we need to create helper table with the below query so that
we get the releavant data according to the pincode.

```sql
    create table helper_2015 as (
        select date_of_registration,
            case when(substring(registered_office_address,length(registered_office_address)-5,length(registered_office_address)) >= '000000') then
                    substring(registered_office_address,length(registered_office_address)-5,length(registered_office_address))
            end
            as mass
            from dataset
        where extract(year from date_of_registration::date) = '2015'
    )
```
we created helper table named as helper_2015 now we can do inner join that table
with the districts table and get the count of the districts and store that output
in different table

```sql
    create table barplot_for_year_2015 as (
        select distinct district as years, count(district) from helper_2015
        inner join districts on mass = pincode
        group by district
        order by district
    )
```

plot grouped barplot by aggregating registration counts over year of registration and
principal business activity.

```sql
    create table grouped_barplot as (
        select distinct years, count(years) as total, count(manufacture) as manufacture,
                count(construction) as construction, count(other) as other
        from (
            select extract(year from date_of_registration::date) as years,
                    principal_business_activity_as_per_cin as activity,
                    case when(substring(principal_business_activity_as_per_cin,1,11) = 'Manufacture') then
                            'Manufacture'
                    end
                    as manufacture,
                    case when(substring(principal_business_activity_as_per_cin,1,12) = 'Construction') then
                            'Construction'
                    end
                    as construction,
                    case when(substring(principal_business_activity_as_per_cin,1,12) != 'Construction' and
                            substring(principal_business_activity_as_per_cin,1,11) != 'Manufacture') then
                                    'other'
                    end
                    as other
                    from dataset
            ) as dd
        group by years
        order by years
    )
```

for above task I used subqueries in which i manipulated the string and looked for manufacture,
construction in the principal activity then I extracted those and create seprated colum for them
rest I altered with other. using this query I passed on parent query in which I extracted the distinct
years and the count of years as total and all the other counts, group by with years
which gave me the desired output so I created table using that query.

<br>

## Author

- [@Vivek Dubey]()

  
