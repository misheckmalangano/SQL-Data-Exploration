# SQL-Data-Exploration CODE
From Fear to Facts: A Deep Dive into COVID-19 in Malawi and SADC

# SQL full code

# --1. Checking the two datasets
SELECT * FROM coviddeaths;
SELECT * FROM covidvaccinations;

# -- 2.Total cases by African country

SELECT location, MAX(total_cases) AS total_cases 
FROM CovidDeaths
WHERE continent = 'Africa'
GROUP BY location
ORDER BY total_cases DESC;


# -- 3. New reported cases over time

SELECT date, SUM(new_cases) AS total_new_cases
FROM coviddeaths
WHERE location = 'Malawi'
GROUP BY date
ORDER BY date;

# -- 4.Peaks in New Cases

SELECT date, (new_cases) AS peak_cases
FROM coviddeaths
WHERE location = 'Malawi'
ORDER BY peak_cases DESC;


# -- 5. Deathrate per 100000 people in Malawi

SELECT 
    ROUND(MAX(total_deaths) / MAX(population) * 100000,2) AS death_rate_per_100k
FROM coviddeaths
WHERE location = 'Malawi';

# -- 6. death percentage in Malawi over a period of time

SELECT location, date, total_cases, total_deaths, ROUND((total_deaths/total_cases)*100, 2) AS death_percentage
FROM coviddeaths
WHERE location = 'Malawi'
ORDER BY date;

# -- 7.death count recorded in Malawi
SELECT location,date, max(total_deaths)
FROM coviddeaths
WHERE location = 'Malawi'
ORDER BY total_deaths;


# -- 8. Monthly Cases in Malawi
SELECT location,year(date) AS year,MONTH (date) AS month,SUM(new_cases) AS monthly_cases
FROM coviddeaths
WHERE location = 'Malawi'
GROUP BY location, year(date), month(date)
ORDER BY year, month;

# -- 9.7-day rolling average of new cases

SELECT location, date, 
       ROUND(AVG(new_cases) OVER (PARTITION BY location ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW),3) 
       AS avg_7day_cases
FROM coviddeaths
WHERE location = 'MALAWI';

# -- 10. Regional Comparizon

SELECT location, ROUND(MAX(total_deaths)/MAX(total_cases),5) AS fatality_rate
FROM coviddeaths
WHERE location IN ('Malawi', 'Zambia','Mozambique','Tanzania','Zimbabwe','Botswana')
GROUP BY location
ORDER BY fatality_rate DESC;

# -- 11. Comparison of deaths per 100,OOO among SADC countries

SELECT location, ROUND((MAX(total_deaths / population) * 100000), 2) AS deaths_per_100k
FROM coviddeaths
WHERE location IN ('Angola', 'Botswana', 'Comoros', 'Democratic Republic of Congo', 'Eswatini',
                   'Lesotho', 'Madagascar', 'Malawi', 'Mauritius', 'Mozambique', 'Namibia',
                   'Seychelles', 'South Africa', 'Tanzania', 'Zambia', 'Zimbabwe')
GROUP BY location
ORDER BY deaths_per_100k DESC;

# -- 12. deaths without prior new cases

SELECT location, date, new_cases, new_deaths
FROM coviddeaths
WHERE location IN ('Angola', 'Botswana', 'Comoros', 'Democratic Republic of Congo', 'Eswatini',
                   'Lesotho', 'Madagascar', 'Malawi', 'Mauritius', 'Mozambique', 'Namibia',
                   'Seychelles', 'South Africa', 'Tanzania', 'Zambia', 'Zimbabwe')
  AND new_deaths > 0
  AND new_cases = 0;

# -- 13. Time to reach 5000 cases

WITH fs AS (SELECT location, MIN(date) AS first_case_date
			  FROM coviddeaths
			  WHERE location IN ('Angola', 'Botswana', 'Comoros', 'Democratic Republic of Congo', 'Eswatini',
                   'Lesotho', 'Madagascar', 'Malawi', 'Mauritius', 'Mozambique', 'Namibia',
                   'Seychelles', 'South Africa', 'Tanzania', 'Zambia', 'Zimbabwe') AND total_cases > 0
			  GROUP BY location),
              
 dy AS (SELECT location, MIN(date) AS date_5k
		  FROM coviddeaths
		  WHERE location IN ('Angola', 'Botswana', 'Comoros', 'Democratic Republic of Congo', 'Eswatini',
                   'Lesotho', 'Madagascar', 'Malawi', 'Mauritius', 'Mozambique', 'Namibia',
                   'Seychelles', 'South Africa', 'Tanzania', 'Zambia', 'Zimbabwe') AND total_cases >= 5000
		  GROUP BY location)
          
SELECT f.location, DATEDIFF(r.date_5k, f.first_case_date) AS days_to_5k
FROM fs f
JOIN dy r ON f.location = r.location
ORDER BY days_to_5k;


