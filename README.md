Data Cleaning

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





🧹 Layoffs Dataset — Data Cleaning Project

This project focuses on cleaning and preparing a layoffs dataset using SQL. The goal is to transform raw data into a clean, structured table suitable for analysis and reporting. The dataset contains information about layoffs across companies, industries, and countries.

🗂 Project Overview

The cleaning process includes:

Removing Duplicates

Standardizing Data Formats

Handling Null or Missing Values

Dropping Irrelevant / Unnecessary Records

🔧 Technologies Used

MySQL

SQL Window Functions (ROW_NUMBER)

Data manipulation queries (UPDATE, DELETE, JOIN, ALTER, TRIM etc.)

📁 Output

A cleaned and structured dataset:

layoffs_staging2


Ready for EDA, visualization, and modeling.


📊 Layoffs Dataset — Exploratory Data Analysis (EDA)

This SQL-based EDA project explores global layoffs trends using aggregations, rankings, pattern identification, and cumulative analysis. The cleaned dataset from the Data Cleaning project serves as the foundation for this analysis.

🎯 Objectives

Identify layoff trends over time

Compare layoffs across industries, companies, and countries

Find most affected organizations

Track layoffs monthly using a rolling cumulative view

🛠 Technologies Used

MySQL

Grouping & Aggregations (SUM, MIN, MAX)

Window Functions (DENSE_RANK, OVER())

CTEs for rolling monthly analysis

📈 Results You Can Derive

Major industries hit hardest by layoffs

Which years had the highest layoffs

Rising or declining layoff patterns

Top companies contributing to workforce reduction yearly
