## Overview of the current landscape of EV recharging stations in France, and what can we learn from it?

This projects starts off from the large consolidated open dataset capturing all registered EV (electric vehicules) charging stations in France, the dataset provides us with very useful starting points such as the name of the companies responsable for each charging point, whether it's free to use or paied, where in France it's located and what types of plugs does it include or doesn't include.

Starting off from these ground observations we will try to glean insights into:
- the EV charging stations market in france.
- the distribution of EV cars through the proxy of EV charging station availability across the french territories. (assuming a logical correlation, which we will contrast with the ownership of EV in france if the data is available to see which regions are overserved, wellserved or underserved in terms of charging stations availability).
- what Electric cars models are catered to the most / least (based on what plug types are most/least available at the charging stations).

==> From these findings we will try to paint a picture regarding what Electric vehicules a french consumer would be better suited to buy / avoid given this particular metric.
==> Which regions should the french government and companies work towards increasing the availability of charging stations, and what plug types should be catered to more.
==> How does the market share breakdown looks like for the top 5/10 companies, and what metrics are driving this share distribution.
==> How will the EV charging station landscape evolve in the next few years?

These metrics are oriented towards business uses, whether the company is already estabilished in the French EV charging market or are exploring it, the French government but also of interest to private EV-owners in France, though the latter 2 are not the main stakeholders targeted in this project.


To answer all these questions, I will be calculating the following set of metrics:

1. Breakdown of the EV charging point ownership (Market share).
2. Number of charging points per Region and % from overall total charging points in France + rank.
3. % of inclusion of plug types and rank from most available to least available.
4. % of free socket types compared to overall, % of paid socket types compared to overall, and testing if there's a correlation / impact on companies market share.
5. (if data is available) how good of a proxy is charging points distribution for EV ownership.
6. Quick overview of what plugs serve which EV models and contrast plug availability to recent EV ownership / sales data in France to calculate best served models and least served models.


Finally: Display everything in a PowerBI dashboard that provides a quick and easy way to absorb the findings and drill into the data / metrics.

The Project Log.md file in this project's folder will display all the steps (chronologically ordered) from the start of the project to its end. Each step is accompanied with details of it's process and the logic behind it.

Dataset schema is available here (original source): https://schema.data.gouv.fr/etalab/schema-irve-statique/latest/documentation.html
and also as PDF in the folder, it covers the definition of each column as well as the data type it contains.

Note:
    - I will be doing this exercise twice, once via Excel, and again via SQL (showcasing formulas) to demonstrate my technical capacity.
    - I have included a column dictionary that translates french column names into English for non-french speakers, It can be found in the Cleaned Excel file in a seperate tab.
    - IMPORTANT: All columns added by me to the dataset will have their headers coloured yellow, Work was mainly kept to these columns unless otherwise stated in the log.
