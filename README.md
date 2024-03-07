# Hotel-Reservation-Analysis

### Project Overview
The primary objective of this project is to leverage a hotel reservation dataset for gaining insights and generating visualizations through the utilization of SQL for data manipulation and Tableau for visualization purposes. The project aims to delve into the dataset, identify interesting patterns, and create impactful visualizations to tell a compelling data story.


### Data Source
The dataset utilized for this analysis is the "Hotel Reservation Analysis in SQL and Tableau.xlsx" file, which contains detailed information about reservations at two types of hotels, Resort and City Hotels.


### Tools
- Excel
- MySQL
- Data visualization tool: Tableau


### Steps

#### 1. Data Import

The dataset was initially converted from xlsx format to csv using Microsoft Excel to facilitate seamless importing into MySQL. Additionally, data cleaning was performed on the countries_codes_and_coordinates.csv dataset, which contains the full names of countries corresponding to the Alpha-3 codes present in the main dataset. This involved using the text-to-column function in Excel to remove inverted commas from the Alpha-3 code column and renaming the column from "Alpha-3 code" to "country_code". This preprocessing step simplified querying the dataset.

#### 2. Data Exploration

The dataset comprises 119,390 rows and 32 columns with varying data types. Key questions explored include:
- Geographic distribution of guests (by countries).
- Busiest month in terms of reservations or cancellations.
- Bookings categorized by market segment.
- Cancellation rates.


#### 3. SQL Analysis

The csv file was imported into MySQL for data manipulation, as some analyses required combining or merging data from multiple columns in the dataset. To determine the top 10 countries of origin for hotel guests, the dataset recorded the first 3 letters of each country as the Alpha-3 code. To derive the full names of each country from their corresponding Alpha-3 codes, another dataset containing the Alpha-3 codes and full country names was obtained from GitHub.
After data cleaning, the two datasets were queried to obtain the full names of the top 10 countries of origin for hotel guests.

``` SQL
SELECT h.country, 
       c.Country,
       COUNT(*) AS Bookings
FROM hotel.hotelreservation AS h
INNER JOIN hotel.countries_codes_and_coordinate AS c
ON h.country = c.country_code
GROUP BY h.country, c.Country
ORDER BY Bookings DESC
LIMIT 10;
```


Also, to determine the number of reservations and cancellations per month across the hotels, the first step involved concatenating the individual components (arrival day, arrival month, and arrival year) of the arrival date into a single string format (YYYY-MM-DD). Additionally, the representation of the arrival month was converted from a string to a number format (e.g., "January" to "01"). Subsequently, calculations for total reservations using the COUNT function, cancelled reservations, and cancellation rate were performed, as illustrated in the query below.

``` SQL
WITH BookingDates AS (
    SELECT
        arrival_date,
        arrival_date - INTERVAL lead_time DAY AS booking_date,
        hotel,
        is_canceled
    FROM (
        SELECT
            CONCAT(
                arrival_date_year, '-',
                CASE 
                    WHEN arrival_date_month = 'January' THEN '01'
                    WHEN arrival_date_month = 'February' THEN '02'
                    WHEN arrival_date_month = 'March' THEN '03'
                    WHEN arrival_date_month = 'April' THEN '04'
                    WHEN arrival_date_month = 'May' THEN '05'
                    WHEN arrival_date_month = 'June' THEN '06'
                    WHEN arrival_date_month = 'July' THEN '07'
                    WHEN arrival_date_month = 'August' THEN '08'
                    WHEN arrival_date_month = 'September' THEN '09'
                    WHEN arrival_date_month = 'October' THEN '10'
                    WHEN arrival_date_month = 'November' THEN '11'
                    WHEN arrival_date_month = 'December' THEN '12'
                    ELSE NULL
                END,
                '-',
                LPAD(arrival_date_day_of_month, 2, '0')
            ) AS arrival_date,
            lead_time,
            hotel,
            is_canceled
        FROM
            hotel.hotelreservation
    ) AS ArrivalDate
)

SELECT hotel, 
	arrival_date,
	COUNT(*) AS total_reservations,
    SUM(is_canceled) AS canceled_reservations,
    (SUM(is_canceled) / COUNT(*)) * 100 AS cancellation_rate
FROM BookingDates
GROUP BY arrival_date, hotel
ORDER BY hotel, arrival_date;
```

