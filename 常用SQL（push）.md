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

## 2.同一天查看酒店订单和订单中酒店的用户数
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

## 3. 行中用户查看行中城市的用户数

`
select count(distinct(x.uid)) from
(select uid,currenttime,regexp_extract(requestparammap,'[{ ]name=([\u4e00-\u9fa5]+)',1) as name from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-06' ) as y
inner join tmp_travel_split_final as x
on x.uid=y.uid where y.currenttime>=x.begin_middle_date and y.name=x.cityname and x.travel_status='行中'
`


## 4. 行中查看常居地城市的用户数

`
select count(distinct(x.uid)) from
(select uid,currenttime,regexp_extract(requestparammap,'[{ ]name=([\u4e00-\u9fa5]+)',1) as name from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-06' ) as y
inner join
(select a.uid,b.cityname,a.begin_middle_date from (select uid,begin_middle_date from tmp_travel_split_final where travel_status='行中') as a inner join mobile_user_new2 as b on a.uid=b.uid )as x
on x.uid=y.uid where y.currenttime>=x.begin_middle_date and y.name=x.cityname
`


## 5. 海南各市16年来旅游人数分布

`
select a.id,count(distinct(a.uid)) from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid,dt from travel_client_access where requesturi='/api/city/get' and dt>='2016-01-01' and dt<='2016-12-31') as a inner join (select regexp_extract(requestparammap,'cityId=([0-9]+)',1) as cityid,uid,dt from travel_client_access where requesturi='/api/city/locate' and dt>='2016-01-01' and dt<='2016-12-31') as b on a.uid=b.uid and a.dt=b.dt where a.id in ('300188','300148','300097','300145','300146','300149','300187','300094') and b.cityid!='300188' and b.cityid!='300148' and b.cityid!='300097' and b.cityid!='300145' and b.cityid!='300146' and b.cityid!='300149' and b.cityid!='300187' and b.cityid!='300094' group by a.id
`

## 6 常居地数据取样语句

`
select a.uid,a.maxtime,a.mintime,a.shijiancha,a.locate_cityid,b.destid,b.destname from (select uid,max(currenttime) as maxtime,min(currenttime) as mintime,datediff(max(currenttime),min(currenttime)) as shijiancha, regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1) as locate_cityid from travel_client_access where dt>='2017-08-04' and dt<='2017-09-04' and requesturi='/api/city/locate' group by uid,regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1)) as a, (select uid,destid,destname from mobile_user_new2 where destname is not null) as b where a.uid=b.uid order by a.uid asc
`

**随机样本b**    
`(select uid,activetime,destid,destname from mobile_user_new2 where destname is not null order by rand() limit 10000) as b`

**1000条**    
`order by rand() limit 10000`

**完整代码取样代码**

`
select a.uid,a.maxtime,a.mintime,a.shijiancha,a.locate_cityid,b.destid,b.destname,b.activetime from (select uid,max(currenttime) as maxtime,min(currenttime) as mintime,datediff(max(currenttime),min(currenttime)) as shijiancha, regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1) as locate_cityid from travel_client_access where dt>='2016-06-01' and dt<='2017-09-10' and requesturi='/api/city/locate' group by uid,regexp_extract(requestparammap,'[{ ]cityId=([0-9]+)',1)) as a, (select uid,activetime,destid,destname from mobile_user_new2 where destname is not null order by rand() limit 10000) as b where a.uid=b.uid order by a.uid asc
`

## 7.14天内到达昌北机场的订单用户

`
select distinct(y.userid) from 
  (select c.uid from (select uid from (select distinct deviceuid from orderinfo_channel where dt>='2017-11-01' and dt<='2017-12-26' and arrcity='南昌' and deptime>='2017-12-13' and deptime<='2017-12-26' and orderStatus NOT IN ('0','12','20','51','91')) as a,(select touch_uid,uid from ods_travel_touch_uid) as b where a.deviceuid=b.touch_uid) as c,(select uid,city_id from tmp_user_locate_result where city_id!='300021') as d where c.uid=d.uid) as x inner join tmp_userid_uid_20171218 as y on x.uid=y.uid
`

## 8. 建立uid与userid关联表

