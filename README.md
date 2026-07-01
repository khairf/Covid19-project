# COVID‑19 Data Exploration & SQL Analysis  
![Project Banner](https://github.com/khairf/Covid19-project/blob/main/covid-cells%20logo.jpg)

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

## How to Run
1. Clone the repository
2. Import both Excel files into SQL Server
3. Run the SQL Scripts in the /sql folder
4. Explore the results and modify queries as needed

---
## SQL Analysis
### **1. Basic Data Exploration**
```sql
SELECT * 
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 3,4;
```

### **2. Selecting Relevant Columns**
```sql
SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1,2;

```
### Infection and Mortality Analysis
### **3. Likelihood of dying if infected in case of the states**
```sql
SELECT Location, date, total_cases, total_deaths,
       (total_deaths/total_cases)*100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE location LIKE '%states%' AND continent IS NOT NULL
ORDER BY 1,2;

```

### **4. Percentage of population infected**
```sql
SELECT Location, date, population, total_cases,
       (total_cases/population)*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
WHERE location LIKE '%states%'
ORDER BY 1,2;

```

### **5. Countries with highest infection rate**
```sql
SELECT Location, Population, MAX(total_cases) AS HighestInfectionCount,
       MAX(total_cases/population)*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
GROUP BY Location, Population
ORDER BY PercentPopulationInfected DESC;

```

### **6. Countries with highest death count**
```sql
SELECT Location, MAX(CAST(total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY Location
ORDER BY TotalDeathCount DESC;

```

### **7. Continent level analysis**
```sql
SELECT continent, MAX(CAST(total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY TotalDeathCount DESC;

```

### **8. Global numbers**
```sql
SELECT SUM(new_cases) AS total_cases,
       SUM(CAST(new_deaths AS int)) AS total_deaths,
       SUM(CAST(new_deaths AS int))/SUM(new_cases)*100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL;

```

### **9. Joining Deaths and Vaccinations Tables**
```sql
SELECT *
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
     ON dea.location = vac.location
    AND dea.date = vac.date;

```

### **10. Rolling Vaccination Totals**
```sql
SELECT dea.continent, dea.location, dea.date, dea.population,
       vac.new_vaccinations,
       SUM(CONVERT(int, vac.new_vaccinations)) OVER
           (PARTITION BY dea.location ORDER BY dea.location, dea.date)
           AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
     ON dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
ORDER BY 2,3;

```

### **11.CTE for Vaccination Percentage**
```sql
WITH PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
AS (
    SELECT dea.continent, dea.location, dea.date, dea.population,
           vac.new_vaccinations,
           SUM(CONVERT(int, vac.new_vaccinations)) OVER
               (PARTITION BY dea.location ORDER BY dea.location, dea.date)
               AS RollingPeopleVaccinated
    FROM PortfolioProject..CovidDeaths dea
    JOIN PortfolioProject..CovidVaccinations vac
         ON dea.location = vac.location
        AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL
)
SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentVaccinated
FROM PopvsVac;

```
### **12. Temp Table Method**
```sql
DROP TABLE IF EXISTS #PercentPopulationVaccinated;

CREATE TABLE #PercentPopulationVaccinated (
    Continent nvarchar(255),
    Location nvarchar(255),
    Date datetime,
    Population numeric,
    New_vaccinations numeric,
    RollingPeopleVaccinated numeric
);

INSERT INTO #PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population,
       vac.new_vaccinations,
       SUM(CONVERT(int, vac.new_vaccinations)) OVER
           (PARTITION BY dea.location ORDER BY dea.location, dea.date)
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
     ON dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;

SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentVaccinated
FROM #PercentPopulationVaccinated;


```

### **13. View Creation**
```sql
CREATE VIEW PercentPopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population,
       vac.new_vaccinations,
       SUM(CONVERT(int, vac.new_vaccinations)) OVER
           (PARTITION BY dea.location ORDER BY dea.location, dea.date)
           AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
     ON dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;


```

