# SQL语句记录
## 1. 查询酒店订单

| 字段             | 值 | 含义 |
|------------------|----|------|
| pay_type         | 0  | 预付 |
| （付款方式）     | 1  | 现付 |
| ppb_order_status | 4  | 确认 |
| （订单状态）     | 5  | 取消 |
|                  | 15 | 退款 |


**SQL语句**  

`
select create_time,b.uid,city_name,hotel_id,hotel_seq,hotel_name,from_date,to_date,room_name,bed_type,pay_type,ppb_order_status from mppb_order_channel as a inner join (select touch_uid,uid from ods_travel_touch_uid where uid='3812B41A-4611-4923-A157-82C19EF2868C' ) as b on a. uid=b.touch_uid WHERE a.dt>='2017-07-01' and a.dt<='2017-08-10' AND a.STATUS IN('0','2') AND a.vid LIKE '91%' AND a.chan_value in('travel_touch','travel_gonglue','travel_client','travelnote_client','travel note_gonglue','gonglue_zhuanti','gonglue_zhuanti2','jd_mt_huaweiqjzn') AND SUBSTRING(a.create_time,1,10)>='2017-07-01' AND SUBSTRING(a.create_time,1,10)<='2017-08-10'
`

同一天查看酒店订单和订单中酒店的用户数
currenttime
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

行中用户查看行中城市

select count(distinct(x.uid)) from
(select uid,currenttime,regexp_extract(requestparammap,'[{ ]name=([\u4e00-\u9fa5]+)',1) as name from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-06' ) as y
inner join tmp_travel_split_final as x
on x.uid=y.uid where y.currenttime>=x.begin_middle_date and y.name=x.cityname and x.travel_status='行中'



行中查看常居地城市
select count(distinct(x.uid)) from
(select uid,currenttime,regexp_extract(requestparammap,'[{ ]name=([\u4e00-\u9fa5]+)',1) as name from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-06' ) as y
inner join
(select a.uid,b.cityname,a.begin_middle_date from (select uid,begin_middle_date from tmp_travel_split_final where travel_status='行中') as a inner join mobile_user_new2 as b on a.uid=b.uid )as x
on x.uid=y.uid where y.currenttime>=x.begin_middle_date and y.name=x.cityname


## 2. 查询浏览的游记的页面位置

`
select * from travel_client_access where dt='2017-08-03' and uid='45523841-53E3-428E-A6F0-C9861FA11F8E' and requesturi='/api/book/getSimplified' and requestparammap like '%type=2%' order by currenttime asc
`

## 3. 查询浏览的酒店的页面位置

`
select * from travel_client_access where dt='2017-08-03' and uid='45523841-53E3-428E-A6F0-C9861FA11F8E' and requesturi='/api/book/element' and requestparammap like '%poiType=2%' order by currenttime asc
`

## 4. 海南旅游目的地占比

`
select a.id,count(distinct(a.uid)) from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid,dt from travel_client_access where requesturi='/api/city/get' and dt>='2016-01-01' and dt<='2016-12-31') as a inner join (select regexp_extract(requestparammap,'cityId=([0-9]+)',1) as cityid,uid,dt from travel_client_access where requesturi='/api/city/locate' and dt>='2016-01-01' and dt<='2016-12-31') as b on a.uid=b.uid and a.dt=b.dt where a.id in ('300188','300148','300097','300145','300146','300149','300187','300094') and b.cityid!='300188' and b.cityid!='300148' and b.cityid!='300097' and b.cityid!='300145' and b.cityid!='300146' and b.cityid!='300149' and b.cityid!='300187' and b.cityid!='300094' group by a.id
`

行前用户
select count(uid) from  
(select a.uid from
  (select uid,max(travel_num) as num from tmp_travel_split_final group by uid) as a inner join tmp_travel_split_final as b on a.uid=b.uid and a.num=b.travel_num where b.travel_status='行前' and b.begin_date<='2017-09-16'
)  
as x
inner join
(select uid,max(currenttime) as currenttime from travel_client_access group by uid)
as y on x.uid=y.uid where y.currenttime<='2017-09-16'


# 5 常居地
select a.uid,a.maxtime,a.mintime,a.shijiancha,a.locate_cityid,b.destid,b.destname from (select uid,max(currenttime) as maxtime,min(currenttime) as mintime,datediff(max(currenttime),min(currenttime)) as shijiancha, regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1) as locate_cityid from travel_client_access where dt>='2017-08-04' and dt<='2017-09-04' and requesturi='/api/city/locate' group by uid,regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1)) as a, (select uid,destid,destname from mobile_user_new2 where destname is not null) as b where a.uid=b.uid order by a.uid asc


随机样本b  
(select uid,activetime,destid,destname from mobile_user_new2 where destname is not null order by rand() limit 10000) as b

1000条  
order by rand() limit 10000



完整代码  

select a.uid,a.maxtime,a.mintime,a.shijiancha,a.locate_cityid,b.destid,b.destname,b.activetime from (select uid,max(currenttime) as maxtime,min(currenttime) as mintime,datediff(max(currenttime),min(currenttime)) as shijiancha, regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1) as locate_cityid from travel_client_access where dt>='2016-06-01' and dt<='2017-09-10' and requesturi='/api/city/locate' group by uid,regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1)) as a, (select uid,activetime,destid,destname from mobile_user_new2 where destname is not null order by rand() limit 10000) as b where a.uid=b.uid order by a.uid asc


