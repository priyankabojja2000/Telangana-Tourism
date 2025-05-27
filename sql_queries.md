## Telangana Tourism SQL Queries



**1. Top 10 districts that have the highest number of domestic visitors overall (2016 - 2019)**
```
SELECT district, 
       SUM(visitors) AS total_visitors
FROM domestic_visitors
GROUP BY district
ORDER BY total_visitors DESC
LIMIT 10;
```


**2. Top 3 districts based on compounded annual growth rate (CAGR) of visitors between (2016 - 2019)**
```
WITH visitor_summary AS (
    SELECT district,
           SUM(CASE WHEN year = 2016 THEN visitors ELSE 0 END) AS visitors_2016,
           SUM(CASE WHEN year = 2019 THEN visitors ELSE 0 END) AS visitors_2019
    FROM domestic_visitors
    GROUP BY district
)
SELECT district,
       visitors_2016,
       visitors_2019,
        ROUND(
        (POWER(CAST(visitors_2019 AS DECIMAL(18,4)) / NULLIF(visitors_2016, 0), 1.0/3) - 1) * 100,
        2
    ) AS CAGR_Percentage
FROM visitor_summary
WHERE visitors_2016 > 0
ORDER BY CAGR_Percentage DESC
LIMIT 3;
```


**3. Bottom 3 districts based on compounded annual growth rate (CAGR) of visitors between (2016 - 2019)**
```
WITH visitor_summary AS (
    SELECT district,
           SUM(CASE WHEN year = 2016 THEN visitors ELSE 0 END) AS visitors_2016,
           SUM(CASE WHEN year = 2019 THEN visitors ELSE 0 END) AS visitors_2019
    FROM domestic_visitors
    GROUP BY district
)
SELECT district,
       visitors_2016,
       visitors_2019,
        ROUND(
        (POWER(CAST(visitors_2019 AS DECIMAL(18,4)) / NULLIF(visitors_2016, 0), 1.0/3) - 1) * 100,
        2
    ) AS CAGR_Percentage
FROM visitor_summary
WHERE visitors_2016 > 0
ORDER BY CAGR_Percentage ASC
LIMIT 3;
```


**4. Peak and low season months for Hyderabad based on the data from 2016 to 2019 for Hyderabad district**

**Peak months for domestic visitors:** 
```
SELECT 
    month,
    SUM(visitors) AS total_visitors
FROM domestic_visitors
WHERE district = 'Hyderabad'
GROUP BY month
ORDER BY total_visitors DESC
LIMIT 3;
```

**Low season months for domestic visitors:**
```
SELECT 
    month,
    SUM(visitors) AS total_visitors
FROM domestic_visitors
WHERE district = 'Hyderabad'
GROUP BY month
ORDER BY total_visitors ASC
LIMIT 3;
```

**Peak months for Foreign visitors:**
```
SELECT 
    month,
    SUM(visitors) AS total_visitors
FROM foreign_visitors
WHERE district = 'Hyderabad'
GROUP BY month
ORDER BY total_visitors DESC
LIMIT 3;
```

**Low season months for foreign visitors:**
```
SELECT 
    month,
    SUM(visitors) AS total_visitors
FROM foreign_visitors
WHERE district = 'Hyderabad'
GROUP BY month
ORDER BY total_visitors ASC
LIMIT 3;
```


**5. Top & bottom 3 districts with high domestic to foreign tourist ratio**

**Top 3 districts:**
```
WITH DomesticTotals AS (
    SELECT district, SUM(visitors) AS domestic_visitors
    FROM domestic_visitors
    GROUP BY district
),
ForeignTotals AS (
    SELECT district, SUM(visitors) AS foreign_visitors
    FROM foreign_visitors
    GROUP BY district
),
RatioCTE AS (
    SELECT 
        d.district,
        d.domestic_visitors,
        f.foreign_visitors,
        ROUND(d.domestic_visitors / NULLIF(f.foreign_visitors, 0), 2) AS domestic_foreign_ratio
    FROM DomesticTotals d
    LEFT JOIN ForeignTotals f ON d.district = f.district
)
SELECT *
FROM RatioCTE
ORDER BY domestic_foreign_ratio DESC
LIMIT 3;
```

**Bottom 3 districts:**
```
WITH DomesticTotals AS (
    SELECT district, SUM(visitors) AS domestic_visitors
    FROM domestic_visitors
    GROUP BY district
),
ForeignTotals AS (
    SELECT district, SUM(visitors) AS foreign_visitors
    FROM foreign_visitors
    GROUP BY district
),
RatioCTE AS (
    SELECT 
        d.district,
        d.domestic_visitors,
        f.foreign_visitors,
        ROUND(d.domestic_visitors / NULLIF(f.foreign_visitors, 0), 2) AS domestic_foreign_ratio
    FROM DomesticTotals d
    LEFT JOIN ForeignTotals f ON d.district = f.district
)
SELECT *
FROM RatioCTE
WHERE foreign_visitors > 0  -- exclude zero foreign visitors
ORDER BY domestic_foreign_ratio ASC
LIMIT 3;
```


