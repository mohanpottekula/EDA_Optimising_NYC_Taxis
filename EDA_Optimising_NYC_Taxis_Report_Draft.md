# Exploratory Data Analysis: Optimising NYC Taxi Operations

Name: `<Your Name>`

Assignment ID: `EDA/02`

## 1. Problem Statement

This analysis uses 2023 NYC yellow taxi trip data to identify patterns in demand, revenue, pricing, route efficiency, and customer behavior. The business objective is to use these patterns to improve taxi availability, route planning, pricing decisions, and overall customer experience.

## 2. Methodology

The raw dataset contained 12 monthly parquet files for 2023, along with the NYC taxi zone shapefile. Since the full dataset is very large, a representative sample was created to make the analysis computationally practical while preserving time-based demand patterns.

### 2.1 Sampling Approach

- Sampling was performed by pickup date and pickup hour.
- A `0.8%` sample was drawn from each date-hour group.
- This preserved hourly and monthly demand variation better than taking one random sample from the full year.

### 2.2 Sample Size

Monthly sample sizes ranged from about `22,727` to `28,262` rows. The combined sampled dataset contained `308,007` rows before cleaning and `304,630` rows after cleaning and outlier treatment.

This final size is appropriate for EDA and remains close to the target range mentioned in the assignment.

## 3. Data Cleaning

### 3.1 Structural Fixes

- Duplicate airport fee columns, `airport_fee` and `Airport_fee`, were merged into one standardized column.
- Unnecessary index-style columns were removed.
- Column names were standardized to lowercase for consistency.

### 3.2 Missing Values

The main missing values were found in:

- `passenger_count`: `3.39%`
- `RatecodeID`: `3.39%`
- `store_and_fwd_flag`: `3.39%`
- `congestion_surcharge`: `3.39%`

Handling decisions:

- Missing and zero `passenger_count` values were replaced with the mode, `1`.
- Missing `RatecodeID` values were replaced with the mode, `1`.
- Missing `congestion_surcharge` values were filled with `0`.
- Missing `store_and_fwd_flag` values were filled with `N`.

These decisions were reasonable because the missing proportions were small and the replacements matched the most common operational case in the data.

### 3.3 Invalid Values

No negative values were found in `fare_amount`, but a very small number of negative values appeared in columns such as `mta_tax`, `improvement_surcharge`, `congestion_surcharge`, `total_amount`, and `extra`. These were treated as invalid records and set to `0`.

### 3.4 Outliers

Some values were clearly unrealistic before treatment. For example:

- Maximum `trip_distance`: `22,528.82`
- Maximum `fare_amount`: `143,163.45`
- Maximum `total_amount`: `143,167.45`

These values are not realistic for standard taxi trips and likely came from reporting or system errors.

Outlier treatment:

- Passenger counts above `6` were removed.
- Trips shorter than `1` minute or longer than `240` minutes were removed.
- `trip_distance`, `fare_amount`, `tip_amount`, and `total_amount` were capped at the `99.5th` percentile.

After treatment:

- Maximum `trip_distance`: `21.82`
- Maximum `fare_amount`: `89.80`
- Maximum `total_amount`: `114.19`
- Maximum `trip_duration_min`: `75.25`

This made the dataset much more realistic for business analysis without discarding normal trip behavior.

## 4. General EDA

### 4.1 Variable Types

Numerical variables included:

- `passenger_count`
- `trip_distance`
- `fare_amount`
- `tip_amount`
- `total_amount`
- `trip_duration_min`

Categorical variables included:

- `vendorid`
- `ratecodeid`
- `store_and_fwd_flag`
- `payment_type`
- `pulocationid`
- `dolocationid`
- `pickup_day_name`
- `pickup_month_name`

Datetime variables:

- `tpep_pickup_datetime`
- `tpep_dropoff_datetime`

### 4.2 Temporal Patterns

Taxi demand is concentrated in the late afternoon and early evening.

Key findings:

- Busiest hour: `18:00`
- Busiest day: `Thursday`
- Busiest month: `October`

Estimated actual trip counts for the five busiest hours:

- `18:00`: `2,678,625`
- `17:00`: `2,554,750`
- `19:00`: `2,395,375`
- `15:00`: `2,345,500`
- `16:00`: `2,344,250`

Interpretation:

- The strongest demand window is the late-afternoon to evening period.
- Supply should be ramped up before the evening peak rather than after it starts.
- Thursday appears to be the strongest weekday for taxi activity, likely reflecting both work and leisure travel.

### 4.3 Revenue Trends

Monthly sampled revenue was highest in:

- `May`: `809,656.92`
- `October`: `806,265.72`

It was lowest in:

- `February`: `626,706.67`
- `August`: `634,866.36`

Quarterly revenue share:

