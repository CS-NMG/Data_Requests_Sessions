hijacking rate by date
```
select
    date, sum(infected_journey)*1.00/sum(visits) as ir
from(select
        date, count(distinct sessionid) as visits, count(case when isinfected = 1 then sessionid end) as infected_journey
    from(select
            date(timestamp) as date, sessionid, max(isinfected) as isinfected
        from stack
        where clienttag = 'PAZDGJ079'
                and timestamp >= '2021-02-14' 
                and timestamp <= '2021-02-28' 
          group by date,sessionid)
    group by date)
group by date
order by date asc;
```


hijacking m/b rate by date
```
select
	date, max(case when tegr = 'x' then ir end) as ir_monitor, max(case when tegr = 'o' then ir end) AS ir_blocked
from(select
		date, tegr, sum(infected_journey)*1.00/sum(visits) as ir
	from(select
			date, tegr, count(distinct sessionid) as visits, count(case when isinfected = 1 then sessionid end) as infected_journey
		from(select
				date(timestamp) as date, tegr, sessionid, max(isinfected) as isinfected
			from stack
			where clienttag = 'PAZDGJ079'
			        and timestamp >= '2021-02-22' 
			        and timestamp <= '2021-02-28' 
			  group by date,tegr,sessionid)
		group by date, tegr)
	group by date,tegr)
group by date
order by date;
```


hijacking rate by specific url
```
select
	href, count(sessionid) as sessions, count(case when isinfected = 1 then sessionid end) as infected_sessions,
	infected_sessions *1.0/sessions AS ir
--	href, date, sum(isinfected)*100.0/count(isinfected)  as ir
from
	(select href, sessionid, max(isinfected) as isinfected
     -- concat(to_char(date(timestamp), 'Month'), to_char(date(timestamp), 'YYYY')) as date
	from stack
	where clienttag = 'CADQPM944' 
	    and timestamp >= '2021-02-03' 
		and device_type in ('pc','mobile')
	    and (href like '%/cart' or
			href like '%/cart/store-service' or
            href like '%/cart/ma-commande' or
			href like '%/cart/paiement' or
            href like '%/cart/confirmation')
--	group by href, sessionid, date) a
--group by href, date
--order by href, date;
	group by href, sessionid) a
group by href;
```


top hijacked urls by stack
```
select
	href, count(sessionid) as sessions, count(case when infection = 1 then sessionid end) as infected_sessions,
	infected_sessions *1.0/sessions AS ir
from (select href, sessionid,  max(isinfected) AS infection
	from stack st
	where clienttag = 'CADQPM944' 
		and timestamp >= '2021-02-03' 
		and device_type in ('pc','mobile')
	group by sessionid, href)
group by href
having sessions > 1000
-- order by ir desc
order by sessions desc
limit 500;
```


hijacking rate per browser
```
select
    browser, count(sessionid) as sessions, count(case when infected = 1 then sessionid end) as infected_sessions,
	infected_sessions*1.0/sessions as ir
from (select browser, sessionid, max(isinfected) as infected
	from stack
	where clienttag = 'CADQPM944' 
		and timestamp >= '2021-02-03' 
		and device_type in ('pc','mobile')
	group by browser, sessionid)
group by browser
order by ir desc
limit 500;
```


top injection types
```
select
    distinct injection, count(distinct session_id) as sessions
from injections
where clienttag = 'CADQPM944' 
  and file_created_at >= '2021-02-03' 
		and device_type in ('pc','mobile')
group by injection
order by sessions desc
limit 500;
```


hijacking rate per test group
```
select
    max(case when tegr='x'then HR END) as HR_monitor, max(case when tegr='o'then HR END) AS HR_blocked
from(
select tegr, count(distinct sessionid) as visits, 
count(case when isinfected = 1 then sessionid end) as infected_journey,
infected_journey/visits as HR
from(
select 
	timestamp, tegr, sessionid, max(isinfected) as isinfected
from stack
where clienttag = 'LODOWS854' and timestamp >= '2021-01-28' 
        AND timestamp <= '2021-02-09' 
  group by timestamp,tegr,sessionid)
group by tegr);
```


