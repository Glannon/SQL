# SQL
SQL查询语句
## 1. 查询酒店订单

| pay_type     | 0 | 预付 | ppb_oeder_status | 4  | 确认 |   |
|--------------|---|------|------------------|----|------|---|
| （付款方式） | 1 | 现付 | （订单状态）     | 5  | 取消 |   |
|              |   |      |                  | 15 | 退款 |   |

使用时修改uid和时间  
**SQL语句**  

`
select create_time,b.uid,city_name,hotel_id,hotel_seq,hotel_name,from_date,to_date,room_name,bed_type,pay_type,ppb_order_status from mppb_order_channel as a , (select touch_uid,uid from ods_travel_touch_uid where uid='3812B41A-4611-4923-A157-82C19EF2868C' ) as b where a. uid=b.touch_uid and a.dt>='2017-07-01' and a.dt<='2017-08-10' AND a.STATUS IN('0','2') AND a.vid LIKE '91%' AND a.chan_value in('travel_touch','travel_gonglue','travel_client','travelnote_client','travel note_gonglue','gonglue_zhuanti','gonglue_zhuanti2','jd_mt_huaweiqjzn') AND SUBSTRING(a.create_time,1,10)>='2017-07-01' AND SUBSTRING(a.create_time,1,10)<='2017-08-10'
`

## 2. 查询浏览的游记的页面位置
位置字段为itemOrder，数值加1为所处位置，使用时修改uid和时间  
`
select * from travel_client_access where dt='2017-08-03' and uid='45523841-53E3-428E-A6F0-C9861FA11F8E' and requesturi='/api/book/getSimplified' and requestparammap like '%type=2%' order by currenttime asc
`

## 3. 查询浏览的酒店的页面位置
位置字段为itemOrder，数值加1为所处位置，使用时修改uid和时间  
`
select * from travel_client_access where dt='2017-08-03' and uid='45523841-53E3-428E-A6F0-C9861FA11F8E' and requesturi='/api/book/element' and requestparammap like '%poiType=2%' order by currenttime asc
`
## 4. 单用户全部访问行为记录

使用时修改uid和时间  

`
select * from travel_client_access where dt>='2017-05-12' and dt<='2017-07-26' and uid='47B360D0-0424-49C0-AE12-77CC94902C8F' order by currenttime asc
`


## 5. 单用户常居地查询
使用时修改uid，dest_name为用户设置常居地，city_name为系统计算常居地

`
select * from mobile_user_new2 where uid='47B360D0-0424-49C0-AE12-77CC94902C8F'
`


## 6. 单用户定位接口查询

使用时修改uid和时间   
`
select * from travel_client_access where dt='2017-09-12' and uid='47B360D0-0424-49C0-AE12-77CC94902C8F' and requesturi='/api/city/locate' order by currenttime asc
`

## 7. 机票查询

查询单用户机票订单，修改时间

`
select b.uid,depcity,arrcity,deptime from orderinfo_channel, (select touch_uid,uid from ods_travel_touch_uid where uid='45523841-53E3-428E-A6F0-C9861FA11F8E') as b where orderinfo_channel.deviceuid=b.touch_uid and dt>='2017-11-01' and dt<='2017-11-01' AND orderstatus NOT IN (0, 12, 20, 51, 91) 	
`

## 8. 火车票查询

查询单用户火车票订单，修改时间     

`
select b.uid,create_time,train_from,train_to,train_start_time,train_end_time,train_seat,train_seat_count,ticket_price,pre_status from ods_train_order_channel , (select touch_uid,uid from ods_travel_touch_uid where uid='45523841-53E3-428E-A6F0-C9861FA11F8E') as b where ods_train_order_channel.gid=b.touch_uid and dt='2017-07-11' and chan_value='gonglue_client'
`

## 9. 查询用户基本信息
根据userid查询，修改userid即可  

`
select uid,userid,c.* from
(select a.uid,a.userid,b.* from user_profile_common a lateral view json_tuple(pmap,'account_realname_info') b where userid='131980484' and dt='2017-11-19') d lateral view json_tuple(c0,'birthday','gender','province','city','age','is_real') c 
`
