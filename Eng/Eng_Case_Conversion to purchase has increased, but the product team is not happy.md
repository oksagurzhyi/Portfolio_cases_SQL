Case: Conversion to purchase has increased, but the product team is not happy



Legend: You are an e-commerce / marketplace analyst



Over the past week, the team has seen an increase in conversion from session to purchase from 3.8% to 4.5%

But the product is not sure that traffic and order quality have improved. You need to check what caused the conversion to increase.



Tables

sessions:

session\_id

user\_id

session\_date

channel - organic, ads, email, referral

device - mobile, desktop, tablet



orders:

order\_id

session\_id

user\_id

order\_date

amount



Need to check:

number of sessions

number of orders

what was the conversion

how did the average check change

what happened to revenue per user

investigate revenue per session

which channels gave growth

and whether the share of cheap orders increased.



We need to understand what has changed with the increase in conversion. Its growth does not mean that the product has become better: it must be considered together with the average check and revenue per session. Because it is possible that there have been more orders, but they have become cheaper and therefore the value has hardly changed.

Next, we check which channels have given the increase in conversion, and then by devices.



After the analysis, we see that the conversion has indeed increased, but its growth cannot be unambiguously interpreted as an improvement in the product. Orders have become more frequent, but cheaper. The average check has decreased, and revenue per session has hardly changed. The product better leads buyers to purchase, but its quality requires further research.



select order\_id

from voda.orders

group by order\_id

having count(order\_id)>1





\--Step 1. Basic metrics by weeks

with sessions\_orders as (

select date\_trunc('week', s.session\_date::date) as week\_start

, s.user\_id

, s.session\_id

, s.channel

, s.device\_type

, o.order\_id

, o.amount

from voda.sessions s

left join   voda.orders o using(session\_id)

order by date\_trunc('week', s.session\_date::date), s.session\_id

)

select  week\_start

, count(distinct session\_id) as session\_cnt

, count(distinct order\_id) as orders\_cnt

, sum(amount) as revenue

, round(1.0\*count(distinct order\_id) /  nullif(count(distinct session\_id),0),2) as conversion\_rate

, round(avg(amount::numeric),2)  as avg\_order\_value

, round(sum(amount::numeric) / count(distinct session\_id),2) as revenue\_per\_session

from  sessions\_orders

group by week\_start



\--Step 2. Exploring channels which have made growth



with sessions\_orders as (

select date\_trunc('week', s.session\_date::date) as week\_start

, s.user\_id

, s.session\_id

, s.channel

, s.device\_type

, o.order\_id

, o.amount

from voda.sessions s

left join   voda.orders o using(session\_id)

order by date\_trunc('week', s.session\_date::date), s.session\_id

)

select  week\_start

, channel

, count(distinct session\_id) as session\_cnt

, count(distinct order\_id) as orders\_cnt

, sum(amount) as revenue

, round(1.0\*count(distinct order\_id) /  nullif(count(distinct session\_id),0),2) as conversion\_rate

, round(avg(amount::numeric),2)  as avg\_order\_value

, round(sum(amount::numeric) / count(distinct session\_id),2) as revenue\_per\_session

from  sessions\_orders

group by week\_start, channel

order by week\_start, channel



\--Step 3. Checking for share of small orders



with sessions\_orders as (

select date\_trunc('week', s.session\_date::date) as week\_start

, s.session\_id

, s.user\_id

, o.order\_id

, o.amount

from voda.sessions s

left join voda.orders o using(session\_id)

)

select  week\_start

, sum(amount) as revenue

, case when amount<20 then 'small\_order'

&#x09;	when amount between 20 and 100 then 'medium order'

&#x09;	else 'large\_order' end as orders\_bucket

, count(order\_id) as orders\_cnt

from sessions\_orders

group by 1,3

order by 1,3





\--Step 4. CT and revenue per session by device

with session\_orders as (

select date\_trunc('week', s.session\_date::date) as week\_start

, s.session\_id

, s.device\_type

, o.order\_id

, o.amount

from voda.sessions s

left join   voda.orders o using(session\_id)

)

select  week\_start

, device\_type

, count(distinct session\_id) as sessions\_cnt

, count(distinct order\_id) as orders\_cnt

, sum(amount) as revenue

, round(1.0\*count(distinct order\_id) /  count(distinct session\_id),2)  as conversion\_rate

, round(1.0\*sum(amount::numeric) / count(distinct session\_id),2) as revenue\_per\_session

, avg(amount) as avg\_order\_value

&#x20;from session\_orders

group by week\_start, device\_type

order by week\_start



\--Step 5.Traffic structure



select  date\_trunc('week', s.session\_date::date) as week\_start

, s.channel

, count(\*) as sessions\_cnt

, round(count(\*) / sum(count(\*)) over (partition by date\_trunc('week', s.session\_date::date)),2) as traffic\_share

from voda.sessions s

group by 1,2

order by 1,2

;

