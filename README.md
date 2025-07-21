Select *
From PortfolioProject1..Vacinations
Where location = 'United States'
order by 3,4

--Select *
--From PortfolioProject1..Vacinations
--order by 3,4

--Select Location, date, total_cases, total_deaths, total_cases/total_deaths * 100 as DeathPercentage
--From PortfolioProject1..Deaths

-- Executing this query gave us a 'operand data type nvarchar is invalid for divide operator'
-- I then decided to convert the two columns into decimals and reapplied the operation

Select Location, date, total_cases, total_deaths,
CONVERT(DECIMAL(15, 3), total_cases) AS 'cases'
    ,CONVERT(DECIMAL(15, 3), total_deaths) AS 'deaths'
    ,CONVERT(DECIMAL(15, 3), (CONVERT(DECIMAL(15, 3), total_deaths) / CONVERT(DECIMAL(15, 3), total_cases)*100)) AS 'DeathPercentage'
from PortfolioProject1..Deaths
where location = 'United States'
order by 1,2

--Now I will show what percent of the US population had covid at any given date
Select Location, date, total_cases, Population,
CONVERT(DECIMAL(15, 3), total_cases) AS 'cases'
    ,CONVERT(DECIMAL(15, 3), population) AS 'pop'
    ,CONVERT(DECIMAL(15, 3), (CONVERT(DECIMAL(15, 3), total_cases) / CONVERT(DECIMAL(15, 3), population)*100)) AS 'CovidCasesByPopulation'
from PortfolioProject1..Deaths
where location = 'United States'
order by 1,2

--Find highest number of covid deaths just in the US


Select Location, Max(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProject1..Deaths
--where location = 'United States'
--where continent is not null
Group by location
order by TotalDeathCount desc

--Find total death count of North America

Select location, Max(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProject1..Deaths
where location = 'North America'
--where continent is null
Group by location
order by TotalDeathCount desc

-- Total Population vs New Vacinations by Date
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
From PortfolioProject1..Vacinations vac
Join PortfolioProject1..Deaths dea
	On dea.location = vac.location
	and dea.date = vac.date
Where dea.continent is not null
Order by 2,3

--total vacinations to date in the us

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
Sum(cast(vac.new_vaccinations as int)) over (Partition by dea.location Order by dea.location, dea.date) as vaccinations_to_date
From PortfolioProject1..Vacinations vac
Join PortfolioProject1..Deaths dea
	On dea.location = vac.location
	and dea.date = vac.date
Where dea.location = 'United States'


--Temp Table
Drop Table if exists #percentpopulationvaccinated
Create Table #percentpopulationvaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
new_vaccinations numeric,
vaccinations_to_date numeric)

Insert into #percentpopulationvaccinated
Select dea.continent, dea.location, dea.date, dea.population, new_vaccinations, 
Sum(cast(vac.new_vaccinations as int)) over (Partition by dea.location Order by dea.location, dea.date) as vaccinations_to_date
From PortfolioProject1..Vacinations vac
Join PortfolioProject1..Deaths dea
	On dea.location = vac.location
	and dea.date = vac.date
Where dea.location = 'United States'

Select *, (vaccinations_to_date/population)*100
from #percentpopulationvaccinated




Create View percentpopulationvaccinated as 
Select dea.continent, dea.location, dea.date, dea.population, new_vaccinations, 
Sum(cast(vac.new_vaccinations as int)) over (Partition by dea.location Order by dea.location, dea.date) as vaccinations_to_date
From PortfolioProject1..Vacinations vac
Join PortfolioProject1..Deaths dea
	On dea.location = vac.location
	and dea.date = vac.date
Where dea.location = 'United States'

Select *
From percentpopulationvaccinated