`
create table tmp_userid_uid as select distinct uid,userid from travel_client_access where dt>='2016-01-01' and userid like '%@qunar%' and uid!='000000000000'
`

## 9. 常居地在某地访问过某地(push)

`
select distinct(x.userid) from  tmp_userid_uid_20171218 as x inner join (select a.uid from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-05' ) as a inner join (select uid from mobile_user_new2 where destid in('300100','300083','703475','300077','300192','300089') or cityid in('300100','300083','703475','300077','300192','300089')) as b on a.uid=b.uid where a.id='300085' ) as y on x.uid=y.uid
`


## 10. 定位在某地访问过某地

`
select distinct(x.userid) from tmp_userid_uid_20171218 as x inner join (select a.uid from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-05' ) as a inner join (select regexp_extract(requestparammap,'cityId=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/locate' and dt>='2017-11-05' ) as b on a.uid=b.uid where a.id in ('300085') and b.id in ('300100','300083','703475','300077','300192','300089')) as y on x.uid=y.uid
`


## 11. 查询所有搜索词pv

`
select regexp_extract(requestparammap,'query=([^, }]+)',1) as query,count(1) from travel_client_access where dt>='2017-11-07' and dt<='2017-11-07' and requesturi='/api/search/dest' group by regexp_extract(requestparammap,'query=([^, }]+)',1)
`



## 12. 查询所有搜索词uv

`
select regexp_extract(requestparammap,'query=([^, }]+)',1) as query,count(uid) as pv,count(distinct uid) as uv from travel_client_access where dt>='2017-11-07' and dt<='2017-11-07' and requesturi='/api/search/dest' group by regexp_extract(requestparammap,'query=([^, }]+)',1)
`



## 13. 选择标签后查看游记

`
select a.label,count(distinct(a.uid)) from
(select regexp_extract(requestparammap,'label=([^, }]+)',1) as label, currenttime,uid,dt from travel_client_access where requesturi='/api/book/search' and requestparammap like '%label=%' and dt='2017-12-01') as a
inner join
(select currenttime,uid,dt from travel_client_access where requesturi='/api/book/getSimplified' and requestparammap like '%type=2%' and dt='2017-12-01') as b
on a.uid=b.uid and a.dt=b.dt where a.currenttime<=b.currenttime group by a.label
`

## 14. 搜索词为城市名+一日游用户

`
select a.dt,a.uid,regexp_extract(requestparammap,'query=([^, }]+)',1) as query,b.distname from travel_client_access as a,poi_detail as b where a.dt>='2017-10-11' and a.dt<='2017-12-10' and a.requesturi='/api/search/dest'  and regexp_extract(requestparammap,'query=([^, }]+)',1)=CONCAT(dist_name,'一日游') or regexp_extract(requestparammap,'query=([^, }]+)',1)='一日游' or regexp_extract(requestparammap,'query=([^, }]+)',1)=CONCAT(dist_name,'周边一日游')
`
## 15. 访问过某个城市或景区的用户
**城市**   
`
select distinct(x.userid) from   tmp_userid_uid_20171218 as x inner join
  (select distinct(uid) from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-05' and dt <='2017-11-05' and
  regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) in ('299914')) as y
 on x.uid=y.uid
`  

**景区、景点**    
`
 select distinct(x.userid) from  tmp_userid_uid_20171218 as x inner join
   (select distinct(uid) from travel_client_access where dt>='2017-11-05' and dt <='2017-11-05' and requesturi='/api/book/element' and requestparammap like '%poiType=4%' and regexp_extract(requestparammap,'[{ ]poiId=([0-9]+)',1) in ('713020')) as y
  on x.uid=y.uid
 `
## 16. 浏览过某地酒店
`
select distinct(userid) from tmp_userid_uid as b inner join (select distinct(uid) from travel_client_access where dt>='2017-11-24' and dt<='2017-11-29' and requesturi='/api/hotel/search' and requestparammap like '%cityId=302161%'  and uid!='000000000000') as a on a.uid=b.uid
`

## 17. 某城市酒店成单量（城市名称）