industry checkout abandonment rates
```
select
	cdv.tag,
	(sum(checkout_visits)-sum(conversions))*100.0/sum(checkout_visits) as checkout_abandonment
from client_data_view cdv
join (select
		tag
	from ecommerce_session_flow_aggr_tbl
	group by tag
	having sum(checkout_sessions) > sum(converted_sessions)
		and sum(checkout_sessions) != 0
	order by tag) eco
using (tag)
--where date_format(cdv.created,'%Y') = '2020'
group by tag
having checkout_abandonment not like '-%'
order by tag;
```


avg page load times
```
select
    clienttag, avg(load_time) as avg
from(select
        distinct l.clienttag, l.sessionid, l.href,l.load_time
     from load_times l 
     join (select
            distinct clienttag, country_name
           from compressed_stack where start_time ='2021-02-21') s
     on l.clienttag=s.clienttag
where s.country_name='United Kingdom'
        and timestamp = '2021-02-21')
group by clienttag
order by avg;
```


avg page tag times
```
select
    clienttag, avg(scriptduration) as avg scriptduration
from(select
     distinct l.clienttag, l.sessionid, l.href, l.scriptduration
from load_times l
group by clienttag)
        AND timestamp = '2021-11-26')
        group by clienttag
        order by avg
```


unique href count
```
select
    date(timestamp) as day, count(distinct href)
from (select s.timestamp, s.href
	from stack s
	join client_domains cd
	    using (clienttag)
	where clienttag = 'LEXJCK305' 
		and timestamp >= '2021-01-01'
		and cd.is_active = '1'
		and cd.agreed_in_contract = '1'
	group by s.timestamp, s.href)
group by day
order by day asc;
```

avg aov per industry and device type
```
select
	industry, device_type, avg(aov)
from client_data_view
join client_configurations_tbl
on tag = clienttag
where is_active = 1
	and is_relationship_paused = 0
	and source != 'Client Data Process'
	and from_date >= now() - interval '1' year
group by industry, device_type;
```


num of sessions for domains
```
select
    date((dateadd('h',timezone_offset,s.start_time))) as date,
    count(distinct sessionid) as sessions,
    count(distinct client_attr1) as client_attr1,
    count(distinct client_attr2) as client_attr2,
    count (distinct uuid) as users
from compressed_stack as s
join CLIENT_CONFIGURATIONS_TBL as c
on s.clienttag = c.clienttag
where s.start_time >= dateadd('h',timezone_offset,'2021-04-18 00:00:00')
--    and s.start_time <= dateadd('h',timezone_offset,'2021-04-15 00:00:00')
    and s.clienttag='HP7F96O2J'
    and domain in ('hp.com','www.hp.com')
group by date((dateadd('h',timezone_offset,s.start_time)))
order by date;
```

          
monitor session count per domain
```
select
    domain, (case when tegr = 'x' then count(distinct sessionid) end) as monitor
from compressed_stack
where clienttag = 'LD7H09A9P'
    and start_time >= '2021-04-17'
    and domain in ('123inkjets.com', '4inkjets.com', 'inkcartridges.com', 'quickship.com', 'suppliesguys.com')
group by domain, tegr
having monitor;
              
select
    domain, count(distinct sessionid) as sessions
from compressed_stack
where clienttag = 'LD7H09A9P'
    and start_time >= '2021-04-17'
    and domain in ('123inkjets.com', '4inkjets.com', 'inkcartridges.com', 'quickship.com', 'suppliesguys.com')
group by domain;
```

urls session count
```
select
    split_part(href,'https://www.marksandspencer.com', 2) AS url, timezone_offset, count(distinct sessionid) AS num_sessions
from stack
left join client_configurations_tbl
    using (clienttag)
    where clienttag = 'SEDHVU295' 
        and dateadd(hour,timezone_offset,start_time) >= '2021-02-20' 
--        and dateadd(hour,timezone_offset,start_time) <= '2021-03-22'
group by url, timezone_offset
order by num_sessions desc
limit 100;
```
