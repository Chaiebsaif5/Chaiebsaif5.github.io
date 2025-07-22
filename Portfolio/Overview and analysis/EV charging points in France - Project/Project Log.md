## 1. Downloaded and converted the original CSV data into Excel and SQL format.
## 2. Cleaned the data:

# a. Checking for errors, missing entries, correct data types and so on.

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


# b. Checking for duplicates:

This dataset contains 2 columns of IDs, one for each local station (overall) located in column J and one for each individual charging point located in column R. Although one would expect stations often hosts multiple charging points and hence duplicate IDs should be expected, this is not the case with this particular dataset as the authors made sure that even for the same station unique IDs are given corresponding to the charging point IDs, however, it's worth checking for duplicates anyways as a good practice.

To do so, first I sorted the CP (charging point) ID column for A to Z, I then created a column next to it titled "Duplicate Check" (column s) with a simple formula of (if(R1=R2,1,0)), matching IDs will be marked as 1 while unique ones will receive a value of 0. I'm happy to report that this dataset contained purely unique IDs without any duplicates, I then did the same for the Unique station IDs in column J, the check was done in the added column H called station duplicates check.

# C. Standardising data:

The Critical columns identified in a. need to have their data standardised , having reviewed them, I identified that columns AD,AG,... needed extra work to make sure their entries all followed the same format. This is a complicated task, hence, to achieve this I decided to deploy the following advanced tools.

 	- Column R: some points were classified by the original data authors as "non  concerné" translating as "irrelevant", these were removed from the dataset as potential unintentional additions by the data collection agents. -- 54 rows removed.
 	- Column AG: "Horaires" or working hours entries had both 24/7 entries, stations that are open only on certain days on certain hours, and others have only hours mentioned for them without any additional context, as such I decided to standardise this column into 4 categories: "24/7", "Time-limited" for those limited in hours, "Days-limited" for those limited in days but not in hours and "Restricted Schedule" for those limited by both. These results are now housed in column AI.
 	==> To accomplish this task I decided to use some advanced Excel logic to showcase Excel capabilities, here's the formula used:
 
"=LET(
    text, LOWER(AG2),
    is_24_7, OR(ISNUMBER(SEARCH({"24/7","24h","00:00"}, text))),
    has_day, OR(ISNUMBER(SEARCH({"mo","tu","we","th","fr","sa","su"}, text))),
    has_time, OR(ISNUMBER(SEARCH({":","-"}, text))),
    IFS(
        is_24_7, "24/7",
        AND(has_day, has_time), "Limited Hours & Days",
        has_day, "Limited Days Only",
        has_time, "Limited Hours Only",
        TRUE, "Unknown"
    )
)"

 	- Column AD: By far needs the most work due to the variety of cells, firstly, if marked as Free in Column Z, I standardised the monetary value to €0. -- 127 Entries
Secondly, to extract the actual price per Kw/H I have to use PowerQuery due to the complexity of the entries in the column, this was done as follows:

    ==> To robustly parse this data, I deployed a sophisticated Power Query script (below). The process was iterative (5 iterations of the code below) due to how many "edge cases" and unique entry types were present in the column


 		Powerquery Script:
