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


行前访问    
`
select regexp_extract(b.bbb,'requesturi=([^, }]+)',1),regexp_extract(b.bbb,'from=([^, }]+)',1),count(distinct(a.uid)) from (select uid,begin_date,begin_middle_date from tmp_travel_split_final where travel_status='during_travel' )as a inner join (select distinct(concat('uid=',uid,',requesturi=',requesturi,',from=',regexp_extract(requestparammap,'from=([^, }]+)',1))) as bbb,dt from travel_client_access where dt>='2017-12-25' and (substring(requesturi,1,5)='/api/') or (substring(requesturi,1,5)='/app/')) as b on a.uid=regexp_extract(b.bbb,'uid=([^, }]+)',1) where b.dt<=a.begin_middle_date and b.dt>=a.begin_date group by regexp_extract(b.bbb,'requesturi=([^, }]+)',1),regexp_extract(b.bbb,'from=([^, }]+)',1)
`  

## 2. 定位城市为常居地城市的定位经纬度
 
`
select a.uid,b.atlng,b.cityid from (select uid,destid from mobile_user_new2 where destid is not null order by rand() limit 100) as a  inner join  (select uid,regexp_extract(requestparammap,'cityId=([^, }]+)',1) as cityid,regexp_extract(requestparammap,'atlng=([^, }]+)',1) as atlng from travel_client_access where dt>='2018-01-01' and requesturi='/api/city/locate') as b on a.uid=b.uid and a.destid=b.cityid
`    

## 3. 分月统计访问城市及省份、国家页面的人数

城市    
`
select substring(dt,1,7),regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1),count(distinct(uid)) from travel_client_access where requesturi='/api/city/get' and dt>='2017-01-01' and dt<='2017-12-31' and (regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1)='302383' or regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1)='316802') group by substring(dt,1,7),regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1)
`    

国家、省份    
`
select substring(dt,1,7),regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1),count(distinct(uid)) from travel_client_access where requesturi='/api/city/search' and dt>='2017-01-01' and dt<='2017-12-31' and (regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1)='316801' or regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1)='300468') group by substring(dt,1,7),regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1)
`  
## 4.同一天查看酒店订单和订单中酒店的用户数
`
select count(distinct(h.uid)) from
(select c.uid,y.dt,y.currenttime from
(select b.uid,create_time,hotel_name,from_date from mppb_order_channel as a inner join ods_travel_touch_uid as b on a.uid=b.touch_uid)
as c
inner join
(select uid,dt,currenttime,regexp_extract(requestparammap,'[{ ]NAME=([\u4e00-\u9fa5]+)',1) as name from travel_client_access where requesturi='/api/book/element')
as y
on c.uid=y.uid and SUBSTRING(c.create_time,1,10)=y.dt where c.hotel_name=y.name and dt<='2017-11-14' and dt>='2017-11-07')
as h
inner join
travel_client_access as x
on h.uid=x.uid and h.dt=x.dt where x.requesturi='/app/api/order/search' and x.dt<='2017-11-14' and x.dt>='2017-11-07' and h.currenttime>=x.currenttime
`

## 5. 行中用户查看行中城市的用户数

`
select count(distinct(x.uid)) from
(select uid,currenttime,regexp_extract(requestparammap,'[{ ]name=([\u4e00-\u9fa5]+)',1) as name from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-06' ) as y
inner join tmp_travel_split_final as x
on x.uid=y.uid where y.currenttime>=x.begin_middle_date and y.name=x.cityname and x.travel_status='行中'
`


## 6. 行中查看常居地城市的用户数

`
select count(distinct(x.uid)) from
(select uid,currenttime,regexp_extract(requestparammap,'[{ ]name=([\u4e00-\u9fa5]+)',1) as name from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-06' ) as y
inner join
(select a.uid,b.cityname,a.begin_middle_date from (select uid,begin_middle_date from tmp_travel_split_final where travel_status='行中') as a inner join mobile_user_new2 as b on a.uid=b.uid )as x
on x.uid=y.uid where y.currenttime>=x.begin_middle_date and y.name=x.cityname
`


## 7. 16年来海南各市旅游人数分布
*执行时间较长*  

`
select a.id,count(distinct(a.uid)) from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid,dt from travel_client_access where requesturi='/api/city/get' and dt>='2016-01-01' and dt<='2016-12-31') as a inner join (select regexp_extract(requestparammap,'cityId=([0-9]+)',1) as cityid,uid,dt from travel_client_access where requesturi='/api/city/locate' and dt>='2016-01-01' and dt<='2016-12-31') as b on a.uid=b.uid and a.dt=b.dt where a.id in ('300188','300148','300097','300145','300146','300149','300187','300094') and b.cityid!='300188' and b.cityid!='300148' and b.cityid!='300097' and b.cityid!='300145' and b.cityid!='300146' and b.cityid!='300149' and b.cityid!='300187' and b.cityid!='300094' group by a.id
`

