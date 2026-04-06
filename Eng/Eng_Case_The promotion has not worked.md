**Case: The promotion has not worked.**



*Legend*: You are an e-commerce analyst. Marketing launched a Free Shipping on Weekends campaign.

Conditions:

The promotion is valid only in 3 countries

Only for the electronics, home, kids categories

Only for users who saw the banner in the application.



After the campaign, marketing reports: The promotion did not work, there are many empty segments in the report.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

The problem may not be in the promotion, but in the fact that the report is not built correctly: some of the combinations "country x category x day" are simply missing, because there are no orders.



To check this, you need to build a full matrix of segments in order to see zeros.



**Tables**:

orders:

order\_date

order\_id

user\_id

country

category

revenue



banner\_impression:

impression\_date

user\_id

country



dims(reference):

countries

categories

dates





1\. Build a "segment grid" via cross join

2\. Add metrics by orders and show zeros

3\. Condition - saw the banner



\--join tables to create a basis for developing required metrics 

with skelet\_table as (

select   d.countries

,  d1.categories 

,  d2.dates::date

from voda.dims d

cross join voda.dims d1 

cross join voda.dims d2

)

, 

banner\_users as (

select distinct b.impression\_date::date as dt

, b.user\_id

, b.country

from voda.banner\_impressions b 

)

, orders\_campaign as (

select o.order\_date::date as dt

, o.order\_id 

, o.user\_id 

, o.country 

, o.category 

, o.revenue

from voda.orders\_c o

join banner\_users b  

&#x09;	on o.order\_date::date = b.dt

&#x09;	and o.user\_id = b.user\_id 

&#x09;	and o.country = b.country

)

select t.dates 

, t.countries

, t.categories

, count(distinct c.order\_id) as orders\_cnt

, count(distinct c.user\_id) as users\_cnt

, round(coalesce(sum(c.revenue::numeric),0),2) as revenue

from skelet\_table t 

left join orders\_campaign c 

&#x09;	on t.dates = c.dt 

&#x09;	and t.countries = c.country 

&#x09;	and t.categories = c.category

group by 1,2,3

order by 1

;



"Empty segments" are not an error in the report and are not proof that the promotion did not work.

In the combinations "country x category x day" zeros were actually obtained for those who saw the banner.

But then you need to investigate:

* whether the audience was correct
* whether the category was interesting
* whether the price and assortment were interesting for buyers.