This query provides an analysis of number of reservations by market segment. 

``` SQL
SELECT hotel, market_segment, COUNT(*) AS segment_count
FROM hotel.hotelreservation
GROUP BY hotel, market_segment
ORDER BY hotel, segment_count DESC;
```


#### 4. Tableau Visualization
Below are the visualizations including charts and a dashboard created using Tableau, showcasing the analysis of the questions posed earlier. These visualizations provide insights into various aspects of the dataset, such as the geographic distribution of guests, the busiest months for reservations and cancellations, bookings by market segment, and cancellation rates.

![Top 10 countries by reservation](https://github.com/Aroglobal1/Hotel-Reservation-Analysis/assets/148555924/b96becad-73d1-4bef-9b28-64ecd6a5a2ae)



![Total reservations and canceled reservations per month](https://github.com/Aroglobal1/Hotel-Reservation-Analysis/assets/148555924/3021c641-5b28-4b62-9508-dc38955d9841)

_The orange colour represents the cancelled reservations while the blue colour represents the total reservations._



![Total reservations and canceled reservations by market segment](https://github.com/Aroglobal1/Hotel-Reservation-Analysis/assets/148555924/b7014357-01fa-4b80-b6d1-fc7ce926749f)

_The orange colour represents the cancelled reservations while the blue colour represents the total reservations._



![Cancellation Rate by Lead time and market segment](https://github.com/Aroglobal1/Hotel-Reservation-Analysis/assets/148555924/bba32e34-b98b-4f79-9a59-2b66363a8182)



![Hotel Reservation Analysis Dashboard](https://github.com/Aroglobal1/Hotel-Reservation-Analysis/assets/148555924/e055911f-ff44-4317-8e0d-94325abb2336)


Here is the link to access the dashboard done on Tableau - https://public.tableau.com/app/profile/gloryarowojolu/viz/HotelReservationAnalysisDashboard/HotelReservationAnalysisDashboard


### Insights and Recommendations
Below are some insights derived from the above visualisations:

1.  From the chart displaying the top 10 countries of origin for hotel guests, Portugal has the highest number with approximately 48,586 reservations, indicating a significant number of guests hailing from the country. The United Kingdom follows with about 12,129 reservations, though with a notable gap between the two countries while the remaining countries follow with considerably lower number of reservations. Understanding this can inform marketing strategies and tailor services to cater to the needs and preferences of guests from specific regions.

2.  The line chart depicts total reservations and cancellations across months, with August showing the highest figures for both. Given this peak period, hotel management should strategize by implementing targeted marketing campaigns and flexible policies to maximize revenue and mitigate the impact of cancellations during this time.

3.  The chart displaying total reservations and cancellations by market segment illustrates a consistent trend of increasing figures from aviation, the lowest, to other market segments, with Online TA exhibiting the highest volume of both total reservations and cancellations. This pattern likely reflects the prevalent preference for online travel agencies for their bookings due to their convenience and ease of use.

4.  The heat map visualization reveals that cancellation rates across various market segments tend to rise with longer lead times, indicating a correlation between lead time and cancellation likelihood. Several factors can contribute to this trend, including the flexibility associated with short-term bookings, uncertainty surrounding long-term reservations, the comparison window provided by Online Travel Agency bookings, and the complexity of group booking dynamics. Hotels can minimize the impact of cancellations by implementing flexible policies, incentivizing early bookings, and engaging in effective communication. Also, targeted marketing campaigns for last-minute deals can help attract short-term bookings and reduce cancellations.
