# Team 3 MIST4610 71552 Group Project 2

## Team Members

1. Brooks Lawson [@blaw05303](https://github.com/blaw05303)
2. Ishaan Prasad [@ishaanprasad1](https://github.com/ishaanprasad1)
3. John Paul [@johnpaul1111-hub](https://github.com/johnpaul1111-hub)
4. Johnathan Lu [@johnathanlu](https://www.github.com/johnathanlu)
5. Madison Jin [@madisonajin-bot](https://github.com/madisonajin-bot)
6. Maxwell Alton Buckingham [@mabuckingham](https://github.com/mabuckingham)

## Description of Dataset (FBI)

The dataset we selected is FBI Crime Data which is on the Snowflake Marketplace and it contains historical data regarding crime across the United States. We chose this specific dataset because it contains large scale data especially useful for detailed analysis. It is a relational database with multiple tables including offense level data, summary statistics, and geography information. It contains hundreds of thousands of rows though due to all of the accumulated information for all 50 states for many years too. Key columns include variables such as year, state / region, offense type, crime counts, rates per 100,000 people, and crime category. The crime counts and year are stored in the integer data, the rates power 100,000 people are stored in Decimal format, and everything else is string data in the VARCHAR format. We proceeded to use these for SQL based querying, as shown later.


## Question 1: South Regional Analysis
## How does crime in Georgia compare to neighboring states over time? 
<img width="1556" height="696" alt="image" src="https://github.com/user-attachments/assets/5bbe7177-e01d-4120-8de7-f8df0f103502" />

Florida’s crime volume heavily skews the overall regional statistics, Florida had much greater volatility in its trend as we see much larger rises and falls in its trend line when compared to the other states. Georgia and North Carolina seem to trend very similarly in  the graph implying that the crime rates in each state are exposed to similar vulnerabilities.

## How much regional crime does the state with the most (Florida) account for compared to the other states in the southeast?

<img width="1785" height="816" alt="image" src="https://github.com/user-attachments/assets/63fe2cde-5b21-4f97-86c6-ddea6e6fd67e" />

Florida accounts for almost half of the crime that is committed in the South Eastern United States. This stacked composition reveals a massive imbalance in regional safety. From an operational standpoint this would indicate that the FBI should place more of their resources in Florida to have the greatest impact on regional safety.

## Interactive Element:

The Regional Analysis Filter allows you to select a year range to analyze by implementing a double sided slider. It applies to both the line chart and the bar chart. Viewing data over 40 years can obscure trends that happen along shorter periods of time. You can adjust to look at the lifetime of different trends and compare that against policy and other factors that might be better shown over a shorter period of time. In the line chart you can see a sharp drop off, like in 2018,  or increase, as seen in the early 90’s,  much better when narrowing the time scope. For the bar graph you can see better the weight that Florida took up better when narrowing the years. With this you can compare better the weight of Florida from time periods that are not directly adjacent. This is a great tool to be able to narrow in on political or economic eras. 

## AI Implementation:
Prompts: 
“I'd like help with refining my Snowflake Dashboard that I'm creating. This is Component 2 in the Group Project 2 Print out. You are meant to take this Python code provided and generate an improved version that I can use for the assignment. AI assistance is permitted for this portion if the modifications that you make are cataloged. I'd like you to work through all the changes in the best way possible to make the cleanest and best looking dashboard and then after completing your changes list out all the components that you changed and what you changed about them. Make the dashboard simple and easy to understand while still making it look professional. Use a University of Georgia theme for the dashboard. There are two questions that are being answered match the looks of the graphs to keep them cohesive. Generate appropriate table names and axis names for the graphs. Determine if there is a better graph type that is better suited at presenting the data. The questions are "How does crime in Georgia compare to neighboring states over time? How much regional crime does the state with the most (Florida) account for compared to the other states in the southeast?". The code for the base version is, …”
“I have a couple improvements that I think could help it look a lot better. The Gray header text is a little hard to read on the background color and it would be great if you made that text darker. The lines that represent the other states stick to the theme very well but the colors are too similar for many of the states and it would be great if you could make them more distinct even if that means shifting away from the theme.”

## Dataset Manipulation:
1. Structural & Boilerplate Cleanup
Removed Snowsight Artifacts: Stripped out the auto-generated @st.fragment decorators, hardcoded timestamps, and refresh buttons.
Simplified Execution: Replaced the complex asynchronous job creation and duplicate column renaming logic with a standard, highly efficient session.sql().to_pandas() flow.

2. Data Querying & Processing
Consolidated SQL Queries: Instead of running two separate, complex SQL queries, the code now runs a single, efficient query to fetch the base state-level data.
Pandas Integration: Shifted the data manipulation for Question 2 (calculating Florida's share versus the rest of the states) out of SQL and into Python using Pandas pivot() and melt() functions. This reduces compute load on Snowflake and prepares the data perfectly for stacked bar charts.

3. Theming & Accessibility
Custom CSS: Injected custom HTML/CSS via st.markdown() to override default Streamlit styling. This established the University of Georgia theme by forcing headers to UGA Red (#BA0C2F) and setting the background to white.
High-Contrast Text: Updated the CSS and added global Altair configurations (configure_title, configure_axis, configure_legend) to ensure all chart text, axes, and paragraph elements render in solid, high-contrast black for readability.

4. Interactivity
Added Dynamic Filtering: Implemented a sidebar using st.sidebar.slider to act as an interactive date range selector.
Analytical Value: This slider dynamically filters the underlying Pandas dataframe before it is passed to the charts, fulfilling the requirement for an interactive element that fundamentally changes the data displayed.

5. Chart Enhancements (Altair Integration)
Chart 1 (Time Trends): Replaced the original st.scatter_chart with an Altair Line Chart (mark_line()), which is the required chart type for displaying time trends.
Chart 1 Colors: Implemented a custom color scale. Georgia was mapped to UGA Red and Florida to Black to fit the theme, while the remaining states were explicitly assigned standard, highly distinct colors (Blue, Orange, Green, Purple) to fix readability issues.
Chart 2 (Proportions): Replaced the generic st.bar_chart with an Altair Stacked Bar Chart (mark_bar()). This visually stacked "Florida" on top of "Other States Total" to accurately answer the question regarding Florida's share of the regional volume.
Chart Tooltips & Labels: Added explicit titles, customized axis labels (with a -45 degree angle for the years to prevent overlapping), and interactive tooltips to both Altair charts.
Syntax Correction: Formatted the Altair array structures vertically to prevent line-truncation issues that were causing "unterminated string literal" syntax errors.

## SQL

SELECT
    YEAR(ts.DATE)::String AS year,
    geo.GEO_NAME AS state,
    SUM(ts.VALUE) AS total_crimes
FROM SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.FBI_CRIME_TIMESERIES ts
JOIN SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.GEOGRAPHY_INDEX geo
    ON ts.GEO_ID = geo.GEO_ID
WHERE geo.LEVEL = 'State'
  AND geo.GEO_NAME IN (
        'Georgia',
        'Alabama',
        'Florida',
        'South Carolina',
        'Tennessee',
        'North Carolina'
  )
GROUP BY year, state
ORDER BY year, state;
This pulls total crimes per year for 6 southeastern states (Georgia, Alabama, Florida, South Carolina, Tennessee, and North Carolina), grouped by year and state — letting you compare overall crime trends side by side across the region over time.

WITH yearly AS (
    SELECT
        YEAR(ts.DATE)::String AS year,
        geo.GEO_NAME AS state,
        SUM(ts.VALUE) AS total_crimes
    FROM SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.FBI_CRIME_TIMESERIES ts
    JOIN SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.GEOGRAPHY_INDEX geo
        ON ts.GEO_ID = geo.GEO_ID
    WHERE geo.LEVEL = 'State'
      AND geo.GEO_NAME IN (
            'Florida',
            'Georgia',
            'Alabama',
            'South Carolina',
            'Tennessee',
            'North Carolina'
      )
    GROUP BY year, state
),

totals AS (
    SELECT
        year,
        SUM(total_crimes) AS regional_total,
        SUM(CASE WHEN state = 'Florida' THEN total_crimes ELSE 0 END) AS florida_total
    FROM yearly
    GROUP BY year
)

SELECT
    year,
    florida_total,
    regional_total,
    ROUND(florida_total / regional_total * 100, 2) AS florida_share_pct
FROM totals
ORDER BY year;

It calculates Florida's share of total crime across 6 southeastern states, year by year — first summing each state's crimes annually, then computing what percentage of the entire region's crime Florida alone accounts for each year, letting you see whether Florida's contribution to regional crime is growing, shrinking, or staying steady over time.

## Question 2: Georgia Historical Analysis

## How do crime rates in Georgia compare decade by decade? 

<img width="1546" height="680" alt="image" src="https://github.com/user-attachments/assets/e55cd429-2991-4032-9ab2-a7c318459069" />

The 1990’s indicated a historical high for criminal activity in Georgia. However, the most significant takeaway is the steady decline of crime throughout the 2000’s and 2010’s. The 2020’s seems to indicate a sharp drop off of crime but that decade has not concluded obviously and therefore the data is incomplete. The reason for low crime in the 1970’s is due to the FBI not including earlier data in the set that they provided to Snowflake. This data is important for operational reasons as it allows us to visualize the effects of macro elements and policy changes over time a long enough time period to see the changes.

## How did the top 3 crime rates change by year for the decade with the most crime?

<img width="1525" height="687" alt="image" src="https://github.com/user-attachments/assets/b398eeb9-c3b7-49a6-8bc2-af5f20f5d42b" />

During Georgia’s most Crime Heavy Decade, the distribution among its top offenses remained relatively constant. This tells us that the high crime rate of the 90’s was likely due to wide spread systematic failures, rather than being driven by a sudden isolated spike in one specific type of offense. By filtering out the highest volume crime, users can more clearly see that even the lower volume top crimes perfectly mirror this steady parallel trend. 

## Interactive Element:
The “Historical Analysis Filters” utilizes a multi select drop down that allows users to add or remove specific crime categories from the line chart in “Yearly Trends of Top 3 Crime Categories in Georgia (1990-1999). This allows users to declutter the visualization to better isolate the trajectory of a single crime type, or to directly compare two categories. This can be helpful if looking at the effects of different policies implemented on different crime categories.

## AI Implementation:
Prompt:
“I have to do the same thing for a new dashboard that I'm creating. I would like for you to help me by remembering the requirements that I set for the previous one you made. The new questions are "How do crime rates in Georgia compare decade by decade? How did the top 3 crime rates change by year for the decade with the most crime?" The new code is…”

## Dataset Manipulation
1. Structural & Boilerplate Cleanup
Removed Snowsight Artifacts: Stripped out the auto-generated @st.fragment decorators, hardcoded timestamps (e.g., "2026-04-27 4:42pm"), and the manual refresh buttons.
Simplified Execution: Replaced the complex asynchronous job creation and duplicate column renaming blocks with standard, clean session.sql().to_pandas() functions wrapped in Streamlit's @st.cache_data for better performance.

2 Data Querying & Processing
Optimized Query 1 (Decades): The original SQL query pulled the VARIABLE_NAME (crime category) and grouped by it, but the Streamlit code just aggregated it all back together by decade anyway. We simplified the SQL query to GROUP BY DECADE directly in Snowflake, returning exactly what was needed and saving compute time.
Streamlined Data Loading: Separated the two queries into distinct, clearly named Python functions (get_decade_data() and get_top_crimes_1990s()) to make the code more readable and modular.

3. Interactivity Added
Multi-Select Filter: The original code had no interactive elements. I added a sidebar with st.sidebar.multiselect() allowing users to filter which of the top 3 crime categories they want to view on the 1990s line chart.
Error Handling: Added a conditional check (if not filtered_df_1990s.empty:) to display a warning message if the user deselects all categories, preventing the chart from throwing a rendering error.

4. Chart Enhancements (Altair Integration)
Chart 1 (Decade Bar Chart): Replaced Streamlit's native st.bar_chart with an Altair Bar Chart (mark_bar()). I explicitly set the bar color to UGA Red (#BA0C2F) to fit the theme and tilted the X-axis labels -45 degrees so the decade strings wouldn't overlap.
Chart 2 (1990s Top Crimes): * Replaced Streamlit's native st.area_chart with an Altair Line Chart (mark_line(point=True)). Area charts with multiple categories can visually obscure data if the areas overlap, so a line chart with data points is much more effective for comparing trends.
Applied a distinct color scale mapping the top 3 categories to high-contrast, theme-appropriate colors: UGA Red, Solid Black, and a distinct Blue (#1f77b4).
Tooltips & Labels: Added explicit titles, customized axis labels, and interactive tooltips to both charts for better analytical readability

5. Theming & Accessibility
Custom CSS Theme: Re-applied the custom HTML/CSS block to ensure the background is white, headers are UGA Red, and paragraph text is solid black.
High-Contrast Charts: Added the global Altair configurations (configure_title, configure_axis, configure_legend) to ensure all chart text renders in solid black, matching the readability improvements from the first dashboard.

## SQL

Part 1
SELECT
    (FLOOR(YEAR(ts.DATE) / 10) * 10)::STRING AS decade,
    ts.VARIABLE_NAME                             AS offense_category,
    SUM(ts.VALUE)                                AS total_crimes
FROM  SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.FBI_CRIME_TIMESERIES  ts
JOIN  SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.GEOGRAPHY_INDEX        geo
      ON ts.GEO_ID = geo.GEO_ID
WHERE geo.LEVEL     = 'State'
  AND geo.GEO_NAME  = 'Georgia'
GROUP BY decade, ts.VARIABLE_NAME
ORDER BY decade, total_crimes DESC; 

This pulls Georgia state-level FBI crime data, groups every year into its decade (1990s, 2000s, etc.), sums up crimes by type for each decade, and returns the results sorted by decade with the highest-volume crime types listed first — giving you a decade-by-decade breakdown of crime categories across Georgia.

Part 2
SELECT
    YEAR(ts.DATE)::STRING AS year,
    ts.VARIABLE_NAME AS crime_category,
    SUM(ts.VALUE) AS total_crimes
FROM SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.FBI_CRIME_TIMESERIES ts
JOIN SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.GEOGRAPHY_INDEX geo
    ON ts.GEO_ID = geo.GEO_ID
JOIN (
    SELECT
        ts.VARIABLE_NAME
    FROM SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.FBI_CRIME_TIMESERIES ts
    JOIN SNOWFLAKE_PUBLIC_DATA_FREE.PUBLIC_DATA_FREE.GEOGRAPHY_INDEX geo
        ON ts.GEO_ID = geo.GEO_ID
    WHERE geo.LEVEL = 'State'
      AND geo.GEO_NAME = 'Georgia'
      AND YEAR(ts.DATE) BETWEEN 1990 AND 1999
    GROUP BY ts.VARIABLE_NAME
    ORDER BY SUM(ts.VALUE) DESC
    LIMIT 3
) top_categories
    ON ts.VARIABLE_NAME = top_categories.VARIABLE_NAME
WHERE geo.LEVEL = 'State'
  AND geo.GEO_NAME = 'Georgia'
  AND YEAR(ts.DATE) BETWEEN 1990 AND 1999
GROUP BY year, ts.VARIABLE_NAME
ORDER BY year, crime_category;

It finds the top 3 most common crime types in Georgia during the 1990s, then tracks those same 3 categories year by year across the decade — giving you a line chart where each line follows one crime category from 1990 to 1999, making it easy to spot trends and compare how those top crimes changed over time.


