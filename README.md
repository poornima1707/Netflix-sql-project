# Netflix Movies and TV Shows Data Analysis using SQL
This project focuses on analysis of Netflix’s movies and TV shows data using SQL to derive meaningful insights and address key business questions.
## Project Overview

In this project, SQL queries are applied to analyze Netflix’s content library, exploring metrics such as content type, genre distribution, country of origin, ratings, and production trends.

## Objectives and Key Questions
- **Analyze Content Distribution**: Examine the breakdown between movies and TV shows to understand the overall composition of Netflix’s library.
- **Identify Popular Ratings**: Find the most frequently assigned ratings for both movies and TV shows, providing insight into content maturity levels.
- **Content Analysis by Release Year, Country, and Duration**: Study the content by release year, origin country, and length to reveal patterns in Netflix's content catalog and regional preferences.
- **Explore and Categorize Content by Keywords**: Group and classify content based on specific keywords or themes to identify trends and unique content types within the catalog.
## Dataset
The data for this project is sourced from the Kaggle dataset:
Dataset Link: <a href=          Movies Dataset
## Schema
```sql
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```
## Business Problems and Solutions

### Q1. Count the Number of Movies vs TV Shows
```sql
SELECT type, COUNT(*)
FROM netflix
GROUP BY 1;
```
### Q2. Find the Most Common Rating for Movies and TV Shows
```sql
SELECT type, rating 
FROM
	(SELECT type , rating ,COUNT(*), RANK() OVER (PARTITION BY type ORDER BY COUNT(*) desc) as ranking
	FROM netflix
	GROUP BY 1,2) as table1
WHERE ranking = 1;
```
### Q3.  List All Movies Released in a Specific Year (e.g., 2020)
```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```
###  Q4. Find the Top 5 Countries with the Most Content on Netflix
```sql
SELECT * FROM 
	(SELECT UNNEST(STRING_TO_ARRAY(country, ' ,')) as country,
	COUNT(*) AS total_content
	FROM netflix
	GROUP BY 1
	)
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```
### Q5. Identify the Longest Movie
```sql
SELECT * FROM netflix
WHERE type = 'Movie' AND duration IS NOT NULL
ORDER BY SPLIT_PART(duration,' ',1)::INT DESC 
LIMIT 1;
```
### Q6. Find Content Added in the Last 5 Years
```sql
SELECT * 
FROM NETFLIX
WHERE TO_DATE(date_added, 'Month DD, YYYY')>= CURRENT_DATE - INTERVAL '5 years';
```
### Q7.  Find All Movies/TV Shows by Director 'Rajiv Chilaka'
```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name ILIKE 'Rajiv Chilaka';
```
### Q8. List All TV Shows with More Than 5 Seasons
```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```
### Q9. Count the Number of Content Items in Each Genre
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```
### Q10. Return top 5 year with highest avg content release
```sql
WITH Avg_table as (
SELECT 
    country,
    release_year,
    COUNT(*) AS total_release,
    ROUND(
        COUNT(*)::numeric /
        (SELECT COUNT(*) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year )
SELECT * FROM Avg_table 
ORDER BY avg_release DESC
LIMIT 5; 
```
### Q11.  List All Movies that are Documentaries
```sql
SELECT * 
FROM netflix
WHERE listed_in ILIKE '%Documentaries%';
```
### Q12. Find All Content Without a Director
```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```
### Q13. . Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years
```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```
### Q14.  Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```
### Q15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords
```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```
## Findings and Conclusion

- **Content Distribution**: The dataset reveals a broad mix of movies and TV shows across various ratings and genres, highlighting the diversity of Netflix’s library.
- **Common Ratings**: Identifying the most frequent ratings offers insights into the primary target audiences and the content maturity levels on the platform.
- **Geographical Insights**: Analyzing content by country, with a focus on the high volume from top countries and the average releases in India, underscores Netflix's regional content distribution strategies.
- **Content Categorization**: Grouping content based on specific keywords provides a clearer view of themes and content types, enhancing the understanding of the nature of offerings available on Netflix.

This analysis provides a view on Netflix's content landscape, supporting strategic content planning and more informed decision-making.