- Q1: `23.75%`
- Q2: `26.82%`
- Q3: `22.62%`
- Q4: `26.81%`

Interpretation:

- Revenue is strongest in Q2 and Q4.
- This suggests that both demand and monetization vary through the year and should be considered in planning and promotional strategy.

### 4.4 Fare, Distance, and Duration

The strongest numerical relationships in the dataset were:

- Correlation between `trip_distance` and `fare_amount`: `0.961`
- Correlation between `trip_duration_min` and `fare_amount`: `0.873`

Interpretation:

- Fare is strongly driven by distance.
- Trip duration also matters, which reflects the effect of congestion and slower travel conditions.

Average fare by passenger count:

- 1 passenger: `18.93`
- 2 passengers: `21.82`
- 3 passengers: `21.54`
- 4 passengers: `22.13`
- 5 passengers: `18.71`
- 6 passengers: `18.95`

Fare does not increase in a strongly linear way with passenger count, which is expected because yellow taxi pricing is trip-based rather than passenger-based.

### 4.5 Payment and Tipping

Payment type distribution:

- Credit card: `240,777`
- Cash: `50,521`
- Unknown: `10,340`
- Dispute: `1,932`
- No charge: `1,060`

Credit card payments dominate the dataset, which is important because recorded tips are mainly available for electronically paid trips.

Average tip amount by distance band:

- `0-1` miles: `2.39`
- `1-2` miles: `3.06`
- `2-5` miles: `4.33`
- `5-10` miles: `7.82`
- `10+` miles: `13.25`

Average tip percentage by distance tier:

- Up to 2 miles: `28.72%`
- 2 to 5 miles: `22.97%`
- 5 to 10 miles: `22.40%`
- 10+ miles: `21.14%`

Interpretation:

- Tip amount rises with distance because fare rises.
- Tip percentage falls as trip length increases, which suggests passengers are less generous proportionally on expensive long trips.

Low-tip and high-tip trip comparison:

- Low-tip trips had a higher average distance: `4.73` vs `2.27`
- Low-tip trips had a higher average fare: `25.24` vs `14.25`
- High-tip trips had a slightly higher credit-card share: `98.70%` vs `96.44%`

This suggests that shorter, lower-fare trips tend to produce better tip percentages.

### 4.6 Geographic Patterns

Top pickup zones:

- `JFK Airport`: `15,417`
- `Upper East Side South`: `14,195`
- `Midtown Center`: `14,070`
- `Upper East Side North`: `12,817`
- `Midtown East`: `10,877`
- `Penn Station/Madison Sq West`: `10,307`
- `LaGuardia Airport`: `10,280`

Top dropoff zones:

- `Upper East Side North`: `13,554`
- `Upper East Side South`: `12,731`
- `Midtown Center`: `11,913`
- `Times Sq/Theatre District`: `9,213`
- `Murray Hill`: `9,046`
- `Midtown East`: `8,675`
- `Lincoln Square East`: `8,640`

Interpretation:

- Manhattan remains the main demand center.
- Airport traffic is also a major source of trips, especially at `JFK Airport` and `LaGuardia Airport`.
- These zones should be the focus of fleet positioning.

## 5. Detailed Insights and Strategies

### 5.1 Operational Efficiency

The slowest high-volume routes were concentrated in busy Manhattan corridors:

- `Penn Station/Madison Sq West -> Times Sq/Theatre District` at `12:00`: `3.05 mph`
- `Times Sq/Theatre District -> Times Sq/Theatre District` at `17:00`: `3.23 mph`
- `Midtown Center -> Times Sq/Theatre District` at `19:00`: `3.30 mph`

Interpretation:

- Dense Midtown corridors are slow but commercially important.
- High demand does not always mean high efficiency.
- These routes need congestion-aware dispatching and realistic ETA expectations.

### 5.2 Weekday and Weekend Demand

Weekday traffic is more sharply concentrated around commute hours, while weekend demand is relatively more spread into leisure-heavy periods.

Interpretation:

- Weekday driver scheduling should target commute peaks.
- Weekend scheduling should remain strong into the evening.

### 5.3 Pickup and Dropoff Imbalance

Highest pickup/dropoff ratios:

- `East Elmhurst`: `9.06`
- `JFK Airport`: `4.65`
- `LaGuardia Airport`: `2.65`
- `Penn Station/Madison Sq West`: `1.52`

Lowest pickup/dropoff ratios:

- `Newark Airport`: `0.00`
- `Windsor Terrace`: `0.03`
- `Whitestone`: `0.03`
- `Greenpoint`: `0.07`

Interpretation:

- Airports behave as strong outbound trip generators.
- Some low-ratio zones are more likely to be destinations than origins.
- Extreme ratio values from low-volume zones should be treated cautiously.

### 5.4 Night Demand

