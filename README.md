IPL_Data_Analysis
Utilized SQL to analyze IPL matches data set, focusing on player’s performances.

-- 1. We will create database schema to load the tables

CREATE DATABASE ipl;

-- 2. Create table to store data

CREATE TABLE ipl.matches( id integer, season integer, city varchar(20), date date, team1 varchar(20), team2 varchar(20), toss_winner varchar(20), toss_decision varchar(20), result varchar(50), dl_applied integer, winner varchar(50), win_by_runs integer, win_by_wickets integer, player_of_the_match varchar(50), venue varchar(20), umpire1 varchar(20), umpire2 varchar(20), umpire3 integer);

CREATE TABLE ipl.deliveries (matchid integer, inning integer, batting_team varchar(20), bowling_team varchar(20), overc integer, ball integer, batsman varchar(20), non_striker varchar(20), bowler varchar(20), is_super_over integer, wide_runs integer, bye_runs integer, legbye_runs integer, noball_runs integer, penalty_runs integer, batsman_runs integer, extra_runs integer, total_runs integer, player_dismissed varchar(20), dismissal_kind varchar(50), fielder varchar(20));

-- 3. Check the tables structure

-- 4. Now, Load the data in tables

SHOW GLOBAL VARIABLES LIKE 'local_infile';

-- SET GLOBAL local_infile = 'ON';
SET GLOBAL local_infile=1;

Now, Begin with the queries:
Shape of Data
-- Shapes are combination of rows and columns -- To know about your table structure (I mean rows and columns) use below query.

select count(*) as No_of_row from ipl.matches;

select count(*) as No_of_columns from information_schema.columns where TABLE_NAME='matches';

Viewing data
select * from ipl.matches;

Select * from ipl.deliveries;

View selected columns
select m.season,m.city,m.date,m.team1,m.team2,m.winner,m.win_by_runs from ipl.matches m where m.season='IPL-2017' limit 5;

Distinct values
select distinct year(m.date) as 'Year of Match' from ipl.matches m order by 1;

select count(distinct player_of_match) as 'No of Matches' from ipl.matches m order by 1;

Find season winner for each season (season winner is the winner of the last match of each season)
select distinctrow m.season, m.winner from ipl.matches m order by m.season desc;

select distinct season, winner from ipl.matches order by season desc;

Find venue of 10 most recently played matches
select DISTINCT m.venue,m.date from ipl.matches m order by m.date desc limit 10;

Case (4,6, single,0)
select distinct batsman,bowler,ball, CASE when total_runs=1 then 'Single' when total_runs=4 then 'Boundry' when total_runs=6 then 'Six' else 'Duck' end as 'Run in words' from ipl.deliveries ;

Data Aggregation
select winner,win_by_wickets,max(win_by_runs) from ipl.matches group by winner order by 3 desc;

How many extra runs have been conceded in ipl
select distinct bowler,sum(extra_runs) from ipl.deliveries group by bowler having sum(extra_runs)>0;

On an average, teams won by how many runs in ipl
select winner,avg(win_by_runs) from ipl.matches group by winner having avg(win_by_runs)>0 order by 2 desc;

How many extra runs were conceded in ipl by SK Warne
select bowler,sum(extra_runs) from ipl.deliveries where bowler='SK Warne' group by bowler;

How many boundaries (4s or 6s) have been hit in ipl
select m.winner, d.total_runs,count(d.total_runs) from ipl.deliveries d inner join ipl.matches m on m.id=d.matchid where d.total_runs in (4,6) and m.winner='Mumbai Indians' group by m.winner, d.total_runs;

How many balls did SK Warne bowl to batsman SR Tendulkar
select batsman,bowler, count(ball) from ipl.deliveries where bowler='TS Mills' and batsman='DA Warner' group by batsman,bowler;

How many matches were played in the month of April
select count(*) from ipl.matches where month(date)='4';

select count(*) from ipl.matches where extract(month from date)=4;

How many matches were played in the March and June
select count(*) from ipl.matches where month(date) in ('3','6');

Total number of wickets taken in ipl (count not null values)
select count(player_dismissed) as 'Wicket' from ipl.deliveries where player_dismissed <>"";

select dismissal_kind,player_dismissed, count(*) from ipl.deliveries where player_dismissed <>"" group by dismissal_kind,player_dismissed order by 3 desc;

Pattern Match ( Like operators % _ )
select Distinct player_of_match from ipl.matches where player_of_match Like '%M%';

select Distinct player_of_match from ipl.matches where player_of_match like 'JJ %';

select distinct player_of_match from ipl.matches where player_of_match like 'K_ P%';

How many teams have word royal in it (could be anywhere in the team name, any case)
SELECT distinct team1 FROM ipl.matches where lower(team1) like lower('%Royal%');

Group by - Maximum runs by which any team won a match per season
select season,max(win_by_runs) from ipl.matches group by season order by 1;

Create score card for each match Id
select batting_team,batsman,sum(batsman_runs) from ipl.deliveries group by batting_team,batsman order by 3 desc;

Top 10 players with max boundaries (4 or 6)
select DISTINCT batsman,count(total_runs) from ipl.deliveries where total_runs in (4,6) group by batsman order by 2 desc limit 10;

Top 20 bowlers who conceded highest extra runs
select bowler,sum(extra_runs) as 'highest extra runs' from ipl.deliveries group by bowler order by 2 desc limit 20;

Top 10 wicket takers
select bowler,count(player_dismissed) as NoWicket_Taken,dismissal_kind from ipl.deliveries where dismissal_kind <>"" group by bowler order by NoWicket_Taken desc limit 10;

Name and number of wickets by bowlers who have taken more than or equal to 100 wickets in ipl
select bowler,count(player_dismissed) as NoWicket_Taken,dismissal_kind from ipl.deliveries where dismissal_kind <>"" group by bowler having count(player_dismissed) >=100 order by NoWicket_Taken desc limit 10;

Top 2 player_of_the_match for each season
select season, player_of_match, CNT from ( select row_number() over (partition by season) as rn,season,player_of_match,Cnt from ( select season,player_of_match, count(player_of_match) as Cnt from ipl.matches group by season,player_of_match order by 1 asc,3 desc ) rw ) Temp where Temp.rn<3;

Window Functions - (CTE) -- Combine column date from matches with table
-- deliveries to get data by year -- what is CTE ? -- A common table expression (CTE) is a named temporary result set that exists within the scope of a
-- single statement and that can be referred to later within that statement, possibly multiple times.

WITH t1 as (select id,season,date,city,team1,team2,winner from ipl.matches), t2 as (select matchid,batting_team,bowling_team from ipl.deliveries) select distinct t1.season,t1.date,t1.city,t1.team1,t1.team2,t2.batting_team,t2.bowling_team, t1.winner from t1 inner join t2 on t1.id=t2.matchid;
