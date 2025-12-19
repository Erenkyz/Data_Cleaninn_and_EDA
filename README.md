# SQL Data Cleaning & Exploratory Data Analysis Projects

# 🧠 Project Overview

 This repository contains two end-to-end SQL data analysis projects focusing on data cleaning and exploratory data analysis (EDA) using MySQL.

The main objectives of these projects are:

Cleaning raw, real-world datasets

Handling duplicates, missing values, and inconsistent data

Performing exploratory analysis to uncover meaningful insights

Demonstrating strong SQL skills for data analyst and data science roles

Both projects follow a structured analytical workflow, starting from raw data inspection and ending with insight-driven analysis.

# 🗂️ Projects Included

## 🔹 Project 1: SQL Data Cleaning

Focus:

Preparing raw data for analysis by improving data quality and consistency.

Key steps performed:

Creating staging tables to preserve raw data

Detecting and removing duplicate records using window functions

Standardizing text fields (company, industry, country)

Converting date columns to proper DATE format

Handling NULL and blank values

Removing irrelevant or unusable rows

 📌 Outcome:

A clean, structured dataset ready for exploratory analysis and reporting.


##🔹 Project 2: Exploratory Data Analysis (EDA)

Focus:

Analyzing cleaned data to identify trends, patterns, and business insights.

Key analyses performed:

Overall layoff statistics and summary metrics

Companies with the highest number of layoffs

Industries and countries most affected

Yearly and monthly layoff trends

Rolling (cumulative) totals over time

Ranking top companies by layoffs per year using window functions

 📌 Outcome:

Actionable insights that highlight how layoffs evolved across time, industries, and regions.

## 🛠️ Tools & Technologies

MySQL

MySQL Workbench

SQL features used:

CTEs (Common Table Expressions)

Window Functions (ROW_NUMBER, DENSE_RANK)

Aggregations (SUM, AVG, COUNT)

Date functions

Joins and data transformations

## 🔍 Methodology

Each project follows the same analytical pipeline:

Raw Data Inspection

Data Cleaning & Transformation

Validation of Cleaned Data

Exploratory Analysis

Trend Analysis & Ranking

Insight Extraction

All SQL queries are documented step by step, with:

Clear explanations

Well-structured SQL code blocks

Query output screenshots for clarity

## 📸 Visual Documentation

For better readability and understanding, query results are documented using:

Screenshot outputs from MySQL Workbench

Visual confirmation of transformations and analysis results

This approach makes the analysis easy to follow even for non-technical stakeholders.

## 📈 Key Skills Demonstrated

Advanced SQL querying

Data cleaning in relational databases

Exploratory data analysis

Analytical thinking and problem solving

Working with real-world, messy datasets

Writing clear and maintainable SQL code



## Data Cleaning

First, I inspected the raw layoffs table to understand the structure of the dataset, column names, and data quality before performing any cleaning operations.

```sql
SELECT *
FROM layoffs;
```

<img width="1004" height="341" alt="image" src="https://github.com/user-attachments/assets/16c03b20-82b5-4832-84c3-f2104dc3c1fb" />


To avoid modifying the original dataset, I created a staging table (layoffs_staging) that has the same structure as the raw table. This allows safe data cleaning and experimentation.

```sql
CREATE TABLE layoffs_staging
LIKE layoffs;
```
```sql
INSERT layoffs_staging
SELECT *
FROM layoffs;
```


I used the ROW_NUMBER() window function to identify potential duplicate rows based on company, industry, location, layoff numbers, date, stage, and funds raised.

```sql
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, 
total_laid_off, percentage_laid_off, 'date') AS ROW_NUM
FROM layoffs_staging;
```


<img width="1103" height="318" alt="image" src="https://github.com/user-attachments/assets/b378313e-8675-4f20-9125-b7154f3a285c" />


Using a Common Table Expression (CTE), I filtered rows where row_num > 1, which represent duplicate records.

```sql
WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, location, 
total_laid_off, percentage_laid_off, `date`, stage,
funds_raised_millions
) AS row_num
FROM layoffs_staging
)
SELECT * 
FROM duplicate_cte
WHERE row_num > 1;
```

<img width="1104" height="361" alt="image" src="https://github.com/user-attachments/assets/840e0d8f-ed08-4b56-bada-517357d960c9" />


Since MySQL does not allow deleting directly from a CTE, I created a second staging table and removed duplicates by keeping only rows where row_num = 1.

```sql
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

```
```sql
INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, location, 
total_laid_off, percentage_laid_off, 'date', stage
, country, funds_raised_millions) AS row_num
FROM layoffs_staging;
```
```sql
DELETE
FROM layoffs_staging2
WHERE row_num > 1;
```

<img width="1190" height="362" alt="image" src="https://github.com/user-attachments/assets/e55a5f43-9545-4b24-b3f2-9ebdf6d5e41c" />


I removed leading and trailing whitespace from company names to ensure consistency.

```sql
SELECT company, TRIM(company)
FROM layoffs_staging2;
```

