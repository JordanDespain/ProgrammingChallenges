# SQL Challenge

The database contains two tables, store_revenue and marketing_data.  Refer to the two CSV
files, store_revenue and marketing_data to understand how these tables have been created.

store_revenue contains revenue by date, brand ID, and location:

 >  create table store_revenue (
 >     id int not null primary key auto_increment,
 >    date datetime,
 >    brand_id int,
 >    store_location varchar(250),
 >    revenue float  
 >  );

marketing_data contains ad impression and click data by date and location:

> create table marketing_data (
>  id int not null primary key auto_increment,
>  date datetime,
>  geo varchar(2),
>  impressions float,
>  clicks float
> );

### Please provide a SQL statement under each question.

* Question #0 (Already done for you as an example)
 Select the first 2 rows from the marketing data
​
>  select *
>  from marketing_data
> limit 2;
​
*  Question #1 
Generate a query to get the sum of the clicks of the marketing data

>  select sum(clicks) as sum_clicks
>  from marketing_data;

*  Question #2
 Generate a query to gather the sum of revenue by store_location from the store_revenue table
​
>  select store_location, sum(revenue) as total_revenue
>  from store_revenue
>  group by store_location;
​
*  Question #3
 Merge these two datasets so we can see impressions, clicks, and revenue together by date
and geo.
 Please ensure all records from each table are accounted for.
​
>  with store_revenue_cte as (
>		  select date, brand_id, split_part(store_location, '-', 2) as geo, revenue
>		  from store_revenue),
>	 store_revenue_agg_cte as(
>		   select date, geo, sum(revenue) as revenue
>		   from store_revenue_cte
>		   group by date, geo),
>	 merged_table_cte as (
>		   select m.date, m.geo, impressions, clicks, revenue
>		   from marketing_data m
>		   full join store_revenue_agg_cte s
>		   on s.date = m.date and s.geo = m.geo),
>	missing_last_date_cte as (
>		  select *
>		  from store_revenue_agg_cte 
>		  union
>		  select date, geo, revenue
>		  from merged_table_cte
>		  where date is not null
>		  order by date)
>	select ml.date, ml.geo, mt.impressions, mt.clicks, ml.revenue
>	from missing_last_date_cte ml
>	left outer join merged_table_cte mt
>	on ml.date = mt.date and ml.geo = mt.geo;
​
* Question #4
 In your opinion, what is the most efficient store and why?

I think in regard to the data we have, I would consider an efficient store to be one that is able to successfully turn impressions into clicks and then those clicks into revenue. So, I summed impressions, clicks, and revenue for each location, and then I created a clicks to impressions ratio and a revenue to clicks ratio. The Minnesota store is definitely the best at turning impressions into clicks with a rate over three times more than the other stores. Unfortunately, we do not have the revenue data for the Minnesota store, so we cannot tell whether these clicks turned into revenue or not. The other three locations have similar clicks to impressions ratios, but the California store, by far, has the best revenue to clicks ratio from the group with available data. So, in my opinion, the California store is the most efficient store.

The query I used:

>  with store_revenue_cte as (
>		   select date, brand_id, split_part(store_location, '-', 2) as geo, revenue
>		   from store_revenue),
>	 store_revenue_agg_cte as(
>		   select date, geo, sum(revenue) as revenue
>		   from store_revenue_cte
>		   group by date, geo),
>	 merged_table_cte as (
>		   select m.date, m.geo, impressions, clicks, revenue
>		   from marketing_data m
>		   full join store_revenue_agg_cte s
>		   on s.date = m.date and s.geo = m.geo),
>	 missing_last_date_cte as (
>		   select *
>		   from store_revenue_agg_cte 
>		   union
>		   select date, geo, revenue
>		   from merged_table_cte
>		   where date is not null
>		   order by date)
>	 select ml.geo, sum(mt.impressions) as total_impressions, sum(mt.clicks) as total_clicks, sum(ml.revenue) as total_revenue,
>	 round(cast(sum(mt.clicks)/sum(mt.impressions) as numeric), 3) as clicks_to_impressions_ratio,
>	 round(cast(sum(ml.revenue)/sum(mt.clicks) as numeric), 3) as revenue_to_clicks_ratio
>	 from missing_last_date_cte ml
>	 left outer join merged_table_cte mt
>	 on ml.date = mt.date and ml.geo = mt.geo
>	 group by ml.geo;
​
* Question #5 (Challenge)
 Generate a query to rank in order the top 10 revenue producing states
​
>  select split_part(store_location, '-', 2) as state, sum(revenue) as total_revenue,
>  rank() over(order by sum(revenue) desc) rank_number
>  from store_revenue
>  group by store_location
limit 10;
