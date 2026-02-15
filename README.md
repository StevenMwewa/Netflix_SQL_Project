# NETFLIX MOVIES & TV SHOWS SQL_Project
This Project aims to analyze the Dataset from Netflix which contains the movies, their ratings , duration, streams and more. The goal was to extract valuable insights and answer various business questions based on the dataset. For a complete work through of the step by step analysis and sql code, check out the README files provided.

## OBJECTIVES 
1. Analyze the distributuion of content types (movies vs TV Shows).
2. Identify the most common ratings for movies and TV Shows.
3. List and analyze content based on release years, countries, and durations.
4. Explore and categorize content based on specific criteria and keywords.

## SCHEMA 
```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
	show_id VARCHAR(6),
	type VARCHAR(10),
	title VARCHAR(150),
	director VARCHAR(208),
	casts	VARCHAR(1000),
	country	VARCHAR(150),
	date_added VARCHAR(50),
	release_year INT,
	rating	VARCHAR(10),
	duration VARCHAR(15),
	listed_in VARCHAR(100),
	description VARCHAR(250)
);
```
## BUSINESS PROBLEMS AND SOLUTIONS

### 1. count the numbers of movies vs tv shows
```sql
SELECT * FROM netflix;

SELECT
	type,
	COUNT(show_id)
FROM netflix
GROUP BY type;
```

### 2. find the most common ratings for both movies and tv shows
### Using CTE
```sql
WITH cte AS
(
	SELECT
		type,
		rating,
		COUNT(*),
		RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
	FROM netflix
	GROUP BY 1, 2
	--ORDER BY 1, 3 DESC;
)
SELECT 
	type,
	rating
FROM cte
WHERE ranking = 1;
```
### Using Subqueries
```sql
SELECT 
	type,
	rating
FROM 
(
	SELECT
		type,
		rating,
		COUNT(*),
		RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
	FROM netflix
	GROUP BY 1, 2
) as t1

WHERE ranking = 1;
```

### 3. List all movies released in a specific year (e.g., 2020)
```sql
SELECT
	title,
	release_year
FROM netflix 
WHERE 
	release_year = 2020 
		AND 
	type = 'Movie';
```
### 4. Find the top 5 countries with the most content on Netflix
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
	COUNT(show_id) as content_num
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

### 5. Identify the longest movie
```sql
SELECT 
	title,
	duration
FROM netflix
WHERE
	type = 'Movie'
	AND
	duration = (SELECT MAX(duration) FROM netflix)
```

### 6. Find content added in the last 5 years
```sql
SELECT
	*
FROM netflix
WHERE
	TO_DATE(date_added, 'Month DD, YYY') >= CURRENT_DATE - INTERVAL '5 years'
```

### 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
```sql
SELECT
	 type,
	 director
FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%'
```
### 8. List all TV shows with more than 5 seasons
```sql
SELECT 
	title,
	duration
FROM netflix
WHERE 
	type = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1)::numeric > 5
GROUP BY 1, 2
```
### 9. Count the number of content items in each genre
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(show_id) as total_content
FROM netflix
GROUP BY 1
```

### 10. Find each year and the average numbers of content release in India on netflix. return top 5 year with highest avg content release!
```sql
 SELECT
 	EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) as year,
	 COUNT(*) as year_content,
	 ROUND(
	 COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country = 'India')::numeric * 100
	 ,2) as avg_content_per_year
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```

### 11. List all movies that are documentaries
```sql
SELECT 
	show_id,
	title,
	listed_in
FROM netflix
WHERE listed_in ILIKE '%documentaries%'
GROUP BY 1, 2, 3
```

### 12. Find all content without a director
```sql
SELECT *
FROM netflix 
WHERE director IS NULL
```

### 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```sql
SELECT *
FROM netflix 
WHERE 
	casts ILIKE '%Salman khan%'
	AND
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
```
### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```sql
SELECT 
	--show_id,
	--casts,
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
	COUNT(*) as total_content
FROM netflix
WHERE country ILIKE '%India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```

### 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.
```sql
WITH new_table 
AS(
SELECT 
	*,
	CASE
		WHEN description ILIKE '%kill%' OR
		description ILIKE '%violence%' THEN 'Bad Content'
		ELSE 'Good Content'
	END category
FROM netflix
)
SELECT 
category,
COUNT(*) as total_content
FROM new_table
GROUP BY 1
```














