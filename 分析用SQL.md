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

行中用户分类    
`
select b.uid,b.requesturi,sum(b.shuliang) from 
(select uid,begin_middle_date from tmp_travel_split_final where travel_status='during_travel' )as a 
inner join (select uid,dt,requesturi,count(uid) as shuliang from travel_client_access 
where dt>='2017-12-25' and (substring(requesturi,1,5)='/api/' or substring(requesturi,1,5)='/app/') group by uid,requesturi,dt) as b 
on a.uid=b.uid where b.dt>=a.begin_middle_date group by b.uid,b.requesturi
`
    
行中用户访问POI分类    
`
select b.uid,b.type,sum(b.shuliang) from 
(select uid,begin_middle_date from tmp_travel_split_final where travel_status='during_travel' )as a 
inner join (select uid,dt,requesturi,regexp_extract(requestparammap,'poiType=([^, }]+)',1) as type,count(uid) as shuliang from travel_client_access where dt>='2017-12-25' and requesturi='/api/book/element' group by uid,dt,regexp_extract(requestparammap,'poiType=([^, }]+)',1)) as b 
on a.uid=b.uid where b.dt>=a.begin_middle_date group by b.uid,b.type
`     
常居地定位经纬度    
`
select a.uid,b.atlng,b.cityid from (select uid,destid from mobile_user_new2 where destid is not null order by rand() limit 100) as a  inner join  (select uid,regexp_extract(requestparammap,'cityId=([^, }]+)',1) as cityid,regexp_extract(requestparammap,'atlng=([^, }]+)',1) as atlng from travel_client_access where dt>='2018-01-01' and requesturi='/api/city/locate') as b on a.uid=b.uid and a.destid=b.cityid
`  

行前访问  
`
select regexp_extract(b.bbb,'requesturi=([^, }]+)',1),regexp_extract(b.bbb,'from=([^, }]+)',1),count(distinct(a.uid)) from (select uid,begin_date,begin_middle_date from tmp_travel_split_final where travel_status='during_travel' )as a inner join (select distinct(concat('uid=',uid,',requesturi=',requesturi,',from=',regexp_extract(requestparammap,'from=([^, }]+)',1))) as bbb,dt from travel_client_access where dt>='2017-12-25' and (substring(requesturi,1,5)='/api/') or (substring(requesturi,1,5)='/app/')) as b on a.uid=regexp_extract(b.bbb,'uid=([^, }]+)',1) where b.dt<=a.begin_middle_date and b.dt>=a.begin_date group by regexp_extract(b.bbb,'requesturi=([^, }]+)',1),regexp_extract(b.bbb,'from=([^, }]+)',1)
`  
