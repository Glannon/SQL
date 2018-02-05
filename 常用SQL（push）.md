# SQL语句记录

## 1.14天内到达昌北机场的订单用户

`
select distinct(y.userid) from 
  (select c.uid from (select uid from (select distinct deviceuid from orderinfo_channel where dt>='2017-11-01' and dt<='2017-12-26' and arrcity='南昌' and deptime>='2017-12-13' and deptime<='2017-12-26' and orderStatus NOT IN ('0','12','20','51','91')) as a,(select touch_uid,uid from ods_travel_touch_uid) as b where a.deviceuid=b.touch_uid) as c,(select uid,city_id from tmp_user_locate_result where city_id!='300021') as d where c.uid=d.uid) as x inner join tmp_userid_uid_20171218 as y on x.uid=y.uid
`

## 2. 建立uid与userid关联表

`
create table tmp_userid_uid as select distinct uid,userid from travel_client_access where dt>='2016-01-01' and userid like '%@qunar%' and uid!='000000000000'
`  

`
create table tmp_userid_uid_20180205 as select distinct uid,userid from (select distinct uid,userid from travel_client_access where dt>='2018-01-15' and userid like '%@qunar%' and uid!='000000000000' union select uid,userid from tmp_userid_uid_20180129) a
`  

## 3. 常居地在某地访问过某地(push)

`
select distinct(x.userid) from  tmp_userid_uid_20171218 as x inner join (select a.uid from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-05' ) as a inner join (select uid from mobile_user_new2 where destid in('300100','300083','703475','300077','300192','300089') or cityid in('300100','300083','703475','300077','300192','300089')) as b on a.uid=b.uid where a.id='300085' ) as y on x.uid=y.uid
`


## 4. 定位在某地访问过某地

`
select distinct(x.userid) from tmp_userid_uid_20171218 as x inner join (select a.uid from (select regexp_extract(requestparammap,'[{ ]id=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/get' and dt>='2017-11-05' ) as a inner join (select regexp_extract(requestparammap,'cityId=([0-9]+)',1) as id,uid from travel_client_access where requesturi='/api/city/locate' and dt>='2017-11-05' ) as b on a.uid=b.uid where a.id in ('300085') and b.id in ('300100','300083','703475','300077','300192','300089')) as y on x.uid=y.uid
`

## 5. 访问过某个城市或景区的用户
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
## 6. 浏览过某地酒店
`
select distinct(userid) from tmp_userid_uid as b inner join (select distinct(uid) from travel_client_access where dt>='2017-11-24' and dt<='2017-11-29' and requesturi='/api/hotel/search' and requestparammap like '%cityId=302161%'  and uid!='000000000000') as a on a.uid=b.uid
`

## 7. 某城市酒店成单量（城市名称）

`
select count(distinct(uid)) from mppb_order_channel as a inner join (select touch_uid,uid from ods_travel_touch_uid where city_name='北京') as b on a. uid=b.touch_uid WHERE a.dt>='2017-07-01' and a.dt<='2017-08-10' AND a.STATUS IN('0','2') AND a.vid LIKE '91%' AND a.chan_value in('travel_touch','travel_gonglue','travel_client','travelnote_client','travel note_gonglue','gonglue_zhuanti','gonglue_zhuanti2','jd_mt_huaweiqjzn') AND SUBSTRING(a.create_time,1,10)>='2017-07-01' AND SUBSTRING(a.create_time,1,10)<='2017-08-10'
`

## 8. 某城市酒店成单量（城市id）

