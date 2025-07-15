1. Downloaded and converted the original CSV data into Excel and SQL format.
2. Cleaned the data:

	a. Formatting validation, checking for errors, missing entries, correct data types and so on.

The data types in this dataset were not too complicated luckily, mainly text strings and numbers (Real) and a few date entries, only one column contained a "custom" data format, column av "created_at" which contains both a date + time.

However, it quickly became apparent that there was barely any standardisation in key columns entries, such as Column AD "Tarification" which captures the charging points costs or the "horaires" column AG which displays the availability time for each charging point. There were also a lot of missing entries for random columns such as plug types (columns U to Y).

I made the decision that all rows missing data in key columns (such as IDs, Costs, Time of availability) will be deleted from the dataset. The columns I deemed critical for this project are the following:
	- Column A and D: Manager name and Operator name, considered missing if both are empty. -- Removed
	- Column P: number of charging points. -- No missing data
	- Column R: Local charging point ID. -- Removed
     Instead of directly removing the empty cells, I checked other ID columns first in case they had data in them that can be copied over to here, as it's quite clear that ids match across all 3 columns, the rows will be removed only if there are no ID entries in any of the columns. 
	- Column T: charging power.
	- Columns U through Y: Plug Types, only if data is missing from ALL at once.
	- Column Z and Column AD: Column Z displays if it's true or false that the station is Free (cost-wise). Column AD displays the actual monetary cost.
	- Columns AE to AG, Columns AJ and AM: All vital to measure "accessibility" in terms of availability in time and the need for reservation before hand ect. as well as the date the station first became open for usage.
	- Column AT: who owns the data for that particular charging point.


Good data governance practices suggest that instead of actually deleting the rows with missing data that they should be moved into a separate tab, this is particularly useful to give feedback to the original dataset creators but also to perform studies on edgecases in case we wanted to unearth some insights into "why" these rows are not properly filled in.

The Critical columns need to have their entries as standardised as possible as well, having reviewed them, I identified that columns AD,AG,... all needed extra work to make sure their entries all followed the same format. This is a complicated task, hence, to achieve this I decided to deploy the following advanced tools: 


	b. Checking for duplicates: 

This dataset contains 2 columns of IDs, one for each local station (overall) located in column J and one for each individual charging point located in column R. Although one would expect stations often hosts multiple charging points and hence duplicate IDs should be expected, this is not the case with this particular dataset as the authors made sure that even for the same station unique IDs are given corresponding to the charging point IDs, however, it's worth checking for duplicates anyways as a good practice. 

To do so, first I sorted the CP (charging point) ID column for A to Z, I then created a column next to it titled "Duplicate Check" (column s) with a simple formula of (if(R1=R2,1,0)), matching IDs will be marked as 1 while unique ones will receive a value of 0. I'm happy to report that this dataset contained purely unique IDs without any duplicates, I then did the same for the Unique station IDs in column J, the check was done in the added column H called station duplicates check.