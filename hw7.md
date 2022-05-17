# ДЗ 7

## DML: агрегация и сортировка, CTE, аналитические функции 

1. Общая сумма очков по годам с помощью group_by
```sql 
select year_game, sum(points) from statistic
group by year_game
order by year_game;
```
![image](https://user-images.githubusercontent.com/41448520/168896943-38bf4b0c-3d29-40cf-943c-d762f19000c0.png)

2. Общая сумма очков по годам и сравнение с предыдущим годом с помощью CTE и LAG
```sql 
with year_points_sum_cte as (
       select year_game,
              sum(points) as year_points_sum
       from statistic
       group by year_game
       order by year_game
)
select cte.year_game,
       cte.year_points_sum,
       lag(cte.year_points_sum, 1) over (order by year_game) as prev_year_sum
from year_points_sum_cte cte;
```
![image](https://user-images.githubusercontent.com/41448520/168897129-6043f0c9-3d17-4867-b9b2-533ec29b1108.png)

3. Добавление к выборке суммы очков по данному году и вклад игрока в процентах с помощью оконной фукнции
```sql 
select s.*,
       sum(points) over (partition by year_game) as year_points_sum,
       floor(s.points*100/sum(points) over (partition by year_game)) as points_percent_contribution
from statistic s
order by year_game;
```
![image](https://user-images.githubusercontent.com/41448520/168897251-a37ebc65-9322-45fb-a4a6-f69c7a14e278.png)

4. Добавление к выборке суммы очков по данному году и вклад игрока в процентах с помощью CTE
```sql 
with year_points_sum_cte as (
       select year_game,
              sum(points) as year_points_sum
       from statistic
       group by year_game
       order by year_game
)
select s.*, y.year_points_sum, floor(s.points*100/y.year_points_sum) as points_percent_contribution
from statistic s
join year_points_sum_cte y on s.year_game = y.year_game
order by s.year_game;
```
![image](https://user-images.githubusercontent.com/41448520/168897395-98c1f006-6d56-47c8-b0c1-ea98bdb388f0.png)
