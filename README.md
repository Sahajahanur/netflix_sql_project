# Netflix Movies & TV Shows — SQL Analysis 🎬

<img width="1386" height="581" alt="image" src="https://github.com/user-attachments/assets/a0766e23-04bd-4829-9098-da36b0f380f6" />


A comprehensive SQL project analyzing Netflix content data to extract business insights around content distribution, ratings, genres, and regional trends.

---

## 📋 Project Overview

| Detail | Info |
|--------|------|
| **Title** | Netflix Movies and TV Shows Data Analysis |
| **Level** | Beginner–Intermediate |
| **Database** | PostgreSQL |
| **Dataset** | [Kaggle — Netflix Movies Dataset](https://www.kaggle.com/) |

---

## 🎯 Objectives

- Analyze the distribution of content types (Movies vs TV Shows)
- Identify the most common ratings for each content type
- List and analyze content by release year, country, and duration
- Explore and categorize content based on specific criteria and keywords

---

## 🗂️ Schema

```sql
DROP TABLE IF EXISTS netflix;

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

---

## 📊 Business Problems & Solutions

### Q1. Count of Movies vs TV Shows
```sql
SELECT
    type,
    COUNT(*) AS total
FROM netflix
GROUP BY type;
```
> Determine the distribution of content types on Netflix.

---

### Q2. Most Common Rating for Movies and TV Shows
```sql
WITH RatingCounts AS (
    SELECT
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```
> Identify the most frequently occurring rating for each content type.

---

### Q3. All Movies Released in a Specific Year
```sql
SELECT *
FROM netflix
WHERE release_year = 2020;
```
> Retrieve all movies released in a specific year.

---

### Q4. Top 5 Countries with Most Content
```sql
SELECT *
FROM (
    SELECT
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```
> Identify the top 5 countries with the highest number of content items.

---

### Q5. Longest Movie on Netflix
```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```
> Find the movie with the longest duration.

---

### Q6. Content Added in the Last 5 Years
```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```
> Retrieve content added to Netflix in the last 5 years.

---

### Q7. All Movies/TV Shows by Director 'Rajiv Chilaka'
```sql
SELECT *
FROM (
    SELECT
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```
> List all content directed by a specific director.

---

### Q8. TV Shows with More Than 5 Seasons
```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```
> Identify TV shows with more than 5 seasons.

---

### Q9. Content Count by Genre
```sql
SELECT
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```
> Count the number of content items in each genre.

---

### Q10. Top 5 Years — Average Content Releases by India
```sql
SELECT
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```
> Calculate and rank years by average content releases from India.

---

### Q11. All Documentary Movies
```sql
SELECT *
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```
> Retrieve all movies classified as documentaries.

---

### Q12. Content Without a Director
```sql
SELECT *
FROM netflix
WHERE director IS NULL;
```
> List content that does not have a director.

---

### Q13. Salman Khan Movies in the Last 10 Years
```sql
SELECT *
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```
> Count movies featuring a specific actor in the last 10 years.

---

### Q14. Top 10 Actors in Indian-Produced Movies
```sql
SELECT
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*) AS appearances
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY appearances DESC
LIMIT 10;
```
> Identify the top 10 actors with the most appearances in Indian content.

---

### Q15. Content Categorized by 'Kill' or 'Violence' Keywords
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
> Categorize content as 'Bad' (contains kill/violence) or 'Good' and count each.

---

## 📌 Findings & Conclusion

| Area | Insight |
|------|---------|
| **Content Distribution** | Diverse range of movies and TV shows across ratings and genres |
| **Common Ratings** | Most common ratings reveal target audience for each content type |
| **Geographical Insights** | Top countries and India's avg releases highlight regional trends |
| **Content Categorization** | Keyword-based categorization reveals nature of available content |

This analysis provides a comprehensive view of Netflix's content library and can help inform content strategy and decision-making.

---

## 🛠️ Tech Stack

- **Database:** PostgreSQL
- **Language:** SQL
- **Concepts:** CTEs, Window Functions, UNNEST, STRING_TO_ARRAY, SPLIT_PART, ILIKE, Date Functions, Aggregations

---

## 🚀 How to Use

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   ```

2. **Create the table** — Run the schema SQL to set up the `netflix` table

3. **Load the dataset** — Import the Kaggle CSV into the table

4. **Run the queries** — Execute any of the 15 business queries above

5. **Explore** — Modify queries to uncover additional insights

---

## 📁 Project Structure

```
netflix-sql-analysis/
│
├── schema.sql          # Table creation script
├── analysis.sql        # All 15 business queries
├── dataset/            # Raw CSV data (from Kaggle)
└── README.md           # Project documentation
```
