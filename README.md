# COVID‑19 Data Exploration & SQL Analysis  
![Project Banner](assets/project-banner.png)

## Overview
This project performs an in‑depth analysis of global COVID‑19 data using SQL Server. Two Excel datasets (COVID deaths and COVID vaccinations) were imported into SQL Server, cleaned, joined, and analyzed using SQL queries. The goal is to understand infection trends, mortality rates, vaccination progress, and continent‑level patterns.

The project includes:
- Data import from Excel into SQL Server  
- Data cleaning and preparation  
- Table joins  
- Exploratory SQL analysis  
- Window functions  
- CTE‑based calculations  
- Temp table usage  
- View creation for BI visualization  

---

## Datasets

### **CovidDeaths**
Contains:
- Location  
- Continent  
- Date  
- Total cases  
- New cases  
- Total deaths  
- New deaths  
- Population  

### **CovidVaccinations**
Contains:
- Location  
- Date  
- New vaccinations  
- Total vaccinations  
- People vaccinated  
- People fully vaccinated  

Both datasets were imported into SQL Server using the SQL Server Import Wizard.

---

## Objectives
- Analyze COVID‑19 infection and death trends globally  
- Calculate mortality rates and infection percentages  
- Identify countries with the highest infection and death counts  
- Compare continent‑level death totals  
- Join deaths and vaccination datasets  
- Compute rolling vaccination totals using window functions  
- Create a SQL view for downstream visualization  

---

## SQL Analysis

### **1. Basic Data Exploration**
```sql
SELECT * 
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 3,4;
