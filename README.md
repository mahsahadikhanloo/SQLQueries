# SQL Covid Project

## Introduction

Analyzing COVID-19 Datasets with SQL Server: Gaining insights into the pandemic through data-driven queries. Data set is from [here](https://ourworldindata.org/covid-deaths) and check out the queries below. The purpose of this SQL Project is to analyze COVID-19 datasets related to deaths and vaccinations. The datasets contain information on the number of COVID-19 cases, deaths, vaccinations, the population of various countries, etc.

Loading the CovidDeaths dataset into SQL Server order by Location and Date:

```sql
select * from CovidDatabase..CovidDeaths
where continent is not null
order by 3,4
```

<image src="/resources/1.jpg"/>

Select the data that we are going to be using, again sort by Location and Date:

```sql
select location, date, total_cases, new_cases, total_deaths, population
from CovidDatabase..CovidDeaths order by 1,2
```
<image src="/resources/2.jpg"/>

Looking at the Total Cases vs Total Deaths and shows the likelihood of dying if you contract Covid in your country:

```sql
select location, date, total_cases, total_deaths, (total_deaths / total_cases)*100 as DeathPercantage
from CovidDatabase..CovidDeaths 
where location like '%states%'
order by 1,2
```

<image src="/resources/3.jpg"/>

Looking at the Total Cases vs Population and shows what percentage of population got Covid:

```sql
select Location, date, population, total_cases, (total_cases / population)*100 as InfectedPercantage
from CovidDatabase..CovidDeaths 
---where location like '%states%'
order by 1,2
```
<image src="/resources/4.jpg"/>

Looking at countries with Highest Infection Rate compared to Population:

```sql
select Location, Population,MAX(total_cases) as HighestInfection_Count,MAX((total_cases / population))*100
as InfectedPercantage
from CovidDatabase..CovidDeaths 
---where location like '%states%'
Group by Location, Population
order by InfectedPercantage desc
```
<image src="/resources/5.jpg"/>

Showing the Countries with Highest Death Count per Population:

```sql
select Location, MAX(cast(total_deaths as int)) as TotalDeath_Count
from CovidDatabase..CovidDeaths 
---where location like '%states%'
where continent is not null
Group by Location
order by TotalDeath_Count desc
```

<image src="/resources/6.jpg"/>

Let's break this down by continent:

```sql
select location, MAX(cast(total_deaths as int)) as TotalDeath_Count
from CovidDatabase..CovidDeaths 
---where location like '%states%'
where continent is null
Group by location
order by TotalDeath_Count desc
```

<image src="/resources/7.jpg"/>

Showing the continent with the Highest Death Count:

```sql
select continent, MAX(cast(total_deaths as int)) as TotalDeath_Count
from CovidDatabase..CovidDeaths 
---where location like '%states%'
where continent is not null
Group by continent
order by TotalDeath_Count desc
```

<image src="/resources/8.jpg"/>

Showing the Global Numbers, the Total Cases, Total Deaths and the Death Percentage group by Date:

```sql
select date, SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths,
SUM(cast(new_deaths as int))/SUM(new_cases)*100
as DeathPercantage
from CovidDatabase..CovidDeaths 
where continent is not null
Group by date
order by 1,2
```

<image src="/resources/9.jpg"/>

Total Cases across the World (same as above query without Group By):

```sql
select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths,
SUM(cast(new_deaths as int))/SUM(new_cases)*100
as DeathPercantage
from CovidDatabase..CovidDeaths 
where continent is not null
order by 1,2
```

Let's load the second table, CovidVaccination:

```sql
select * from CovidDatabase..CovidVaccination 
```

Join the two tables:

```sql
select *
from CovidDatabase..CovidDeaths dea
Join CovidDatabase..CovidVaccination vac
	on dea.location = vac.location
	and dea.date = vac.date
```

Looking at Total Population vs Vaccination, and using CTE:

```sql
With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeople_Vaccinated )
as
(
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int,vac.new_vaccinations)) over (partition by dea.location order by dea.location, dea.date)
as RollingPeople_Vaccinated 
from CovidDatabase..CovidDeaths dea
inner Join CovidDatabase..CovidVaccination vac
	on dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
---order by 2, 3
)

select * , (RollingPeople_Vaccinated / population)*100
from PopvsVac
```

Building the TEMP TABLE:

```sql
Drop Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_Vaccinations numeric,
RollingPeople_Vaccinated numeric
)
Insert into #PercentPopulationVaccinated
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int,vac.new_vaccinations)) over (partition by dea.location order by dea.location, dea.date)
as RollingPeople_Vaccinated 
from CovidDatabase..CovidDeaths dea
inner Join CovidDatabase..CovidVaccination vac
	on dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
---order by 2, 3
```

```sql
select * , (RollingPeople_Vaccinated / population)*100
from #PercentPopulationVaccinated
```

Creating View to store data for later visualizations:

```sql
Create View PercentPopulationVaccinated as
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int,vac.new_vaccinations)) over (partition by dea.location order by dea.location, dea.date)
as RollingPeople_Vaccinated 
from CovidDatabase..CovidDeaths dea
inner Join CovidDatabase..CovidVaccination vac
	on dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
---order by 2, 3

select * from PercentPopulationVaccinated
```