`
select x.dist_id,count(distinct(uid)) from (select dist_name,dist_id from poi_detail where dist_id in ('299914')) as x
inner join
(select a.uid,a.city_name from mppb_order_channel as a  WHERE a.dt>='2017-07-01' and a.dt<='2017-08-01' AND a.STATUS IN('0','2') AND a.vid LIKE '91%' AND a.chan_value in('travel_touch','travel_gonglue','travel_client','travelnote_client','travel note_gonglue','gonglue_zhuanti','gonglue_zhuanti2','jd_mt_huaweiqjzn') AND SUBSTRING(a.create_time,1,10)>='2017-08-01' AND SUBSTRING(a.create_time,1,10)<='2017-07-01' ) as y
on x.dist_name=y.city_name group by x.dist_id
`
## 9. 14天内有新加坡酒店订单的用户（入住时间）
`
select distinct(d.userid)  from (select b.uid,city_name,from_date from mppb_order_channel as a inner join (select touch_uid,uid from ods_travel_touch_uid ) as b on a. uid=b.touch_uid WHERE a.dt>='2017-11-01' and a.dt<='2017-12-26' AND a.STATUS IN('0','2') AND a.vid LIKE '91%' AND a.chan_value in('travel_touch','travel_gonglue','travel_client','travelnote_client','travel note_gonglue','gonglue_zhuanti','gonglue_zhuanti2','jd_mt_huaweiqjzn') and a.from_date>='2017-12-13' and a.from_date<='2017-12-26' and a.city_name='新加坡')as c  inner join tmp_userid_uid_20171218 as d on c.uid=d.uid
`
## 10. 常居地在成都
`
select distinct(a.userid) from tmp_userid_uid_20171218 as a inner join (select uid from mobile_user_new2 where destid in ('300085') or cityid in ('300085')) as b on a.uid=b.uid
`
## 11. 常居地or近5天定位在成都重庆的12月以后登录过的 浏览过泸沽湖景区的用户

常居地  
`
select distinct(x.userid) from (select distinct(a.userid) from tmp_userid_uid_20171218 as a inner join (select c.uid from (select uid from mobile_user_new2 where destid in ('300085','299979') or cityid in ('300085','299979')) as b inner join (select distinct(uid) from travel_client_access where dt>='2017-12-01' and dt <='2017-12-28' and requesturi='/api/book/element' and requestparammap like '%poiType=4%' and regexp_extract(requestparammap,'[{ ]poiId=([0-9]+)',1) in ('720460')) as c on b.uid=c.uid) as d on a.uid=d.uid) as x  inner join (select distinct(userid) from travel_client_access where dt>='2017-12-01' and dt <='2017-12-28' and userid like '%@qunar%') as y on x.userid=y.userid
`
  
定位  
`
select distinct(x.userid) from (select distinct(a.userid) from tmp_userid_uid_20171218 as a inner join (select c.uid from (select uid from travel_client_access where dt>='2017-12-24' and dt <='2017-12-28' and requesturi='/api/city/located' and (requestparammap like '%cityId=300085%' or requestparammap like '%cityId=299979%')) as b inner join (select distinct(uid) from travel_client_access where dt>='2017-12-01' and dt <='2017-12-28' and requesturi='/api/book/element' and requestparammap like '%poiType=4%' and regexp_extract(requestparammap,'[{ ]poiId=([0-9]+)',1) in ('720460')) as c on b.uid=c.uid) as d on a.uid=d.uid) as x  inner join (select distinct(userid) from travel_client_access where dt>='2017-12-01' and dt <='2017-12-28' and userid like '%@qunar%') as y on x.userid=y.userid
`
## 12.从suggest进入某城市游记用户
`
select distinct(a.userid) from tmp_userid_uid_20171218 as a inner join (select distinct(uid) from travel_client_access where requesturi='/api/book/search' and requestparammap like '%from=destsuggest%' and requestparammap like '%type=2%' and dt>='2017-12-01' and dt<='2017-12-28' and requestparammap like '%keyword=新加坡%') as b on a.uid=b.uid
`

## 13. push聪游榜打开人数
`
select dt,count(userid),count(distinct userid) from access_info where dt>='2018-01-25' and dt<='2018-01-25' and modelname='touch' and uri like '%/smartlist/6982704%' and parammap like '%pushcyb%' group by dt order by dt asc
`