`
select count(distinct(uid)) from mppb_order_channel as a inner join (select touch_uid,uid from ods_travel_touch_uid where city_name='北京') as b on a. uid=b.touch_uid WHERE a.dt>='2017-07-01' and a.dt<='2017-08-10' AND a.STATUS IN('0','2') AND a.vid LIKE '91%' AND a.chan_value in('travel_touch','travel_gonglue','travel_client','travelnote_client','travel note_gonglue','gonglue_zhuanti','gonglue_zhuanti2','jd_mt_huaweiqjzn') AND SUBSTRING(a.create_time,1,10)>='2017-07-01' AND SUBSTRING(a.create_time,1,10)<='2017-08-10'
`

## 18. 某城市酒店成单量（城市id）

`
select x.dist_id,count(distinct(uid)) from (select dist_name,dist_id from poi_detail where dist_id in ('299914')) as x
inner join
(select a.uid,a.city_name from mppb_order_channel as a  WHERE a.dt>='2017-07-01' and a.dt<='2017-08-01' AND a.STATUS IN('0','2') AND a.vid LIKE '91%' AND a.chan_value in('travel_touch','travel_gonglue','travel_client','travelnote_client','travel note_gonglue','gonglue_zhuanti','gonglue_zhuanti2','jd_mt_huaweiqjzn') AND SUBSTRING(a.create_time,1,10)>='2017-08-01' AND SUBSTRING(a.create_time,1,10)<='2017-07-01' ) as y
on x.dist_name=y.city_name group by x.dist_id
`
## 19. 14天内有新加坡酒店订单的用户（入住时间）
`
select distinct(d.userid)  from (select b.uid,city_name,from_date from mppb_order_channel as a inner join (select touch_uid,uid from ods_travel_touch_uid ) as b on a. uid=b.touch_uid WHERE a.dt>='2017-11-01' and a.dt<='2017-12-26' AND a.STATUS IN('0','2') AND a.vid LIKE '91%' AND a.chan_value in('travel_touch','travel_gonglue','travel_client','travelnote_client','travel note_gonglue','gonglue_zhuanti','gonglue_zhuanti2','jd_mt_huaweiqjzn') and a.from_date>='2017-12-13' and a.from_date<='2017-12-26' and a.city_name='新加坡')as c  inner join tmp_userid_uid_20171218 as d on c.uid=d.uid
`
## 20. 常居地在成都
`
select distinct(a.userid) from tmp_userid_uid_20171218 as a inner join (select uid from mobile_user_new2 where destid in ('300085') or cityid in ('300085')) as b on a.uid=b.uid
`
## 21. 常居地or近5天定位在成都重庆的12月以后登录过的 浏览过泸沽湖景区的用户

常居地  
`
select distinct(x.userid) from (select distinct(a.userid) from tmp_userid_uid_20171218 as a inner join (select c.uid from (select uid from mobile_user_new2 where destid in ('300085','299979') or cityid in ('300085','299979')) as b inner join (select distinct(uid) from travel_client_access where dt>='2017-12-01' and dt <='2017-12-28' and requesturi='/api/book/element' and requestparammap like '%poiType=4%' and regexp_extract(requestparammap,'[{ ]poiId=([0-9]+)',1) in ('720460')) as c on b.uid=c.uid) as d on a.uid=d.uid) as x  inner join (select distinct(userid) from travel_client_access where dt>='2017-12-01' and dt <='2017-12-28' and userid like '%@qunar%') as y on x.userid=y.userid
`
  
定位
`
select distinct(x.userid) from (select distinct(a.userid) from tmp_userid_uid_20171218 as a inner join (select c.uid from (select uid from travel_client_access where dt>='2017-12-24' and dt <='2017-12-28' and requesturi='/api/city/located' and (requestparammap like '%cityId=300085%' or requestparammap like '%cityId=299979%')) as b inner join (select distinct(uid) from travel_client_access where dt>='2017-12-01' and dt <='2017-12-28' and requesturi='/api/book/element' and requestparammap like '%poiType=4%' and regexp_extract(requestparammap,'[{ ]poiId=([0-9]+)',1) in ('720460')) as c on b.uid=c.uid) as d on a.uid=d.uid) as x  inner join (select distinct(userid) from travel_client_access where dt>='2017-12-01' and dt <='2017-12-28' and userid like '%@qunar%') as y on x.userid=y.userid
`
