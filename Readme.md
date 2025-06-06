
# Power BI Stock and Portfolio Report

---

## Table of Contents

- [Executive Summary](#executive-summary)
- [ERD Diagram](#erd-diagram)
- [Fact Activity Table](#fact-activity-table)
- [Dimension Date Table](#dimension-date-table)
- [Dimension Exchange Table](#dimension-exchange-table)
- [Model View](#model-view)
- [Connected Model](#connected-model)
- [Dax Measures](#dax-measures)
- [Page 1](#page-1)
- [Page 2](#page-2)

---

## Executive Summary

This report explains a stock market dashboard developed to analyze the NYSE and NASDAQ markets, as well as a portfolio built from those markets.


This project transforms raw market data into a structured model, creates DAX measures, and ultimately builds report visuals.

Page 1 provide insights into the exchanges in the dataset while page 2 provides information for the investment portfolio and individual securities (open and close price of the day), visual information about the securities(stocks), company name, industry, location, info about the exchanges, date dimension .

## ERD Diagram

![ERD_Diagram](images/ERD_Diagram.png)

Notice how the Fact table has 5 basic measures

### Creating Queries 

This section shows how 4 queries were created in order to bring to power Bi the factActivity table, and the dimensional tables called dimDate, dimExchange and dimSecurity.

## Fact Activity Table 

The factActivity table was created by bringing the complete directory called  fact activity which contains 7 csv files

### Here the steps to prepare this table 

<details><summary>Click to expand </summary>

### Entering Power Query editor

![Power_Query](images/PowerQueryEditor.png)

After entering the Power Query editor select the new sources tool from the ribbon and select the path of the folder that contains the multiple csv files

![folder_source](images/Folder_Source.png)

![folder](images/fact_activity_folder.png)

![folder_path](images/fact_activity_folder_path.png)

Finally, the files can be combined

![combining_files](images/transform_data.png)

After selecting the files, click 'Combine' and then 'OK'

![combine_confirmation](images/combine_confirmation.png)


After that, we can see that in addition to the factActivity query, helper queries have been created, within this we find parameters , functions and a Transform Sample File query, which helps for pre-processing individual files( transformation steps like promoting headers), also meant to reuse transformation logic(function) so power query applies the same steps on every file in the directory.

![main_and_helper_queries](images/main_and_helper_queries.png)

All transformations are now done in the "Transform Sample File" query, as these will be applied to each individual file in the folder

the first step is to detect the data type of the data set

![detect_data_type](images/detect_data_type.png)

now the data types have been changed effectively

![data_types_changed](images/data_types_changed.png)

Now we need to reshape the data in the Measure column. The objective is to fill each category of this column with their corresponding value from the "Value Column", so effectively grouping by and aggregating by Open, High, Low, Close , Adjclose. To do this just select the Pivot Column option on the Transform tab and select Value as the Values Column

![pivot_column](images/pivot_column.png)

![pivot_result](images/result_pivot.png)

## Creating Price Range and Price Change

Price Range = Total Price Movement for the entire day 

This can be considered a measure of volatility

Select the High and Low columns, then click the Standard button and choose Subtract.

![subtract](images/subtract.png)

after that, the column is renamed to Price Range

![renaming_subtract_column](images/PricetRange.png)

do the same for the Open and Close to get the Price Change, which tells us if the stock is going upwards or downwards (bullish or bearish)

![price_change](images/PriceChange.png)

Finally just remove the Open, High, Low and AdjClose 

![removing_columns](images/Removing_Columns.png)

Once all of this have been done on the Transform Sample File query , then just go to the main qury called FactActivity and remove the "Change Type" step

![remove_change_type](images/remove_chage_type_step.png)

after that just remove the Source_Name column, which is not needed for the report

![remove_source_name](images/remove_Source_Name.png)

</details>
---

## Dimension Date Table 

<details><summary>Click to expand </summary>

Now let's address the Dim Date table

In this case the Date table comes from a text file

![dim_date](images/dimDate_text.png)

Then simply promote the firt row as header

![promote_first_row](images/promote_first_row.png)

by doing this notice hot the step Change Type is auto generated. As a good practice , select the data type as Locale, so no matter where one can be in the world, this file will be interpret with the date and time of in this case Canada

![locale](images/locale.png)

Now we are just adding Year Column, Name of Month, Month and Day

![date_columns](images/adding_date_columns.png)

![added_dates_colums](images/added_dates_colums.png)

</details>
---


## Dimension Exchange Table


This table is contained in an excel file

<details><summary>Click to expand </summary>

![dim_exchange](images/dim_exchange.png)

then just connect to sheet one and click ok

![sheet1_dim_exchange](images/sheet1_dim_exchange.png)

change the name of the query to Dim exchange. Notice how the Promoted header and Change Type steps were done automatically

![rename_dimExchange](images/rename_to_dimExchange.png)

Notice that since the real headers are in row 6 we need to first of all remove the automatic steps of Promoted headers and change type, then the row that contains Exchange_ID and the other fields in that row will go to row seven. So after that just simply remove the first 6 rows, then use the promote first row to header option

![removing_top_rows_dim_exhcange](images/removing_top_rows_dim_exchange.png)

Now we have the correct headers

![correct_headers](images/correct_headers.png)

since the Exchange_Id is expected to be numeric, just force the column to be of type whole number, this will throw error for the non numeric rows of this column, so just simply select the column, and use remove errors

![whole_number](images/forcing_whole_number.png)

![remove_errors](images/remove_errors.png)

To remove the null values in the second row of Type, Location and Currency, simply select them and fill down , since the type, location and currency are the same

![fill_down](images/fill_down.png)

</details>
---

### Dimension Security Table 

The Dimennsion Security table come from an Excel file, similar as the Dimension Exchange table, we select again the sheet1 and this is the result, finally the query was renamed to dimSecurity

<details><summary>Click to expand </summary>

![renaming_dim_security](images/renaming_dim_security.png)


A primary key is needed to be added here since this table does not have one, so an index colum from one is added

![adding_index_column](images/adding_index_column.png)

then this column is renamed to SecurityID

![security_id](images/Security_ID_column.png)

now we need to create a FK. The NASDAQ has a PK of 1 and NYSW of 2 (by looking at the dimExchange table)

So let us create a column using a condition

![conditional_column](images/conditional_column.png)

then the data type of this new column is changed to a whole number. So this new column ( Exchange_ID) is the one used to relate the DimSecurity table witht the dimExchange table

DateAdded , Exchange and Description columns are removed form dimSecurity

![removing_columns_dimSecurity](images/removing_columns_dimSecurity.png

)

### Cleaning the Headquarters column

The objective is to have a separate column for address, city, state, zip code and country

since some of the addresses contain commas , using "," as the delimiter will not work as expected, therefore the splitting will not be consistent.  

So first the split is set to the Right-most delimeter

![right_most_delimeter](images/right_most_delimeter.png)

delete the Change type step and then split the new column that now contain country and zip code, using a delimeter by space, using the right most delimeter again

![country_and_zip_code](images/country_and_zipcod_splitting.png)

and delete the Changed Type step again, since at the end all the data type colums will be set

Then the new columns headquarters2.1 and headquartes2.2 are renamed to Zip Code and Country respectively

the same process is perform to extract the state and after the city and finally the only piece of information left is the address.

The result is the following:

![address_splitting](images/address_splitting_result.png)

finally all these columns get trimmed since they were been split by delimeter and some may have extra spaces

![trim](images/trim.png)

## Merging the Portfolio Weighting 

Since the Portfolio Weighting comes from another file, it has to be merged to the dimSecurity table

![portfolio_weighting](images/portfolio_weighting.png)

![portfolio_weighting_query](images/portfolio_weighting_query.png)

Both dimSecurity and Portfolio Weighting share the Security_ID column, so they can be merged together

Since the Portfolio Weighting is not required in the model disable the load option(notice that the query now is in italics)

![disable_load](images/diable_load.png)

Finally the merge can be done

![merge_queries](images/merge_queries.png)

Then just the Weighting column is expanded

![weighting_expanded](images/weighting_expanded.png)

![expansion_result](images/expansion_result.png)

A final thing to do is to move the Exchange_ID and the Security_ID to the left of the table 

![rearranging](images/rearranging.png)

finally just click close and apply, and this will load the queries into the data model

![loading](images/loading.png)

</details>

## Model View 

![model_view](images/model_view.png)

So here we connect Date with Date from dimDate to FactActivity,
Security_id from FactActivity to Security ID of dimSecurity. And finally Exchange_ID of dimSecurity with Exchange_ID of dimEchange 


## Connected Model

![model](images/connected_model.png)

## Dax Measures

- Weighting = RELATED(dimSecurity[Weighting])
- Average Daily Volume = AVERAGEX(dimDate, [Total Volume])
- Average Price Change = AVERAGE(factActivity[PriceChange])
- Average Price Range = AVERAGE(factActivity[PriceRange])
- Count Securities = COUNT(dimSecurity[Security_ID])
- Current Close Price = CALCULATE([Total Close Price], - LASTNONBLANK(dimDate[Date], [Total Close Price]))
- Current Portfolio Value = [Current Close Price] * [Total Weighting]
- Max Daily Volume = MAXX(dimDate, [Total Volume])
- Min Daily Volume = MINX(dimDate, [Total Volume])
- Total Close Price = SUM(factActivity[Close])
- Total Portfolio Value = SUM(factActivity[Portfolio Value])
- Total Volume = SUM(factActivity[Volume])
- Total Weighting = SUM( dimSecurity[Weighting])
- YTD Avg Daily Volume = CALCULATE([Average Daily Volume], DATESYTD(dimDate[Date]))
- YTD Max Daily Volume = CALCULATE([Max Daily Volume], DATESYTD(dimDate[Date]))
- YTD Min Daily Volume = CALCULATE([Min Daily Volume], DATESYTD(dimDate[Date]))

![dax_measures_model_view](images/dax_measures_view_model.png)


## Page 1 

![dashboard1](images/dashboard1.png)

## Page 2
![dashboard2](images/dashboard2.png)