"
let
    // 1. Load your source table. This name should match your actual table name.
    Source = Excel.CurrentWorkbook(){[Name="consolidation_etalab_schema_irve_statique_v_2_3_1_20250712"]}[Content],

    // 2. Add a new 'Tarif_Cleaned' column to safely work with the data.
    // This step handles nulls, lowercasing, standardizing decimal points, and converting centimes.
    Add_Cleaned_Column = Table.AddColumn(Source, "Tarif_Cleaned", each
        let
            // Start with the original 'tarification' field, but handle potential errors/nulls
            rawText = try Text.From([tarification]) otherwise "",
            
            // Basic cleaning: lowercase and trim whitespace
            lowerTrimmed = Text.Lower(Text.Trim(rawText)),
            
            // Standardize decimal separators from comma to dot
            dotFixed = Text.Replace(lowerTrimmed, ",", "."),
            
            // Robustly find and replace "cts" patterns (e.g., "49cts", "36 cts/kwh") with euro values
            // Regex: (\d+(?:\.\d+)?) captures the number, [ ]?cts matches an optional space then "cts"
            centsConverted = Text.RegexReplace(
                dotFixed,
                "(\d+(?:\.\d+)?)[ ]?cts",
                each try Text.From(Number.FromText(_{1}) / 100) otherwise _{0}
            )
        in
            centsConverted
    , type text),

    // 3. Extract all valid numeric parts from the 'Tarif_Cleaned' column into a list
    Add_Numeric_Tokens = Table.AddColumn(Add_Cleaned_Column, "Tokens", each
        let
            txt = [Tarif_Cleaned],
            // Split the text by any character that is NOT a digit or a period.
            // This isolates number-like strings.
            delimiters = "€/\\-– ,;:()abcdefghijklmnopqrstuvwxyz+",
            parts = Text.SplitAny(txt, delimiters),
            
            // Select only the parts that are valid numbers and convert them
            numbers = List.Transform(
                List.Select(parts, each Value.Is(try Number.FromText(_) otherwise null, type number)),
                each Number.FromText(_)
            )
        in
            numbers
    , type list),

    // 4. Create Price_1, Price_2, and Price_3 columns from the tokens list
    Add_Price1 = Table.AddColumn(Add_Numeric_Tokens, "Price_1", each try [Tokens]{0} otherwise null, type number),
    Add_Price2 = Table.AddColumn(Add_Price1, "Price_2", each try [Tokens]{1} otherwise null, type number),
    Add_Price3 = Table.AddColumn(Add_Price2, "Price_3", each try [Tokens]{2} otherwise null, type number),

    // 5. Add 'Has_Augmentation' column by checking for a "+" symbol
    Add_Augmentation_Flag = Table.AddColumn(Add_Price3, "Has_Augmentation", each Text.Contains([Tarif_Cleaned], "+"), type logical),

    // 6. Add 'Tarif_Type' classification based on the number of prices and the augmentation flag
    Add_Tarif_Type = Table.AddColumn(Add_Augmentation_Flag, "Tarif_Type", each
        let
            priceCount = List.Count([Tokens]),
            hasPlus = [Has_Augmentation]
        in
            if priceCount = 0 then "Unclassified"
            else if hasPlus then "Tiered with augmentation"
            else if priceCount = 1 then "Simple fixed cost"
            else "Time-based pricing" 
    , type text),

    // 7. Add 'Basis of cost calculation' column by checking for keywords
    Add_Cost_Basis = Table.AddColumn(Add_Tarif_Type, "Basis of cost calculation", each
        let
            txt = [Tarif_Cleaned]
        in
            if Text.Contains(txt, "kwh") then "cost calculated per kwh consumed"
            else if Text.Contains(txt, "min") then "cost calculated per minute of usage"
            else if Text.Contains(txt, "heure") or Text.Contains(txt, "/h") then "cost calculated per hour of usage"
            else if List.Count([Tokens]) > 0 then "Unclassified unit"
            else null
    , type text),

    // 8. Add 'Occupation_charge' flag by checking for the keyword "occupation"
    Add_Occupation_Charge = Table.AddColumn(Add_Cost_Basis, "Occupation_charge", each
        if Text.Contains([Tarif_Cleaned], "occupation") then "Occupation charge detected" else null
    , type text),

    // 9. Final cleanup: remove the intermediate columns we don't need in the final output
    Final_Table = Table.RemoveColumns(Add_Occupation_Charge, {"Tarif_Cleaned", "Tokens"})

in
    Final_Table
"

This script first isolates up to three distinct price points from each entry. It also detects whether a price is an add-on charge by checking for a "+" symbol. Based on this, it classifies each charging point's pricing model into one of three categories:

        - "Simple fixed cost": For entries with only one price.
        - "Time-based pricing": For entries with multiple, distinct prices that vary based on conditions like time of day.
        - "Tiered with augmentation": For prices structured as a base fee plus an add-on (e.g., "base price € + add-on €").

        These new classifications are necessary for visualising the true cost landscape across the french departments. The results of this process are now located in columns BE through BH.

The script also creates two additional classification columns:
-column BI: Determines if the cost is calculated per minute/hour of use or per kWh consumed. 

-column BJ: specifically flags any entry that mentions an "occupation" charge—a fee for occupying the charging bay beyond the actual charging session.

- Column A, Column D and column G: Standardised company names to avoid splitting the data across multiple name varieties, such as: 
"TotalEnergies Marketing France" and "TotalEnergies Charging Services".
( This step was completed in the "Inclusive scenario dataset as well, more on this below).

all names changed are saved in a sperate tab called "Name Standardisation" in their respective tables.

    - Standardised 209 names in Column A.
    - Standardised 27 names in Column D.
    - Standardised 27 names in Column G.

## 3. Defining and calculating the metrics:

The first step in our exploratory data analysis is to define a set of metrics that encapsulates key information communicated by our dataset. These metrics are oriented towards business uses, whether they be already estabilished in the French EV market or are exploring it, the French government but also of interest to private EV-owners in France, though the latter 2 are not the main stakeholders targeted in this project.


    Note: All these metrics will be calculated twice, once under a "strict" scenario using our cleaned and vetted dataset (that has all the critical columns filled out and is standardised) and again under an "inclusive" scenario that just uses the overall dataset as is from the source with limited standardisation (such as making sure a company mentions are all unified under one name rather than x varieties).

    ==> The use of the 2 scenarios is to highlight the best scope for each metric, some metrics are less affected by the missing information in the columns than others. For example, Market Share Breakdown results will be valid even in the "Inclusive" scenario and they will be highlighted in the Dashboard.

# a. Market Share breakdown:

As explained in the cleaning phase of the data, Each individual single charging station has a manager, an operator, the commercial network name (who's logo is displayed in the station's banner) and finally a Data owner, in many cases, a single company or private individual can occupy and play several of these roles at once.

Anothor consideration is that the share of the market calculated differs significantly based on whether we're caluclating from a station ownership point of vue or from the number of Charging points offered. For the latter, it may not make sense to calculate the market share for individual operators or station managors if the result is too granular.

    - Ownership breakdown (Top 10):

     A.1.From Station ownership PoV:

     - By commerical network. 
     - By manager. -- Calculated
     - By operator. -- Calculated
     - By data owner.

     A.2. From Number of charging points POV:

     - By commerical network.
     - By manager.
     - By operator.
     - By data owner.

==> I did a similar calculation in the overall (not cleaned) dataset to review if the subset results match the overall or not, the intention of this comparison is to highlight priority targets to backfill the missing data for.