<img width="263" height="327" alt="image" src="https://github.com/user-attachments/assets/8a8d3dad-cd1e-414d-a410-a5bb70b7ae36" />


Different variations of the same industry (e.g., "Crypto", "Crypto Currency") were standardized into a single category.

```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```

<img width="1119" height="318" alt="image" src="https://github.com/user-attachments/assets/a3274fc4-bb5c-44bb-95d6-9737a6860fbb" />


Trailing punctuation (such as periods) was removed from country names for consistency.


<img width="1185" height="362" alt="image" src="https://github.com/user-attachments/assets/5b0aad53-9114-420b-a69c-51dad6ea6937" />



The date column was originally stored as text. I converted it into a proper DATE data type for time-based analysis.

```sql
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
```

```sql
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```



<img width="123" height="336" alt="image" src="https://github.com/user-attachments/assets/877d8e3c-5bac-49a5-901d-a9d101187c81" />



Blank industry values were converted to NULL, and missing industry values were filled using data from other rows with the same company name.

```sql
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';
```

```sql

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
  ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;
```

<img width="1154" height="306" alt="image" src="https://github.com/user-attachments/assets/e0e95001-75cb-4a1b-bfb7-42b2aa98c27e" />

Rows where both total_laid_off and percentage_laid_off were NULL provided no analytical value and were removed.

```sql
DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

<img width="1102" height="340" alt="image" src="https://github.com/user-attachments/assets/7b964c1b-2515-47c3-ad7a-2fb87d99c91a" />


## Exploratory_Data_Analysis

This query displays all records from the cleaned staging table to verify the final dataset before analysis.

```sql
SELECT *
FROM layoffs_staging2;
```

<img width="1105" height="340" alt="image" src="https://github.com/user-attachments/assets/5dfd751c-e742-4852-bc50-a4b9d46e8936" />


I calculated the maximum number of layoffs and minimum layoff percentage to understand the range of the data.

```sql
SELECT MAX(total_laid_off), MIN(percentage_laid_off)
FROM layoffs_staging2;
```

<img width="355" height="53" alt="image" src="https://github.com/user-attachments/assets/c7a18db6-69ed-43b9-9395-441343e926dd" />

This query identifies companies where the entire workforce was laid off, sorted by funds raised.

```sql
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;
```
<img width="1153" height="342" alt="image" src="https://github.com/user-attachments/assets/a6c2735b-9db8-4b87-905e-0c0bcf9195fc" />

Aggregated total layoffs per company to identify organizations with the highest impact.

```sql
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
```

<img width="1159" height="324" alt="image" src="https://github.com/user-attachments/assets/b4314667-51b6-47d4-9abb-c09ac6a4258b" />

This query identifies the time range covered by the dataset.

```sql
SELECT MIN(`date`), MAX(`date`)
FROM layoffs_staging2;
```

<img width="226" height="50" alt="image" src="https://github.com/user-attachments/assets/5eab2fca-abc4-442e-8bee-27341bc340e5" />

These queries show which industries and countries were most affected by layoffs.

```sql
SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;
```

<img width="273" height="342" alt="image" src="https://github.com/user-attachments/assets/d442fcac-a46e-4a6a-bb39-bb33f05c9960" />

```sql
SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;
```

<img width="352" height="346" alt="image" src="https://github.com/user-attachments/assets/c226f495-8e59-4137-a9d7-6c90e72b7ff5" />

I analyzed layoffs by year and month, including a rolling cumulative total to observe long-term trends.

```sql

SELECT YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY YEAR(`date`)
ORDER BY 1 DESC;
```

```sql
WITH Rolling_Total AS
 (
 SELECT SUBSTRING(`date`,1,7) AS `MONTH`, SUM(total_laid_off) AS total_off
 FROM layoffs_staging2
 WHERE SUBSTRING(`date`,1,7) IS NOT NULL
 GROUP BY `MONTH`
 ORDER BY 1 ASC
 )
 SELECT `MONTH`, total_off
 ,SUM(total_off) OVER(ORDER BY `MONTH`) AS Rolling_Total
 FROM Rolling_Total;
```

<img width="279" height="131" alt="image" src="https://github.com/user-attachments/assets/ecad81eb-f140-48b2-abe8-e6e672a7cb88" />


<img width="269" height="349" alt="image" src="https://github.com/user-attachments/assets/95840ef2-c0f1-4c60-8135-d6e15f82903b" />

Using window functions, I ranked the top 5 companies with the highest layoffs for each year.

```sql
WITH company_year (company, years, total_laid_off) AS
(
SELECT company, YEAR (`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company, YEAR (`date`)
), Company_Year_Rank AS
(SELECT *, 
DENSE_RANK() OVER (partition by years ORDER BY total_laid_off DESC) AS Ranking
FROM company_year
WHERE years IS NOT NULL
)
SELECT *
FROM Company_Year_Rank
WHERE Ranking <= 5 
;
```
<img width="352" height="345" alt="image" src="https://github.com/user-attachments/assets/13c42242-2819-4da8-959b-6b5b51623eca" />