Top night pickup zones:

- `East Village`
- `JFK Airport`
- `West Village`
- `Clinton East`
- `Lower East Side`

Top night dropoff zones:

- `East Village`
- `Clinton East`
- `Murray Hill`
- `East Chelsea`
- `Gramercy`

Revenue share:

- Day: `87.44%`
- Night: `12.56%`

Interpretation:

- Night demand is concentrated in nightlife-heavy Manhattan zones and key airport corridors.
- Night operations are smaller than daytime operations, but they still form a meaningful niche segment.

### 5.5 Pricing Analysis

Average fare per mile per passenger:

- 1 passenger: `8.300`
- 2 passengers: `4.221`
- 3 passengers: `2.626`
- 4 passengers: `2.711`
- 5 passengers: `1.525`
- 6 passengers: `1.274`

This shows that solo riders effectively bear the highest cost per passenger.

Average fare per mile by day:

- Monday: `8.137`
- Tuesday: `8.517`
- Wednesday: `8.571`
- Thursday: `8.721`
- Friday: `8.052`
- Saturday: `8.532`
- Sunday: `7.716`

Thursday shows the highest average fare per mile, while Sunday shows the lowest.

Vendor comparison by distance tier:

- Up to 2 miles:
  - Creative Mobile Technologies: `9.652`
  - VeriFone: `10.850`
- 2 to 5 miles:
  - Creative Mobile Technologies: `6.378`
  - VeriFone: `6.524`
- More than 5 miles:
  - Creative Mobile Technologies: `4.415`
  - VeriFone: `4.473`

Interpretation:

- Short trips have the biggest fare-per-mile differences across vendors.
- Long-trip pricing is much more similar.

### 5.6 Passenger Trends

Average passenger count by day:

- Monday: `1.342`
- Tuesday: `1.319`
- Wednesday: `1.319`
- Thursday: `1.331`
- Friday: `1.389`
- Saturday: `1.455`
- Sunday: `1.442`

Interpretation:

- Weekend trips carry slightly more passengers on average.
- This is consistent with leisure, family, and group travel.

### 5.7 Surcharges

Surcharge prevalence:

- `improvement_surcharge`: `99.97%`
- `mta_tax`: `99.37%`
- `congestion_surcharge`: `89.64%`
- `extra`: `60.35%`
- `airport_fee`: `8.42%`
- `tolls_amount`: `8.10%`

Interpretation:

- Most base surcharges are nearly universal.
- Airport fees and tolls are much more route-specific.
- Some zone-level surcharge extremes are driven by low counts and should not be over-interpreted.

## 6. Key Insights

The analysis points to five major business drivers:

1. Demand is strongly time-driven, with the heaviest pressure in the late afternoon and early evening.
2. Manhattan and airport zones dominate overall trip volume.
3. Fare is strongly influenced by both distance and trip duration.
4. Night demand is smaller than daytime demand but remains important in nightlife and airport corridors.
5. Short trips are valuable because they generate high fare-per-mile and higher tip percentages.

## 7. Recommendations

### 7.1 Routing and Dispatch

- Increase driver availability before the `5 PM` to `7 PM` peak.
- Use congestion-aware dispatch rules in slow Midtown and Times Square corridors.
- Evaluate routes using both demand and efficiency, not trip count alone.

### 7.2 Cab Positioning

- Keep strong cab availability near `JFK Airport`, `LaGuardia Airport`, `Midtown Center`, and the `Upper East Side`.
- Build a separate night positioning strategy around `East Village`, `West Village`, `Clinton East`, and other nightlife zones.
- Rebalance cabs away from low-origin zones and toward pickup-heavy zones.

### 7.3 Pricing

- Protect yield during high-demand evening periods.
- Monitor short-trip pricing carefully because short trips produce the highest fare per mile.
- Use targeted promotions during softer periods such as Sunday or quieter off-peak hours.
- Benchmark short-distance fares closely across vendors.

## 8. Assumptions and Limitations

- The analysis is based on a `0.8%` stratified sample, not the full dataset.
- Scaled hourly trip counts are estimates.
- Some zone-level extremes are based on low volumes and should be interpreted carefully.
- Recorded tip data may understate cash-tip behavior.
- Outlier treatment improves realism but may remove a few legitimate edge cases.

## 9. Conclusion

The 2023 NYC yellow taxi data shows a network shaped by strong evening demand, dense Manhattan activity, major airport traffic, and pricing that is heavily influenced by both distance and congestion. The most important operating opportunities lie in aligning supply with evening peaks, maintaining strong presence in high-demand Manhattan and airport zones, and protecting revenue on short, high-yield trips. A focused strategy built around time-based staffing, location-based positioning, and congestion-aware dispatching would improve both service efficiency and revenue performance.
