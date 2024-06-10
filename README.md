# Analyzing Education & Census Data

## Objective
- Policymakers need to make more informed decisions on education by investigating how external factors may influence performance in state assessment exams for public high school students.

## Tools Used
- Google Big Query
- SQliteonline

## Project Tasks

#### 1) Explore Tables :
- Start by exploring each table separately to understand the data.
```sh
  SELECT * 
  FROM `stoked-producer-388612.12.census_data` 
  LIMIT 20;
```
![image](https://github.com/HimanshuBaswal/Census-High_School_SQL-Project/assets/74957804/72a95014-f705-40ab-8e4a-b47dd484dc42)

```sh
  SELECT * 
  FROM `stoked-producer-388612.12.public_hs` 
  LIMIT 100;
```
![image](https://github.com/HimanshuBaswal/Census-High_School_SQL-Project/assets/74957804/6fc6e901-909c-4c24-80c6-15ed5cbfc55d)


#### 2) Public High Schools Count :
- Determine the number of public high schools in each zip code.
```sh
  SELECT zip_code, COUNT(*) as school_count
  FROM `stoked-producer-388612.12.public_hs`
  GROUP BY zip_code;
```
![image](https://github.com/HimanshuBaswal/Census-High_School_SQL-Project/assets/74957804/62bd9143-c04d-43ac-bc11-d5d77761a2b9)

- Determine the number of public high schools in each state.
```sh
  SELECT state_code, COUNT(*) as school_count_by_state
  FROM `stoked-producer-388612.12.public_hs`
  GROUP BY state_code;
```
![image](https://github.com/HimanshuBaswal/Census-High_School_SQL-Project/assets/74957804/8ba0c261-7153-465f-9284-9bcc016d3dcf)

- Use the locale_code column to understand urbanization levels. Use the CASE statement to display the corresponding locale_text and locale_size in your query result.
- Hint: Use the substr() function to examine parts of the locale_code for determining locale_text and locale_size.
> The locale_code column uses numerical codes to represent different types of areas where the high schools are located. These codes can be broken down into two parts:
> The *first digit* represents the *locale_text*, indicating the general type of area (e.g., city, suburb, town, rural). It ranges for 1 to 4.
> The *second digit* represents the *locale_size*, indicating the size of the area (e.g., large, midsize, small).
> So, we will translate the *locale_code* from **public_hs** table into useful description to understand the level of urbanization for each high school.

```sh
  SELECT
    school_id,
    -- first digit of local_code is locale_text
    CASE
      WHEN SUBSTR(locale_code,1,1) = '1' THEN 'City'
      WHEN SUBSTR(locale_code,1,1) = '2' THEN 'Suburb'
      WHEN SUBSTR(locale_code,1,1) = '3' THEN 'Town'
      WHEN SUBSTR(locale_code,1,1) = '4' THEN 'Rural'
      ELSE 'Unknown'
    END AS locale_text,
    
    CASE
      WHEN SUBSTR(locale_code,2,1) = '1' THEN 'Large'
      WHEN SUBSTR(locale_code,2,1) = '2' THEN 'Midsize'
      WHEN SUBSTR(locale_code,2,1) = '3' THEN 'Small'
      ELSE 'Unknown'
    END AS locale_size
FROM `stoked-producer-388612.12.public_hs`;
```
![image](https://github.com/HimanshuBaswal/Census-High_School_SQL-Project/assets/74957804/4df6c1ae-3902-45a4-bd5c-52a0ceeff80b)

#### 3) Income Analysis : 
- Calculate the minimum, maximum, and average median household income of the nation.
```sh
SELECT 
  MIN(SAFE_CAST(median_household_income AS INT64)) as min_income,
  MAX(SAFE_CAST(median_household_income AS INT64)) as max_income,
  AVG(SAFE_CAST(median_household_income AS INT64)) as avg_income
FROM `stoked-producer-388612.12.census_data`
WHERE median_household_income IS NOT NULL
AND SAFE_CAST(median_household_income AS INT64) IS NOT NULL;
```
![image](https://github.com/HimanshuBaswal/Census-High_School_SQL-Project/assets/74957804/39f04fd8-0fc1-4b74-9370-79a91d553daf)

- Calculate the minimum, maximum, and average median household income for each state.
> Utilize the SAFE_CAST function, which returns NULL instead of failing when the conversion cannot be performed.
```sh
SELECT 
  state_code,
  MIN(SAFE_CAST(median_household_income AS INT64)) as min_income,
  MAX(SAFE_CAST(median_household_income AS INT64)) as max_income,
  AVG(SAFE_CAST(median_household_income AS INT64)) as avg_income
FROM `stoked-producer-388612.12.census_data`
WHERE median_household_income IS NOT NULL
AND SAFE_CAST(median_household_income AS INT64) IS NOT NULL
GROUP BY state_code;
```
![image](https://github.com/HimanshuBaswal/Census-High_School_SQL-Project/assets/74957804/11f62923-4508-4377-a9cd-adb38463bb1c)

#### 4) Joint Analysis :
- Join tables to analyze further.
- Investigate if characteristics of the zip-code area, such as median household income, influence students' performance in high school.
```sh
WITH income_ranges AS (
  SELECT
    h.school_id,
    l.zip_code,
    l.median_household_income,
    CASE 
        WHEN SAFE_CAST(l.median_household_income AS INT64) < 50000 THEN '<$50K'
        WHEN SAFE_CAST(l.median_household_income AS INT64) BETWEEN 50000 AND 100000 THEN '<$50K-$100K'
        ELSE '>$100K'
    END AS income_range,
    h.pct_proficient_math,
    h.pct_proficient_reading
  FROM `stoked-producer-388612.12.census_data` l 
  JOIN `stoked-producer-388612.12.public_hs` h ON l.zip_code = h.zip_code
)

SELECT 
  income_range,
  AVG(SAFE_CAST(pct_proficient_math AS INT64)) AS pct_proficient_math,
  AVG(SAFE_CAST(pct_proficient_reading AS INT64)) AS pct_proficient_reading
FROM income_ranges
GROUP BY income_range;
```
- *Hint*: Use the CASE statement to divide median_household_income into income ranges (e.g., <$50k, $50k-$100k, $100k+) and find the average exam scores for each range.
![image](https://github.com/HimanshuBaswal/Census-High_School_SQL-Project/assets/74957804/6455f038-f5f1-445b-a0fa-46599697c673)

#### 5) Intermediate Challenge :
- Determine if students perform better on the math or reading exam on average.
- Find the number of states where students perform better in math exams compared to reading exams, and vice versa.
```sh
WITH avg_exam_scores AS (
  SELECT
  state_code,
  AVG(SAFE_CAST(pct_proficient_math AS INT64)) AS avg_math_pct,
  AVG(SAFE_CAST(pct_proficient_reading AS INT64)) AS avg_reading_pct,
  CASE 
    WHEN AVG(SAFE_CAST(pct_proficient_math AS INT64)) > AVG(SAFE_CAST(pct_proficient_reading AS INT64)) THEN 'Math'
    WHEN AVG(SAFE_CAST(pct_proficient_math AS INT64)) < AVG(SAFE_CAST(pct_proficient_reading AS INT64)) THEN 'Reading'
    ELSE 'No Exam Data'
  END AS Higher_Performance_In,
  FROM `stoked-producer-388612.12.public_hs`
  GROUP BY state_code
)

SELECT Higher_Performance_In,
COUNT(*) AS number_of_states
FROM avg_exam_scores
GROUP BY Higher_Performance_In;
```
![image](https://github.com/HimanshuBaswal/Census-High_School_SQL-Project/assets/74957804/5fa0b04c-ddc9-4a88-8e10-ccc2314f3134)

- *Hint*: Use the WITH clause to create a temporary table of average exam scores for each state, including a column indicating whether the math or reading average is higher. Include an option for "No Exam Data" for states without standardized assessments.

#### 6) Advanced Challenge :
- Calculate the average proficiency on state assessment exams for each zip code.
- Compare the average proficiency to other zip codes within the same state.
- Note: Exam standards may vary by state, so limit comparisons within states. Some states may not have exams.

- *Hint*: Use the WITH clause to create a temporary table of exam score statistics for each state (e.g., min/max/avg) and then join it to each zip-code level data for comparison.