**6. Top & bottom 5 districts based on 'population to tourist footfall ratio' in 2019**

**Top 5 districts:**
```
SELECT 
    p.district,
    p.Est_population_2019,
    COALESCE(d.total_domestic, 0) AS domestic_visitors,
    COALESCE(f.total_foreign, 0) AS foreign_visitors,
    (COALESCE(d.total_domestic, 0) + COALESCE(f.total_foreign, 0)) AS total_visitors,
    ROUND((COALESCE(d.total_domestic, 0) + COALESCE(f.total_foreign, 0)) / p.Est_population_2019, 2) AS footfall_ratio
FROM 
    district_population p
LEFT JOIN 
    (SELECT district, SUM(visitors) AS total_domestic
     FROM domestic_visitors
     WHERE year = 2019
     GROUP BY district) d ON p.district = d.district
LEFT JOIN 
    (SELECT district, SUM(visitors) AS total_foreign
     FROM foreign_visitors
     WHERE year = 2019
     GROUP BY district) f ON p.district = f.district
ORDER BY 
    footfall_ratio DESC
    LIMIT 5;
```
    
**Bottom 5 districts:**
```
SELECT 
    p.district,
    p.Est_population_2019,
    COALESCE(d.total_domestic, 0) AS domestic_visitors,
    COALESCE(f.total_foreign, 0) AS foreign_visitors,
    (COALESCE(d.total_domestic, 0) + COALESCE(f.total_foreign, 0)) AS total_visitors,
    ROUND((COALESCE(d.total_domestic, 0) + COALESCE(f.total_foreign, 0)) / p.Est_population_2019, 2) AS footfall_ratio
FROM 
    district_population p
LEFT JOIN 
    (SELECT district, SUM(visitors) AS total_domestic
     FROM domestic_visitors
     WHERE year = 2019
     GROUP BY district) d ON p.district = d.district
LEFT JOIN 
    (SELECT district, SUM(visitors) AS total_foreign
     FROM foreign_visitors
     WHERE year = 2019
     GROUP BY district) f ON p.district = f.district
ORDER BY 
    footfall_ratio ASC
    LIMIT 5;
```


**7 & 8. Projected numbers of domestic and foreign tourists in Hyderabad for 2025 based on past growth rates, and projected revenue for Hyderabad in 2025 considering the average spend per tourist**

**Domestic Visitors:**
```
WITH cte AS (
    SELECT 
        district,
        SUM(CASE WHEN year = 2016 THEN visitors ELSE 0 END) AS visitors_2016,
        SUM(CASE WHEN year = 2019 THEN visitors ELSE 0 END) AS visitors_2019
    FROM domestic_visitors
    WHERE district = 'Hyderabad'
    GROUP BY district
),
cte2 AS (
    SELECT 
        visitors_2019,
        POWER(visitors_2019 / NULLIF(visitors_2016, 0), 1.0 / 3) - 1 AS CAGR
    FROM cte
)
SELECT 
    visitors_2019 AS dom_visitors_2019,
    visitors_2019 * 1200 AS rev_dom_visitors_2019,
    ROUND(visitors_2019 * POWER(1 + AGR, 6)) AS dom_visitors_2025,
    ROUND(visitors_2019 * POWER(1 + AGR, 6)) * 1200 AS rev_dom_visitors_2025
FROM cte2;
```

**Foreign Visitors:**
```
WITH cte AS (
  SELECT district,
    SUM(CASE WHEN year = 2016 THEN visitors ELSE 0 END) AS visitors_2016,
    SUM(CASE WHEN year = 2019 THEN visitors ELSE 0 END) AS visitors_2019
  FROM foreign_visitors
  GROUP BY district 
  HAVING district = "Hyderabad"
),
cte2 AS (
  SELECT 
    visitors_2019 AS for_visitors_2019,
    (POWER((visitors_2019 / visitors_2016), 1.0/3) - 1) AS CAGR
  FROM cte
)
SELECT 
  for_visitors_2019,
  for_visitors_2019 * 5600 AS rev_for_visitors_2019, 
  ROUND(for_visitors_2019 * POWER((1 + AGR), 6)) AS for_visitors_2025,
  ROUND(for_visitors_2019 * POWER((1 + AGR), 6)) * 5600 AS rev_for_visitors_2025
FROM cte2;
```