## 8. 计算常居地数据取样语句
*执行时间较长*  
**全部用户**    

`
select a.uid,a.maxtime,a.mintime,a.shijiancha,a.locate_cityid,b.destid,b.destname from (select uid,max(currenttime) as maxtime,min(currenttime) as mintime,datediff(max(currenttime),min(currenttime)) as shijiancha, regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1) as locate_cityid from travel_client_access where dt>='2017-08-04' and dt<='2017-09-04' and requesturi='/api/city/locate' group by uid,regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1)) as a, (select uid,destid,destname from mobile_user_new2 where destname is not null) as b where a.uid=b.uid order by a.uid asc
`  

**随机样本b（随机10000用户）**      

`(select uid,activetime,destid,destname from mobile_user_new2 where destname is not null order by rand() limit 10000) as b`  

**抽样10000条（抽取10000用户）**    

`order by rand() limit 10000`  

**取样（抽样）代码**  

`
select a.uid,a.maxtime,a.mintime,a.shijiancha,a.locate_cityid,b.destid,b.destname,b.activetime from (select uid,max(currenttime) as maxtime,min(currenttime) as mintime,datediff(max(currenttime),min(currenttime)) as shijiancha, regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1) as locate_cityid from travel_client_access where dt>='2016-06-01' and dt<='2017-09-10' and requesturi='/api/city/locate' group by uid,regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1)) as a, (select uid,activetime,destid,destname from mobile_user_new2 where destname is not null order by rand() limit 10000) as b where a.uid=b.uid order by a.uid asc
`
## 9. 查询所有搜索词pv、uv
`
select regexp_extract(requestparammap,'query=([^, }]+)',1) as query,count(uid) as pv,count(distinct uid) as uv from travel_client_access where dt>='2017-11-07' and dt<='2017-11-07' and requesturi='/api/search/dest' group by regexp_extract(requestparammap,'query=([^, }]+)',1)
`
## 10. 选择标签后查看游记

`
select a.label,count(distinct(a.uid)) from
(select regexp_extract(requestparammap,'label=([^, }]+)',1) as label, currenttime,uid,dt from travel_client_access where requesturi='/api/book/search' and requestparammap like '%label=%' and dt='2017-12-01') as a
inner join
(select currenttime,uid,dt from travel_client_access where requesturi='/api/book/getSimplified' and requestparammap like '%type=2%' and dt='2017-12-01') as b
on a.uid=b.uid and a.dt=b.dt where a.currenttime<=b.currenttime group by a.label
`

## 11. 搜索词为城市名+一日游用户

`
select a.dt,a.uid,regexp_extract(requestparammap,'query=([^, }]+)',1) as query,b.distname from travel_client_access as a,poi_detail as b where a.dt>='2017-10-11' and a.dt<='2017-12-10' and a.requesturi='/api/search/dest'  and regexp_extract(requestparammap,'query=([^, }]+)',1)=CONCAT(dist_name,'一日游') or regexp_extract(requestparammap,'query=([^, }]+)',1)='一日游' or regexp_extract(requestparammap,'query=([^, }]+)',1)=CONCAT(dist_name,'周边一日游')
`
## 12.从suggest进入某城市游记用户
`
select distinct(a.userid) from tmp_userid_uid_20171218 as a inner join (select distinct(uid) from travel_client_access where requesturi='/api/book/search' and requestparammap like '%from=destsuggest%' and requestparammap like '%type=2%' and dt>='2017-12-01' and dt<='2017-12-28' and requestparammap like '%keyword=新加坡%') as b on a.uid=b.uid
`
## 13. 搜索结果页PV、uv
`
select count(uid),count(distinct(uid)) from travel_client_access where requesturi='/api/book/search' and requestparammap like '%from=search%' and dt>='2018-01-01' and dt<='2018-01-01'
`  

## 14. 2月1日新用户查看的第一篇游记来源于发现页
`
select count(distinct(uid)) from 
(select uid from mobile_user_new2 where substring(activetime,1,10)='2018-02-01') as a inner join 
(select uid from 
(select uid,max(currenttime) as time from travel_client_access where requesturi='/api/book/getSimplified' and dt='2018-02-01') as b 
inner join 
(select uid,currenttime from travel_client_access where requesturi='/api/book/getSimplified' and dt='2018-02-01' 
and (regexp_extract(requestparammap,'from=([^, }]+)',1)='explore' or regexp_extract(requestparammap,'from=([^, }]+)',1)='exploreResult')) as c 
on b.uid=c.uid and b.time=c.currenttime) as d on a.uid=d.uid
`  

