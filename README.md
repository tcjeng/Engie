# Engie
## BACKGROUND
The Electric Reliability Council of Texas (ERCOT) is a nonprofit organization responsible for managing the flow of electric power to over 27 million customers across Texas, covering approximately 90% of the state's electric load. As the independent system operator (ISO) for the region, ERCOT oversees one of the largest power grids in the United States, comprising more than 54,100 miles of transmission lines and 1,250 generation units, including Private Use Networks.
ERCOT plays a critical role in ensuring grid reliability through efficient scheduling of power supply, financial settlements for the competitive wholesale bulk-power market, and administration of retail switching for approximately 8 million premises in competitive retail choice areas. Operating under the oversight of the Public Utility Commission of Texas (PUCT) and the Texas Legislature, ERCOT facilitates market participation through registration, training, communication, and user guides.
This project aims to analyze the performance of Battery Energy Storage Systems (BESS) within the Day Ahead Market (DAM) and Real-Time Market (SCED) operated by ERCOT. The goal is to identify optimization strategies and assess the capacity utilization of BESS resources by examining market participation and performance trends. Also, based on client’s interest, we are going to explore opportunities for Energy market participation for BESS in order to increase revenue.

## DATA PREPARATION
The data preparation process involved loading and transforming multiple datasets related to energy offers, generation, and load data. The datasets were read from Parquet files stored in Amazon S3 and locally.
### DAM (Day-Ahead Market) 
The energy data was sourced from ERCOT's DAM (Day-Ahead Market) 60-day reports. 
#### Energy Offered Awarded Data Processing 
The data was preprocessed by first converting the `delivery_date` column to a datetime format and then combining it with the `hour_ending` column to create a `date_time` column representing the specific timestamp for each record. All records were labeled as "ENERGY" under a new column named `as_type`. Resources were categorized as either "BESS" (Battery Energy Storage System) or "NON-BESS" based on patterns detected in the `settlement_point` column. Additional columns related to various ancillary services were added with default values of 0 or null where applicable. To enhance consistency and clarity, relevant columns were renamed, such as changing `settlement_point` to `settlement_point_name` and `qse_name` to `qse`. Finally, the columns were reordered to maintain consistency for further processing.
#### Generation Source Data Processing  
The generation data was loaded from a local Parquet file into AWS SageMaker workspace for Python and concatenated with the processed energy data. To ensure compatibility between the datasets, the `resource_name` column in the energy data was cast to a text format before concatenation. The combined dataset, referred to as `concatenated_dam_energy_gen`, was prepared for further analysis.
#### Load Resource Data Processing  
The load data was also sourced from a Parquet file and underwent several transformations to ensure consistency and readiness for analysis, similar to the generation data. A new column, `ecrs_awarded`, was calculated by combining the `ecrssd_awarded` and `ecrsmd_awarded` columns, followed by the removal of redundant columns. Default values of 0 were assigned to columns related to energy quantity and price, which were not relevant to load data. A `resource_type` column was created to categorize resources as "BESS" or "NON-BESS" based on the `isBESS` indicator, and missing values were filled with 0 where applicable. 
One of the issues with the original Load Data reports from ERCOT is that it does not specify which load resource is linked to a battery storage (BESS), while the load resource names don’t exactly match the gen resource names. This presented a challenge for merging the data to get a holistic view of the charge and discharge cycles. After examining the data, we came up with a solution to develop a custom process for mapping the battery-storage resources from Gen reports with the corresponding Load resources based on the name prefixes.
Additionally, a `qse` column was added by performing a left join with a filtered and unique subset of load resource names from the offer data to enhance data completeness.
#### Load Offer Data Processing  
A separate load offer dataset was processed to extract unique resource names and their associated `qse` values. This information was merged with the load data to ensure completeness.
#### Combining Datasets  
The final dataset, `concatenated_dam_energy_gen_load`, was created by concatenating the processed energy, generation, and load data. The columns were aligned for consistency, and appropriate renaming was applied to ensure compatibility across all datasets. The resulting dataset was saved as a CSV file for further analysis.
The entire process aimed to standardize the various datasets, ensuring compatibility and completeness for subsequent modeling and analysis.

### SCED 
For the data preparation process, multiple steps were undertaken to clean, filter, and merge the SCED generation and load datasets to obtain a comprehensive dataset suitable for analysis. 
First, the SCED generation data files for different months of 2024 were read from Parquet files stored in an S3 bucket. The files were loaded separately for each pair of months, including January-February, March-April, May-June, July-August, September-October, and November-December. The data was filtered to include only rows where the “resource_type” column was labeled as “PWRSTR” to focus on relevant power storage resources. Subsequently, unnecessary columns beginning with “sced2_curve_”, “sced1_curve_”, and “submitted_tpo_” were dropped to simplify the dataset and retain only essential information.
After cleaning, all the individual data frames were concatenated vertically to form a single, consolidated SCED generation dataset. The timestamp column, “sced_time_stamp,” was converted to a datetime format to facilitate further processing. From this timestamp, the “delivery_date” and “hour_ending” columns were derived to align with the SCED load data. Specifically, the “delivery_date” was adjusted for entries corresponding to midnight (00:00:00) by subtracting one day from the original date. Additionally, the “hour_ending” column was corrected to account for timestamp values between 01:00:00 and 23:00:00 by reducing the hour by one.
Similar to the generation dataset, unnecessary columns starting with “sced_bid_to_buy_curve” were removed. The dataset was further filtered to include only rows corresponding to Battery Energy Storage System (BESS) resources by matching the “resource_name” column with a list of BESS load names.
The SCED generation and load datasets were merged on the common columns “delivery_date”, “hour_ending”, and “resource_name” using a full join operation to ensure that all relevant entries were retained. The resulting dataset was processed to replace any missing values with zeros. 

### METHODOLOGY & ANALYSIS
#### Tools:
Amazon S3: Used to store parquet files for DAM and SCED datasets
Amazon SageMaker workspace for Python: Used to work with data preparation, exploration, wrangling
Local Python instance: Used when exploring the smaller csv files and running data simulations.
Tableau: Used for visualizations, analysis, interactive dashboards and final presentation Story
#### Methodology:
Merge DAM datasets and SCED datasets for markets comparison
Merge GEN datasets and LOAD datasets for resource analysis
Map power storage resources between GEN and LOAD datasets for merging purpose (Appendix 2)
#### Analysis:
Comparison between DAM and SCED markets 
Market capacity, revenue and settlement point pricing analysis in day-ahead market
Intra-week and intra-day BESS capacity allocation revenue, pricing and energy market strategy analysis
Simulation of optimal charging and discharging points with SOC constraints(Code attached in Appendix 3)


## RECOMMENDATIONS
Prioritize energy market participation over ancillary services, especially in June & September where Energy award prices are consistently higher than average prices for Ancillary Services
Exploit peak hourly price patterns for Energy to maximize revenues, by allocating BESS capacity to Energy between 5-9pm, Monday through Saturday.

## CONCLUSION
- DAM average revenues were impacted by the increase in average monthly capacity and decrease in the energy prices in 2024
- The energy prices (SPP) in DAM were consistently higher than those of Ancillary services
- Intra-day peak prices for both energy and ancillary services were observed between 5-9pm
- A shift towards energy allocation for top battery QSE is observed during peak energy price hours in 2024.
- A slight increase in energy market allocation would result in a significant uptick in revenue contribution for batteries.
