# Analyzing Education & Census Data

## Objective
- Policymakers need to make more informed decisions on education by investigating how external factors may influence performance in state assessment exams for public high school students.

## Tools Used
- Google Big Query

## Project Tasks

1) Explore Tables : Start by exploring each table separately to understand the data.
2) Public High Schools Count :
- Determine the number of public high schools in each zip code.
- Determine the number of public high schools in each state.
- Use the locale_code column to understand urbanization levels. Use the CASE statement to display the corresponding locale_text and locale_size in your query result.

- Hint: Use the substr() function to examine parts of the locale_code for determining locale_text and locale_size.

3) Income Analysis
- Calculate the minimum, maximum, and average median household income of the nation.
- Calculate the minimum, maximum, and average median household income for each state.

4) Joint Analysis
- Join tables to analyze further.
- Investigate if characteristics of the zip-code area, such as median household income, influence students' performance in high school.
- Hint: Use the CASE statement to divide median_household_income into income ranges (e.g., <$50k, $50k-$100k, $100k+) and find the average exam scores for each range.

5) Intermediate Challenge
- Determine if students perform better on the math or reading exam on average.
- Find the number of states where students perform better in math exams compared to reading exams, and vice versa.
- Hint: Use the WITH clause to create a temporary table of average exam scores for each state, including a column indicating whether the math or reading average is higher. Include an option for "No Exam Data" for states without standardized assessments.

6) Advanced Challenge
- Calculate the average proficiency on state assessment exams for each zip code.
- Compare the average proficiency to other zip codes within the same state.
- Note: Exam standards may vary by state, so limit comparisons within states. Some states may not have exams.
Hint: Use the WITH clause to create a temporary table of exam score statistics for each state (e.g., min/max/avg) and then join it to each zip-code level data for comparison.
