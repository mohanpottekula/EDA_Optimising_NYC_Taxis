# Exploratory Data Analysis: Optimising NYC Taxi Operations

Name: `<Your Name>`

Assignment ID: `EDA/02`

## 1. Introduction

New York City yellow taxis operate in a fast-moving environment where demand changes by hour, day, location, and season. Because of this, taxi operators cannot rely on a fixed strategy throughout the year. They need to understand where demand is strongest, when trips are most frequent, which routes are efficient or inefficient, and how pricing and customer behavior change across different kinds of trips.

The purpose of this analysis is to study the 2023 NYC yellow taxi trip data and use it to identify patterns that can support better operational decisions. The main focus is on demand trends, revenue trends, trip characteristics, geographical concentration, tipping behavior, and route efficiency. Based on these findings, the report proposes recommendations on dispatching, cab positioning, and pricing strategy.

## 2. Data and Analytical Approach

The dataset consisted of 12 monthly parquet files for 2023, along with the taxi zone shapefile used for geographic analysis. Since the full-year data was too large to combine directly in memory, a sampled dataset was created first.

To make sure the sample still reflected actual travel patterns, sampling was done separately for each pickup date and pickup hour. A `0.8%` sample was taken from every date-hour group. This was a better approach than taking one random sample from the entire year, because it preserved the hourly structure of taxi demand across all months.

After sampling, the combined dataset contained `308,007` rows. Once missing values, invalid values, and outliers were handled, the final analytical dataset contained `304,630` rows. This size was manageable for analysis while still large enough to preserve clear patterns.

## 3. Data Preparation and Cleaning

The first step in cleaning the dataset was to fix structural issues. One of the clearest problems was the presence of two airport fee columns, `airport_fee` and `Airport_fee`, which appeared to represent the same information with different capitalization. These were combined into a single `airport_fee` column. Unnecessary index-like columns were removed, and column names were standardized for consistency.

The dataset also contained a small but noticeable amount of missing data. The main fields affected were `passenger_count`, `RatecodeID`, `store_and_fwd_flag`, and `congestion_surcharge`, each with around `3.39%` missing values. Missing and zero values in `passenger_count` were replaced with the most common value, which was `1`. Missing `RatecodeID` values were also filled with the mode, `1`. Missing `store_and_fwd_flag` values were set to `N`, and missing `congestion_surcharge` values were replaced with `0`. These decisions were reasonable because the missing proportion was small and the replacements matched the most common trip profile in the dataset.

Another issue was the presence of a few negative values in columns such as `mta_tax`, `improvement_surcharge`, `total_amount`, `congestion_surcharge`, and `extra`. These values were clearly invalid in context, so they were corrected to `0`. No negative values were found in `fare_amount`, which was a good sign.

Outlier analysis showed that some records had impossible or highly unrealistic values. For example, before treatment the maximum `trip_distance` was `22,528.82`, while the maximum `fare_amount` and `total_amount` were both above `143,000`. These values are not credible for normal NYC taxi operations and were likely caused by data-entry or system errors. To make the dataset suitable for business analysis, passenger counts above `6` were removed, trips shorter than `1` minute or longer than `240` minutes were filtered out, and very large values in `trip_distance`, `fare_amount`, `tip_amount`, and `total_amount` were capped at the `99.5th` percentile. After treatment, the highest values were much more realistic: `21.82` miles for trip distance, `89.80` for fare amount, and `114.19` for total amount.

Overall, the cleaning process improved the reliability of the dataset without removing the natural variation needed for exploratory analysis.

## 4. General Exploratory Analysis

The variables in the dataset naturally fell into three broad types. Numerical variables included `trip_distance`, `fare_amount`, `tip_amount`, `total_amount`, `passenger_count`, and `trip_duration_min`. Categorical variables included fields such as `vendorid`, `ratecodeid`, `payment_type`, `store_and_fwd_flag`, and the pickup and dropoff location identifiers. Pickup and dropoff timestamps were treated as datetime variables and later used to derive features such as hour of day, day of week, month, and quarter.

### 4.1 Demand by Time

The clearest pattern in the data was the concentration of demand in the late afternoon and early evening. The busiest hour in the sampled data was `18:00`, which is consistent with after-work travel and evening activity. The busiest day was `Thursday`, and the busiest month was `October`.

When the hourly counts were scaled using the sampling fraction, the five busiest hours were all clustered in the afternoon and evening:

- `18:00`: `2,678,625`
- `17:00`: `2,554,750`
- `19:00`: `2,395,375`
- `15:00`: `2,345,500`
- `16:00`: `2,344,250`

This is an important operational finding. It suggests that the strongest pressure on taxi supply does not occur randomly through the day. Instead, it builds toward the evening, meaning drivers should already be positioned and available before the peak begins.

### 4.2 Revenue Trends

Revenue varied noticeably across the year. Based on the sampled data, the strongest revenue months were `May` at `809,656.92` and `October` at `806,265.72`, while the weakest were `February` at `626,706.67` and `August` at `634,866.36`. This shows that revenue opportunity is not evenly distributed through the year.

