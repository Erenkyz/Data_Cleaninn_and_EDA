Data Cleaning

First, I inspected the raw layoffs table to understand the structure of the dataset, column names, and data quality before performing any cleaning operations.

```sql
SELECT *
FROM layoffs;
```
<img width="1004" height="341" alt="image" src="https://github.com/user-attachments/assets/16c03b20-82b5-4832-84c3-f2104dc3c1fb" />






























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
