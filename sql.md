```sql
--Tips : 1. In having clause, we can not use aliases. 2. In Union, pay attention that we should use those two sub queries. it is better to not union there queries having Order By CLAUSE.
--how the data looks ike
select * from dbo.Data1

select * from dbo.Data2

--Number of rows in each dataset
select count(*) from dbo.Data1
select count(*) from dbo.Data2

--retrieve data for the state Jharkhand and Bihar
select * from dbo.Data1 where State='Jharkhand' or State='Bihar'
-- alternative: select * from dbo.Data1 where State in ('Jharkhand','Bihar')

--Total popuation of India
select sum(Population) as "Population" from dbo.Data2

--average sex ratio in each state
select State, round(AVG(Sex_Ratio),0) as "avg of Sex Ratio" from dbo.Data1 Group By State order by [avg of Sex Ratio] desc

--average growth for each State
select State, AVG(Growth)*100 as "Growth Average" from dbo.Data1
Group by State 

--average litercy rate higher than 90/100 for each state 
select State,round(avg(Literacy),0) as "avg of literacy" from dbo.Data1 group by State 
having round(avg(Literacy),0)>90
order by [avg of literacy] desc

--Top 3 state with highest average growth rate
select top 3 state, AVG(growth)*100 as "Growth Average" from dbo.Data1 group by State order by [Growth Average] desc 

--Bottom 3 state with lowest sex ratio
select top 3 state, round(avg(sex_ratio),0) as "avg of sex ratio" from dbo.Data1 group by State order by [avg of sex ratio] 

--Bottom and top 3 state in  literacy rate (join two different result with union )
select * from (select top 3 state,avg(literacy) as "avg of literacy" from dbo.Data1 group by State order by [avg of literacy] desc) as a
union
select * from (select top 3 state,avg(literacy) as "avg of literacy" from dbo.Data1 group by State order by [avg of literacy] ) as b
order by [avg of literacy] desc

--state starting with letter a
select distinct(State) from dbo.Data1
where State like 'a%'

--state starting with letter a or b
select distinct(state) from dbo.Data1 
where State like 'a%' or state like 'b%'

---state starting with letter a and ending with letter b
select distinct(state) from dbo.Data1 where State like 'a%' and state like '%m'

--number of males and females in a state
select c.district, c.state, c.population, c.sex_ratio, round(((c.Sex_Ratio*c.Population)/(1+c.sex_ratio)),0) as Female, round((c.population/(1+c.sex_ratio)),0) as Male from
(select d1.district, d1.state, d1.Sex_ratio/1000 as sex_ratio, d2.population
from dbo.Data1 as d1
inner join dbo.Data2 as d2
on d1.district=d2.District) c

--- number of male and females in each state
select d.state, sum(d.female) as "Total Female", sum(d.male) as "Total male" from
(select c.district, c.state, round(((c.Sex_Ratio*c.Population)/(1+c.sex_ratio)),0) as Female, round((c.population/(1+c.sex_ratio)),0) as Male from
(select d1.district, d1.state, d1.sex_ratio/1000 as sex_ratio , d2.population
from dbo.Data1 as d1
inner join dbo.Data2 as d2 on d1.district=d2.district) c) d
group by d.state

--total literate people in each district
select t.district, t.state, t.population, t.literacy, round((t.population*(t.literacy/100)),0) as "number of literate" from 
(select d1.district, d1.state, d1.Literacy, d2.population
from dbo.Data1 as d1
inner join dbo.Data2 as d2 on d1.district=d2.district) as t

--total literate people for each state
select f.state, sum(f.number_of_literate) as "number of literate people" from
(select t.district, t.state, t.population, t.literacy, round((t.population*(t.literacy/100)),0) as "number_of_literate" from 
(select d1.district, d1.state, d1.Literacy, d2.population
from dbo.Data1 as d1
inner join dbo.Data2 as d2 on d1.district=d2.district) as t) as f
group by f.state

--population in previous census for each state
select c.state, sum(previous_population) as Previous_population, sum(current_population) as Current_population from
(select t.district, t.state, t.growth, t.population as "current_population", round(t.population/(t.growth+1),0) as "previous_population" from
(select d1.District, d1.state, d1.growth, d2.population 
from dbo.Data1 as d1
inner join dbo.Data2 as d2 on d1.District=d2.District) as t) as c
group by c.state


--total of population in previous census for each state
select sum(m.Previous_population) Previous_population, sum(m.Current_population) Current_population from
(select c.state, sum(previous_population) as Previous_population, sum(current_population) as Current_population from
(select t.district, t.state, t.growth, t.population as "current_population", round(t.population/(t.growth+1),0) as "previous_population" from
(select d1.District, d1.state, d1.growth, d2.population 
from dbo.Data1 as d1
inner join dbo.Data2 as d2 on d1.District=d2.District) as t) as c
group by c.state)  m

--average growth of population for each state
select state, avg(growth) from dbo.Data1
group by state

--how much area per population has been reduced from the current census as compare to the previous census for each district
select w.district, w.state, w.current_population, w.previous_population, (w.area_perperson_current-w.area_perperson_previous)/w.area_perperson_previous as "reduction of area per person" from 
(select q.district, q.state, q.current_population, (q.area_km2/q.current_population) as "area_perperson_current", q.previous_population, (q.Area_km2/q.previous_population) as "area_perperson_previous" from
(select t.district, t.state, t.growth, t.Area_km2, t.population as "current_population", round(t.population/(t.growth+1),0) as "previous_population" from
(select d1.District, d1.state, d1.growth, d2.Area_km2, d2.population 
from dbo.Data1 as d1
inner join dbo.Data2 as d2 on d1.District=d2.District) as t) as q) w

--how much area per population has been reduced from the total current census as compare to the total previous census
select ((k.india_area/k.current_population)-(k.india_area/k.previous_population))/(k.india_area/k.previous_population) as "area_reduction_per_person" from
(select o.india_area, e.previous_population, e.current_population from
(select '1' as keyy , n.* from
(select sum(m.Previous_population) Previous_population, sum(m.Current_population) Current_population from
(select c.state, sum(previous_population) as Previous_population, sum(current_population) as Current_population from
(select t.district, t.state, t.growth, t.population as "current_population", round(t.population/(t.growth+1),0) as "previous_population" from
(select d1.District, d1.state, d1.growth, d2.population 
from dbo.Data1 as d1
inner join dbo.Data2 as d2 on d1.District=d2.District) as t) as c
group by c.state)  m ) as n) as e inner join
--total area of India
(select '1' as keyy, s.* from
(select sum(area_km2) as "India_area"  from dbo.Data2) s) as o on o.keyy=e.keyy) k


--top 3 districts from each state which has the highest litracy rate
select w.* from
(select d1.*,
rank() over(partition by state order by literacy desc) as rank_literacy
from dbo.Data1 as d1) as w
where w.rank_literacy<4 ```