Quarterly revenue shares supported the same conclusion:

- Q1: `23.75%`
- Q2: `26.82%`
- Q3: `22.62%`
- Q4: `26.81%`

Q2 and Q4 made the largest contribution to total revenue, which suggests stronger business conditions in those periods. From an operator’s point of view, this means staffing and planning decisions should consider seasonal variation rather than assuming one steady pattern across the year.

### 4.3 Fare, Distance, and Duration

The financial variables behaved largely as expected. The relationship between `trip_distance` and `fare_amount` was extremely strong, with a correlation of `0.961`. The relationship between trip duration and fare was also strong, with a correlation of `0.873`.

This indicates that both distance and time matter in explaining how much a trip earns. Distance is the strongest direct driver, but trip duration is also important because congestion increases the time spent on a trip and therefore affects fare and driver productivity.

Average fare by passenger count did not increase in a simple linear way:

- 1 passenger: `18.93`
- 2 passengers: `21.82`
- 3 passengers: `21.54`
- 4 passengers: `22.13`
- 5 passengers: `18.71`
- 6 passengers: `18.95`

This is not surprising because taxi fares are mainly trip-based rather than charged per passenger. Passenger count is therefore more useful for understanding demand type than for explaining the fare itself.

### 4.4 Payment and Tipping

Payment behavior was dominated by credit card usage. Out of the cleaned sample, `240,777` trips were paid by credit card and `50,521` by cash, with the remaining trips falling into smaller categories such as unknown, dispute, or no-charge cases.

Tip amount increased with trip distance, which was expected because longer trips usually have higher fares:

- `0-1` miles: `2.39`
- `1-2` miles: `3.06`
- `2-5` miles: `4.33`
- `5-10` miles: `7.82`
- `10+` miles: `13.25`

However, tip percentage told a different story. Average tip percentage was highest for the shortest trips and then declined as distance increased:

- Up to 2 miles: `28.72%`
- 2 to 5 miles: `22.97%`
- 5 to 10 miles: `22.40%`
- 10+ miles: `21.14%`

This suggests that passengers tend to tip more generously, in percentage terms, on short and moderate trips than on long expensive ones.

When low-tip trips were compared with high-tip trips, the difference became clearer. Low-tip trips had a higher average distance (`4.73` vs `2.27`) and a higher average fare (`25.24` vs `14.25`). High-tip trips also had a slightly stronger credit card presence. Taken together, this suggests that shorter and cheaper trips are often better from a tipping perspective.

### 4.5 Geographic Concentration

The taxi market was heavily concentrated in a relatively small number of zones. The leading pickup zones were:

- `JFK Airport`: `15,417`
- `Upper East Side South`: `14,195`
- `Midtown Center`: `14,070`
- `Upper East Side North`: `12,817`
- `Midtown East`: `10,877`
- `Penn Station/Madison Sq West`: `10,307`
- `LaGuardia Airport`: `10,280`

The leading dropoff zones were:

- `Upper East Side North`: `13,554`
- `Upper East Side South`: `12,731`
- `Midtown Center`: `11,913`
- `Times Sq/Theatre District`: `9,213`
- `Murray Hill`: `9,046`
- `Midtown East`: `8,675`
- `Lincoln Square East`: `8,640`

These results show that Manhattan remains the central area of yellow taxi demand, while airports continue to be very important trip origins. This has clear implications for positioning and dispatch strategy.

## 5. Detailed Analysis

### 5.1 Route Efficiency

One of the more useful operational findings came from comparing route speeds across zones and hours. The slowest high-volume routes were mostly located in dense Manhattan areas:

- `Penn Station/Madison Sq West -> Times Sq/Theatre District` at `12:00`: `3.05 mph`
- `Times Sq/Theatre District -> Times Sq/Theatre District` at `17:00`: `3.23 mph`
- `Midtown Center -> Times Sq/Theatre District` at `19:00`: `3.30 mph`

These routes are not weak routes in terms of demand. In fact, they are busy and commercially important. The problem is that heavy congestion reduces efficiency. This means that high demand alone should not be used as the only indicator of route attractiveness. Operators also need to consider how long drivers are tied up in slow-moving corridors.

### 5.2 Weekday, Weekend, and Night Demand

The weekday pattern was more commute-driven, while the weekend pattern was more spread into leisure hours. This is a fairly natural result for a city like New York and supports the idea of having different staffing logic for weekdays and weekends.

Night demand also showed a clear geographical pattern. The top night pickup zones were `East Village`, `JFK Airport`, `West Village`, `Clinton East`, and `Lower East Side`. The main night dropoff zones included `East Village`, `Clinton East`, `Murray Hill`, and `East Chelsea`.

Although night trips contributed less revenue overall than day trips, they still represented a meaningful segment:

- Day revenue share: `87.44%`
- Night revenue share: `12.56%`

