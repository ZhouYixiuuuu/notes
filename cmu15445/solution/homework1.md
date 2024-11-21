# homework1

`sudo rlwrap sqlite3 imdb-cmudb2022.db`

字符串拼接

```sqlite
select premiered,primary_title || ' (' || original_title || ')'
from titles
where type = 'movie' AND genres like '%Action%' AND original_title != primary_title
order by premiered DESC,primary_title
limit 10;
```





```sql
--这是错误的代码
select distinct(title), ifnull(titles.ended,2023)-premiered AS years_running
from titles, akas
where akas.title_id = titles.title_id AND premiered is not null AND type = 'tvSeries'
order by years_running DESC, title
limit 20;
```



```sql
with tvSeries_list AS (
	select primary_title, ifnull(ended, 2023) AS ended_new, premiered
    from titles
    where type = 'tvSeries' AND premiered is not null
)
select primary_title, tvSeries_list.ended_new - premiered AS years_running
from tvSeries_list 
order by years_running DESC, primary_title
limit 20;
```



q4

```sqlite
select CAST(born / 10 * 10 AS text) || 's' as decade, count(distinct(people.person_id))
from crew INNER JOIN people on crew.person_id = people.person_id
where category = 'director' AND born is not null AND born >= 1900
group by decade
order by decade;
```



q5

```sqlite
select type, round(avg(rating),2) as average, min(rating), max(rating)
from akas, titles, ratings 
where akas.title_id = titles.title_id AND ratings.title_id = titles.title_id AND language = 'de' AND (types = 'imdbDisplay' OR types = 'original')
group by type
order by average;
```



q6

```sqlite
with batman_list (person_id,name) AS (
	select distinct(people.person_id), name
    from crew INNER JOIN people on people.person_id = crew.person_id
    where category = 'actor' AND characters like '%Batman%'
)
select name, round(avg(rating), 2) as average
from crew
INNER JOIN batman_list
on batman_list.person_id = crew.person_id
INNER JOIN ratings
on ratings.title_id = crew.title_id
group by name
order by average DESC, name
limit 10;
```



q7

```sql
select count(distinct(people.person_id))
from people 
INNER JOIN crew
on people.person_id = crew.person_id
where (crew.category = 'actor' OR crew.category = 'actress')
AND born = (select premiered
from titles
where primary_title = 'The Prestige' AND premiered is not null);
```

q8

```sql
--姓Rose的actress呆过的crew编号
with Rose_actress AS (
	select distinct(crew.title_id)
    from crew INNER JOIN people
    on crew.person_id = people.person_id
    where category = 'actress' AND people.name like 'Rose%'
)

select distinct(people.name) AS n
from crew INNER JOIN people
on crew.person_id = people.person_id
where category = 'director' AND title_id in Rose_actress
order by n;
```

q9

```sql
--先找出每一个crew最长runting_minutes的作品
with table1 (name, category, died, person_id, runtime_minutes, primary_title, runtime_rank) AS (
	select 
        people.name,
        crew.category,
        people.died,
    	crew.person_id,
    	titles.runtime_minutes,
    	titles.primary_title,
        ROW_NUMBER() OVER (partition by crew.person_id order by titles.runtime_minutes DESC, titles.title_id ASC) AS runtime_rank
    from crew
    INNER JOIN people on people.person_id = crew.person_id
    INNER JOIN titles on titles.title_id = crew.title_id
    where died is not null AND runtime_minutes is not null
),
table2 AS (
	select name, category, died, person_id, runtime_minutes, primary_title,
	ROW_NUMBER() OVER (partition by category order by died, name) AS death_rank
    from table1
    where runtime_rank = 1
)
select category, name, died, primary_title, runtime_minutes, death_rank
from table2
where death_rank <= 5
order by category;
```

