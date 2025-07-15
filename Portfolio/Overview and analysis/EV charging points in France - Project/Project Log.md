1. Downloaded and converted the original CSV data into Excel and SQL format.
2. Cleaned the data:

	a. Checking for errors, missing entries, correct data types and so on.

The data types in this dataset were not too complicated luckily, mainly text strings and numbers (Real) and a few date entries, only one column contained a "custom" data format, column av "created_at" which contains both a date + time.

However, it quickly became apparent that there was barely any standardisation in key columns entries, such as Column AD "Tarification" which captures the charging points costs or the "horaires" column AG which displays the availability time for each charging point. There were also a lot of missing entries for random columns such as plug types (columns U to Y).

I made the decision that all rows missing data in key columns (such as IDs, Costs, Time of availability) will be deleted from the dataset. The columns I deemed critical for this project are the following:
	- Column A and D: Manager name and Operator name, considered missing if both are empty. -- 348 rows removed
	- Column P: number of charging points. -- No missing data
	- Column R: Local charging point ID. -- 48934 rows removed
	Instead of directly removing the empty cells, I checked the other ID column "I" first in case it had data in them that can be copied over to here, thereby giving the Charging point the same ID at it's local station, it proved to be possible for some entries but not all, also, stations with an ID of 0 were also considered as empty and removed.
	- Column T: charging power. -- No missing data.
	- Columns U through Y: Plug Types, only if data is missing from ALL at once. -- 1282 rows removed
	- Column Z and Column AD: Column Z displays if it's true or false that the station is Free (cost-wise). Column AD displays the actual monetary cost. Removed entries only if both are empty, backfilled the Boolean column if actual cost data is present in column AD. -- 3980 rows removed

	==> When attempting to check items with no exact cost but marked as Not Free (False) on column Z, I found that it affected a massive chunk of the data (71382 rows), removing this large amount of rows, in addition t the previously removed observations on column R may inaccurately skew geographical analysis on the distribution of the charging points across the French territory, as such, I concluded that I will be running 2geographical analyses on this project, one of the perfectly clean data, and another with the inclusion of the imperfect rows that still include geographical information, I will compare both to see if I can spot a location related cause for this data imperfection. -- 71382 Rows removed

	- Columns AE to AG, Columns AJ and AM: All vital to measure "accessibility" in terms of availability in time and the need for reservation before hand ect. as well as the date the station came online. 	- No missing data in columns AJ to AH.
			- For the date of coming online, a lot of rows are missing this observation (Column AM). However, we do have access to the date the agents visited the point and provided the measurements of this dataset, It is logical to therefore assume that the station was in operation at that date or before and hence it could be used as a good proxy (Column AO). It was also observed at this point that a few dates entered are erroneous (1930) and these rows were removed -- 2 Rows
			
	- Column AT: who owns the data for that particular charging point. This column is vitale for Ethical reasons. -- No missing rows


Good data governance practices suggest that instead of actually deleting the rows with missing data that they should be moved into a separate tab, this is particularly useful to give feedback to the original dataset creators but also to perform studies on edgecases in case we wanted to unearth some insights into "why" these rows are not properly filled in.


	- For Column A and D: if the Manager name is empty yet the Operator name is present, The operator will be considered as the manager for the purposes of our study, hence I backfilled the empty column A cells from column D.


	b. Checking for duplicates: 

This dataset contains 2 columns of IDs, one for each local station (overall) located in column J and one for each individual charging point located in column R. Although one would expect stations often hosts multiple charging points and hence duplicate IDs should be expected, this is not the case with this particular dataset as the authors made sure that even for the same station unique IDs are given corresponding to the charging point IDs, however, it's worth checking for duplicates anyways as a good practice. 

To do so, first I sorted the CP (charging point) ID column for A to Z, I then created a column next to it titled "Duplicate Check" (column s) with a simple formula of (if(R1=R2,1,0)), matching IDs will be marked as 1 while unique ones will receive a value of 0. I'm happy to report that this dataset contained purely unique IDs without any duplicates, I then did the same for the Unique station IDs in column J, the check was done in the added column H called station duplicates check.

	C. Standardising data:

The Critical columns identified in a. need to have their data standardised , having reviewed them, I identified that columns AD,AG,... needed extra work to make sure their entries all followed the same format. This is a complicated task, hence, to achieve this I decided to deploy the following advanced tools.

	- Column AD: if marked as Free in Column Z, I standardised the monetary value to €0. -- 127 Entries