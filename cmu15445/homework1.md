# homework1

`sudo rlwrap sqlite3 imdb-cmudb2022.db`

```sqlite
select premiered,primary_title || '(' || original_title || ')'
from titles
where type = 'movie' AND genres like '%Action%' AND original_title != primary_title
order by premiered DESC,primary_title
limit 10;
```





```sql
select distinct(title), ifnull(titles.ended,2023)-premiered AS years_running
from titles, akas
where akas.title_id = titles.title_id AND premiered is not null AND type = 'tvSeries'
order by years_running DESC, title
limit 20;
```



```sqlite
select CAST(born / 10 * 10 AS text) || 's' as decade, count(*)
from crew INNER JOIN people on crew.person_id = people.person_id
where category = 'director' AND born is not null
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
	select crew.person_id, name
    from crew INNER JOIN people on people.person_id = crew.person_id
    where category = 'actor' AND characters like '%Batman%'
    group by crew.person_id
)
select name, round(avg(rating), 2) as average
from crew
INNER JOIN batman_list
on batman_list.person_id = crew.person_id
INNER JOIN ratings
on ratings.title_id = crew.title_id
group by name
order by average DESC
limit 10;
```