This means night operations should not be treated as the main revenue engine, but they are still important enough to justify a targeted strategy.

### 5.3 Pickup and Dropoff Imbalance

The pickup-to-dropoff ratio provided another useful perspective on zone behavior. The highest ratios were seen in:

- `East Elmhurst`: `9.06`
- `JFK Airport`: `4.65`
- `LaGuardia Airport`: `2.65`
- `Penn Station/Madison Sq West`: `1.52`

The very low ratios were seen in places such as `Newark Airport`, `Windsor Terrace`, `Whitestone`, and `Greenpoint`.

These results suggest that airports and major travel hubs act as strong trip-origin zones, while some other areas function more as destinations. At the same time, the smallest-volume zones should be interpreted carefully because a small number of rides can exaggerate the ratio.

### 5.4 Pricing and Vendor Comparison

Average fare per mile per passenger declined sharply as the number of passengers increased:

- 1 passenger: `8.300`
- 2 passengers: `4.221`
- 3 passengers: `2.626`
- 4 passengers: `2.711`
- 5 passengers: `1.525`
- 6 passengers: `1.274`

This shows that from the passenger’s point of view, solo travel is the most expensive on a per-person basis.

Average fare per mile also varied by day:

- Monday: `8.137`
- Tuesday: `8.517`
- Wednesday: `8.571`
- Thursday: `8.721`
- Friday: `8.052`
- Saturday: `8.532`
- Sunday: `7.716`

Thursday had the highest average fare per mile, while Sunday had the lowest. This suggests some difference in the quality of revenue across the week, not just in the number of trips.

The vendor comparison by distance tier showed that short trips had the biggest differences:

- Up to 2 miles:
  - Creative Mobile Technologies: `9.652`
  - VeriFone: `10.850`
- 2 to 5 miles:
  - Creative Mobile Technologies: `6.378`
  - VeriFone: `6.524`
- More than 5 miles:
  - Creative Mobile Technologies: `4.415`
  - VeriFone: `4.473`

This suggests that short-distance pricing is where vendor differences are most visible, while long-distance pricing is much more similar.

### 5.5 Passenger and Surcharge Patterns

Passenger count by day of week was slightly higher on weekends:

- Friday: `1.389`
- Saturday: `1.455`
- Sunday: `1.442`

Compared with earlier weekdays, this points toward more group or leisure travel on weekends.

Surcharge prevalence also showed a useful pricing pattern:

- `improvement_surcharge`: `99.97%`
- `mta_tax`: `99.37%`
- `congestion_surcharge`: `89.64%`
- `extra`: `60.35%`
- `airport_fee`: `8.42%`
- `tolls_amount`: `8.10%`

This shows that some surcharges are effectively built into the standard structure of most taxi trips, while others are linked to specific trip types such as airport or toll-road trips.

## 6. Main Insights

Looking across the full analysis, five points stand out clearly.

First, demand is strongly time-driven, and the most important demand window is the late afternoon and early evening. Second, Manhattan and airport zones dominate both volume and strategic importance. Third, revenue is shaped by both distance and duration, which means congestion matters not only operationally but also financially. Fourth, night demand is smaller than daytime demand but still important in nightlife and airport corridors. Fifth, short trips are especially valuable because they generate high fare per mile and relatively high tip percentages.

## 7. Recommendations

The most immediate recommendation is to improve dispatching around the evening peak. Driver availability should be increased before `5 PM` to `7 PM`, not after the rush has already started. Routes around Midtown, Penn Station, and Times Square should be monitored carefully because they combine strong demand with very slow speeds.

The second recommendation is to position cabs more aggressively in high-demand zones. Airports, Midtown, Upper East Side areas, and major Manhattan business and entertainment districts should receive priority. A separate night positioning plan should be used for areas such as `East Village`, `West Village`, `Clinton East`, and airport corridors, where nighttime demand remains meaningful.

The third recommendation concerns pricing. High-demand evening periods should be protected from unnecessary discounting. Short trips deserve special attention because they generate the highest fare per mile and show the clearest vendor pricing differences. Softer periods such as Sunday and other low-yield windows can be used for targeted promotions instead of broad price reductions across the whole network.

## 8. Assumptions and Limitations

This analysis is based on a `0.8%` stratified sample rather than the full raw dataset, so scaled values should be treated as estimates. Some zone-level extremes are based on small trip counts and should not be over-interpreted. In addition, tip values mainly reflect recorded tips and may not fully capture cash-tip behavior. Finally, the outlier treatment improves realism for analysis, but it may remove a small number of unusual but valid trips.

## 9. Conclusion

The 2023 NYC yellow taxi data shows a system shaped by strong evening demand, heavy Manhattan concentration, major airport pickup activity, and pricing that depends closely on both distance and traffic conditions. The clearest opportunity for a taxi operator is to match supply more closely to those time-and-location patterns. A strategy built around better peak-hour staffing, stronger positioning in airport and core Manhattan zones, and smarter dispatching through congested corridors would improve both efficiency and revenue performance.
