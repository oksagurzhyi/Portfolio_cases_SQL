Case: Conversion increased, no money



Legend: You are an e-commerce analyst. You launched an AV test of a new cart design.



In a week:

CR (purchase/session) increased from 2.0 to 2.3%

Average check decreased from 50 to 44

Revenue per session almost unchanged

DAU unchanged

Returns unchanged.



Management says: Conversion increased - that's very good.



Data:

Table ab\_sessions:

session\_id

user\_id

variant (A/B)

revenue (0 if there was no purchase).



Calculate by variants:

sessions

purchasers with revenue greater than 0

CR

AOV

RPV (revenue per variant).



Why can conversion increase but RPV not? RPV consists of CR and AOV, and if conversion increases and average revenue decreases, the effect can be mutually compensated. In this situation, CR is better, but the money per session is almost the same. They won the frequency of purchase, but lost its size.



Segmentation into cheap and expensive buyers allows you to check what caused the conversion to increase.

Most often, this may be that CR increased in cheap purchases, but decreased (did not increase) in expensive ones. Or - CR increased in both cheap and expensive purchases, then the change in AOV may be due to the fact that expensive ones began to buy more "narrowly" - fewer items in the basket, fewer additional items.



That is, in this case, the decisive metric for business is not conversion, but average revenue by options (or profit).







select date\_trunc('day', a.session\_date::date) as day\_start

, a.variant

, count(distinct a.session\_id) as sessions\_cnt

, round(sum(a.revenue::numeric),2) as revenue

, count(distinct a.user\_id) as buyers

, round(1.0\*count(distinct a.user\_id) / count(distinct a.session\_id),2) as CR

, round(1.0\*sum(a.revenue::numeric) / count(distinct a.user\_id),2) as AOV

,  round(1.0\*sum(a.revenue::numeric) / count(distinct a.session\_id),2) as revenue\_per\_variant

from voda.ab\_sessions a

where a.revenue >0

group by 1,2

order by 2,1

;



\--week

select date\_trunc('week', a.session\_date::date) as week\_start

, a.variant

, count(distinct a.session\_id) as sessions\_cnt

, round(sum(a.revenue::numeric),2) as revenue

, count(distinct a.user\_id) as buyers

, round(1.0\*count(distinct a.user\_id) / count(distinct a.session\_id),2) as CR

, round(1.0\*sum(a.revenue::numeric) / count(distinct a.user\_id),2) as AOV

,  round(1.0\*sum(a.revenue::numeric) / count(distinct a.session\_id),2) as revenue\_per\_variant

from voda.ab\_sessions a

where a.revenue >0

group by 1,2

order by 2,1

;





\--checking for low and hith spenders and CR

with segmenting as (

select date\_trunc('day', a.session\_date::date) as day\_start

, a.variant

, count(distinct a.session\_id) as sessions\_cnt

, count(distinct a.user\_id) as buyers

, round(sum(a.revenue::numeric),2) as revenue

, count(distinct case when a.revenue <30 then a.user\_id end) as low\_spenders\_cnt

, sum(case when a.revenue <30 then a.revenue end) as low\_spenders\_revenue

, count(distinct case when a.revenue >=30 then a.user\_id end) as high\_spenders\_cnt

, sum( case when a.revenue >=30 then a.revenue end) as high\_spenders\_revenue

from voda.ab\_sessions a

where a.revenue >0

group by 1,2

order by 2,1

)

select  day\_start

, variant

, round(1.0\*low\_spenders\_cnt / buyers, 2) as CR\_low\_spenders

, round(1.0\*high\_spenders\_cnt / buyers,2) as CR\_high\_spenders

from segmenting

;

