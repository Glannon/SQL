## 1. 行中用户访问页面及来源
`
select b.requesturi,b.fromm,count(distinct(a.uid)) from 
(select uid,begin_middle_date from tmp_travel_split_final where travel_status='during_travel' )as a 
inner join (select uid,dt,requesturi,regexp_extract(requestparammap,'from=([^, }]+)',1) as fromm from travel_client_access 
where dt>='2017-12-25' and substring(requesturi,1,5)='/api/') as b 
on a.uid=b.uid where b.dt>=a.begin_middle_date group by b.fromm,b.requesturi
`    

行中用户POI访问来源及类型  
`
select b.requesturi,b.fromm,b.type,count(distinct(a.uid)) from 
(select uid,begin_middle_date from tmp_travel_split_final where travel_status='during_travel' )as a inner join 
(select uid,dt,requesturi,regexp_extract(requestparammap,'poiType=([^, }]+)',1) as type,
regexp_extract(requestparammap,'from=([^, }]+)',1) as fromm from travel_client_access 
where dt>='2017-12-25' and requesturi='/api/book/element') as b on a.uid=b.uid where b.dt>=a.begin_middle_date 
group by b.fromm,b.requesturi,b.type
`
