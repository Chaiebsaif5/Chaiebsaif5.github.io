## List of metrics calculated with definition:
# 1. Market Share breakdown:

As explained in the cleaning phase of the data, Each individual single charging station has a manager, an operator, the commercial network name (who's logo is displayed in the station's banner) and finally a Data owner, in many cases, a single company or private individual can occupy and play several of these roles at once.

Anothor consideration is that the share of the market calculated differs significantly based on whether we're caluclating from a station ownership point of vue or from the number of Charging points offered. For the latter, it may not make sense to calculate the market share for individual operators or station managors if the result is too granular.

    - Ownership breakdown (Top 10):

     A.1.From Station ownership PoV:

     - By commerical network.
     - By manager.
     - By operator.
     - By data owner.

     A.2. From Number of charging points POV:

     - By commerical network.
     - By manager.
     - By operator.
     - By data owner.

# 2. Station Density per 100 km² and per 100 000 Inhabitants
Why? Charging‑point counts are useful, but raw numbers don’t account for geographic or population differences.

How? Join your consolidated_commune or department codes with shapefile/polygons to compute area, and with INSEE population data. Then calculate:

Stations per 100 km²

Stations (or charging points) per 100 000 inhabitants

Insight: Highlights both “charging deserts” in sparsely populated rural areas and potential urban under‑provision.

# 3. Temporal Growth Trends
Why? You have created_at (or proxy visit date) for each station. Examining installation dates over time shows market momentum.

How? Aggregate installations by month or quarter; plot cumulative total.

Insight: Identify periods of rapid expansion (e.g., policy changes, subsidy roll‑outs) or slowdowns.

# 4. Price Distribution & Outliers
Why? You’ve parsed up to three price points (Price_1, Price_2, Price_3). Looking at the distribution—mean, median, IQR—helps you understand affordability and identify premium or “budget” stations.

How? Calculate descriptive stats per Tarif_Type or per region. Flag outliers (z‑score > 3).

Insight: Spot regions or operators charging significantly above/below market.

# 5. Availability & Accessibility Index
Why? Not all stations are equally usable—some require booking (reservation), have limited hours (Working hours standardised), or aren’t PMR‑accessible (accessibilite_pmr).

How? Define a simple score (e.g., +1 point if 24/7, +1 if reservation = False, +1 if PMR access = True). Sum into an “Accessibility Score” 0–3.

Insight: Map or rank stations by accessibility to highlight user‑friendly vs restrictive locations.

# 6. Average Points per Station & Skew
Why? Some “stations” are just a single point, others have dozens of sockets. Understanding this mix can shape infrastructure planning.

How? For each station ID, average its nbre_pdc, and examine the distribution (mean, median, skewness).

Insight: Tell whether the network leans toward many small stations or fewer large hubs.

# 7. Operator Diversification Index
Why? How many different operators does each manager or network partner with? A high “diversification” suggests open‑network approaches; low suggests vertical integration.

How? Group by nom_amenageur and count distinct nom_operateur values; and vice versa.

Insight: Strategic alliances, franchising models, or monopolistic tendencies.

# 8. Free‐to‑Paid Transition Analysis
Why? Some regions or networks may be shifting from free to paid models (or vice versa).

How? Compare the proportion of free stations installed before vs after a key policy date (e.g., national subsidy change).

Insight: Impact of regulations on user cost.

# 9. Plug‑Type Diversity Score per Region
Why? Regions with many plug types may better serve a wider variety of EVs.

How? For each region, count how many of the five plug‑type columns are non‑null, then normalize (0–5).

Insight: Target regions lacking certain connectors.

# 10. Nearest‑Neighbor Distance Analysis
Why? Beyond counts, the physical spacing between stations affects the “coverage” experience.

How? Use lat/long to compute each station’s distance to its nearest neighbor (Haversine). Summarize by mean/median per region.

Insight: Detect clusters vs gaps even within well‑served departments.

11. Correlation Matrix of Key Variables
Why? See which factors most strongly co‑vary—e.g., does higher average price correlate with larger station size? Or does accessibility score correlate with network size?

How? Compute Pearson or Spearman correlations among numeric fields (nbre_pdc, Price_1, Accessibility Score, station density, etc.).

Insight: Data‑driven hints at causal relationships worth exploring.