11月1日至12月10日到达昌北机场的订单用户

	select c.uid from (select uid from (select distinct deviceuid from orderinfo_channel where dt>='2017-11-01' and dt<='2017-12-10' and arrcity='南昌' and deptime>='2017-11-01' and deptime<='2017-12-10' and orderStatus NOT IN ('0','12','20','51','91')) as a,(select touch_uid,uid from ods_travel_touch_uid) as b where a.deviceuid=b.touch_uid) as c,(select uid,city_id from tmp_user_locate_result where city_id!='300021') as d where c.uid=d.uid

建表

drop table tmp_travel_userid_uid
create table tmp_userid_uid as select distinct uid,userid from travel_client_access where dt>='2016-01-01' and userid like '%@qunar%'


/api/poi_search/hotel/


常居地在某地访问过某地
select x.userid from   tmp_userid_uid_20171205 as x inner join (select a.uid from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-05' ) as a inner join (select uid from mobile_user_new2 where destid in('300100','300083','703475','300077','300192','300089') or cityid in('300100','300083','703475','300077','300192','300089')) as b on a.uid=b.uid where a.id='300085' ) as y on x.uid=y.uid


常居地在某地访问过某地
select x.userid from   tmp_userid_uid_20171205 as x inner join (select a.uid from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-05' ) as a inner join (select uid from mobile_user_new2 where destid in('300085','5942410') or cityid in('300085','5942410')) as b on a.uid=b.uid where a.id='300100' ) as y on x.uid=y.uid

定位在某地访问过某地
select distinct(x.userid) from tmp_userid_uid_20171205 as x inner join (select a.uid from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-05' ) as a inner join (select regexp_extract(requestparammap,'cityId=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/locate' and dt>='2017-11-05' ) as b on a.uid=b.uid where a.id in ('300085') and b.id in ('300100','300083','703475','300077','300192','300089')) as y on x.uid=y.uid

定位在某地访问过某地
select distinct(x.userid) from tmp_userid_uid_20171205 as x inner join (select a.uid from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-05' ) as a inner join (select regexp_extract(requestparammap,'cityId=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/locate' and dt>='2017-11-05' ) as b on a.uid=b.uid where a.id='300100' and b.id='300085') as y on x.uid=y.uid


查询所有搜索词pv
select regexp_extract(requestparammap,'query=([^, }]+)',1) as query,count(1) from travel_client_access where dt>='2017-11-07' and dt<='2017-11-07' and requesturi='/api/search/dest' group by regexp_extract(requestparammap,'query=([^, }]+)',1)




查询所有搜索词uv
select regexp_extract(requestparammap,'query=([^, }]+)',1) as query,count(uid) as pv,count(distinct uid) as uv from travel_client_access where dt>='2017-11-07' and dt<='2017-11-07' and requesturi='/api/search/dest' group by regexp_extract(requestparammap,'query=([^, }]+)',1)




选择标签后查看游记
select a.label,count(distinct(a.uid)) from
(select regexp_extract(requestparammap,'label=([^, }]+)',1) as label, currenttime,uid,dt from travel_client_access where requesturi='/api/book/search' and requestparammap like '%label=%' and dt='2017-12-01') as a
inner join
(select currenttime,uid,dt from travel_client_access where requesturi='/api/book/getSimplified' and requestparammap like '%type=2%' and dt='2017-12-01') as b
on a.uid=b.uid and a.dt=b.dt where a.currenttime<=b.currenttime group by a.label


城市名+一日游
select dt,uid,query,cityid from
(select dt,uid,regexp_extract(requestparammap,'query=([^, }]+)',1) as query, regexp_extract(requestparammap,'cityId=([0-9]+)',1) as cityid from travel_client_access where dt>='2017-10-11' and dt<='2017-12-10' and requesturi='/api/search/dest' and requestparammap like '%query=%') as a,
 (select distinct dist_id,dist_name from poi_detail) as b
 where (a.query=CONCAT(dist_name,'一日游') or a.query='一日游' or a.query=CONCAT(dist_name,'周边一日游'))


select a.dt,a.uid,regexp_extract(requestparammap,'query=([^, }]+)',1) as query,b.distname from travel_client_access as a,poi_detail as b where a.dt>='2017-10-11' and a.dt<='2017-12-10' and a.requesturi='/api/search/dest'  and regexp_extract(requestparammap,'query=([^, }]+)',1)=CONCAT(dist_name,'一日游') or regexp_extract(requestparammap,'query=([^, }]+)',1)='一日游' or regexp_extract(requestparammap,'query=([^, }]+)',1)=CONCAT(dist_name,'周边一日游')

select count(* ) from travel_client_access where dt>='2017-11-01' and dt<='2017-11-30' and regexp_extract(requestparammap,'keyword=([^, }]+)',1)='一日游' and requesturi='/api/book/search'


多个POI访问pv与uv

select * from travel_client_access where dt='2017-12-01' and requesturi='/api/book/element' and requestparammap like '%poiType=4%' limit 100



访问过某个城市或景区的用户
select distinct(x.userid) from   tmp_userid_uid_20171211 as x inner join
  (select distinct(uid) from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-05' and dt <='2017-11-05' and
  regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) in ('299914')) as y
 on x.uid=y.uid


 select distinct(x.userid) from  tmp_userid_uid_20171211 as x inner join
   (select distinct(uid) from travel_client_access where dt>='2017-11-05' and dt <='2017-11-05' and requesturi='/api/book/element' and requestparammap like '%poiType=4%' and regexp_extract(requestparammap,'[{ ]poiId=([0-9]+)',1) in ('713020')) as y
  on x.uid=y.uid
